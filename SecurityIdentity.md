# Security & Identity

> **📌 Scope note:** AWS tests these with *"determine WHEN to use"* — you pick the right service / policy type / federation method, you don't author policy JSON or call crypto APIs. Domain 1 (Security) is the **heaviest exam domain at 30%**, so this chapter stays broad. If it's not here, the exam doesn't ask it.

### **SECTION 1: IAM (IDENTITY AND ACCESS MANAGEMENT)**

*The Foundation of AWS Security.*

### **1. IAM Overview**

- **The Rule:** Control **who** can access **what** in AWS. Global service (not region-specific), free.
- **Core Components:** Users (people/services, long-term credentials), Groups (collections of users), Roles (temporary credentials for services/federated users), Policies (JSON permission documents).

---

### **2. IAM Policies (The Permissions)**

- A policy is a JSON document with **Effect** (Allow/Deny), **Action** (e.g. `s3:GetObject`), **Resource** (ARN), and an optional **Condition** (IP range, MFA, time). You won't write JSON on the exam — you pick *which type* of policy and reason about *what wins* (see §7).

---

### **3. Policy Types**

### **A. Identity-Based Policies**

- Attached to **users, groups, or roles**. Types: AWS Managed (pre-built, e.g. `AdministratorAccess`), Customer Managed (your custom policies), Inline (embedded in one identity, not reusable).

### **B. Resource-Based Policies**

- Attached to **resources** (S3 bucket policy, SQS queue policy). Used for cross-account access and letting services access resources.

**Exam Example:** "S3 bucket allows CloudFront to access" → Bucket Policy (Resource-based).

- Resource policy : you attach it to an item to determine who's allowed to access it and what can it exactly do with it, it doesnt give any power to the item, for that you need an IAM Role.

---

### **4. Permission Boundaries**

**The Rule:** Set **maximum permissions** a user/role can have (Even if policies grant more).

**Use Case:** Delegate admin permissions safely (Admin can't escalate beyond boundary).

**Exam Trigger:** "Prevent users from gaining admin access" → Permission Boundaries.

---

### **5. IAM Roles (The Temporary Credentials)**

**The Rule:** Temporary credentials (no password), **assumed** by users/services. When assumed, STS returns temporary credentials (Access Key, Secret Key, Session Token) that expire in 15 min – 12 hours.

- **Common Roles:** EC2 Instance Role (via Instance Profile), Lambda Execution Role, Cross-Account Role, Service Role (a role an AWS service assumes on your behalf).

**Exam Trigger:** "EC2 needs S3 access without hardcoded keys" → IAM Role (Instance Profile).

---

### **6. IAM Best Practices**

- **Root Account:** Lock it. Enable MFA. Don't use for daily tasks.
- **Least Privilege:** Grant minimum permissions needed.
- **Use Roles:** Never embed credentials in code.
- **Enable MFA:** Especially for privileged users.
- **Rotate Credentials:** Use IAM Credential Report to audit.
- **Use Groups:** Don't attach policies directly to users.

---

### **7. IAM Evaluation Logic**

**The Rule:** **Explicit Deny > Explicit Allow > Implicit Deny (Default)**. Check explicit Deny first (→ Deny), then explicit Allow (→ Allow), else default Deny.

**Exam Trap:** "User has Allow in one policy, Deny in another" → **Deny wins**.

---

### **SECTION 2: STS (SECURITY TOKEN SERVICE)**

*Temporary credentials via AssumeRole.*

### **1. STS Overview**

- **The Rule:** Issue **temporary credentials** (Access Key, Secret Key, Session Token) for IAM Roles.
- **Global Service:** Single endpoint `sts.amazonaws.com` (Regional endpoints also available).

---

### **2. Key STS API Calls**

- **AssumeRole:** Assume a role within your account or cross-account. Most common. Used by EC2 Instance Profiles, Lambda Execution Roles, cross-account access.
- **AssumeRoleWithWebIdentity:** Exchange a web identity token (Google, Facebook, Amazon) for temporary AWS credentials. Cognito Identity Pools uses this behind the scenes. Exam shows this API name when discussing mobile/web apps accessing AWS.
- **AssumeRoleWithSAML:** Exchange a SAML assertion from corporate IdP (Active Directory) for temporary AWS credentials. Exam trigger: "enterprise SSO to AWS Console via SAML."

**Exam Trap:** Don't confuse the three. Web identity = social/OpenID logins. SAML = corporate federation. Plain AssumeRole = AWS-to-AWS or CLI usage.

---

### **SECTION 3: IAM IDENTITY CENTER (FORMERLY AWS SSO)**

*Single sign-on across AWS accounts.*

### **1. IAM Identity Center Overview**

- **The Rule:** **Centralized SSO** for multiple AWS accounts in an Organization plus business apps (Salesforce, Slack, custom SAML).
- **Identity Source:** Built-in identity store, Active Directory (AWS Managed AD or AD Connector), or external IdP (Okta, Azure AD).
- **Permission Sets:** Define permissions once, assign to users/groups across accounts. User signs in once via the portal and gets STS temporary credentials per account.

**Exam Trigger:** "SSO across AWS accounts" → IAM Identity Center. "Centralized access management for Organization" → IAM Identity Center.

**Exam Trap:** IAM Identity Center replaces the old "AWS SSO" name. Same service, new name. Exam may use either.

---

### **SECTION 3B: AWS DIRECTORY SERVICE (FEDERATING AD)**

*The guide explicitly tests "when to federate a directory service with IAM roles."*

- **AWS Managed Microsoft AD:** A real, AWS-run Active Directory in your VPC. Pick it when you need actual AD features in AWS (LDAP, GPOs, trust with on-prem AD) — e.g. Windows workloads, SQL Server, .NET apps.
- **AD Connector:** A *proxy* (no directory of its own) that redirects auth to your **existing on-prem AD**. Pick it to let AWS use on-prem AD without copying/syncing users to the cloud.
- **Simple AD:** Small, standalone, Samba-based directory. Pick it for basic/low-cost needs with no on-prem AD and no advanced AD features.

**Exam Trigger:** "Use existing on-prem Active Directory for AWS" → **AD Connector** (proxy) or set up a **trust** with Managed Microsoft AD. "Need a full managed AD in AWS for Windows/SQL workloads" → AWS Managed Microsoft AD. Then federate to AWS access via **IAM Identity Center** (or SAML → IAM roles).

---

### **SECTION 4: KMS (KEY MANAGEMENT SERVICE)**

*Managed encryption keys.*

### **1. KMS Overview**

- **The Rule:** Create and manage **encryption keys**. Integrated with most AWS services.
- **Encryption:** Data at rest (EBS, S3, RDS) and in transit.
- **Audit:** All key usage logged in CloudTrail.

---

### **2. KMS Key Types**

- **AWS Managed Keys:** Created/managed by AWS (e.g., `aws/s3`), auto-rotated yearly, free, limited control (can't delete or change key policy).
- **Customer Managed Keys (CMK):** You create and manage; optional rotation (auto-yearly or manual); $1/month per key + API calls; full control (key policy, enable/disable, delete).

**Exam Trigger:** "Need to rotate key on demand" → Customer Managed Key.

---

### **3. KMS Key Policies**

**The Rule:** Control who can use/manage keys (like IAM policies, but for keys). Default key policy gives the root account full access; custom key policy gives explicit control (needed for cross-account access).

**Exam Trigger:** "Grant cross-account access to KMS key" → Custom Key Policy.

---

### **4. Envelope Encryption**

**The Rule:** Encrypt your data with a **Data Key**, then encrypt that Data Key with a KMS **Master Key**. The bulk data is encrypted locally; only the small key goes to KMS.

- **Why:** Reduces network overhead — KMS itself can only directly encrypt small payloads (≤4 KB), so large data uses envelope encryption.

**Exam Trigger:** "Encrypt large files efficiently with KMS" → Envelope Encryption.

---

### **5. KMS Throttling (The One Limit That's Tested)**

**Exam Trap:** "S3 uploads failing with throttling errors, using SSE-KMS" → you've hit the **KMS request rate limit** (KMS is called on every object). Solution: request a limit increase, or switch to **SSE-S3** (no per-request KMS call).

---

### **6. KMS vs. CloudHSM**

| **Feature** | **KMS** | **CloudHSM** |
| --- | --- | --- |
| **Tenancy** | Shared / multi-tenant | **Single-tenant** dedicated hardware |
| **Compliance** | FIPS 140-2 Level 2 | **FIPS 140-2 Level 3** |
| **Key Control** | AWS manages HSM infrastructure | **You control keys entirely** |
| **HA** | AWS-managed (Multi-AZ built-in) | You deploy CloudHSM cluster across AZs |
| **Integration** | Native with almost all AWS services | Custom apps, Oracle TDE, SSL offload on EC2 |

**Exam Trigger:** "FIPS 140-2 Level 3" → CloudHSM. "Customer-managed encryption keys with full control" → CloudHSM. "Regulatory compliance requiring dedicated HSM" → CloudHSM.

---

### **SECTION 4B: ACM (AWS CERTIFICATE MANAGER)**

*Free SSL/TLS certificates.*

### **1. ACM Overview**

- **The Rule:** Provision, manage, and deploy **public and private SSL/TLS certificates** (free for public ones), with automatic renewal before expiry. Integrates with ALB, NLB, CloudFront, API Gateway, Elastic Beanstalk.

---

### **2. ACM Key Rules**

- **Cannot install ACM certificates directly on EC2.** Must use CloudHSM or import certificates manually to EC2.
- **Regional Service:** Certificates are region-specific. Exception: **CloudFront requires ACM certificate in us-east-1** (N. Virginia) regardless of where your app runs.
- **Validation:** Domain validation via DNS (recommended, add CNAME record) or email.

**Exam Trigger:** "Free SSL for ALB" → ACM. "HTTPS on CloudFront with custom domain" → ACM certificate in **us-east-1**.

**Exam Trap:** "SSL certificate on EC2" → NOT ACM. Use CloudHSM for SSL offload on EC2 or import certificate manually.

---

### **SECTION 4C: CloudHSM**

*Dedicated Hardware Security Module.*

### **1. CloudHSM Overview**

- **The Rule:** AWS provisions **dedicated, single-tenant HSM hardware** in your VPC; you manage keys entirely. **FIPS 140-2 Level 3** compliant (KMS is only Level 2).

---

### **2. CloudHSM Use Cases**

- SSL/TLS offload on EC2 (ACM can't do this), Oracle TDE, custom key store for KMS (KMS API + CloudHSM hardware), regulatory requirements mandating a dedicated HSM. Deploy a cluster across AZs for HA.

**Exam Trigger:** "FIPS 140-2 Level 3" → CloudHSM. "Dedicated HSM" → CloudHSM. "SSL offload on EC2" → CloudHSM.

---

### **SECTION 5: SECRETS MANAGER**

*Securely store and rotate secrets.*

### **1. Secrets Manager Overview**

- **The Rule:** Store secrets (DB passwords, API keys) with **auto-rotation** of credentials. Integrates with RDS, Redshift, DocumentDB (rotation via Lambda). Versioning allows rollback.

**Secrets Manager vs. Systems Manager Parameter Store:**

| **Feature** | **Secrets Manager** | **Parameter Store** |
| --- | --- | --- |
| **Auto-Rotation** | **Yes** (Built-in for RDS) | No (Manual via Lambda) |
| **Cost** | $0.40/secret/month + API calls | Free (Standard), Paid (Advanced) |
| **Use Case** | Database credentials, Auto-rotation | App config, Free secrets |

**Exam Trigger:** "Auto-rotate RDS password" → Secrets Manager.

---

### **SECTION 6: COGNITO**

*User authentication and authorization.*

### **1. Cognito User Pools (The Identity Provider)**

**The Rule:** Managed user directory (sign-up, sign-in, MFA, password recovery) for app authentication (email/password, social login, SAML). Integrates with API Gateway and ALB. On sign-in, returns **JWT tokens** (ID, Access, Refresh) the app uses to authorize API requests.

**Exam Trigger:** "App needs user sign-up/sign-in" → Cognito User Pools.

---

### **2. Cognito Identity Pools (The AWS Credentials)**

**The Rule:** Grant **temporary AWS credentials** so users access AWS services directly from the app. User authenticates (User Pool, Google, Facebook) → app calls the Identity Pool with the token → it returns STS temporary credentials for S3, DynamoDB, etc.

**Exam Trigger:** "Mobile app needs to upload to S3 without backend" → Cognito Identity Pools.

---

### **Cognito User Pools vs. Identity Pools**

| **Feature** | **User Pools** | **Identity Pools** |
| --- | --- | --- |
| **Purpose** | User authentication (Sign-up/Sign-in) | AWS credentials (Access AWS services) |
| **Output** | JWT tokens | Temporary AWS credentials |
| **Use Case** | Authenticate users | Grant AWS access to users |

**Exam Pattern:** User Pools + Identity Pools together (User Pool authenticates → Identity Pool grants AWS access).

---

### **SECTION 7: WAF & SHIELD**

*Web Application Firewall and DDoS Protection.*

### **1. AWS WAF (Web Application Firewall)**

**The Rule:** Protect web apps from common exploits (SQL injection, XSS, rate limiting). Deploys on CloudFront, ALB, API Gateway, AppSync.

- **Rules:** IP Match, Geo Match, Rate Limiting (block IPs over a threshold, e.g. 2,000 req/5 min), String Match (SQL keywords/scripts), Managed Rules (OWASP Top 10, Bot control).

**Exam Trigger:** "Block SQL injection attacks" → WAF. "Rate limit API requests" → WAF Rate Limiting.

---

### **2. AWS Shield**

**The Rule:** DDoS protection.

- **Shield Standard:** Auto-enabled for **all** customers, **free**. Layer 3/4 protection (SYN floods, UDP reflection) covering CloudFront, Route 53, Global Accelerator.
- **Shield Advanced:** Enhanced DDoS protection, **$3,000/month**. Covers EC2, ELB, CloudFront, Route 53, Global Accelerator; 24/7 DDoS Response Team (DRT); cost protection (refunds scaling costs during attacks); real-time metrics. For mission-critical/high-value targets.

**Exam Trigger:** "DDoS protection for all customers" → Shield Standard (Free). "Enhanced protection + DRT support" → Shield Advanced.

---

### **SECTION 8: GUARDDUTY, INSPECTOR, MACIE**

*Threat Detection and Security Assessment.*

### **1. GuardDuty**

**The Rule:** Intelligent **threat detection** — monitors VPC Flow Logs, CloudTrail, DNS Logs. Detects compromised instances (crypto mining, outbound DDoS), reconnaissance (port scanning), and compromised credentials (API calls from unusual locations). Findings go to the console + EventBridge.

**Exam Trigger:** "Detect compromised instances automatically" → GuardDuty.

---

### **2. Inspector**

**The Rule:** **Vulnerability scanning** for EC2, ECR images, and Lambda — OS CVEs, network exposure (open ports/reachability), and software vulnerabilities in container images. Outputs findings with risk scores.

**Exam Trigger:** "Scan EC2 for vulnerabilities" → Inspector.

---

### **3. Macie**

**The Rule:** **Data security** for S3 — uses ML to discover sensitive data (credit cards, SSN, API keys) and flag public buckets with sensitive data. Findings go to the console + EventBridge.

**Exam Trigger:** "Discover PII in S3 buckets" → Macie.

---

### **GuardDuty vs. Inspector vs. Macie**

| **Service** | **Purpose** | **What It Monitors** | **Exam Trigger** |
| --- | --- | --- | --- |
| **GuardDuty** | Threat detection | VPC Flow Logs, CloudTrail, DNS | "Detect compromised instances" |
| **Inspector** | Vulnerability scanning | EC2, ECR, Lambda | "Scan for OS vulnerabilities" |
| **Macie** | Data security | S3 (PII, Sensitive data) | "Discover PII in S3" |

---

### **SECTION 9: SECURITY HUB**

*Centralized security findings.*

### **1. Security Hub Overview**

- **The Rule:** **Aggregates security findings** from GuardDuty, Inspector, Macie, Firewall Manager, and third-party tools into one dashboard, across all Organization accounts.
- **Automated Compliance Checks:** CIS AWS Foundations Benchmark, PCI DSS, AWS Foundational Security Best Practices. Consolidated findings with severity scores + EventBridge for automated remediation.

**Exam Trigger:** "Centralized security findings" → Security Hub. "Security posture dashboard" → Security Hub. "Compliance dashboard for CIS/PCI DSS" → Security Hub.

---

### **SECTION 10: FIREWALL MANAGER**

*Centralized firewall rule management across accounts.*

### **1. Firewall Manager Overview**

- **The Rule:** **Centrally manage** WAF rules, Shield Advanced protections, Security Groups, and Network Firewall rules across **all accounts** in an Organization. Requires Organizations + AWS Config. New accounts automatically inherit the rules.

**Exam Trigger:** "Manage WAF rules across all accounts" → Firewall Manager. "Centralized firewall management" → Firewall Manager. "Ensure all accounts have Shield Advanced" → Firewall Manager.

**Exam Trap:** Firewall Manager **manages rules**, it is not a firewall itself. It orchestrates WAF, Shield, Security Groups, and Network Firewall.

---

### **SECTION 11: AWS NETWORK FIREWALL**

*Managed firewall for VPC traffic inspection.*

### **1. Network Firewall Overview**

- **The Rule:** Managed **firewall service** deployed in your VPC, inspecting traffic entering/leaving VPCs. Supports stateful + stateless rules, deep packet inspection (L3–L7), IPS/IDS, and domain name filtering. Sits in a dedicated firewall subnet with traffic routed through it via route tables.

**Network Firewall vs. NACLs vs. Security Groups:**

| **Feature** | **Security Groups** | **NACLs** | **Network Firewall** |
| --- | --- | --- | --- |
| **Level** | Instance level | Subnet level | VPC level |
| **Inspection** | Port/Protocol | Port/Protocol/IP | **Deep packet inspection** |
| **Stateful** | Yes | No | Yes (and stateless rules) |
| **IDS/IPS** | No | No | **Yes** |
| **Domain Filtering** | No | No | **Yes** |

**Exam Trigger:** "Inspect traffic entering/leaving VPC" → Network Firewall. "IDS/IPS for VPC" → Network Firewall. "Deep packet inspection in VPC" → Network Firewall.

**Exam Trap:** NACLs are basic allow/deny by IP/port. Network Firewall does deep inspection, domain filtering, and IPS. Very different.

---

### **Exam Summary Cheat Sheet — Practice (Fill In Yourself)**

1. **EC2 needs S3 access without keys?** →
2. **Explicit Deny vs. Allow?** →
3. **Auto-rotate RDS password?** →
4. **Free secret storage?** →
5. **User sign-up/sign-in?** →
6. **Mobile app uploads to S3 without backend?** →
7. **Block SQL injection?** →
8. **Rate limit API?** →
9. **Free DDoS protection?** →
10. **Detect compromised instances?** →
11. **Scan EC2 for vulnerabilities?** →
12. **Discover PII in S3?** →
13. **Cross-account S3 access?** →
14. **Encrypt large files efficiently?** →
15. **Free SSL/TLS for ALB or CloudFront?** →
16. **CloudFront HTTPS with custom domain?** →
17. **SSL certificate on EC2?** →
18. **FIPS 140-2 Level 3 / dedicated HSM?** →
19. **SSO across multiple AWS accounts?** →
20. **Centralized security findings dashboard?** →
21. **Manage WAF rules across all accounts?** →
22. **Deep packet inspection / IDS/IPS in VPC?** →
23. **AssumeRoleWithWebIdentity?** →
24. **AssumeRoleWithSAML?** →
25. **Use existing on-prem Active Directory with AWS?** →
26. **Full managed AD in AWS (Windows/SQL workloads)?** →

---

### **Exam Summary Cheat Sheet — Answer Key**

1. **EC2 needs S3 access without keys?** → IAM Role (Instance Profile).
2. **Explicit Deny vs. Allow?** → Deny wins.
3. **Auto-rotate RDS password?** → Secrets Manager.
4. **Free secret storage?** → Parameter Store (Standard).
5. **User sign-up/sign-in?** → Cognito User Pools.
6. **Mobile app uploads to S3 without backend?** → Cognito Identity Pools.
7. **Block SQL injection?** → WAF.
8. **Rate limit API?** → WAF Rate Limiting.
9. **Free DDoS protection?** → Shield Standard.
10. **Detect compromised instances?** → GuardDuty.
11. **Scan EC2 for vulnerabilities?** → Inspector.
12. **Discover PII in S3?** → Macie.
13. **Cross-account S3 access?** → Bucket Policy (Resource-based).
14. **Encrypt large files efficiently?** → Envelope Encryption (KMS).
15. **Free SSL/TLS for ALB or CloudFront?** → ACM.
16. **CloudFront HTTPS with custom domain?** → ACM in **us-east-1**.
17. **SSL certificate on EC2?** → NOT ACM. Use CloudHSM or import manually.
18. **FIPS 140-2 Level 3 / dedicated HSM?** → CloudHSM.
19. **SSO across multiple AWS accounts?** → IAM Identity Center.
20. **Centralized security findings dashboard?** → Security Hub.
21. **Manage WAF rules across all accounts?** → Firewall Manager.
22. **Deep packet inspection / IDS/IPS in VPC?** → AWS Network Firewall.
23. **AssumeRoleWithWebIdentity?** → Cognito Identity Pools / social login federation.
24. **AssumeRoleWithSAML?** → Corporate IdP (Active Directory) federation to AWS.
25. **Use existing on-prem Active Directory with AWS?** → AD Connector (proxy) or trust with AWS Managed Microsoft AD.
26. **Full managed AD in AWS (Windows/SQL workloads)?** → AWS Managed Microsoft AD.

---

# **REAL EXAM SCENARIOS**

### Scenario 1

**The Situation:** A developer hardcoded AWS credentials (Access Key, Secret Key) in a Lambda function to access S3. The security team flags this as a violation during code review.

**The Options:**

A. Store credentials in environment variables.

B. Store credentials in Secrets Manager and fetch at runtime.

C. Create an IAM Role with S3 permissions and attach it to the Lambda function (Execution Role).

D. Encrypt credentials with KMS.

**The Logic:**

- **Trap:** Environment variables (Option A) still expose credentials (Visible in console).
- **Trap:** Secrets Manager (Option B) works but adds complexity (Fetching, cost). Credentials still exist.
- **The Fix:** **Option C**. **IAM Role** (Execution Role) grants Lambda permissions. No credentials in code/config. AWS automatically provides temporary credentials via metadata. Best practice. Option B is for DB passwords, not AWS API access.

---

### Scenario 2

**The Situation:** Company A (Account 111111111111) wants to allow Company B (Account 222222222222) to access an S3 bucket in Account A. Company B's users should authenticate with their own AWS credentials.

**The Options:**

A. Create IAM users in Account A for Company B's users.

B. Share Access Keys with Company B.

C. Create an IAM Role in Account A with a Trust Policy allowing Account B, and grant S3 permissions.

D. Make the S3 bucket public.

**The Logic:**

- **Trap:** IAM users (Option A) are hard to manage (Create users for each person in Company B).
- **Trap:** Sharing keys (Option B) is insecure (Credentials leak risk).
- **Trap:** Public bucket (Option D) is insecure (Anyone can access).
- **The Fix:** **Option C**. Create **IAM Role** in Account A with:
    1. **Trust Policy:** Allow Account B to assume role (`Principal: "arn:aws:iam::222222222222:root"`).
    2. **Permission Policy:** Grant S3 access.
    Company B users run `aws sts assume-role` → Get temporary credentials → Access S3. Secure, auditable, no shared credentials.

---

### Scenario 3

**The Situation:** A web application connects to an RDS PostgreSQL database using a hardcoded password. The security policy requires passwords to be **rotated every 30 days**. The team manually updates the password in the code each month, causing downtime during deployments.

**The Options:**

A. Store password in SSM Parameter Store and manually rotate via Lambda.

B. Store password in Secrets Manager and enable automatic rotation every 30 days.

C. Use IAM Database Authentication.

D. Store password in environment variables and rotate manually.

**The Logic:**

- **Trap:** Parameter Store (Option A) requires manual rotation logic (Lambda function, updating code).
- **Trap:** IAM Auth (Option C) works but not all apps support it (Requires code changes).
- **Trap:** Environment variables (Option D) still require manual rotation + redeployment.
- **The Fix:** **Option B**. **Secrets Manager** has **built-in RDS rotation** (No Lambda needed for RDS). Enable rotation → Secrets Manager automatically updates RDS password every 30 days. App fetches current password from Secrets Manager (No hardcoded password). Zero downtime.

---

### Scenario 4

**The Situation:** A public REST API (API Gateway) is being abused by a bot making **10,000 requests per minute** from a single IP, causing high costs and degraded performance for legitimate users. The company wants to **block IPs** exceeding 100 requests per 5 minutes.

**The Options:**

A. Use API Gateway Usage Plans with API Keys.

B. Use AWS WAF with a rate-based rule on API Gateway.

C. Use Lambda Authorizer to count requests in DynamoDB.

D. Use CloudWatch Alarms to alert when traffic is high.

**The Logic:**

- **Trap:** Usage Plans (Option A) require API keys (Bot can generate multiple keys or not use keys at all).
- **Trap:** Lambda Authorizer (Option C) is custom code (Complex, adds latency).
- **Trap:** CloudWatch (Option D) is monitoring, not prevention.
- **The Fix:** **Option B**. **WAF Rate-Based Rule**: "Block any IP exceeding 100 requests in 5 minutes." Attach WAF Web ACL to API Gateway. Automatic blocking. No custom code. AWS-managed.

---

### Scenario 5

**The Situation:** A company has **500 S3 buckets** with years of uploaded files. They need to **identify which buckets contain PII** (Social Security Numbers, Credit Cards) to comply with GDPR. Manually reviewing files is impractical.

**The Options:**

A. Use GuardDuty to scan S3 buckets.

B. Use Macie to automatically discover PII in S3.

C. Use Inspector to scan S3 objects.

D. Write Lambda function to scan files with regex.

**The Logic:**

- **Trap:** GuardDuty (Option A) detects threats, not PII (Monitors logs, not data content).
- **Trap:** Inspector (Option C) scans EC2/ECR/Lambda, not S3 data content.
- **Trap:** Lambda (Option D) is custom code (Complex, requires maintaining regex patterns).
- **The Fix:** **Option B**. **Macie** uses ML to automatically scan S3 buckets for PII (SSN, Credit Cards, Passport numbers, etc.). Identifies sensitive data, classifies risk, flags public exposure. Generates findings. Purpose-built for this use case.
