# Networking — Speed Refresher (10 min)

> All exam-critical facts from `Networking.md`, no prose, no scenarios. Read top-to-bottom once, then take the quiz in `Recall.md`.

---

## 1. VPC Anatomy

- VPC = **one Region**. Subnet = **one AZ**.
- CIDR: min `/28` (16 IPs), max `/16` (65,536). Up to **5 CIDR blocks** per VPC.
- AWS reserves **5 IPs per subnet** (.0 net, .1 router, .2 DNS, .3 future, .255 broadcast).
  - `/28` subnet → 16 − 5 = **11 usable**.
- **Public subnet** = route table has `0.0.0.0/0 → IGW`. **Private** = does not.
- **Trap:** Instance has Public IP but no internet → check Route Table for IGW route.
- Longest Prefix Match wins (more specific route).

---

## 2. Security — SG vs NACL ⭐ (most common topic)

| Feature | Security Group | NACL |
|---|---|---|
| Level | ENI | Subnet |
| State | **Stateful** | **Stateless** (need inbound + outbound) |
| Rules | **Allow only** | **Allow + Deny** |
| Order | All evaluated | **Lowest # wins** |
| Default | Deny in / Allow out | Default NACL: allow all · Custom NACL: **deny all** |
| Trigger | "Timeout" | "Permission denied" |
| Use case | App security | **Block specific IP** |

- NACL stateless → must allow **ephemeral ports 1024-65535** for return traffic.
- **Trap:** Block specific IP → **NACL** (SG can't deny).
- **Trap:** Custom NACL → starts deny-all, must add rules.

**AWS Network Firewall** = VPC-level deep packet inspection / IDS/IPS / domain filtering (L3-L7, Suricata). Beyond SG/NACL.

---

## 3. Connectivity Out

- **IGW**: 1 per VPC, HA, lets public subnets reach internet.
- **NAT Gateway**: managed, in **public subnet**, up to 45 Gbps, **zonal** (1 per AZ for HA), hourly + data fees. Outbound only for **private** IPv4.
- **NAT Instance**: legacy EC2; **disable Source/Dest Check**. Not a Bastion.
- **Egress-Only IGW**: IPv6 outbound only (IPv6 has no private range).

---

## 4. VPC Endpoints (PrivateLink)

| Type | Services | Mechanism | Cost |
|---|---|---|---|
| **Gateway Endpoint** | **S3, DynamoDB only** | Route table entry | **Free** |
| **Interface Endpoint** | Everything else (SQS, SNS, CW, KMS, …) | ENI + DNS | **$$$** (hourly + data) |

**AWS PrivateLink (Endpoint Service)** = expose YOUR service to other VPCs/accounts privately. Pattern: **provider NLB + Endpoint Service → consumer Interface Endpoint**. No peering, no IGW, no NAT, no public IP.

---

## 5. Connecting Networks

- **VPC Peering**: 2 VPCs, private IP, **not transitive**, **no CIDR overlap**, same or cross-region.
- **Transit Gateway (TGW)**: hub-and-spoke, **transitive**, scales to thousands of VPCs + on-prem. **TGW Peering** = cross-region.
- **Site-to-Site VPN**: VGW (AWS side) + CGW (your router). Over **public internet**, encrypted, **1.25 Gbps per tunnel**.
- **ECMP** (multi-tunnel bandwidth boost): **only with TGW + VPN**, NOT with VGW.
- **VPN CloudHub**: branch-offices hub-and-spoke through one VGW.
- **Direct Connect (DX)**: dedicated fiber, no internet, 1/10/100 Gbps, weeks to set up, **private but NOT encrypted**.
- **DX + VPN**: run S2S VPN over DX for encryption (compliance).
- **DX Resiliency**:
  - High = 2 connections, 2 locations.
  - **Maximum = 2 connections × 2 locations = 4 total** (mission-critical).
- **Client VPN**: end-user remote access (OpenVPN client → VPC). User-to-AWS.
- **Route 53 Resolver** (hybrid DNS):
  - **Inbound Endpoint** = on-prem queries AWS private zones.
  - **Outbound Endpoint** = AWS queries on-prem DNS.
  - Resolver Rules route domain → DNS.
- **VPC Sharing (RAM)**: share subnets across accounts in an Org — no peering needed.

---

## 6. Monitoring

- **VPC Flow Logs**: metadata only (src/dst IP, port, ALLOW/DENY). **NO payload**. Ships to **S3 or CloudWatch Logs**.

---

## 7. Bastion Host vs Session Manager

| Feature | Bastion | **Session Manager (SSM)** |
|---|---|---|
| Where | EC2 in public subnet | SSM feature |
| Port 22 | Open | **None** |
| Infra | You patch it | None |
| Access | SG (IP) | **IAM** |
| Logging | DIY | **CloudWatch / S3 built-in** |

- **Most secure private access / no SSH / audit shell commands → Session Manager.** Bastion is the trap answer.

---

## ⚡ 20 Trigger → Answer (read aloud once)

1. Block specific IP → **NACL**
2. Free private S3 access → **Gateway Endpoint**
3. Private SQS/Kinesis/CW → **Interface Endpoint**
4. Connect 50 VPCs → **Transit Gateway**
5. Connect 2 VPCs → **VPC Peering** (no CIDR overlap)
6. Private instance outbound internet → **NAT Gateway in public subnet**
7. Public IP but no internet → **Route Table missing IGW route**
8. Bastion → public-subnet EC2, SSH from your IP only
9. NAT Instance → legacy, disable Source/Dest Check
10. Peering broken → check Route Tables BOTH sides + SGs
11. Expose your service privately to other VPCs → **NLB + PrivateLink Endpoint Service**
12. Encrypt Direct Connect → **S2S VPN over DX**
13. Highest DX resiliency → **2 connections × 2 locations (4 total)**
14. IPv6 outbound only → **Egress-Only IGW**
15. Deep packet inspection / IDS/IPS → **AWS Network Firewall**
16. Increase VPN bandwidth → **ECMP via TGW** (not VGW)
17. Hybrid DNS → **Route 53 Resolver** (Inbound/Outbound endpoints)
18. Share subnets across accounts → **RAM**
19. Remote user VPN → **Client VPN**
20. Custom NACL blocks all → deny by default + need **ephemeral ports 1024-65535**
21. Connect branch offices through AWS → **VPN CloudHub**
22. Most secure private access → **SSM Session Manager**

---

## 🎯 Memorize these 5 packet flows

1. **Private EC2 → internet:** Private subnet → NAT GW (public subnet) → IGW → Internet.
2. **Private EC2 → S3 privately:** Private subnet → Gateway Endpoint → S3 (no internet).
3. **Private EC2 → SQS privately:** Private subnet → Interface Endpoint (ENI) → SQS.
4. **On-prem → AWS encrypted dedicated:** On-prem → CGW → DX + VPN over DX → VGW/TGW → VPC.
5. **Other VPC → your service privately:** Consumer VPC Interface Endpoint → PrivateLink → NLB → your service.

If you can sketch these on paper, networking is yours.


1- B
2- B
3- 