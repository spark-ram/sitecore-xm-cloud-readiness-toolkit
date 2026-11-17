# Sitecore XM Cloud Migration Risk Register Template

Use this register to identify, quantify, and manage technical, operational, and organizational risks throughout discovery, build, content migration, and production cutover to **Sitecore XM Cloud**.

---

## Risk Scoring Rubric

### Likelihood (1–5)
- **1 (Rare):** <10% probability of occurring during the program.
- **2 (Unlikely):** 10%–25% probability.
- **3 (Possible):** 26%–50% probability.
- **4 (Likely):** 51%–75% probability.
- **5 (Almost Certain):** >75% probability unless proactive mitigation is executed.

### Impact (1–5)
- **1 (Negligible):** Minor inconvenience; resolved within normal sprint capacity (<2 days delay).
- **2 (Minor):** Localized rework or minor scope adjustment (2–5 days delay).
- **3 (Moderate):** Noticeable timeline slip (1–2 weeks) or moderate budget increase.
- **4 (Major):** Significant architectural rework, missed launch milestone (>2 weeks), or degraded user experience.
- **5 (Critical):** Program halt, data loss, severe security/compliance breach, or prolonged production outage.

### Composite Risk Score (`Likelihood × Impact`)
- **15 – 25 (Critical / Red):** Immediate executive attention and mandatory pre-build mitigation plan required.
- **8 – 14 (High / Amber):** Active mitigation tracked weekly by Technical Lead and Delivery Manager.
- **1 – 7 (Low / Green):** Monitor periodically during standard sprint ceremonies.

---

## Active Risk Register

| Risk ID | Category | Risk Statement (Cause -> Event -> Impact) | Likelihood (1–5) | Impact (1–5) | Score (`L×I`) | Rating | Preventive Mitigation Strategy | Contingency Plan (If Triggered) | Early Warning Trigger | Risk Owner | Status |
| :--- | :--- | :--- | :---: | :---: | :---: | :---: | :--- | :--- | :--- | :--- | :--- |
| `RSK-001` | **Architecture / API** | High volume of complex GraphQL queries on multi-component pages exceeds Experience Edge complexity or rate limits, causing intermittent 429/timeout errors at launch. | `3` | `4` | `12` | `High` | Optimize GraphQL fragments, enforce query depth limits, implement Next.js ISR/SSG caching, and run load tests against Experience Edge before go-live. | Add an edge caching/aggregation proxy layer (e.g., Cloudflare/Vercel Data Cache) with extended stale-while-revalidate headers. | GraphQL query complexity warnings or 429 responses during Wave 1 load testing. | `[Lead Architect]` | `Open` |
| `RSK-002` | **Scope / Legacy Debt** | Undocumented custom .NET pipeline processors or HTTP modules are discovered mid-build, delaying legacy CD decommissioning. | `4` | `4` | `16` | `Critical` | Export and audit full `/sitecore/admin/showconfig.aspx` and `Web.config` diffs during Wave 0 before estimating component builds. | Isolate uncovered legacy routes behind CDN path-based routing to legacy CD while refactoring in a fast-follow sprint. | Discovery of unmapped `.dll` patches or `httpRequestBegin` overrides during Wave 0 code audit. | `[Technical Lead]` | `Open` |
| `RSK-003` | **Content Migration** | Uncleaned legacy content trees (>250k items and unoptimized media) cause prolonged migration script runtimes and broken internal links. | `4` | `3` | `12` | `High` | Enforce content freeze windows, archive obsolete items prior to migration, offload large videos to DAM/CDN, and run automated link-integrity checks. | Migrate active top-level pages first for go-live and batch-migrate deep historical archives post-cutover. | Dry-run migration exceeds 8 hours or produces >2% broken media/link errors. | `[Content Ops Lead]` | `Open` |
| `RSK-004` | **Search & SEO** | Replacing Solr with a new SaaS search provider alters URL structures, faceted navigation, or metadata rendering, causing temporary organic search ranking drops. | `3` | `5` | `15` | `Critical` | Build a comprehensive 1:1 301-redirect map, validate canonical tags, structured data (JSON-LD), and XML sitemaps in UAT prior to DNS cutover. | Immediately deploy edge redirect rules for top 1,000 organic landing pages and submit updated sitemaps to search consoles. | Missing canonical tags, mismatched status codes, or >5% crawl errors in pre-launch SEO audit. | `[SEO / Front-End Lead]` | `Open` |
| `RSK-005` | **Team Readiness** | Development team experienced primarily in server-side C#/MVC faces a steep learning curve with Next.js, TypeScript, and GraphQL, reducing early sprint velocity. | `3` | `3` | `9` | `High` | Deliver targeted Next.js/JSS workshops in Wave 0, establish strict component starter templates, and pair senior headless engineers with C# developers. | Bring in temporary headless front-end specialists to establish core architecture patterns during Waves 0–1. | Wave 0 sprint velocity falls >25% below baseline forecast. | `[Delivery Manager]` | `Open` |
| `RSK-006` | **Integrations** | Third-party CRM/ERP API endpoints lack rate capacity or webhook resilience when decoupled from legacy Sitecore queue mechanisms. | `2` | `4` | `8` | `High` | Introduce lightweight message queuing (e.g., Azure Service Bus / SQS) between Next.js API routes/webhooks and downstream ERP/CRM endpoints. | Enable graceful degradation UI states (e.g., async confirmation message) and replay failed payloads from dead-letter queues. | Downstream API latency >2s or error rate >1% during integration stress tests. | `[Integration Lead]` | `Open` |
| `RSK-007` | **Governance / Licensing** | Delays in procuring third-party composable SaaS contracts (Front-end Host, Search, CIAM) block environment setup for Wave 1. | `3` | `4` | `12` | `High` | Present architectural decisions requiring approval at the end of discovery and initiate InfoSec/procurement reviews immediately. | Provision free/developer tiers of Vercel/Search/Auth0 for Wave 0–1 engineering while enterprise contracts finalize. | Procurement approval remains pending >10 business days after discovery readout. | `[Program Sponsor]` | `Open` |

---

## Official Documentation to Validate

Review current operational limits and SLAs in the official Sitecore Documentation (`doc.sitecore.com`) when calibrating risk likelihoods:
1. **Experience Edge Rate Limits & Fair Use Policies:** `[Placeholder: Sitecore Documentation -> XM Cloud -> Experience Edge -> Rate Limits and Throttling]`
2. **XM Cloud Backup, Restore, and Disaster Recovery Procedures:** `[Placeholder: Sitecore Documentation -> XM Cloud -> Administration -> Backup and Restore]`
3. **XM Cloud Service Level Agreement (SLA) & Status Monitoring:** `[Placeholder: Sitecore Trust Center / Sitecore Cloud Portal Status Documentation]`
