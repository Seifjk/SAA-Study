# Content Delivery & DNS

> **📌 Scope note:** AWS tests these with *"determine WHEN to use"* — you pick the routing policy / edge service, you don't configure TTLs, health-check intervals, or write edge functions. If it's not here, the exam doesn't ask it.

### **SECTION 1: ROUTE 53 (THE DNS SERVICE)**

*Managed DNS and Domain Registration.*

### **1. Route 53 Overview**

- **The Rule:** Highly available, scalable **DNS service**. Functions: domain registration, DNS routing (domain → IP), and health checks.

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

- Returns a **single resource** (one IP, or multiple IPs the client picks randomly). For a single web server / no routing logic. No health check support.

---

### **B. Weighted Routing**

- Distributes traffic **by percentage** across resources (e.g., 70% us-east-1, 30% eu-west-1). For A/B testing, gradual migration, Blue/Green. Health checks supported.

**Exam Trigger:** "Route 20% traffic to new version" → Weighted Routing.

---

### **C. Latency-Based Routing**

- Routes to the resource with **lowest network latency** for the user (Route 53 measures user→region latency). For global performance. Health checks supported.

**Exam Trigger:** "Route users to closest region for best performance" → Latency-Based Routing.

---

### **D. Failover Routing**

- Active/Passive: route to **Primary**, fail to **Secondary** if unhealthy. Requires a **Health Check** on Primary. For DR/HA.

**Exam Trigger:** "Automatic failover to secondary site" → Failover Routing.

---

### **E. Geolocation Routing**

- Routes by **user's geographic location** (continent/country/state). For localization, compliance, content restriction. Has a default fallback location.

**Exam Trigger:** "Show different content by country" → Geolocation Routing.

---

### **F. Geoproximity Routing**

- Routes by **proximity** to resources, with an optional **bias** to expand/shrink a region's traffic (positive attracts, negative repels). Requires Route 53 Traffic Flow.

**Exam Trigger:** "Shift traffic to specific region with bias" → Geoproximity Routing.

---

### **G. Multi-Value Answer Routing**

- Returns **up to 8 IPs**, only healthy ones; client picks one. Simple load balancing with health checks (not an ELB replacement).

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

**The Rule:** Monitor endpoint health (HTTP/HTTPS/TCP); Route 53 routes only to healthy endpoints.

- **Types:** Endpoint (monitor an IP/domain), Calculated (combine checks with AND/OR), CloudWatch Alarm (monitor alarm state).

**Exam Trigger:** "Route only to healthy resources" → Health Checks. "Combine multiple health checks into one" → Calculated.

---

### **5. Traffic Flow**

**The Rule:** Visual editor for complex routing policies (Chains multiple policies).

**Use Case:** Combine Geolocation + Weighted + Failover in one configuration.

**Exam Trigger:** "Complex routing logic with multiple policies" → Traffic Flow.

---

### **6. Public vs. Private Hosted Zones**

### **A. Public Hosted Zone**

- Records routed **on the internet** (public DNS), queryable by anyone. Created automatically when you register a domain with Route 53. E.g., `example.com` → ALB public IP.

### **B. Private Hosted Zone**

- Records routed **within one or more VPCs** (internal DNS) — must be **associated with VPCs** (can span multiple VPCs, even cross-account). For internal service discovery, private endpoints, microservices routing. E.g., `db.internal.example.com` → RDS private IP.

**Exam Trap:** "Resolve internal DNS names within a VPC" → Private Hosted Zone (must associate with the VPC).

**Exam Trigger:** "Internal DNS for VPC resources" → Private Hosted Zone. "Internet-facing DNS" → Public Hosted Zone.

---

### **SECTION 2: CLOUDFRONT (THE CDN)**

*Global Content Delivery Network.*

### **1. CloudFront Overview**

- **The Rule:** Caches content at **Edge Locations** (216+ globally) — low latency, reduced origin load, AWS Shield Standard DDoS protection included.
- **Content:** Static (images/CSS/JS), dynamic (API responses), video streaming. **Origins:** S3, ALB, NLB, EC2, custom HTTP servers.

---

### **2. CloudFront Architecture**

**Flow:** User request → DNS routes to nearest Edge Location → Edge checks cache: hit → return cached; miss → fetch from Origin, cache, return. Subsequent requests served from cache until TTL expires.

**TTL:** Default 24 hours, configurable per object (0 sec – 1 year). **Invalidation** manually clears cache (costs money) — prefer versioned filenames (e.g., `logo_v2.png`).

---

### **3. CloudFront Origins**

### **A. S3 as Origin**

- **Use Case:** Static website, media files, software downloads.
- **Origin Access Control (OAC):** Restricts S3 bucket access **only** to CloudFront (Bucket Policy allows the CloudFront service principal); supports SSE-KMS buckets and all regions. OAI is legacy/deprecated — always use **OAC**.

**Exam Trigger:** "Serve S3 content via CloudFront, Block direct S3 access" → OAC.

---

### **B. ALB/EC2 as Origin**

- **Use Case:** Dynamic/personalized content (APIs). ALB/EC2 must be publicly accessible; Security Group should allow traffic from the published **CloudFront IP ranges**.

**Exam Trigger:** "Serve dynamic API via CloudFront" → ALB as Origin.

---

### **4. CloudFront Security**

### **A. Geo Restriction**

- Block/allow users from specific countries via whitelist (allow only) or blacklist (block).

**Exam Trigger:** "Block access from certain countries" → Geo Restriction.

---

### **B. Signed URLs & Signed Cookies**

- Restrict access to **premium content**. **Signed URL** = signature for **individual files**; **Signed Cookie** = signature for **multiple files**.
- **Flow:** User authenticates with your app → app generates a signed URL/cookie with the CloudFront private key → CloudFront validates the signature and serves content.

**Exam Trigger:** "Restrict access to premium content" → Signed URLs/Cookies.

---

### **C. Field-Level Encryption**

- Encrypts specific sensitive POST fields (credit card, SSN) at the edge so only the origin can decrypt them. **Exam Trigger:** "Encrypt specific sensitive form fields at the edge" → Field-Level Encryption. (Niche — recognize the name, don't memorize internals.)

---

### **5. CloudFront Functions vs. Lambda@Edge**

*One of the most commonly tested CloudFront topics on the exam.*

### **A. CloudFront Functions**

- Lightweight **JavaScript**, **viewer request/response only**, sub-ms, cheapest. **No network access, no request-body access.** For simple transforms: header manipulation, URL rewrites/redirects, cache-key normalization, simple token checks.

---

### **B. Lambda@Edge**

- **Node.js/Python**, runs on **all 4 event types** (incl. origin request/response), **can make network calls and read/write the request body**. For complex logic: full auth (OIDC/SAML), dynamic origin selection, content generation at the edge.

---

### **C. Comparison Table**

| **Feature** | **CloudFront Functions** | **Lambda@Edge** |
| --- | --- | --- |
| **Language** | JavaScript only | Node.js, Python |
| **Event Types** | Viewer request/response ONLY | All 4 (Viewer + Origin) |
| **Max Execution** | Sub-millisecond | 5 sec (Viewer) / 30 sec (Origin) |
| **Scale** | Millions req/sec | Thousands req/sec |
| **Network Access** | No | Yes |
| **Body Access** | No | Yes (Read/Write) |
| **Cost** | Very cheap | ~6x more expensive |
| **Use Case** | Simple transforms, Rewrites, Headers | Complex logic, External calls, Body manipulation |

**In one line:** CloudFront Functions = simple, cheap, massive-scale viewer-only edits (headers, URL rewrites); Lambda@Edge = powerful but ~6× pricier — pick it only when you need **network/external calls, request-body access, or origin-side logic** (e.g. auth, dynamic origin selection).

**Exam Trigger:** "Simple URL rewrite or header manipulation" → CloudFront Functions. "Need network access or origin-level processing" → Lambda@Edge. "Cheapest edge compute for high-volume requests" → CloudFront Functions.

---

### **6. CloudFront Origin Groups (Origin Failover)**

**The Rule:** Configure a **Primary** and **Secondary origin** for automatic failover. If the primary returns a configured error status (e.g., 500, 502, 503, 504), CloudFront automatically retries the secondary, which serves the content with no error to the user.

- Primary and Secondary can be **different origin types** (e.g., Primary = ALB, Secondary = S3 static error page). For HA and fallback content.

**Exam Trigger:** "CloudFront high availability" or "Origin failover" → Origin Groups (Primary + Secondary origin).

---

### **7. CloudFront vs. S3 Cross-Region Replication**

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

- **The Rule:** Provides **2 static Anycast IPs** routing traffic through the AWS global network to the optimal endpoint. Unlike CloudFront (caches content), Global Accelerator **proxies connections** (no caching).
- **Flow:** User connects to an Anycast IP (same globally) → traffic routed via AWS Edge Locations to the nearest healthy endpoint (ALB, NLB, EC2, Elastic IP) over the AWS private network.

---

### **2. Global Accelerator Benefits**

- 2 static Anycast IPs (easy to whitelist); fast failover (< 1 min) to healthy endpoints; AWS backbone for lower latency/packet loss; AWS Shield Standard included.
- **Use Cases:** Gaming (UDP, low latency), IoT, VoIP, non-HTTP protocols (CloudFront is HTTP/HTTPS only).

---

### **3. Global Accelerator vs. CloudFront**

| **Feature** | **CloudFront** | **Global Accelerator** |
| --- | --- | --- |
| **Purpose** | Cache content (CDN) | Proxy connections (No cache) |
| **Protocols** | HTTP/HTTPS | TCP, UDP, HTTP, HTTPS |
| **IPs** | Dynamic (Edge location IPs) | **2 Static Anycast IPs** |
| **Use Case** | Static content, Caching | Dynamic content, Non-HTTP, Gaming, IoT |
| **Failover** | Not applicable (Origin handles) | **Fast failover** (< 1 min) |

**In one line:** CloudFront = **caches** content at the edge (keeps a copy near the user) for HTTP/HTTPS; Global Accelerator = **no cache**, just routes live connections over the AWS backbone — pick it when you need **static anycast IPs, TCP/UDP (gaming/IoT/VoIP), or fast cross-region failover**.

**Exam Trigger:** "Static IP required, UDP traffic, Gaming" → Global Accelerator. "Cache static content" → CloudFront.

---

### **Exam Summary Cheat Sheet — Practice (Fill In Yourself)**

1. **Map root domain to ALB?** →
2. **Route 20% traffic to new version?** →
3. **Route to fastest region?** →
4. **Automatic failover to DR site?** →
5. **Show different content by country?** →
6. **Serve S3 content via CloudFront, Block direct access?** →
7. **Restrict access to premium content?** →
8. **Block access from specific countries?** →
9. **Cache static content globally?** →
10. **Static IP, UDP, Gaming, Fast failover?** →
11. **Replicate S3 to specific region?** →
12. **Manually clear CloudFront cache?** →
13. **Simple URL rewrite or header manipulation at edge?** →
14. **Complex edge logic with network access or body manipulation?** →
15. **CloudFront high availability / origin failover?** →
16. **Internal DNS within a VPC?** →
17. **Internet-facing DNS resolution?** →

---

### **Exam Summary Cheat Sheet — Answer Key**

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
13. **Simple URL rewrite or header manipulation at edge?** → CloudFront Functions (Cheap, sub-ms, viewer only).
14. **Complex edge logic with network access or body manipulation?** → Lambda@Edge (All 4 event types, up to 30 sec).
15. **CloudFront high availability / origin failover?** → Origin Groups (Primary + Secondary origin).
16. **Internal DNS within a VPC?** → Route 53 Private Hosted Zone (Must associate with VPC).
17. **Internet-facing DNS resolution?** → Route 53 Public Hosted Zone.

---

> ### My Attempt — Jun 4 (15/17)
> 1. Alias ✅
> 2. Weighted routing ✅
> 3. ~~Geo-proximity~~ ❌ → **Latency-based routing** (fastest = network latency, not proximity+bias)
> 4. Failover Routing ✅
> 5. Geolocation ✅
> 6. ~~Field-Level Encryption~~ ❌ → **OAC** (FLE = encrypt form fields; block direct S3 = OAC)
> 7. Signed cookie/URL ✅ *(CloudFront term is "Signed", not "pre-signed")*
> 8. Geo Restriction ✅
> 9. CloudFront ✅
> 10. Global Accelerator ✅
> 11. S3 CRR ✅
> 12. Cache invalidation ✅
> 13. CloudFront Functions ✅
> 14. Lambda@Edge ✅
> 15. Origin Groups (primary + secondary) ✅
> 16. Private Hosted Zone ✅
> 17. Public Hosted Zone ✅
>
> **Re-drill:** #3 Latency-based (not Geoproximity) · #6 OAC (not Field-Level Encryption).

---

# **REAL EXAM SCENARIOS**

### Scenario 1

**The Situation:** A company hosts a website on an Application Load Balancer. They want users to access the site using the **root domain** (`example.com`) instead of a subdomain (`www.example.com`). They try to create a CNAME record but Route 53 rejects it.

**The Options:**

A. Use an A Record pointing to the ALB's IP address.

B. Use a CNAME Record pointing to the ALB's DNS name.

C. Use an Alias Record pointing to the ALB's DNS name.

D. Put the ALB behind CloudFront and create a CNAME from the root domain to the CloudFront distribution.

**The Logic:**

- **Trap A — A Record to the ALB's IP:** An A record needs a *fixed* IP. An ALB's IP addresses are managed by AWS and change over time — hardcoding one will break. Not supported.
- **Trap B — CNAME on the root domain:** The DNS standard **forbids a CNAME at the zone apex** (root domain) — it conflicts with the required SOA/NS records. This is why Route 53 rejected it. CNAME only works on subdomains.
- **Trap D — CloudFront + CNAME at the root:** Adding CloudFront doesn't fix the actual problem — a **CNAME still can't live at the zone apex**, so this is rejected for the exact same reason as B. It also bolts on a CDN layer the question never asked for (over-engineered). The apex restriction is about the *record type*, not what it points to.
- **The Fix — Option C:** A Route 53 **Alias record** is AWS-specific and **works at the root domain**. It points to the ALB's DNS name and Route 53 auto-resolves to the ALB's current IPs. It's also free (no query charge). Alias = the answer whenever you need apex-domain → AWS resource.

---

### Scenario 2

**The Situation:** A SaaS application has users worldwide. The application runs in **three regions**: `us-east-1`, `eu-west-1`, and `ap-southeast-1`. The company wants users automatically routed to the **fastest region** based on their location.

**The Options:**

A. Use Weighted Routing with equal weights.

B. Use Latency-Based Routing.

C. Use Geolocation Routing to map continents to regions.

D. Use CloudFront.

**The Logic:**

- **Trap A — Weighted Routing, equal weights:** Weighted routing splits traffic by *percentage*, ignoring where the user is. A user in Tokyo could be sent to `eu-west-1` — the opposite of "fastest region."
- **Trap C — Geolocation Routing:** Routes by the user's *geographic location*, which is not the same as *network latency*. Geography ≠ speed — a path can be physically near but network-slow. Close distractor, but Latency-Based is the precise answer.
- **Trap D — CloudFront:** CloudFront caches content at edge locations. It doesn't *route users to the fastest application backend region* — and dynamic SaaS traffic isn't cacheable anyway. Wrong tool.
- **The Fix — Option B:** **Latency-Based Routing** uses AWS's measured network latency from the user to each region and sends them to the **lowest-latency** one. It optimizes for real performance, dynamically.

---

### Scenario 3

**The Situation:** A streaming service hosts video files in S3 and serves them via CloudFront. Only **paid subscribers** should access videos. Free users should be blocked. The company wants to prevent URL sharing (User shares URL → Another user watches free).

**The Options:**

A. Use S3 bucket policies to allow only authenticated users.

B. Use CloudFront Geo Restriction.

C. Use WAF to block unauthorized IPs.

D. Use CloudFront with Signed URLs that expire after 1 hour.

**The Logic:**

- **Trap A — S3 bucket policy for authenticated users:** When CloudFront serves the content, the *end user* never talks to S3 — **CloudFront** does. An S3 policy can't see or authenticate the individual viewer. Wrong layer.
- **Trap B — CloudFront Geo Restriction:** Geo Restriction allows/blocks whole **countries**. It cannot tell a paying subscriber from a free user in the same country. Wrong granularity.
- **Trap C — WAF blocking unauthorized IPs:** WAF filters by IP/request patterns — it has no concept of "this person paid." It can't distinguish subscriber from free user.
- **The Fix — Option D:** **CloudFront Signed URLs** carry a cryptographic signature your app generates (with a private key) only *after* the user authenticates as a paying subscriber, and they **expire after 1 hour**. A shared URL stops working when it expires — combating URL sharing. Per-user, time-limited access = Signed URLs.

---

### Scenario 4

**The Situation:** A gaming company runs multiplayer servers on EC2 behind Network Load Balancers in **four regions**. Players complain about lag when connecting over the public internet. The game client requires **static IPs** to connect (Hardcoded in client config). The company needs to improve latency and provide failover.

**The Options:**

A. Use CloudFront to cache game data.

B. Use Global Accelerator with 2 Anycast IPs pointing to NLBs.

C. Use Route 53 Latency-Based Routing with health checks.

D. Use Elastic IPs for each NLB and distribute to clients.

**The Logic:**

- **Trap A — CloudFront:** CloudFront caches **HTTP/HTTPS** content at edge. Real-time multiplayer traffic is TCP/UDP and not cacheable — CloudFront doesn't apply.
- **Trap C — Route 53 Latency-Based Routing:** Route 53 hands back *DNS answers* (which can be multiple, changing IPs) — it doesn't give the **static IPs** the game client hardcodes, and DNS routing doesn't accelerate the network path.
- **Trap D — Elastic IPs per NLB:** EIPs *are* static, but you'd hand clients a different IP per region, with no global anycast and no fast cross-region failover. Solves "static" but not "latency" or "failover."
- **The Fix — Option B:** **Global Accelerator** gives **2 static anycast IPs** (perfect for hardcoded clients), routes traffic over the **AWS global backbone** (lower latency than the public internet), supports **TCP/UDP** for gaming, and **fails over to healthy endpoints in under a minute**. Hits every requirement.

---

### Scenario 5

**The Situation:** A website uses CloudFront to serve static assets (CSS, JS, Images) from S3. When developers deploy updates, users see **old cached versions** for hours (24-hour TTL). The company pays $500/month for CloudFront invalidations. They want a better solution.

**The Options:**

A. Use versioned filenames (e.g., `style_v2.css`) and update HTML references.

B. Reduce the CloudFront TTL to 1 minute so updates appear quickly.

C. Run a `/*` wildcard invalidation automatically on every deployment.

D. Set `Cache-Control: no-cache` on the assets so CloudFront revalidates with the origin on every request.

**The Logic:**

- **Trap B — Drop TTL to 1 minute:** Updates would appear within a minute, but it forces near-constant cache misses, hammering the origin and gutting most of CloudFront's performance/cost benefit. Trades a deploy problem for a permanent load problem.
- **Trap C — Wildcard `/*` invalidation each deploy:** It *works* technically, but `/*` invalidations are the **most expensive** way to do this (you're already paying \$500/mo for far less) — automating them makes the bill worse, not better. Treating the symptom, not the cause.
- **Trap D — `no-cache` for revalidation:** The asset is revalidated on **every** request, so the edge constantly phones home to the origin — you lose the cache benefit and add latency. Right for truly dynamic content, wrong for static CSS/JS that rarely changes.
- **The Fix — Option A:** Use **versioned filenames** (`style_v1.css` → `style_v2.css`). A changed file ships under a *new* name, the HTML references it, and CloudFront treats it as a fresh object (cache miss → cache). No invalidations needed, **free**, instant updates, long TTLs stay intact. Industry best practice for CDN cache busting.
