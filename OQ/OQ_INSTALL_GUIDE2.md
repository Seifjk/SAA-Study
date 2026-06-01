# OQ OpenShift Cluster — Install Guide

A from-scratch guide to installing the **OQ** OpenShift cluster, written so you understand *why* every step exists, not just *what to type*. Use it as a teaching doc first and a runbook second.

---

## 0. The 60-second summary

We're installing a small OpenShift 4.20.10 cluster called **OQ** on 8 nodes (3 masters + 4 infra + 1 VM worker) to act as a sandbox alongside the hub. Because nothing automates this on our hardware (no AWS, no vSphere integration), we have to set up *everything that the cluster needs from outside itself*: IPs, DNS, load balancers, switch VLANs. OpenShift's installer then takes over and brings up the cluster software.

The install flow looks like this:

```
NetBox (IPAM)  ─┐
FreeIPA (DNS)  ─┼─►  agent-config.yaml  ──►  openshift-install  ──►  agent.iso  ──►  Boot 8 nodes  ──►  Running cluster
F5 (LBs)       ─┤    install-config.yaml
Switches (VLAN)─┘
```

Each piece on the left has to exist *before* you boot the nodes. The rest of this document explains each box.

> **Status (2026-05-31).** External prerequisites are complete and verified: NetBox IPAM ✅ (verified via API), F5 LBs ✅ (confirmed by network team), VLANs 178/179/180 + firewall ✅ (confirmed by network team), FreeIPA DNS records ✅ (created — see §6.4). The `install/OQ/` repo folder is scaffolded and validated. **Remaining before boot:** stage the 4.20.10 binaries in `OQ/bin/`, fill the node MAC addresses from XClarity, build the ISO. Nothing has been executed toward the actual install yet.

---

## 1. What kind of cluster install is this?

OpenShift has four broad install paths. Knowing which one we're using changes what we have to provide.

| Method | What it automates | When you'd use it |
|---|---|---|
| **IPI** (Installer-Provisioned Infra) | Everything: VMs, networking, DNS, LBs | Public clouds (AWS, Azure, GCP) and supported vSphere |
| **UPI** (User-Provisioned Infra) | Nothing — you bring servers + DNS + LBs + ignition delivery | Old method, lots of manual work |
| **Assisted Installer** | Discovery + most prerequisites — you upload an ISO | Mostly used via console.redhat.com |
| **Agent-based** (what we use) | Same end result as Assisted, but the entire installer is bundled into one self-contained ISO you boot | Air-gapped or restricted networks, on-prem hardware where you still want UX similar to Assisted |

**Why Agent-based here?** Our environment has no cloud integration, and we want a single ISO we can mount through XClarity (the Lenovo BMC) on each baremetal node. The Agent installer is the modern Red Hat answer for that scenario.

**Why `platform: none`?** "Platform" in OCP config tells the installer "what provider should manage VIPs and LBs for you?" `baremetal` would manage them via keepalived/MetalLB internally, but it requires PXE + BMC credentials per node and assumes a specific topology. `none` means "I'm handling that externally" — we give OpenShift an existing DNS + LB and it doesn't care how they're implemented. It's more work for us but it fits our environment because we already have F5 + FreeIPA.

---

## 2. Cluster topology (and why each role exists)

We have three node roles in OQ. They share the OS and most components but differ in what gets scheduled on them.

```
          ┌─────────────────────────────────────────────┐
          │              External clients               │
          └─────────────────────┬───────────────────────┘
                                │
            ┌───────────────────┴───────────────────┐
            │                                       │
       api.oq                                  *.apps.oq
   (API LB → 6443)                       (Ingress LB → 80/443)
            │                                       │
   ┌────────┴───────────┐               ┌───────────┴─────────────┐
   │      MASTERS       │               │       INFRA NODES       │
   │  (control plane)   │               │ (ingress controllers,   │
   │                    │               │  monitoring, logging)   │
   │ b-master-004       │               │ b-infra-006             │
   │ b-master-005       │               │ b-infra-007             │
   │ m-master-004       │               │ m-infra-006             │
   │                    │               │ m-infra-007             │
   │ Run kube-apiserver,│               │                         │
   │ etcd, scheduler,   │               │                         │
   │ controller-manager │               │                         │
   └────────────────────┘               └─────────────────────────┘
                                                                    
                                                                    
                                  ┌─────────────────────────────────┐
                                  │       WORKER NODES              │
                                  │  m-vm-029 (VM, sandbox tenant)  │
                                  │  Run user workloads             │
                                  └─────────────────────────────────┘
```

- **Masters (3)**: Run the cluster's control plane (kube-apiserver, etcd, scheduler, controller-manager). 3 is the minimum for etcd quorum; we don't run user workloads on them.
- **Infra (4)**: A dedicated worker pool reserved for cluster-infrastructure pods — the OpenShift Ingress Controllers (HAProxy), monitoring (Prometheus), logging, image registry, etc. We separate them so tenant workloads can't starve cluster-internal services. They aren't a built-in OCP role; we tag them with `node-role.kubernetes.io/infra=""` after install.
- **Workers (1)**: Where user workloads run. On OQ that's just `m-vm-029` initially. More can be added later.

The 8 nodes you're about to install all use the same RHCOS image. The role differs because:
1. `install-config.yaml` says `controlPlane.replicas: 3` and `compute.replicas: 5`.
2. `agent-config.yaml` declares each host as `role: master` or `role: worker` (infra is a sub-type of worker, applied later via labels).

---

## 3. The three networks (VLANs 178/179/180)

OQ has three dedicated networks. Each has a specific traffic class and lives on its own VLAN.

| Name | VLAN | IPv4 | IPv6 | NetBox prefix | What flows on it |
|---|---|---|---|---|---|
| **Node Network** | 178 | `10.1.48.0/24` | `2001:678:108c:130::/64` | 10503/10504 | All cluster traffic: kubelet↔API, pod↔pod (via OVN), API/Ingress data |
| **Live Migration** | 179 | `10.1.49.0/24` | `2001:678:108c:131::/64` | 10505/10506 | KubeVirt VM live-migration traffic (memory pages copying between nodes) |
| **MetalLB Pool** | 180 | `10.1.50.0/24` | `2001:678:108c:132::/64` | 10507/10508 | (a) the cluster's API + Ingress VIPs on the F5; (b) future Kubernetes Service `type: LoadBalancer` IPs handed out by MetalLB |

**Why three separate networks?**
1. **Separation of failure domains** — if a VM live-migration storm saturates one network, the cluster control plane on the Node Network keeps working.
2. **Security** — workloads in the MetalLB range can be firewalled differently from cluster-internal traffic.
3. **Performance** — live-migration is bursty and bandwidth-hungry; isolating it keeps p99 latency on the Node Network predictable.

**On the host side**, the Node Network maps to `bond0.178` (a VLAN 178 sub-interface of the LACP bond `bond0`), and Live Migration maps to `bond1.179`. The hub cluster uses VLAN 8 and 10 — same idea, different numbers. MetalLB doesn't get a host interface; it's announced via BGP/L2 by the in-cluster MetalLB operator later.

---

## 4. NetBox — IPAM source of truth

NetBox is the system-of-record for IPs, prefixes, and physical infrastructure. Before any other team touches anything for OQ, the IPs need to live in NetBox.

### 4.1 Why this matters

- **Collision avoidance** — if I don't reserve `10.1.48.111`, someone else may give it to a different host next week.
- **Hand-off to network team** — when we ask them to configure F5 LBs, they need to see those IPs are owned in IPAM, not just "an idea Seif had".
- **Auditability** — there's a single place to grep if you wonder "what's at 10.1.50.10?".

### 4.2 What we reserved (and why)

For each of the 8 OQ nodes, two new virtual interfaces with two IPs each:

| Interface | Network | Purpose |
|---|---|---|
| `bond0.178` | OQ Node Network | The IP OCP will use as the node's primary address (kubelet, pod-network attachment point) |
| `bond1.179` | OQ Live Migration | The IP used for VM live-migration if/when KubeVirt is added |

Plus 4 standalone IPs (no device assignment) in the MetalLB prefix:

| Address | DNS name | Role |
|---|---|---|
| `10.1.50.10` + v6 `…:320a` | `api.oq.tsrs.corp` | API VIP (kube-apiserver entry point) |
| `10.1.50.11` + v6 `…:320b` | `apps.oq.tsrs.corp` | Ingress VIP (HTTP/HTTPS apps entry point) |

The VIPs have `role=vip` in NetBox so it's obvious they're not "real" host IPs.

### 4.3 The "last octet from XCC" rule

Every Lenovo baremetal node in our environment has three IPv4 addresses with **the same last octet**, just different `/24` networks:

| Interface | Network | Example: `b-master-004` |
|---|---|---|
| `XCC` (BMC management) | `10.0.0.0/23` | `10.0.0.111` |
| `bond0.<NodeVLAN>` (data) | hub `10.1.8.0/23`, OQ `10.1.48.0/24` | hub: `10.1.8.111`; OQ: `10.1.48.111` |
| `bond1.<LiveMigVLAN>` (live-mig) | hub `10.1.10.0/23`, OQ `10.1.49.0/24` | hub: `10.1.10.111`; OQ: `10.1.49.111` |

The XCC IP is set when the server is racked. Everything else inherits its last octet. That's what the colleague meant by *"nimm die letzte Nummer vom Baremetal Interface und pack das dann hinten dran"* — the BMC's last octet is the node's identity across all networks.

For OQ, this gave us:

| Node | XCC | bond0.178 (Node) | bond1.179 (LiveMig) |
|---|---|---|---|
| b-master-004 | 10.0.0.111 | 10.1.48.111 | 10.1.49.111 |
| m-master-004 | 10.0.0.112 | 10.1.48.112 | 10.1.49.112 |
| b-master-005 | 10.0.0.113 | 10.1.48.113 | 10.1.49.113 |
| m-infra-006 | 10.0.0.114 | 10.1.48.114 | 10.1.49.114 |
| b-infra-006 | 10.0.0.115 | 10.1.48.115 | 10.1.49.115 |
| m-infra-007 | 10.0.0.116 | 10.1.48.116 | 10.1.49.116 |
| b-infra-007 | 10.0.0.117 | 10.1.48.117 | 10.1.49.117 |
| m-vm-029 (VM) | 10.0.0.82 | 10.1.48.82 | 10.1.49.82 |

### 4.4 IPv6 mapping

Every IPv4 host IP has an IPv6 sibling using a **dual-stack, IPv4-mapped style**. The last 4 hex groups of the v6 address encode the v4 in hex:

```
v4:  10.1.48.111         →  hex: 0a 01 30 6f  →  groups: a01:306f
v6:  2001:678:108c:130:0:ffff:a01:306f
     └──── OQ Node v6 prefix ────┘  └ marker ┘  └ encoded v4 ┘
```

This is a deliberate convention: it makes any v6 address trivially readable as a v4 address. Hub uses the same scheme on its own prefix (`2001:678:108c:108`); OQ just lives on `…:130`. You can verify in Python:

```python
import ipaddress
ipaddress.IPv6Address('2001:678:108c:130:0:ffff:a01:306f')  # contains 10.1.48.111
```

> **Gotcha — the Live-Migration v6 suffix encodes the *node-network* (`.48`) octets, not the `.49` ones.**
> In NetBox each host's `bond1.179` address pairs IPv4 `10.1.49.<n>` with an IPv6 whose `ffff:` suffix still
> encodes `10.1.48.<n>`. Example for `b-master-004`: `bond1.179 = 10.1.49.111` **and**
> `2001:678:108c:131:0:ffff:a01:306f` (suffix `a01:306f` = `10.1.48.111`, the host's stable identity). So the
> same `ffff:a01:30xx` suffix appears on both the `…:130` (node) and `…:131` (live-mig) prefixes — `agent-config.yaml`
> is written to match NetBox exactly.

---

## 5. Load Balancers

### 5.1 Why we need them at all

With `platform: none`, OpenShift does not advertise its own VIPs. So we need an external load balancer (here: F5) for two reasons:

1. **API HA** — the kube-apiserver runs on all 3 masters. Clients (kubectl, kubelet, controllers) need one stable address that fronts all three. If you point `kubectl` at a single master and that master reboots, you've lost the cluster from that client's perspective.
2. **Ingress entry point** — application HTTP traffic enters the cluster via the OCP Router (HAProxy) running on the infra nodes. The LB picks any healthy infra node for each connection.

### 5.2 The four LB services for OQ

| Service | VIP | VS Port | Real Servers | Real Port | Reason |
|---|---|---|---|---|---|
| OCP OQ API | `10.1.50.10` (+ v6) | 6443 | 3 masters | 6443 | kube-apiserver HA |
| OCP OQ MCS | `10.1.50.10` (+ v6) | 22623 | 3 masters | 22623 | Machine Config Server delivers Ignition to nodes during install and scale |
| OCP OQ Ingress HTTP | `10.1.50.11` (+ v6) | 80 | 4 infra nodes | 80 | HTTP routes |
| OCP OQ Ingress HTTPS | `10.1.50.11` (+ v6) | 443 | 4 infra nodes | 443 | HTTPS routes (TLS termination on cluster side, LB is pure TCP) |

**Why "API" and "MCS" share a VIP**: nodes resolve `api-int.oq.tsrs.corp` to the same VIP for both. They talk 6443 for the API and 22623 for Ignition. F5 picks the right pool by port.

**Why MCS only matters during install (sort of)**: `22623` is hit when a node first boots (to fetch its Ignition config) and every time a node reboots after a MachineConfig change. So you keep the LB rule permanently, but day-to-day clients never see it.

**Why VIPs come from the MetalLB range, not the Node Network**: convention from hub. The hub cluster's API VIP (`api.hub.tsrs.corp`) lives in `10.1.12.0/x`, which is hub's equivalent of our MetalLB range. The Node Network is reserved for actual node IPs; VIPs (whether F5-owned or MetalLB-owned) live separately. Keeps the bookkeeping clean and lets the in-cluster MetalLB later allocate from the same range without colliding with node IPs.

**Why ingress real servers are the infra nodes**: in the hub cluster, the `IngressController` resource has:

```yaml
spec:
  nodePlacement:
    nodeSelector:
      matchLabels:
        node-role.kubernetes.io/infra: ""
```

…which pins the HAProxy pods to infra nodes. Same approach on OQ. The LB sends traffic only to infra IPs because those are the only nodes running the router.

### 5.3 Health checks

The colleague named three F5 health-check profiles to attach:

| Profile | Used by |
|---|---|
| `HCM_OCP_API_6080` | API LB rows |
| `HCM_OCP_MC_22623` | MCS LB rows |
| `HCM_OCP-Ingress` | Ingress HTTP + HTTPS rows |

These are pre-defined monitors on the F5 — they know how to probe an OCP backend correctly. We just reference them by name.

---

## 6. DNS in FreeIPA

OpenShift wants its cluster FQDN resolvable from anywhere a client could be — your laptop, the masters themselves during install, the LB during health-checks. FreeIPA is the org's authoritative DNS.

### 6.1 The 4 records

In zone `oq.tsrs.corp`, all four entries are A + AAAA pairs:

| Name | Type | Target |
|---|---|---|
| `api` | A / AAAA | API VIP (`10.1.50.10` / `2001:678:108c:132:0:ffff:a01:320a`) |
| `api-int` | A / AAAA | Same as `api` |
| `*.apps` | A / AAAA | Ingress VIP (`10.1.50.11` / `…:320b`) |
| `apps` | A / AAAA | Same as `*.apps` |

### 6.2 Why each one

- **`api`**: external clients (you running `oc login`, CI runners, Argo CD on hub).
- **`api-int`**: internal cluster clients (kubelet, controllers) talking to the API. OCP separates the two names so you *could* split the LBs externally vs internally. We don't, so both point at the same VIP — but you still need both records because the installer hardcodes `api-int` for in-cluster traffic.
- **`*.apps`**: every Route the cluster admits (e.g. `console-openshift-console.apps.oq.tsrs.corp`) must resolve to the Ingress VIP. Wildcard saves us from creating one record per Route.
- **`apps`**: some clients (older browsers, some HTTP libraries) don't handle wildcard apex matching well. The `apps` record alone ensures `apps.oq.tsrs.corp` itself resolves.

### 6.3 Why FreeIPA specifically

Because our org runs FreeIPA as identity/DNS, that's where authoritative zones live. The hub `kubelet` resolvers in `agent-config.yaml` point at FreeIPA servers (`10.1.31.11`, `10.1.31.10`, and their v6 siblings). When OQ nodes boot, they'll do the same. If the FreeIPA records aren't in place by the time install finishes, kubelet won't find `api-int` and the cluster won't bootstrap.

### 6.4 How they were created (done 2026-05-31)

The zone `oq.tsrs.corp` **already existed** in FreeIPA (it had been created earlier; only the records were missing, because the VIPs weren't known yet). There is **no Ansible/AWX automation for FreeIPA DNS** — records are added by hand with the `ipa` CLI, which needs a Kerberos ticket for an admin with DNS-write rights. The four record pairs were created with:

```bash
kinit admin   # or your IPA admin principal with DNS write

ipa dnsrecord-add oq.tsrs.corp api      --a-rec=10.1.50.10 --aaaa-rec=2001:678:108c:132:0:ffff:a01:320a
ipa dnsrecord-add oq.tsrs.corp api-int  --a-rec=10.1.50.10 --aaaa-rec=2001:678:108c:132:0:ffff:a01:320a
ipa dnsrecord-add oq.tsrs.corp apps     --a-rec=10.1.50.11 --aaaa-rec=2001:678:108c:132:0:ffff:a01:320b
ipa dnsrecord-add oq.tsrs.corp '*.apps' --a-rec=10.1.50.11 --aaaa-rec=2001:678:108c:132:0:ffff:a01:320b
```

Verify:

```bash
for n in api api-int apps test.apps; do
  printf "%-12s A=%s AAAA=%s\n" "$n" \
    "$(dig +short @10.1.31.11 A $n.oq.tsrs.corp)" \
    "$(dig +short @10.1.31.11 AAAA $n.oq.tsrs.corp)"
done
```

Use **A/AAAA, not CNAME**, for `api`/`api-int`. Quote `'*.apps'` so the shell doesn't glob it. PTR/reverse records are **not** required for bootstrap and were not created.

---

## 7. Switch + Firewall (network team)

You don't do this — but understanding what they're doing helps you write the ticket clearly.

| Component | Device | What's needed | Why |
|---|---|---|---|
| **Switch ports** for the 8 nodes | the access switch each node is patched into | VLAN 178 and 179 trunked tagged | The nodes will tag-out frames on `bond0.178` and `bond1.179`; the switch needs to expect those tags |
| **Firewall L3 interfaces** | `tcfwadm` (central firewall, zone path `tcadmin > tccloud`) | One interface per VLAN (178, 179, 180), each with the gateway IP — `.1` of each subnet, plus the v6 `::1` | Otherwise nodes have IPs but can't reach anything off-subnet — DNS lookups fail, installer can't pull container images, kubelet can't reach FreeIPA |
| **Firewall rules** | same | Allow OQ Node Network → FreeIPA, mirror registries, NTP, hub Argo, internet for pull-secret, etc. | Same baseline policy as hub. The network team has a template. |

VLAN 180 (MetalLB) doesn't need trunking to node ports because nodes don't have an interface on it — MetalLB announces those IPs via BGP/L2 from inside the cluster. But the firewall still needs an L3 interface on VLAN 180 so external clients can route to the VIPs and to whatever MetalLB hands out for Services later.

**Zone vocabulary** (handy for reading colleague messages):
- `tcadmin` — the admin / management zone where central firewalls and IPAM live.
- `tccloud` — the tenant cloud zone where OQ (and hub) lives. New VLANs get attached here.
- `tcadmin > tccloud` notation = "route/policy from admin into cloud" — the direction the firewall is being asked to bridge.

---

## 8. The install repo

Location: `/home/rs4798/Desktop/Infrastructure_FMO/Openshift/install/`

Current layout:

```
install/
├── README.md
├── Sovereign_Cloud/                ← hub cluster's install artifacts (do NOT edit)
│   ├── env.sh
│   ├── prepare.sh
│   ├── generate.sh
│   ├── manifests/
│   │   ├── install-config.yaml
│   │   └── agent-config.yaml
│   └── additional_nodes/
│       └── baremetal_lenovo/
│           ├── README.md
│           └── gen_nodes-config.sh
└── OQ/                             ← created + adapted for OQ — scaffolded 2026-05-31 (env.sh, prepare.sh, generate.sh, manifests/, additional_nodes/); see §8.x and §9
```

### 8.1 `env.sh`

Just sets `PATH` (and optional, commented-out proxies):

```bash
SCRIPT_DIR=$( cd -- "$( dirname -- "${BASH_SOURCE[0]}" )" &> /dev/null && pwd )
export PATH=$SCRIPT_DIR/bin:$PATH
# (proxies optional, commented)
```

For OQ this file is **unchanged** from hub. The OCP version is pinned in `prepare.sh`, not here.

### 8.2 `prepare.sh`

Downloads + extracts three binaries into `bin/`:

| Binary | Why |
|---|---|
| `openshift-install` | Generates the agent ISO and waits for install completion |
| `oc` | Once install completes, you `oc login` to operate the cluster |
| `nmstatectl` | Validates the YAML network configs in `agent-config.yaml` before they get baked into the ISO (catches typos like wrong VLAN id) |

The version is pinned at the **top of `prepare.sh`** (`ocp_version="4.20.10"` for OQ) and each binary is checked against it. Our environment is air-gapped, so the `curl` lines are commented out — pre-stage the three archives in `bin/` (`openshift-install-linux.tar.gz`, `openshift-client-linux.tar.gz`, `nmstatectl-linux-x64.zip`) and the script extracts them in place.

### 8.3 `manifests/install-config.yaml`

Cluster-wide config. Key fields, with the OQ values we'll use:

```yaml
apiVersion: v1
baseDomain: tsrs.corp                     # → cluster FQDN = oq.tsrs.corp
metadata:
  name: oq                                # ← cluster name
compute:
- name: worker
  replicas: 5                             # 4 infra + 1 sandbox worker (infra is a label, applied post-install)
  hyperthreading: Enabled
controlPlane:
  name: master
  replicas: 3                             # 3 masters
  hyperthreading: Enabled
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14                   # pod IP pool — internal to cluster, never seen outside
    hostPrefix: 23
  - cidr: fd02::/48                       # v6 pod pool
    hostPrefix: 64
  serviceNetwork:
  - 172.30.0.0/16                         # ClusterIP pool — internal Service IPs
  - fd03::/112
  machineNetwork:
  - cidr: 10.1.48.0/24                    # OQ Node Network (must match agent-config bond0.178 IPs)
  - cidr: 2001:678:108c:130::/64
  networkType: OVNKubernetes              # CNI
platform:
  none: {}                                # external LB + DNS
fips: false
sshKey: 'ssh-ed25519 ...'                 # for emergency SSH access to nodes
pullSecret: '{ ... }'                     # Red Hat registry credentials
```

**What does the installer do with this?**
- Picks the install topology (3+5)
- Allocates internal Service / Pod ranges (these have to not overlap with anything on your real networks)
- Knows the cluster's outward identity (`oq.tsrs.corp`)
- Verifies the pull secret can fetch container images

### 8.4 `manifests/agent-config.yaml`

Per-host networking. This is what makes Agent-based different from old UPI — the network state for *every* node is described once, declaratively, and baked into the ISO. Each host block:

```yaml
- hostname: "b-master-004.bm.tsrs.corp"
  role: "master"
  rootDeviceHints:
    deviceName: "/dev/disk/by-path/..."   # which disk OCP installs onto
  interfaces:
    - macAddress: "..."                    # 4 NIC MACs — order matters
      name: "fe1"
    ...
  networkConfig:
    interfaces:
      - name: bond0                        # bond over fe1/fe2 (active-backup — see note below)
        type: bond
        link-aggregation:
          mode: active-backup
          options: { miimon: '100' }
          port: [fe1, fe2]
      - name: bond0.178                    # tagged sub-interface, OQ Node Network
        type: vlan
        vlan: { base-iface: bond0, id: 178 }
        ipv4: { enabled: true, dhcp: false, address: [{ip: 10.1.48.111, prefix-length: 24}] }
        ipv6: { enabled: true, dhcp: false, address: [{ip: '2001:678:108c:130:0:ffff:a01:306f', prefix-length: 64}] }
      - name: bond1                        # second LACP bond over lm1/lm2
      - name: bond1.179                    # OQ Live Migration sub-interface
    dns-resolver:
      config:
        search: [bm.tsrs.corp]
        server: [10.1.31.11, 10.1.31.10, '2001:678:108c:11f::11', '2001:678:108c:11f::10']
    routes:
      config:
        - destination: 0.0.0.0/0
          next-hop-address: 10.1.48.1      # OQ Node Network gateway
          next-hop-interface: bond0.178
```

Plus, at file top:
```yaml
rendezvousIP: "10.1.48.111"               # one master that bootstraps the install
```

**What is `rendezvousIP`?** During install the agent-installer needs one node to act as a coordination point — it pulls the bootstrap config and orchestrates the others. After bootstrap finishes, this role disappears. The IP must be a master and must be reachable from all other nodes.

**Why VLANs at all?** Because we use bonded NICs and a single physical pair carries multiple VLANs (178 for data, 179 for live-mig). The Linux kernel demuxes by tag.

**Why `active-backup`, not LACP/`802.3ad`?** The proven hub *install* manifest bonds with `active-backup`, which needs no switch-side setup (it just uses one port at a time). `802.3ad` (LACP) only works if the switch ports are configured as a port-channel — our VLAN ticket asked for tagged trunking, not LAG. So OQ's `agent-config.yaml` uses `active-backup` to match hub. (The `gen_nodes-config.sh` helper still emits `802.3ad` blocks for the separate *add-nodes* flow; the MAC values are identical, so we only copy MACs from it.) If the network team later configures LACP port-channels on the OQ node ports, switch every bond `mode` to `802.3ad` + `lacp_rate: fast`.

### 8.5 `generate.sh`

The runner. After you've edited the two manifests:

```bash
mkdir -p $SCRIPT_DIR/install/
cp $SCRIPT_DIR/manifests/* $SCRIPT_DIR/install/
cd $SCRIPT_DIR/install
$SCRIPT_DIR/bin/openshift-install agent create image
```

That last command:
1. Reads `install-config.yaml` + `agent-config.yaml`.
2. Renders all the Kubernetes manifests (Cluster, ClusterImageSet, NMStateConfigs, etc.).
3. Bakes them into an ISO together with RHCOS and a small agent service.
4. Writes `install/agent.x86_64.iso` (~1.2 GB).

That single ISO is everything every node needs.

### 8.6 `additional_nodes/baremetal_lenovo/gen_nodes-config.sh`

A helper script that generates a `nodes-config.yaml` from an XClarity export (`allNodes.json`). For the **initial install** we don't strictly need this — we can write `agent-config.yaml` by hand. But the script is convenient because XClarity's JSON already has each server's MAC addresses and BMC IP, which are the per-host details we'd otherwise type by hand.

For OQ the script has already been adapted (in `OQ/additional_nodes/baremetal_lenovo/gen_nodes-config.sh`):
- VLAN IDs: `bond0.8` → `bond0.178`, `bond1.10` → `bond1.179`
- IPv4 math: XCC `10.0.0.<last>` → bond0 `10.1.<0+48>.<last>` (`+8`→`+48`); added bond1 `10.1.<0+49>.<last>` (`+49`)
- IPv6 prefix: `108c:108` → `108c:130` (node) and `108c:131` (live-mig); the `ffff:`-mapped suffix uses the **node-net** address on both (see the §4.4 gotcha)
- Prefix-length: `23` → `24` (OQ node/live-mig are `/24`, not hub's `/23`)
- The live-migration interface now carries explicit v4 + v6 addresses (hub's version left it address-less)

**Use it only to harvest MACs.** It emits `802.3ad` blocks (for the add-nodes flow); the authoritative initial-install file is the hand-written `manifests/agent-config.yaml` (active-backup). Copy the four `macAddress:` lines per host from its `nodes-config.yaml` output into `agent-config.yaml` — the IPs there are already correct.

---

## 9. The actual install — step by step

Here's the order. Steps in **bold** are blocking — you can't do step N+1 until step N is done.

### Preparation (status as of 2026-05-31)
1. **NetBox** ✅ — all IPs reserved and **verified via API** (32 node IPs + 4 VIPs; VLANs 178/179/180; prefixes 10503–10508). MetalLB corrected to `10.1.50.0/24` + `2001:678:108c:132::/64`.
2. **LB ticket** ✅ — F5 built and confirmed by the network team (4 services; VIPs `.10`/`.11`; real-server pools = 3 masters / 4 infra).
3. **Switch/Firewall ticket** ✅ — VLANs 178/179/180 + L3 gateways + firewall rules confirmed by the network team.
4. **FreeIPA** ✅ — zone already existed; the 4 record pairs (`api`, `api-int`, `apps`, `*.apps`) created 2026-05-31 (see §6.4).

### Repo bring-up
5. ✅ `OQ/` created + adapted — `env.sh`, `prepare.sh`, `generate.sh`, `manifests/install-config.yaml`, `manifests/agent-config.yaml` (all 8 hosts, IPs/VLANs/routes filled, MACs pending), and the adapted `gen_nodes-config.sh`. Validated: 8 hosts, no IP collisions, every address matches NetBox.
6. ✅ `OQ/prepare.sh` pins `ocp_version="4.20.10"` (the version lives here, not in `env.sh`).
7. ⛔ Stage the 4.20.10 `openshift-install` / `oc` / `nmstatectl` archives in `OQ/bin/`, then run `OQ/prepare.sh` (air-gapped — downloads are commented out).

### Generate manifests
8. **Pull XClarity `allNodes.json`** for the 3 Lenovo masters + 4 Lenovo infra nodes (see `additional_nodes/baremetal_lenovo/README.md` step 1).
9. Run the adapted `gen_nodes-config.sh` against `allNodes.json` → produces host blocks with MACs + IPs.
10. Hand-write the `m-vm-029` block (no XClarity, single-NIC or dual-NIC depending on the VM definition) and merge into `agent-config.yaml`. Set `rendezvousIP`.
11. Validate: `nmstatectl gc agent-config.yaml` (catches syntax + invalid VLAN configs).

### Build the ISO
12. `OQ/generate.sh` → produces `OQ/install/agent.x86_64.iso`.

### Stage the ISO
13. Copy to the install web server (the BM nodes mount it over HTTP):
    ```
    scp ./agent.x86_64.iso 10.0.148.10:/home/$USER/openshift-install.iso
    ssh 10.0.148.10 sudo mv /home/$USER/openshift-install.iso /var/www/html
    ```

### Boot the nodes
14. For each **Lenovo** (b-master-004, b-master-005, m-master-004, b-infra-006, b-infra-007, m-infra-006, m-infra-007):
    - XClarity → Server → Actions → Launch Remote Control
    - Media → Mount Media File from Network → URL `http://10.0.148.10/openshift-install.iso`
    - Boot Order → select the mounted ISO → Restart server immediately
15. For **m-vm-029** (VM): mount the ISO in its hypervisor and reboot.
16. Wait. Each node boots, runs the agent, downloads RHCOS, lays it on disk, reboots. 2–3 reboots, 5–6 min per node total. The rendezvous master coordinates.

### Watch from your workstation
```bash
cd OQ/install
../bin/openshift-install --dir=. agent wait-for bootstrap-complete --log-level=info
../bin/openshift-install --dir=. agent wait-for install-complete    --log-level=info
```
The first command returns when etcd is up on the 3 masters and the temporary bootstrap node has gone away. The second returns when all operators are `Available=True`.

You now have `OQ/install/auth/kubeconfig` and `OQ/install/auth/kubeadmin-password`.

---

## 10. Post-install (the cluster is up, now make it ours)

### 10.1 CSRs
Workers join with a self-signed certificate request that an admin has to approve:
```bash
oc get csr | grep Pending
oc adm certificate approve <csr-name>
```
You'll see ~2 per node (kubelet-serving + node-bootstrapper). Approve them all.

### 10.2 Labels

Following the hub convention (see `Sovereign_Cloud/additional_nodes/baremetal_lenovo/README.md`):

```bash
# Zone (datacenter)
oc label node b-master-004.bm.tsrs.corp topology.kubernetes.io/zone=dc-biere
oc label node m-master-004.bm.tsrs.corp topology.kubernetes.io/zone=dc-magdeburg
# … etc.

# Infra role (so workloads tolerating/selecting infra land on these)
for n in b-infra-006 b-infra-007 m-infra-006 m-infra-007; do
  oc label node $n.bm.tsrs.corp node-role.kubernetes.io/infra=""
  oc label node $n.bm.tsrs.corp node-role.kubernetes.io/worker-   # remove "worker" role
done

# VM nodes get an extra label so KubeVirt knows where to schedule
oc label node m-vm-029.bm.tsrs.corp tsrs.corp/vm-networking=true
```

### 10.3 IngressController targeting
Patch the default `IngressController` to run only on infra:
```bash
oc -n openshift-ingress-operator patch ingresscontroller default --type=merge -p '
spec:
  replicas: 4
  nodePlacement:
    nodeSelector:
      matchLabels:
        node-role.kubernetes.io/infra: ""
    tolerations:
      - effect: NoSchedule
        key: node-role.kubernetes.io/infra
        operator: Exists
'
```
(With Argo CD this gets driven from `cluster-apps` — for OQ's initial install, do it imperatively, then bring under GitOps in step 10.5.)

### 10.4 MetalLB
Install the MetalLB operator and create an `IPAddressPool` over `10.1.50.0/24` (minus the `.10`/`.11` which the F5 owns). Plus a corresponding `L2Advertisement` or `BGPAdvertisement` so the rest of the network learns those addresses.

### 10.5 GitOps onboarding
Add OQ as a cluster in the hub Argo's `cluster-apps` app-of-apps and `cluster-specifics`. From this point on, *don't* `oc apply` to OQ — commit + PR instead.

### 10.6 Add the cluster to your shell helpers
Edit `~/.profile_dev`:
```bash
oq() {
  export ENVDIR="$HOME/session/oq" ENVNAME="OQ" \
         URL="https://api.oq.tsrs.corp:6443" \
         USER="rs4798" COLOR="\[\033[0;35m\]"
  source "$HOME/.config_env"
}
```
Then `source ~/.profile_dev && oq` to switch context.

### 10.7 NetBox cleanup
The 8 nodes still have their old `10.1.8.X` (hub-network) primary IPs in NetBox. Once OQ is healthy and the old IPs aren't reachable any more, delete those records and update `primary_ip4`/`primary_ip6` to the OQ ones.

---

## 11. Cheat sheet (the values you'll keep needing)

| | OQ |
|---|---|
| Cluster name | `oq` |
| Base domain | `tsrs.corp` |
| Cluster FQDN | `oq.tsrs.corp` |
| OCP version | `4.20.10` |
| Node Network | VLAN 178, `10.1.48.0/24`, `2001:678:108c:130::/64`, gw `.1` |
| Live Migration | VLAN 179, `10.1.49.0/24`, `2001:678:108c:131::/64`, gw `.1` |
| MetalLB | VLAN 180, `10.1.50.0/24`, `2001:678:108c:132::/64` |
| API VIP | `10.1.50.10` / `2001:678:108c:132:0:ffff:a01:320a` |
| Ingress VIP | `10.1.50.11` / `2001:678:108c:132:0:ffff:a01:320b` |
| Rendezvous IP (proposed) | `10.1.48.111` (b-master-004) |
| FreeIPA DNS servers | `10.1.31.11`, `10.1.31.10`, v6 `…11f::11`, `…11f::10` |
| FreeIPA zone to create | `oq.tsrs.corp` |
| Install web server (ISO host) | `10.0.148.10` |

---

## 12. Glossary

- **Agent-based installer** — the modern OCP installer flow that produces a single self-contained ISO. Replaces the old UPI/PXE dance.
- **Bootstrap node** — a short-lived node (often co-located on rendezvous) that initialises etcd before masters take over. Lives ~30 minutes during install, then is discarded.
- **CSR** (CertificateSigningRequest) — Kubernetes object representing a node's request to join the cluster. Admin approves it; cluster issues the kubelet cert.
- **Ignition** — the Fedora CoreOS configuration format. Each node fetches its Ignition once at first boot to write disk layout, systemd units, etc.
- **MCS** — Machine Config Server. Hosted on masters, port 22623. Serves Ignition to nodes.
- **Operator** — a Kubernetes-native software pattern: a Deployment that runs a controller that reconciles CRDs (Custom Resources) toward a desired state. OCP is mostly built out of Operators.
- **Platform: none** — install-config setting meaning "no cloud/baremetal provider integration; user-managed DNS + LB".
- **RHCOS** — Red Hat Enterprise Linux CoreOS. The immutable OS that runs on every OCP node.
- **Route** — OpenShift's term for an Ingress rule. Created in a namespace, admitted by the IngressController, advertised via the wildcard DNS.
- **VIP** — Virtual IP. An address that doesn't belong to a single host but is served by an LB or VRRP-like mechanism.
- **VLAN** — IEEE 802.1Q tag on Ethernet frames. Multiple VLANs share one physical wire; the switch and kernel demux by tag.

---

## 13. When something goes wrong

| Symptom | Where to look first |
|---|---|
| Node never appears in `oc get nodes` after boot | Check it can reach `api-int.oq.tsrs.corp:22623` and `:6443` — usually a DNS or firewall issue |
| `wait-for bootstrap-complete` hangs | `journalctl -u release-image.service` on the rendezvous node; usually pull-secret or registry reach |
| Ingress works on one node but not the LB | F5 health-check is failing because the route hasn't been admitted yet — the operator may still be coming up |
| CSR list empty even though node is up | Node didn't reach the API server; same as line 1 |
| Cluster Operator stuck `Available=False` | `oc get co` shows which one; `oc describe co/<name>` has events |

For a deeper bootstrap log dump:
```
ssh -i ~/.ssh/<key> core@<rendezvous-ip>
sudo journalctl -u bootkube.service
```

---
