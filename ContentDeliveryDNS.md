# Content Delivery & DNS

### **SECTION 1: ROUTE 53 (THE DNS SERVICE)**

*Managed DNS and Domain Registration.*

### **1. Route 53 Overview**

- **The Rule:** Highly available and scalable **DNS service** (Domain Name System).
- **Functions:**
    - Domain Registration (Buy domains).
    - DNS Routing (Translate domain → IP).
    - Health Checks (Monitor endpoint health).

---

### **2. DNS Record Types (The Basics)**

- **A Record:** Map domain to **IPv4** address (e.g., `example.com` → `192.0.2.1`).
- **AAAA Record:** Map domain to **IPv6** address.
- **CNAME Record:** Map alias to another domain (e.g., `www.example.com` → `example.com`).
    - **Cannot** be used for **root domain** (e.g., `example.com`).
- **Alias Record (AWS-Specific):** Map domain to **AWS resource** (ELB, CloudFront, S3, API Gateway).
    - **Can** be used for **root domain** (e.g., `example.com` → ALB).
    - **Free** (No charge for queries).
    - **Preferred over CNAME** for AWS resources.

**Exam Trap:** "Map root domain to ALB" → Use **Alias Record** (CNAME doesn't work for root).

---

### **3. Routing Policies (The Intelligence)**

*How Route 53 decides which IP to return.*

### **A. Simple Routing**

- **The Rule:** Return **single resource** (One IP or multiple IPs, client picks randomly).
- **Use Case:** Single web server or no routing logic needed.
- **Health Check:** Not supported.

---

### **B. Weighted Routing**

- **The Rule:** Distribute traffic **by percentage** across multiple resources.
- **Example:** 70% to `us-east-1`, 30% to `eu-west-1`.
- **Use Case:** A/B testing, Gradual migration, Blue/Green deployment.
- **Health Check:** Supported (Skip unhealthy endpoints).

**Exam Trigger:** "Route 20% traffic to new version" → Weighted Routing.

---

### **C. Latency-Based Routing**

- **The Rule:** Route traffic to resource with **lowest network latency** for the user.
- **How It Works:** Route 53 measures latency from user location to each AWS region → Routes to fastest.
- **Use Case:** Global applications (Optimize performance).
- **Health Check:** Supported.

**Exam Trigger:** "Route users to closest region for best performance" → Latency-Based Routing.

---

### **D. Failover Routing**

- **The Rule:** Active/Passive setup. Route to **Primary** resource. If unhealthy → Route to **Secondary** (Disaster Recovery).
- **Requirements:** Must configure **Health Check** on Primary.
- **Use Case:** DR, High availability.

**Exam Trigger:** "Automatic failover to secondary site" → Failover Routing.

---

### **E. Geolocation Routing**

- **The Rule:** Route based on **user's geographic location** (Continent, Country, State).
- **Use Case:** Content localization (Show UK site to UK users), Compliance (Data residency), Restrict content.
- **Default Location:** Fallback if no match.

**Exam Trigger:** "Show different content by country" → Geolocation Routing.

---

### **F. Geoproximity Routing**

- **The Rule:** Route based on **proximity** to resources. Optionally shift traffic with **bias** (Expand/shrink geographic region).
- **Bias:** Positive (Attract more traffic), Negative (Repel traffic).
- **Use Case:** Fine-grained traffic shifting (More complex than Geolocation).
- **Requires:** Route 53 Traffic Flow.

**Exam Trigger:** "Shift traffic to specific region with bias" → Geoproximity Routing.

---

### **G. Multi-Value Answer Routing**

- **The Rule:** Return **multiple IPs** (Up to 8). Client picks one. Route 53 only returns **healthy** resources.
- **Use Case:** Simple load balancing with health checks (Not a replacement for ELB).
- **Health Check:** Supported.

**Exam Trigger:** "Return multiple healthy IPs" → Multi-Value Answer.

---

### **Routing Policy Comparison Table**

| **Policy** | **Use Case** | **Health Check** | **Exam Trigger** |
| --- | --- | --- | --- |
| **Simple** | Single resource, No logic | No | "Basic DNS" |
| **Weighted** | A/B testing, Traffic distribution | Yes | "20% to new version" |
| **Latency-Based** | Global apps, Performance | Yes | "Route to fastest region" |
| **Failover** | Active/Passive DR | **Yes (Required)** | "Automatic failover" |
| **Geolocation** | Localization, Compliance | Yes | "Different content by country" |
| **Geoproximity** | Proximity + Bias | Yes | "Shift traffic with bias" |
| **Multi-Value** | Multiple healthy IPs | Yes | "Return multiple IPs" |

---

### **4. Health Checks**

**The Rule:** Monitor endpoint health (HTTP, HTTPS, TCP). Route 53 routes only to healthy endpoints.

**Types:**

- **Endpoint Health Check:** Monitor IP/domain (e.g., Web server at `203.0.113.5:80`).
- **Calculated Health Check:** Combine multiple health checks (AND/OR logic).
- **CloudWatch Alarm:** Monitor CloudWatch alarm state.

**Configuration:**

- **Interval:** 30 sec (Standard) or 10 sec (Fast).
- **Failure Threshold:** Number of consecutive failures to mark unhealthy (Default: 3).
- **String Matching:** Check response body for specific string.

**Exam Trigger:** "Route only to healthy resources" → Health Checks.

---

### **5. Traffic Flow**

**The Rule:** Visual editor for complex routing policies (Chains multiple policies).

**Use Case:** Combine Geolocation + Weighted + Failover in one configuration.

**Exam Trigger:** "Complex routing logic with multiple policies" → Traffic Flow.

---

### **SECTION 2: CLOUDFRONT (THE CDN)**

*Global Content Delivery Network.*

### **1. CloudFront Overview**

- **The Rule:** Cache content at **Edge Locations** (216+ globally). Reduces latency by serving content close to users.
- **Content Types:** Static (Images, CSS, JS), Dynamic (API responses), Video streaming.
- **Origins:** S3, ALB, NLB, EC2, Custom HTTP servers.

**Benefits:**

- Low latency (Content served from nearest edge).
- Reduced load on origin (Cache at edge).
- DDoS protection (AWS Shield Standard included).

---

### **2. CloudFront Architecture**

**Flow:**

1. User requests `www.example.com/logo.png`.
2. DNS routes to **nearest Edge Location**.
3. Edge checks cache:
    - **Cache Hit:** Returns cached content.
    - **Cache Miss:** Fetches from **Origin**, caches, returns to user.
4. Subsequent requests served from cache (Until TTL expires).

**TTL (Time to Live):**

- Default: 24 hours.
- Configurable per object (0 sec - 1 year).
- **Invalidation:** Manually clear cache (Costs money). Use versioned filenames instead (e.g., `logo_v2.png`).

---

### **3. CloudFront Origins**

### **A. S3 as Origin**

**Use Case:** Static website, Media files, Software downloads.

**Origin Access Control (OAC):**

- **The Rule:** Restrict S3 bucket access **only** to CloudFront (Block direct access).
- **How:** Bucket Policy allows CloudFront service principal.
- **Supports:** SSE-KMS encrypted buckets, all S3 regions.
- **Note:** OAI (Origin Access Identity) is **legacy/deprecated**. Always use **OAC** on the exam.

**Exam Trigger:** "Serve S3 content via CloudFront, Block direct S3 access" → OAC.

---

### **B. ALB/EC2 as Origin**

**Use Case:** Dynamic content (APIs), Personalized content.

**Security:**

- ALB/EC2 must be **publicly accessible** (CloudFront connects over internet).
- **Security Group:** Allow traffic from **CloudFront IP ranges** (AWS publishes list).

**Exam Trigger:** "Serve dynamic API via CloudFront" → ALB as Origin.

---

### **4. CloudFront Security**

### **A. Geo Restriction**

- **The Rule:** Block/Allow users from specific countries.
- **Whitelist:** Only allow specific countries.
- **Blacklist:** Block specific countries.

**Exam Trigger:** "Block access from certain countries" → Geo Restriction.

---

### **B. Signed URLs & Signed Cookies**

**The Rule:** Restrict access to **premium content** (Require authentication).

**Signed URL:**

- URL with signature (Token).
- Use for **individual files** (e.g., Download single video).

**Signed Cookie:**

- Cookie with signature.
- Use for **multiple files** (e.g., All videos in course).

**How It Works:**

1. User authenticates with your app.
2. App generates Signed URL/Cookie using CloudFront private key.
3. User accesses CloudFront with signed URL/Cookie.
4. CloudFront validates signature → Serves content.

**Exam Trigger:** "Restrict access to premium content" → Signed URLs/Cookies.

---

### **C. Field-Level Encryption**

**The Rule:** Encrypt sensitive data (Credit card, SSN) at edge **before** sending to origin.

**How It Works:**

- CloudFront encrypts specific POST fields using public key.
- Only origin's private key can decrypt.
- Protects data in transit through application layers.

**Exam Trigger:** "Encrypt sensitive form data at edge" → Field-Level Encryption.

---

### **5. CloudFront vs. S3 Cross-Region Replication**

| **Feature** | **CloudFront** | **S3 CRR** |
| --- | --- | --- |
| **Purpose** | Cache at edge (Global CDN) | Replicate to specific regions |
| **Latency** | Lowest (216+ edge locations) | Low (Specific regions) |
| **Use Case** | Static content, Global audience | Compliance, DR, Low-latency reads in specific regions |
| **TTL** | Files cached (TTL-based) | Real-time replication |

**Exam Trigger:** "Serve static content globally with low latency" → CloudFront. "Replicate S3 to specific region for compliance" → S3 CRR.

---

### **SECTION 3: GLOBAL ACCELERATOR**

*Improve global application availability and performance.*

### **1. Global Accelerator Overview**

- **The Rule:** Provides **2 static Anycast IPs** that route traffic through AWS global network to optimal endpoint.
- **Difference from CloudFront:**
    - **CloudFront:** Caches content at edge (CDN).
    - **Global Accelerator:** Proxies connections to application (No caching).

**How It Works:**

1. User connects to **Anycast IP** (Same IP globally).
2. Traffic routed through **AWS Edge Locations** to nearest healthy endpoint (ALB, NLB, EC2, Elastic IP).
3. Uses AWS private network (Faster, more reliable than public internet).

---

### **2. Global Accelerator Benefits**

- **Static IPs:** 2 Anycast IPs (Don't change, easy to whitelist).
- **Fast Failover:** Automatic reroute to healthy endpoint (< 1 minute).
- **Performance:** Use AWS backbone network (Lower latency, less packet loss).
- **DDoS Protection:** AWS Shield Standard included.

**Use Cases:**

- Gaming (UDP traffic, Low latency).
- IoT.
- VoIP.
- Non-HTTP protocols (CloudFront is HTTP/HTTPS only).

---

### **3. Global Accelerator vs. CloudFront**

| **Feature** | **CloudFront** | **Global Accelerator** |
| --- | --- | --- |
| **Purpose** | Cache content (CDN) | Proxy connections (No cache) |
| **Protocols** | HTTP/HTTPS | TCP, UDP, HTTP, HTTPS |
| **IPs** | Dynamic (Edge location IPs) | **2 Static Anycast IPs** |
| **Use Case** | Static content, Caching | Dynamic content, Non-HTTP, Gaming, IoT |
| **Failover** | Not applicable (Origin handles) | **Fast failover** (< 1 min) |

**Exam Trigger:** "Static IP required, UDP traffic, Gaming" → Global Accelerator. "Cache static content" → CloudFront.

---

### **Exam Summary Cheat Sheet (Memorize This)**

1. **Map root domain to ALB?** → Alias Record (NOT CNAME).
2. **Route 20% traffic to new version?** → Weighted Routing.
3. **Route to fastest region?** → Latency-Based Routing.
4. **Automatic failover to DR site?** → Failover Routing (Requires Health Check).
5. **Show different content by country?** → Geolocation Routing.
6. **Serve S3 content via CloudFront, Block direct access?** → Origin Access Control (OAC).
7. **Restrict access to premium content?** → Signed URLs/Cookies.
8. **Block access from specific countries?** → CloudFront Geo Restriction.
9. **Cache static content globally?** → CloudFront.
10. **Static IP, UDP, Gaming, Fast failover?** → Global Accelerator.
11. **Replicate S3 to specific region?** → S3 Cross-Region Replication (NOT CloudFront).
12. **Manually clear CloudFront cache?** → Invalidation (Costs money; use versioned names).

---

# **REAL EXAM SCENARIOS**

### **Scenario 1: The "Root Domain Problem" (Alias Record)**

**The Situation:** A company hosts a website on an Application Load Balancer. They want users to access the site using the **root domain** (`example.com`) instead of a subdomain (`www.example.com`). They try to create a CNAME record but Route 53 rejects it.

**The Options:**

A. Use an A Record pointing to the ALB's IP address.

B. Use an Alias Record pointing to the ALB's DNS name.

C. Use a CNAME Record pointing to the ALB's DNS name.

D. Use a TXT Record.

**The Logic:**

- **Trap:** CNAME (Option C) **cannot** be used for root domain (DNS standard limitation).
- **Trap:** A Record (Option A) requires static IP. ALB IP changes (Not supported).
- **The Fix:** **Option B**. **Alias Record** is AWS-specific. Works for root domain. Points to ALB DNS name. Route 53 automatically resolves to ALB's current IPs. Free (No query charges).

---

### **Scenario 2: The "Global Performance" (Latency-Based Routing)**

**The Situation:** A SaaS application has users worldwide. The application runs in **three regions**: `us-east-1`, `eu-west-1`, and `ap-southeast-1`. The company wants users automatically routed to the **fastest region** based on their location.

**The Options:**

A. Use Weighted Routing with equal weights.

B. Use Geolocation Routing to map continents to regions.

C. Use Latency-Based Routing.

D. Use CloudFront.

**The Logic:**

- **Trap:** Weighted Routing (Option A) distributes evenly (Ignores performance).
- **Trap:** Geolocation (Option B) routes by geography (Not actual latency). A user in Brazil might be closer to `us-east-1` than `sa-east-1`.
- **Trap:** CloudFront (Option D) is for caching, not routing to application backends.
- **The Fix:** **Option C**. **Latency-Based Routing** measures actual network latency from user to each region. Routes to region with lowest latency. Optimizes performance dynamically.

---

### **Scenario 3: The "Premium Content" (Signed URLs)**

**The Situation:** A streaming service hosts video files in S3 and serves them via CloudFront. Only **paid subscribers** should access videos. Free users should be blocked. The company wants to prevent URL sharing (User shares URL → Another user watches free).

**The Options:**

A. Use S3 bucket policies to allow only authenticated users.

B. Use CloudFront with Signed URLs that expire after 1 hour.

C. Use CloudFront Geo Restriction.

D. Use WAF to block unauthorized IPs.

**The Logic:**

- **Trap:** Bucket Policy (Option A) doesn't work if CloudFront is serving (CloudFront accesses S3, not end user).
- **Trap:** Geo Restriction (Option C) blocks countries, not individual users.
- **Trap:** WAF (Option D) can't identify paid users vs. free users.
- **The Fix:** **Option B**. **Signed URLs** include a signature (Generated by app using private key). URL expires after 1 hour. User authenticates → App generates Signed URL → User accesses CloudFront with URL → CloudFront validates signature. If user shares URL, it expires or requires re-authentication.

---

### **Scenario 4: The "Static IP Requirement" (Global Accelerator)**

**The Situation:** A gaming company runs multiplayer servers on EC2 behind Network Load Balancers in **four regions**. Players complain about lag when connecting over the public internet. The game client requires **static IPs** to connect (Hardcoded in client config). The company needs to improve latency and provide failover.

**The Options:**

A. Use CloudFront to cache game data.

B. Use Route 53 Latency-Based Routing with health checks.

C. Use Global Accelerator with 2 Anycast IPs pointing to NLBs.

D. Use Elastic IPs for each NLB and distribute to clients.

**The Logic:**

- **Trap:** CloudFront (Option A) is for HTTP/HTTPS caching (Gaming uses UDP/TCP, no caching).
- **Trap:** Route 53 (Option B) provides multiple IPs (Not static single IP).
- **Trap:** Elastic IPs (Option D) provide static IPs but don't optimize routing or failover.
- **The Fix:** **Option C**. **Global Accelerator** provides **2 static Anycast IPs**. Traffic routed through AWS global network (Lower latency than public internet). Automatic failover to healthy endpoints (< 1 minute). Supports TCP/UDP (Gaming). Players connect to same IPs globally.

---

### **Scenario 5: The "Cache Invalidation" (Versioned Filenames)**

**The Situation:** A website uses CloudFront to serve static assets (CSS, JS, Images) from S3. When developers deploy updates, users see **old cached versions** for hours (24-hour TTL). The company pays $500/month for CloudFront invalidations. They want a better solution.

**The Options:**

A. Reduce CloudFront TTL to 1 minute.

B. Use versioned filenames (e.g., `style_v2.css`) and update HTML references.

C. Increase CloudFront invalidation budget.

D. Disable caching entirely.

**The Logic:**

- **Trap:** Low TTL (Option A) increases origin load (Frequent cache misses). Reduces caching benefit.
- **Trap:** More invalidations (Option C) costs more money (Doesn't solve problem).
- **Trap:** No caching (Option D) defeats CloudFront purpose (High latency, high origin load).
- **The Fix:** **Option B**. **Versioned filenames** (e.g., `style_v1.css`, `style_v2.css`). When file changes, upload with new name. Update HTML to reference new filename. CloudFront fetches new file (Cache miss), caches it. Old file remains cached but unused. No invalidations needed. Free. Best practice.
