# VPC — Visual Reference (Mermaid Diagrams)

> Visual companion to [Networking.md](./Networking.md). Render in VS Code (Markdown Preview Enhanced or "Markdown Preview Mermaid Support" plugin) or directly on GitHub.

## Table of Contents

1. [VPC Anatomy](#1-vpc-anatomy--the-building-blocks)
2. [Public vs Private Subnet](#2-public-vs-private-subnet--what-makes-them-different)
3. [NAT Gateway Flow](#3-nat-gateway-flow--private-ec2-reaches-the-internet)
4. [Security Groups vs NACLs](#4-security-groups-vs-nacls)
5. [VPC Endpoints — Gateway vs Interface](#5-vpc-endpoints--gateway-vs-interface)
6. [PrivateLink — Expose YOUR Service](#6-privatelink--exposing-your-service-to-other-vpcs)
7. [VPC Peering](#7-vpc-peering--bidirectional-non-transitive)
8. [Transit Gateway](#8-transit-gateway--hub-and-spoke)
9. [Hybrid Connectivity — VPN vs DX vs DX+VPN](#9-hybrid-connectivity--vpn-vs-dx-vs-dxvpn)
10. [DX Maximum Resiliency](#10-dx-maximum-resiliency)
11. [Bastion Host vs Session Manager](#11-bastion-host-vs-session-manager)
12. [Hybrid DNS — Route 53 Resolver](#12-hybrid-dns--route-53-resolver)
13. [Full 3-Tier App Reference Architecture](#13-the-full-picture--3-tier-app-in-one-vpc)
14. [Egress-Only IGW (IPv6)](#14-egress-only-internet-gateway-ipv6)
15. [VPN CloudHub](#15-vpn-cloudhub--multi-branch-hub-and-spoke)
16. [AWS Network Firewall](#16-aws-network-firewall--deep-packet-inspection)
17. [Client VPN](#17-client-vpn--remote-user-access)
18. [The 4 Mental Rules for the Exam](#18-the-4-mental-rules-to-take-to-the-exam)

---

## 1. VPC anatomy — the building blocks

```mermaid
graph TB
    subgraph Region["AWS Region (e.g., us-east-1)"]
        subgraph VPC["VPC — 10.0.0.0/16 (logical datacenter)"]
            subgraph AZA["Availability Zone A"]
                PubA["Public Subnet A<br/>10.0.1.0/24"]
                PrivA["Private Subnet A<br/>10.0.10.0/24"]
            end
            subgraph AZB["Availability Zone B"]
                PubB["Public Subnet B<br/>10.0.2.0/24"]
                PrivB["Private Subnet B<br/>10.0.20.0/24"]
            end
        end
    end

    style VPC fill:#e8f4f8,stroke:#2c7da0
    style PubA fill:#d4f1c5,stroke:#52b788
    style PubB fill:#d4f1c5,stroke:#52b788
    style PrivA fill:#fcdada,stroke:#c1121f
    style PrivB fill:#fcdada,stroke:#c1121f
```

**Key rules:**
- VPC = **one Region**
- Subnet = **one AZ** (cannot span AZs)
- Green = public (has route to IGW). Red = private (no IGW route).
- AWS reserves 5 IPs per subnet (`.0`, `.1`, `.2`, `.3`, `.255`).

---

## 2. Public vs Private subnet — what makes them different

```mermaid
graph LR
    subgraph VPC["VPC"]
        subgraph PubSub["🟢 PUBLIC SUBNET"]
            PubEC2["EC2<br/>Public IP: 54.x.x.x<br/>Private: 10.0.1.5"]
            PubRT["Route Table:<br/>10.0.0.0/16 → local<br/>0.0.0.0/0 → IGW ✅"]
        end
        subgraph PrivSub["🔴 PRIVATE SUBNET"]
            PrivEC2["EC2<br/>No Public IP<br/>Private: 10.0.10.5"]
            PrivRT["Route Table:<br/>10.0.0.0/16 → local<br/>0.0.0.0/0 → NAT GW"]
        end
        IGW(["IGW"])
    end

    PubEC2 -.routes via.-> PubRT
    PrivEC2 -.routes via.-> PrivRT
    PubRT --> IGW
    IGW --> Internet((Internet))

    style PubSub fill:#d4f1c5
    style PrivSub fill:#fcdada
```

**The only difference:** the route table. Public subnet has `0.0.0.0/0 → IGW`. Private doesn't.

---

## 3. NAT Gateway flow — private EC2 reaches the internet

```mermaid
graph LR
    subgraph VPC["VPC"]
        subgraph PubSub["🟢 PUBLIC SUBNET"]
            NAT["NAT Gateway<br/>EIP: 54.x.x.x"]
        end
        subgraph PrivSub["🔴 PRIVATE SUBNET"]
            EC2["EC2<br/>10.0.10.5<br/>(no public IP)"]
        end
        IGW(["IGW"])
    end
    Internet((Internet<br/>8.8.8.8))

    EC2 -->|"1.src=10.0.10.5<br/>dst=8.8.8.8"| NAT
    NAT -->|"2. NAT rewrites src<br/>src=54.x.x.x<br/>dst=8.8.8.8"| IGW
    IGW -->|"3."| Internet
    Internet -.->|"4. reply<br/>dst=54.x.x.x"| IGW
    IGW -.->|"5."| NAT
    NAT -.->|"6. NAT looks up table<br/>forwards to 10.0.10.5"| EC2

    style PubSub fill:#d4f1c5
    style PrivSub fill:#fcdada
```

**Why NAT GW must be in a public subnet:** it needs `0.0.0.0/0 → IGW` itself to forward the rewritten packet.

---

## 4. Security Groups vs NACLs

```mermaid
graph TB
    subgraph Subnet["Subnet (NACL applies HERE — at subnet boundary)"]
        NACL["🛡️ NACL<br/>• Stateless<br/>• Allow + DENY<br/>• Rule order matters<br/>• Need rules BOTH ways"]
        subgraph Instance["EC2 Instance"]
            ENI["ENI"]
            SG["🛡️ Security Group<br/>• Stateful<br/>• Allow ONLY<br/>• All rules evaluated<br/>• Return traffic auto"]
        end
    end

    Internet((Internet)) -->|"1. inbound<br/>checked by NACL first"| NACL
    NACL -->|"2. then by SG"| SG
    SG --> ENI

    style NACL fill:#fff3cd
    style SG fill:#d1ecf1
```

**Order on inbound:** Internet → **NACL** → **SG** → EC2.
**Order on outbound:** EC2 → SG → NACL → Internet.

**Exam triggers:**
- "Block a specific IP" → NACL (only one with Deny)
- "Timeout" → usually SG misconfiguration
- "Permission denied" → usually NACL
- "Custom NACL, instances can't talk" → deny-all default, add allow + ephemeral 1024-65535

---

## 5. VPC Endpoints — Gateway vs Interface

```mermaid
graph LR
    subgraph VPC["VPC"]
        subgraph PrivSub["🔴 PRIVATE SUBNET"]
            EC2["EC2"]
            IE["Interface Endpoint<br/>(ENI with private IP)<br/>💰 hourly + data"]
        end
        GE["Gateway Endpoint<br/>(route table entry)<br/>✅ FREE"]
    end

    EC2 -->|"S3 / DynamoDB"| GE
    EC2 -->|"SQS / SNS / CW /<br/>KMS / Secrets / etc."| IE

    GE -->|via AWS<br/>private network| S3[(S3 / DynamoDB)]
    IE -->|via AWS<br/>private network| OtherSvc[(SQS / SNS / etc.)]

    style PrivSub fill:#fcdada
    style GE fill:#c8e6c9
    style IE fill:#ffe0b2
```

**Memorize:** Gateway Endpoint = **S3 + DynamoDB only, free, route table entry**. Interface Endpoint = **everything else, costs $$, creates an ENI**.

---

## 6. PrivateLink — exposing YOUR service to other VPCs

```mermaid
graph LR
    subgraph ConsumerVPC["👤 CONSUMER VPC (customer)"]
        CEC2["EC2"]
        CIE["Interface Endpoint<br/>(ENI in consumer)"]
    end

    subgraph ProviderVPC["🏢 PROVIDER VPC (you)"]
        ES["VPC Endpoint Service"]
        NLB["NLB"]
        AppEC2["Your App EC2s"]
    end

    CEC2 --> CIE
    CIE -->|AWS private<br/>network| ES
    ES --> NLB
    NLB --> AppEC2

    style ConsumerVPC fill:#e1f5fe
    style ProviderVPC fill:#fff9c4
```

**Pattern to memorize:** Provider NLB + Endpoint Service ← Consumer Interface Endpoint. No peering, no public IPs, works across accounts.

---

## 7. VPC Peering — bidirectional, non-transitive

```mermaid
graph LR
    VPCA["VPC A<br/>10.0.0.0/16"]
    VPCB["VPC B<br/>10.1.0.0/16"]
    VPCC["VPC C<br/>10.2.0.0/16"]

    VPCA <-->|"Peering A↔B"| VPCB
    VPCB <-->|"Peering B↔C"| VPCC
    VPCA -.->|"❌ NOT transitive<br/>A cannot reach C"| VPCC

    style VPCA fill:#e1f5fe
    style VPCB fill:#e1f5fe
    style VPCC fill:#e1f5fe
```

**Rules:**
- Non-transitive (A→B and B→C does NOT give A→C)
- No CIDR overlap
- Must update **route tables on BOTH sides**
- Same region or cross-region

---

## 8. Transit Gateway — hub-and-spoke

```mermaid
graph TB
    TGW(("🌐 Transit Gateway<br/>(transitive hub)"))
    VPCA["VPC A"]
    VPCB["VPC B"]
    VPCC["VPC C"]
    VPCD["VPC D"]
    VPN["Site-to-Site VPN<br/>(on-prem)"]
    DX["Direct Connect<br/>(on-prem)"]

    TGW <--> VPCA
    TGW <--> VPCB
    TGW <--> VPCC
    TGW <--> VPCD
    TGW <--> VPN
    TGW <--> DX

    style TGW fill:#ff9800,color:#fff
```

**Use it when:** 3+ VPCs need to talk, or you want a clean place to hang on-prem connectivity. Replaces the explosion of N×(N-1)/2 peering connections.

---

## 9. Hybrid connectivity — VPN vs DX vs DX+VPN

```mermaid
graph LR
    subgraph OnPrem["🏢 On-Premises"]
        CGW["Customer Gateway<br/>(your router)"]
    end

    subgraph AWS["AWS"]
        VGW["VGW / TGW"]
        VPC["VPC"]
    end

    CGW -->|"Site-to-Site VPN<br/>🔒 Encrypted (IPSec)<br/>📶 Over public internet<br/>⚡ 1.25 Gbps/tunnel"| VGW
    CGW -->|"Direct Connect<br/>🚫 NOT encrypted<br/>🔌 Dedicated fiber<br/>⚡ 1/10/100 Gbps<br/>⏱️ Weeks to set up"| VGW
    CGW -->|"DX + VPN<br/>🔒 Encrypted (IPSec)<br/>🔌 Over dedicated fiber<br/>= Compliance + speed"| VGW
    VGW --> VPC

    style OnPrem fill:#fff3cd
    style AWS fill:#e1f5fe
```

**Triggers:**
- "Encrypted, cheap" → VPN
- "Consistent latency, high bandwidth" → DX
- "Encrypted + dedicated" → DX + VPN over DX
- "Increase VPN bandwidth" → ECMP (only with **TGW** + VPN, NOT plain VGW)

---

## 10. DX maximum resiliency

```mermaid
graph TB
    subgraph OnPrem["🏢 On-Premises"]
        Router["Your Router"]
    end

    subgraph DXLoc1["DX Location 1"]
        DX1["Connection 1"]
        DX2["Connection 2"]
    end

    subgraph DXLoc2["DX Location 2"]
        DX3["Connection 3"]
        DX4["Connection 4"]
    end

    subgraph AWSRegion["AWS Region"]
        VPC["VPC"]
    end

    Router --> DX1 --> AWSRegion
    Router --> DX2 --> AWSRegion
    Router --> DX3 --> AWSRegion
    Router --> DX4 --> AWSRegion

    style DXLoc1 fill:#ffe0b2
    style DXLoc2 fill:#ffe0b2
```

**Max resiliency = 2 connections × 2 locations = 4 total.** Survives device + location failure.

---

## 11. Bastion Host vs Session Manager

```mermaid
graph TB
    subgraph PubSub["🟢 PUBLIC SUBNET"]
        Bastion["Bastion Host EC2<br/>📌 You patch it<br/>📌 SG: SSH from YOUR IP only<br/>📌 Port 22 OPEN"]
    end

    subgraph PrivSub["🔴 PRIVATE SUBNET"]
        Target["Target EC2<br/>(SSM Agent installed)"]
    end

    Admin1["👤 Admin (Bastion)"] -->|"SSH :22 ❌ open port"| Bastion
    Bastion -->|"SSH"| Target

    Admin2["👤 Admin (SSM)"] -->|"IAM-authenticated<br/>✅ no open ports<br/>📋 logged to CW/S3"| SSM["AWS SSM<br/>Session Manager"]
    SSM -. "via SSM Agent" .-> Target

    style PubSub fill:#d4f1c5
    style PrivSub fill:#fcdada
    style SSM fill:#c8e6c9
```

**Exam trigger:** "Most secure access to private instances" → **Session Manager** (always). Bastion is the trap answer.

---

## 12. Hybrid DNS — Route 53 Resolver

```mermaid
graph LR
    subgraph OnPrem["🏢 On-Premises"]
        OnPremDNS["On-Prem DNS<br/>(corp.internal)"]
    end

    subgraph AWS["AWS VPC"]
        IE["Inbound Endpoint<br/>(ENI for on-prem<br/>to query AWS)"]
        OE["Outbound Endpoint<br/>(ENI for AWS<br/>to query on-prem)"]
        PHZ["Route 53<br/>Private Hosted Zone<br/>(aws.internal)"]
    end

    OnPremDNS -->|"resolve aws.internal"| IE
    IE --> PHZ
    OE -->|"resolve corp.internal"| OnPremDNS

    style OnPrem fill:#fff3cd
    style AWS fill:#e1f5fe
```

**Inbound endpoint** = on-prem queries AWS. **Outbound endpoint** = AWS queries on-prem.

---

## 13. The full picture — 3-tier app in one VPC

```mermaid
graph TB
    Internet((Internet))

    subgraph VPC["VPC 10.0.0.0/16"]
        IGW(["IGW"])

        subgraph PubSubA["🟢 Public Subnet AZ-A"]
            ALB1["ALB"]
            NAT["NAT GW (EIP)"]
        end
        subgraph PubSubB["🟢 Public Subnet AZ-B"]
            ALB2["ALB"]
        end

        subgraph PrivAppA["🔴 Private App Subnet AZ-A"]
            App1["App EC2"]
        end
        subgraph PrivAppB["🔴 Private App Subnet AZ-B"]
            App2["App EC2"]
        end

        subgraph PrivDBA["⚫ Isolated DB Subnet AZ-A"]
            DB1["RDS Primary"]
        end
        subgraph PrivDBB["⚫ Isolated DB Subnet AZ-B"]
            DB2["RDS Standby<br/>(Multi-AZ)"]
        end

        GE["S3 Gateway Endpoint"]
    end

    Internet --> IGW
    IGW --> ALB1
    IGW --> ALB2
    ALB1 --> App1
    ALB2 --> App2
    App1 --> DB1
    App2 --> DB1
    DB1 -. "sync replication" .- DB2
    App1 -->|patches| NAT
    App2 -->|patches| NAT
    NAT --> IGW
    App1 -->|"private S3 access"| GE
    GE --> S3[(S3)]

    style PubSubA fill:#d4f1c5
    style PubSubB fill:#d4f1c5
    style PrivAppA fill:#fcdada
    style PrivAppB fill:#fcdada
    style PrivDBA fill:#424242,color:#fff
    style PrivDBB fill:#424242,color:#fff
```

**Read this diagram as the AWS gold-standard reference architecture.** If a question shows you a deployment, mentally match it to this — anything missing is usually the answer.

> **Production HA note:** For full AZ-failure resilience, deploy **one NAT Gateway per AZ** (the diagram shows one for simplicity). NAT GW is **zonal** — if its AZ fails, instances in other AZs lose internet unless they have their own NAT GW. Exam trigger: *"highly available NAT"* → one per AZ.

---

## 14. Egress-Only Internet Gateway (IPv6)

```mermaid
graph LR
    subgraph VPC["VPC (dual-stack: IPv4 + IPv6)"]
        subgraph PrivSub["🔴 PRIVATE SUBNET (IPv6)"]
            EC2["EC2<br/>IPv6: 2001:db8::1"]
        end
        EIGW(["Egress-Only IGW<br/>(IPv6 only)"])
    end

    EC2 -->|"outbound IPv6"| EIGW
    EIGW -->|"✅ outbound allowed"| Internet((IPv6 Internet))
    Internet -. "❌ inbound BLOCKED" .-> EIGW

    style PrivSub fill:#fcdada
    style EIGW fill:#fff3cd
```

**Why it exists:** IPv6 has no private address range — every IPv6 is publicly routable. **Egress-Only IGW** is the IPv6 equivalent of NAT GW — outbound only, no inbound.

**Exam trigger:** *"IPv6 instances need outbound internet but must not be reachable from the internet"* → **Egress-Only IGW**.

---

## 15. VPN CloudHub — multi-branch hub-and-spoke

```mermaid
graph TB
    subgraph AWS["AWS"]
        VGW["VGW<br/>(hub)"]
    end

    Branch1["🏢 Branch Office 1<br/>(CGW)"]
    Branch2["🏢 Branch Office 2<br/>(CGW)"]
    Branch3["🏢 Branch Office 3<br/>(CGW)"]

    Branch1 <-->|"Site-to-Site VPN"| VGW
    Branch2 <-->|"Site-to-Site VPN"| VGW
    Branch3 <-->|"Site-to-Site VPN"| VGW

    Branch1 -. "branch-to-branch<br/>via VGW" .-> Branch2
    Branch2 -. "branch-to-branch<br/>via VGW" .-> Branch3

    style AWS fill:#e1f5fe
    style VGW fill:#ff9800,color:#fff
```

**The trick:** each branch only sets up ONE VPN to AWS, and AWS routes branch-to-branch traffic for them. No mesh of branch-to-branch VPNs needed.

**Exam trigger:** *"Connect multiple branch offices to each other through AWS"* → **VPN CloudHub**.

---

## 16. AWS Network Firewall — deep packet inspection

```mermaid
graph LR
    Internet((Internet))

    subgraph VPC["VPC"]
        subgraph FWSub["🛡️ FIREWALL SUBNET"]
            FW["AWS Network Firewall<br/>📌 Stateful L3-L7<br/>📌 IDS/IPS (Suricata)<br/>📌 Domain filtering"]
        end
        subgraph PrivSub["🔴 PRIVATE SUBNET"]
            EC2["EC2"]
        end
        IGW(["IGW"])
    end

    Internet --> IGW
    IGW --> FW
    FW -->|"inspected traffic"| EC2
    EC2 --> FW
    FW --> IGW

    style FWSub fill:#fff3cd
    style PrivSub fill:#fcdada
```

**Key point:** Network Firewall lives in its **own dedicated subnet**, and route tables direct traffic *through* it before it reaches workload subnets.

**Network Firewall vs SG/NACL:**
- SG/NACL = simple IP/port allow/deny (L3/L4).
- **Network Firewall = packet payload inspection, IDS/IPS, domain-name filtering, malicious-pattern detection (L3-L7).**

**Exam triggers:** *"Inspect traffic for malicious payloads"*, *"IDS/IPS at VPC level"*, *"Filter outbound by domain name"* → **AWS Network Firewall**.

---

## 17. Client VPN — remote user access

```mermaid
graph LR
    User1["👤 Remote Employee<br/>(OpenVPN client)"]
    User2["👤 Remote Employee"]

    subgraph AWS["AWS"]
        CVE["Client VPN Endpoint<br/>(associated with subnets)"]
        subgraph VPC["VPC"]
            EC2["Private EC2"]
            RDS[(RDS)]
        end
    end

    User1 -->|"TLS tunnel<br/>(OpenVPN)"| CVE
    User2 -->|"TLS tunnel"| CVE
    CVE --> EC2
    CVE --> RDS

    style AWS fill:#e1f5fe
    style VPC fill:#fcdada
```

**Difference from Site-to-Site VPN:**
- **Client VPN** = **user-to-AWS** (individual remote workers, OpenVPN).
- **Site-to-Site VPN** = **network-to-network** (whole office to AWS, IPSec).

**Exam trigger:** *"Remote employees need VPN access to private AWS resources"* → **Client VPN**.

---

## 18. The 4 mental rules to take to the exam

1. **A subnet is "public" because of its route table, not its name.** `0.0.0.0/0 → IGW` makes it public.
2. **A private EC2 needs NO public IP.** The NAT GW lends it its own EIP via translation.
3. **NACL = subnet level, stateless, deny-capable. SG = ENI level, stateful, allow-only.**
4. **Peering ≠ transitive. TGW = transitive.** If the question says 3+ VPCs need to talk, the answer is TGW.
