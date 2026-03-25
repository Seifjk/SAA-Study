# Security & Identity

### **SECTION 1: IAM (IDENTITY AND ACCESS MANAGEMENT)**

*The Foundation of AWS Security.*

### **1. IAM Overview**

- **The Rule:** Control **who** can access **what** in AWS.
- **Global Service:** Not region-specific.
- **Free:** No charge for IAM.

**Core Components:**

- **Users:** Individual people or services (Long-term credentials).
- **Groups:** Collection of users (Apply policies to multiple users).
- **Roles:** Temporary credentials for AWS services or federated users.
- **Policies:** JSON documents defining permissions (Attached to users/groups/roles).

---

### **2. IAM Policies (The Permissions)**

**Structure:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {"IpAddress": {"aws:SourceIp": "203.0.113.0/24"}}
    }
  ]
}
```

**Key Elements:**

- **Effect:** `Allow` or `Deny`.
- **Action:** What can be done (e.g., `s3:GetObject`, `ec2:*`).
- **Resource:** Which resources (ARN format).
- **Condition:** Optional (IP range, MFA, Time, etc.).

---

### **3. Policy Types**

### **A. Identity-Based Policies**

- **The Rule:** Attached to **users, groups, or roles**.
- **Types:**
    - **AWS Managed:** Pre-built by AWS (e.g., `AdministratorAccess`).
    - **Customer Managed:** Custom policies you create.
    - **Inline:** Directly embedded in a single user/group/role (Not reusable).

### **B. Resource-Based Policies**

- **The Rule:** Attached to **resources** (e.g., S3 bucket policy, SQS queue policy).
- **Use Case:** Grant cross-account access, Allow services to access resources.

**Exam Example:** "S3 bucket allows CloudFront to access" → Bucket Policy (Resource-based).

---

### **4. Permission Boundaries**

**The Rule:** Set **maximum permissions** a user/role can have (Even if policies grant more).

**Use Case:** Delegate admin permissions safely (Admin can't escalate beyond boundary).

**Exam Trigger:** "Prevent users from gaining admin access" → Permission Boundaries.

---

### **5. IAM Roles (The Temporary Credentials)**

**The Rule:** Temporary credentials (No password). **Assumed** by users/services.

**Common Roles:**

- **EC2 Instance Role:** Grants permissions to applications running on EC2 (Via Instance Profile).
- **Lambda Execution Role:** Grants permissions to Lambda function.
- **Cross-Account Role:** Allows users in Account A to access resources in Account B.
- **Service Role:** Allows AWS services (e.g., CodeDeploy) to perform actions.

**How It Works:**

1. User/Service assumes role.
2. STS (Security Token Service) returns temporary credentials (Access Key, Secret Key, Session Token).
3. Use credentials to access AWS resources.
4. Credentials expire (15 min - 12 hours, configurable).

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

**The Rule:** **Explicit Deny > Explicit Allow > Implicit Deny (Default)**.

**Flow:**

1. Check for **explicit Deny** → If found, **Deny**.
2. Check for **explicit Allow** → If found, **Allow**.
3. If neither → **Deny** (Default).

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

- **The Rule:** **Centralized SSO** for multiple AWS accounts in an Organization and business applications (Salesforce, Slack, custom SAML apps).
- **Identity Source:** Built-in identity store, or integrate with **Active Directory** (AWS Managed AD or AD Connector) or external IdP (Okta, Azure AD).
- **Permission Sets:** Define permissions once, assign to users/groups across multiple accounts.

**How It Works:**

1. User signs in once via IAM Identity Center portal.
2. Sees all assigned AWS accounts and applications.
3. Clicks an account → Gets temporary credentials via STS.

**Exam Trigger:** "SSO across AWS accounts" → IAM Identity Center. "Centralized access management for Organization" → IAM Identity Center.

**Exam Trap:** IAM Identity Center replaces the old "AWS SSO" name. Same service, new name. Exam may use either.

---

### **SECTION 4: KMS (KEY MANAGEMENT SERVICE)**

*Managed encryption keys.*

### **1. KMS Overview**

- **The Rule:** Create and manage **encryption keys**. Integrated with most AWS services.
- **Encryption:** Data at rest (EBS, S3, RDS) and in transit.
- **Audit:** All key usage logged in CloudTrail.

---

### **2. KMS Key Types**

### **A. AWS Managed Keys**

- **The Rule:** Created/managed by AWS (e.g., `aws/s3`, `aws/rds`).
- **Rotation:** Automatic (Every year).
- **Cost:** Free.
- **Control:** Limited (Can't delete, Can't change key policy).

### **B. Customer Managed Keys (CMK)**

- **The Rule:** You create and manage.
- **Rotation:** Optional (Automatic every year or manual).
- **Cost:** $1/month per key + API calls.
- **Control:** Full (Set key policy, Enable/disable, Delete).

**Exam Trigger:** "Need to rotate key on demand" → Customer Managed Key.

---

### **3. KMS Key Policies**

**The Rule:** Control **who** can use/manage keys (Like IAM policies but for keys).

**Default Key Policy:** Allows root account full access (IAM policies can grant access).

**Custom Key Policy:** Explicit control (Useful for cross-account access).

**Exam Trigger:** "Grant cross-account access to KMS key" → Custom Key Policy.

---

### **4. Encryption Context**

**The Rule:** Additional authentication data (Key-value pairs) used during encryption.

**Use Case:** Ensure data is decrypted only in correct context (e.g., `user-id: 12345`).

**Audit:** Logged in CloudTrail (Helps identify who decrypted what).

---

### **5. Envelope Encryption**

**The Rule:** Encrypt data with **Data Key**, Encrypt Data Key with **Master Key** (KMS).

**Flow:**

1. Call KMS `GenerateDataKey` → Get plaintext Data Key + encrypted Data Key.
2. Use plaintext Data Key to encrypt data.
3. Store encrypted data + encrypted Data Key.
4. Discard plaintext Data Key.
5. To decrypt: Call KMS `Decrypt` on encrypted Data Key → Get plaintext Data Key → Decrypt data.

**Why:** Reduce network overhead (Encrypt large data locally, only send keys to KMS).

**Exam Trigger:** "Encrypt large files efficiently" → Envelope Encryption.

---

### **6. KMS Limits**

- **API Calls:** 5,500/sec (shared across encrypt/decrypt/generate operations) - Soft limit, can request increase.
- **Throttling:** If exceeded → `ThrottlingException`.

**Exam Trap:** "S3 uploads failing with throttling error, Using SSE-KMS" → KMS API limit exceeded. Solution: Request limit increase or use SSE-S3.

---

### **7. KMS vs. CloudHSM**

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

- **The Rule:** Provision, manage, and deploy **public and private SSL/TLS certificates**. Free for public certificates.
- **Auto-Renewal:** ACM automatically renews certificates before expiry (No manual work).
- **Integration:** ALB, NLB, CloudFront, API Gateway, Elastic Beanstalk.

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

- **The Rule:** AWS provisions **dedicated HSM hardware** in your VPC. You manage keys entirely.
- **FIPS 140-2 Level 3** compliant (KMS is only Level 2).
- **Single-Tenant:** Hardware is not shared with other customers.

---

### **2. CloudHSM Use Cases**

- **SSL/TLS offload** on EC2 (ACM can't do this).
- **Oracle TDE** (Transparent Data Encryption).
- **Custom key store** for KMS (Bridge: KMS API + CloudHSM hardware).
- **Regulatory requirements** mandating dedicated HSM.

**HA Setup:** Deploy **CloudHSM cluster** across multiple AZs for high availability.

**Exam Trigger:** "FIPS 140-2 Level 3" → CloudHSM. "Dedicated HSM" → CloudHSM. "SSL offload on EC2" → CloudHSM.

---

### **SECTION 5: SECRETS MANAGER**

*Securely store and rotate secrets.*

### **1. Secrets Manager Overview**

- **The Rule:** Store secrets (DB passwords, API keys). **Auto-rotate** credentials.
- **Integration:** RDS, Redshift, DocumentDB (Auto-rotation via Lambda).
- **Versioning:** Track secret versions (Rollback if needed).

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

**The Rule:** Managed user directory. Sign-up, Sign-in, MFA, Password recovery.

**Use Case:** App needs user authentication (Email/password, Social login, SAML).

**Integration:** API Gateway, ALB (For authentication).

**Flow:**

1. User signs up/signs in → Cognito User Pool.
2. Cognito returns **JWT tokens** (ID token, Access token, Refresh token).
3. App uses tokens to authorize API requests.

**Exam Trigger:** "App needs user sign-up/sign-in" → Cognito User Pools.

---

### **2. Cognito Identity Pools (The AWS Credentials)**

**The Rule:** Grant **temporary AWS credentials** to users (Access AWS services directly from app).

**Use Case:** Mobile/web app needs to upload to S3, read DynamoDB.

**Flow:**

1. User authenticates with Cognito User Pool (or Google, Facebook).
2. App calls Cognito Identity Pool with token.
3. Identity Pool returns **temporary AWS credentials** (via STS).
4. App uses credentials to access S3, DynamoDB, etc.

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

**The Rule:** Protect web apps from common exploits (SQL injection, XSS, Rate limiting).

**Deployments:** CloudFront, ALB, API Gateway, AppSync.

**Rules:**

- **IP Match:** Block/Allow specific IPs/CIDR ranges.
- **Geo Match:** Block/Allow countries.
- **Rate Limiting:** Block IPs exceeding request threshold (e.g., 2,000 req/5 min).
- **String Match:** Block requests with specific strings (SQL keywords, Scripts).
- **Managed Rules:** Pre-configured rule sets (OWASP Top 10, Bot control).

**Exam Trigger:** "Block SQL injection attacks" → WAF. "Rate limit API requests" → WAF Rate Limiting.

---

### **2. AWS Shield**

**The Rule:** DDoS protection.

### **A. Shield Standard**

- **The Rule:** Automatically enabled for **all** AWS customers. **Free**.
- **Protection:** Layer 3/4 (Network/Transport layer). SYN floods, UDP reflection.
- **Coverage:** CloudFront, Route 53, Global Accelerator.

### **B. Shield Advanced**

- **The Rule:** Enhanced DDoS protection. **$3,000/month**.
- **Features:**
    - Protection for EC2, ELB, CloudFront, Route 53, Global Accelerator.
    - **DDoS Response Team (DRT):** 24/7 support.
    - **Cost Protection:** Refund for scaling costs during attack.
    - **Real-time metrics and reports.**
- **Use Case:** Mission-critical apps, High-value targets.

**Exam Trigger:** "DDoS protection for all customers" → Shield Standard (Free). "Enhanced protection + DRT support" → Shield Advanced.

---

### **SECTION 8: GUARDDUTY, INSPECTOR, MACIE**

*Threat Detection and Security Assessment.*

### **1. GuardDuty**

**The Rule:** Intelligent **threat detection**. Monitors VPC Flow Logs, CloudTrail, DNS Logs.

**Detects:**

- Compromised instances (Crypto mining, Outbound DDoS).
- Reconnaissance (Port scanning).
- Compromised credentials (API calls from unusual locations).

**Output:** Findings in GuardDuty console + EventBridge (Trigger automated response).

**Exam Trigger:** "Detect compromised instances automatically" → GuardDuty.

---

### **2. Inspector**

**The Rule:** **Vulnerability scanning** for EC2, ECR images, Lambda.

**Scans For:**

- OS vulnerabilities (CVEs).
- Network exposure (Open ports, reachability).
- Software vulnerabilities in container images.

**Output:** Findings with risk scores (Prioritize remediation).

**Exam Trigger:** "Scan EC2 for vulnerabilities" → Inspector.

---

### **3. Macie**

**The Rule:** **Data security** for S3. Uses ML to discover sensitive data (PII, Credentials).

**Detects:**

- Credit card numbers, SSN, API keys in S3 buckets.
- Public buckets with sensitive data.

**Output:** Findings in Macie console + EventBridge.

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

- **The Rule:** **Aggregates security findings** from GuardDuty, Inspector, Macie, Firewall Manager, and third-party tools into a **single dashboard**.
- **Automated Compliance Checks:** CIS AWS Foundations Benchmark, PCI DSS, AWS Foundational Security Best Practices.
- **Cross-Account:** Works across all accounts in an Organization.

**Output:** Consolidated findings with severity scores + EventBridge integration for automated remediation.

**Exam Trigger:** "Centralized security findings" → Security Hub. "Security posture dashboard" → Security Hub. "Compliance dashboard for CIS/PCI DSS" → Security Hub.

---

### **SECTION 10: FIREWALL MANAGER**

*Centralized firewall rule management across accounts.*

### **1. Firewall Manager Overview**

- **The Rule:** **Centrally manage** WAF rules, Shield Advanced protections, Security Groups, Network Firewall rules across **all accounts** in an Organization.
- **Prerequisite:** Requires AWS Organizations and AWS Config enabled.
- **Auto-Apply:** New accounts in the Organization automatically get the rules.

**Exam Trigger:** "Manage WAF rules across all accounts" → Firewall Manager. "Centralized firewall management" → Firewall Manager. "Ensure all accounts have Shield Advanced" → Firewall Manager.

**Exam Trap:** Firewall Manager **manages rules**, it is not a firewall itself. It orchestrates WAF, Shield, Security Groups, and Network Firewall.

---

### **SECTION 11: AWS NETWORK FIREWALL**

*Managed firewall for VPC traffic inspection.*

### **1. Network Firewall Overview**

- **The Rule:** Managed **firewall service** deployed in your VPC. Inspects traffic entering and leaving VPCs.
- **Features:**
    - **Stateful and stateless** rules.
    - **Deep packet inspection** (Layer 3 through Layer 7).
    - **Intrusion Prevention System (IPS)** and intrusion detection.
    - Domain name filtering (Allow/block specific domains).
- **Deployment:** Sits in a dedicated firewall subnet. Traffic routed through it via VPC route tables.

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

### **Exam Summary Cheat Sheet (Memorize This)**

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

---

# **REAL EXAM SCENARIOS**

### **Scenario 1: The "Hardcoded Keys" (IAM Role)**

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

### **Scenario 2: The "Cross-Account Access" (IAM Role + Trust Policy)**

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

### **Scenario 3: The "Auto-Rotate Password" (Secrets Manager)**

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

### **Scenario 4: The "API Rate Limiter" (WAF)**

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

### **Scenario 5: The "Sensitive Data Discovery" (Macie)**

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
