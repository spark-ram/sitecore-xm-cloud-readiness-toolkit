# XM Cloud Architecture Decision Matrix

This matrix helps solution architects evaluate **Retain**, **Modernize**, and **Replace** options across 12 core architectural domains when transitioning from **Sitecore Experience Manager (XM)** or **Sitecore Experience Platform (XP)** to **Sitecore XM Cloud**.

---

## How to Use This Matrix

1. **Identify Current State:** Locate your existing implementation pattern for each domain.
2. **Evaluate Constraints:** Compare options against **Verified Platform Constraints** (what XM Cloud natively supports or prohibits) and **Practitioner Guidance** (trade-offs in cost, speed, and operational overhead).
3. **Document Selection:** Record the chosen option and architectural rationale in [`docs/migration-assessment-template.md`](migration-assessment-template.md).

---

## Domain Comparison Matrix

### 1. Front-End Presentation & Rendering

| Approach | Pattern Description | XM Cloud Compatibility | Pros | Cons | Best Fit Scenario |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Retain** | Keep Sitecore MVC (`.cshtml`) or WebForms (`.aspx`) views running on Content Delivery (CD) servers. | **Not Supported.** XM Cloud has no public CD tier for server-side views. | Zero front-end rewrite effort (only valid if staying on XM/XP). | Blocks migration to XM Cloud completely. | Organizations remaining on self-hosted/PaaS XM/XP. |
| **Modernize** | Refactor existing JSS (React/Next.js) or ASP.NET Core Rendering SDK components to consume Experience Edge GraphQL. | **Supported (Recommended).** | Preserves existing component logic; aligns with XM Cloud native architecture. | Requires updating SDK versions, layout service endpoints, and image/link resolvers. | Existing Sitecore 10.x JSS or headless implementations. |
| **Replace** | Rebuild presentation layer from scratch using **Next.js** (React) with TypeScript and Tailwind/component design system. | **Supported (Standard Path).** | Full compatibility with XM Cloud Pages, Static Site Generation (SSG), and Incremental Static Regeneration (ISR). | Highest front-end engineering effort; requires full UI/UX QA cycle. | Legacy MVC, WebForms, or older SXA Razor solutions. |

### 2. Content Delivery & Edge Layer

| Approach | Pattern Description | XM Cloud Compatibility | Pros | Cons | Best Fit Scenario |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Retain** | Self-managed Azure App Services / VMs running Sitecore CD roles + SQL Web database. | **Not Supported.** | Direct control over IIS, `Web.config`, and local caches. | High infrastructure maintenance; incompatible with XM Cloud SaaS. | Non-XM Cloud topologies only. |
| **Modernize** | Adopt **Sitecore Experience Edge** (included with XM Cloud) as the globally distributed GraphQL content delivery API. | **Native / Mandatory.** | Eliminates CD server patching, SQL Web scaling, and cache-clearing bottlenecks. | Requires adapting queries to GraphQL schema limits and complexity budgets. | All XM Cloud migrations. |
| **Replace** | Build a custom middleware content cache/aggregator in front of Experience Edge for complex multi-source data joining. | **Supported (Complementary).** | Shields front-end from multiple API calls; handles custom rate-limit buffering. | Adds extra infrastructure and cache invalidation complexity. | High-traffic enterprises merging CMS content with real-time ERP/PIM feeds. |

### 3. Front-End Hosting & Edge Runtime

| Approach | Pattern Description | XM Cloud Compatibility | Pros | Cons | Best Fit Scenario |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Retain** | Host front-end on existing on-premises IIS or standard Windows Virtual Machines. | **Discouraged.** | Leverages existing data center contracts. | Poor support for modern Next.js ISR, edge middleware, and global CDN scaling. | Strict on-premises regulatory edge cases (rare). |
| **Modernize** | Host Next.js on managed container/PaaS platforms (Azure Container Apps, AWS Amplify, Azure Static Web Apps). | **Supported.** | Keeps hosting within existing enterprise cloud commitments (Azure/AWS). | Requires manual setup of ISR cache persistence, preview builds, and CDN routing. | Enterprises with strict single-cloud procurement policies. |
| **Replace** | Adopt a purpose-built front-end edge cloud (**Vercel** or **Netlify**). | **Supported (Most Common).** | First-class Next.js support, instant preview deployments, zero-config ISR, and tight XM Cloud Deploy integration. | Additional SaaS vendor contract and usage-based bandwidth/function billing. | Teams prioritizing developer velocity and turnkey Next.js operations. |

### 4. Site Search

| Approach | Pattern Description | XM Cloud Compatibility | Pros | Cons | Best Fit Scenario |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Retain** | Query local Apache Solr or SolrCloud directly via `Sitecore.ContentSearch` APIs. | **Not Supported.** XM Cloud does not expose public Solr endpoints. | Familiar C# LINQ query syntax for legacy developers. | Cannot be used for front-end site search in XM Cloud. | Legacy XP/XM solutions prior to migration. |
| **Modernize** | Migrate existing Solr indexes to **SearchStax Site Search** or **Algolia** using webhook-driven or crawler indexing. | **Supported.** | Fast transition for teams already familiar with Solr concepts; predictable indexing pipelines. | Requires building front-end search UI components via REST/GraphQL APIs. | Teams seeking a straightforward managed search replacement. |
| **Replace** | Adopt **Sitecore Search** or **Coveo** for AI-powered discovery, facets, and behavioral ranking. | **Supported.** | Purpose-built headless search widgets, real-time indexing, and advanced merchandising/relevance tuning. | Higher licensing cost and configuration effort compared to basic keyword search. | Content-heavy, commerce, or multi-brand sites needing advanced search relevance. |

### 5. Digital Asset Management (DAM)

| Approach | Pattern Description | XM Cloud Compatibility | Pros | Cons | Best Fit Scenario |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Retain** | Store all images, PDFs, and videos inside the Sitecore XM Cloud Media Library. | **Supported (With Limits).** | Zero extra licensing; works out-of-the-box with Experience Edge CDN. | Increases environment size; lacks enterprise rights management, video streaming, or asset workflows. | Small-to-medium sites with under 20 GB of static images and PDFs. |
| **Modernize** | Offload videos to a dedicated video host (YouTube, Vimeo, Brightcove) while keeping lightweight images in XM Cloud Media Library. | **Supported.** | Reduces media footprint significantly without purchasing a full enterprise DAM. | Assets are split across two repositories for authors to manage. | Mid-sized marketing sites with moderate video usage. |
| **Replace** | Integrate **Sitecore Content Hub DAM** or a third-party DAM (Cloudinary, Bynder, Aprimo) via XM Cloud DAM connectors. | **Supported.** | Single source of truth for omnichannel assets, DRM, automated renditions, and zero CMS bloat. | Additional licensing cost and asset migration effort. | Global enterprises with >50 GB of assets, strict brand governance, or multiple channels. |

### 6. Personalization & Experimentation

| Approach | Pattern Description | XM Cloud Compatibility | Pros | Cons | Best Fit Scenario |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Retain** | Use Sitecore XP xDB contact facets, pattern cards, and server-side rule engine. | **Not Supported.** xDB and xConnect do not exist in XM Cloud. | Deep historical integration with legacy XP Marketing Control Panel. | Requires re-architecting all personalization rules. | Legacy XP solutions prior to migration. |
| **Modernize** | Use **XM Cloud Embedded Personalization** (included with XM Cloud) for page-level audience variants (geo, UTM, device, referrer). | **Native.** | Included in XM Cloud licensing; integrated directly inside XM Cloud Pages visual editor. | Limited to session/context conditions; does not store long-term cross-session customer profiles. | Organizations needing lightweight, campaign-driven page personalization. |
| **Replace** | Adopt **Sitecore Personalize & CDP** (or Optimizely / Adobe Target) for behavioral segmentation, A/B/n testing, and omnichannel decisioning. | **Supported.** | Full Customer Data Platform (CDP) profiles, real-time event streaming, and server/edge experimentation. | Additional SaaS subscription and front-end SDK instrumentation effort. | Data-mature organizations replacing active XP xDB personalization and A/B testing. |

### 7. Web Forms & Lead Capture

| Approach | Pattern Description | XM Cloud Compatibility | Pros | Cons | Best Fit Scenario |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Retain** | Use legacy Sitecore Forms (MVC) or Web Forms for Marketers (WFFM) with SQL Forms database. | **Not Supported.** | Familiar drag-and-drop builder for legacy XP authors. | Relies on server-side MVC postbacks incompatible with Experience Edge. | Legacy XP/XM solutions prior to migration. |
| **Modernize** | Use **XM Cloud Forms** (SaaS visual form builder included with XM Cloud) or build custom React forms submitting to serverless APIs. | **Supported.** | Native SaaS integration (XM Cloud Forms) or complete UI/validation control (custom React). | XM Cloud Forms has evolving feature parity compared to complex custom .NET submit actions. | Standard lead-gen, contact, and newsletter forms. |
| **Replace** | Embed or API-integrate native forms from your Customer Relationship Management (CRM) / Marketing Automation platform (Marketo, HubSpot, Pardot). | **Supported.** | Direct lead attribution, progressive profiling, and compliance handling inside the CRM without middleware. | Styling third-party embeds to match the design system can require extra CSS effort. | B2B and demand-generation sites where leads route directly to marketing automation. |

### 8. Identity & User Authentication

| Approach | Pattern Description | XM Cloud Compatibility | Pros | Cons | Best Fit Scenario |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Retain** | Use Sitecore `Core` database ASP.NET Membership for both CM authors and public extranet users. | **Partially Supported (CM Only via Cloud Portal).** | No external identity migration needed for basic author accounts. | Public extranet membership cannot enforce item security on Experience Edge. | Public websites with zero gated/authenticated user areas. |
| **Modernize** | Federate CM author logins via **Sitecore Cloud Portal SSO** (OIDC/SAML) while moving extranet logins to NextAuth / Auth.js. | **Supported.** | Centralizes employee access governance; keeps front-end authentication lightweight. | Requires configuring OIDC/SAML claims mapping and front-end session handling. | Organizations with moderate gated content requirements. |
| **Replace** | Implement a dedicated Customer Identity and Access Management (CIAM) platform (**Auth0**, **Microsoft Entra External ID**, **Okta CIC**). | **Supported.** | Enterprise-grade MFA, social login, self-service password reset, and JWT validation at the edge. | Additional licensing and user migration effort from Sitecore `Core` DB. | Customer portals, loyalty programs, and B2B extranets. |

### 9. Custom Business Logic & Extensibility

| Approach | Pattern Description | XM Cloud Compatibility | Pros | Cons | Best Fit Scenario |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Retain** | Deploy custom `.dll` assemblies and `App_Config` patches onto CM and CD servers. | **Restricted (CM Only, Discouraged).** | Reuses existing C# code. | Cannot run on Experience Edge; heavy CM customization risks SaaS build failures and upgrade friction. | Minimal, isolated CM authoring utilities only. |
| **Modernize** | Refactor runtime C# business logic into serverless functions (**Azure Functions**, **AWS Lambda**, **Vercel Functions**) triggered by **XM Cloud Webhooks** or front-end API routes. | **Supported (Recommended).** | Decouples custom logic from CMS upgrades; scales independently; supports modern languages (TypeScript/C#). | Requires distributed logging, secret management, and retry handling. | All custom integrations, notifications, and dynamic API orchestration. |
| **Replace** | Adopt an Integration Platform as a Service (**iPaaS**) such as **Sitecore Connect**, **MuleSoft**, or **Workato** for multi-system orchestration. | **Supported.** | Visual workflow orchestration, pre-built ERP/CRM connectors, and enterprise error monitoring. | Additional iPaaS licensing and governance overhead. | Enterprises with complex, multi-system data synchronization requirements. |

### 10. Content Serialization & CI/CD

| Approach | Pattern Description | XM Cloud Compatibility | Pros | Cons | Best Fit Scenario |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Retain** | Use Team Development for Sitecore (TDS) or Unicorn YAML files with custom PowerShell deployment scripts. | **Not Supported Directly.** | Familiar to legacy Sitecore teams. | XM Cloud Deploy expects Sitecore Content Serialization (SCS) format. | Legacy solutions prior to migration. |
| **Modernize** | Convert Unicorn/TDS configurations to **Sitecore Content Serialization (SCS)** and deploy via **XM Cloud Deploy App** / Sitecore CLI. | **Native / Mandatory.** | Official SaaS deployment pipeline with automated environment provisioning and item sync. | Requires migrating serialization rules and testing item dependency graphs. | All XM Cloud migrations. |
| **Replace** | Pair SCS + XM Cloud Deploy with split-repository or monorepo CI/CD (GitHub Actions / Azure DevOps) separating CMS and Next.js deployments. | **Supported (Recommended).** | Enables front-end teams to deploy UI updates independently without triggering CMS builds. | Requires coordinating release versioning across CMS schema and front-end builds. | Mature product teams with frequent front-end release cadences. |

### 11. Caching & Performance Optimization

| Approach | Pattern Description | XM Cloud Compatibility | Pros | Cons | Best Fit Scenario |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Retain** | Rely on Sitecore in-memory HTML/Data caches cleared on `publish:end`. | **Not Applicable.** | Handled automatically inside monolithic CD instances. | Does not exist in headless Experience Edge architectures. | Legacy monolithic topologies only. |
| **Modernize** | Combine **Experience Edge CDN caching** with **Next.js Static Site Generation (SSG)** and **Incremental Static Regeneration (ISR)**. | **Supported (Recommended).** | Sub-100ms global page loads; high resilience during traffic spikes. | Stale content windows possible unless paired with webhook-triggered on-demand revalidation. | Marketing, editorial, and catalog pages with predictable update cycles. |
| **Replace** | Implement Edge-Side Includes / Distributed Edge Caching (Cloudflare Workers, Akamai EdgeWorkers) with webhook-driven cache purging. | **Supported.** | Fine-grained geo-routing, A/B edge bucketing, and instant global purge control. | Requires specialized edge-computing expertise and configuration management. | High-volume global brands with complex regional routing and strict SLA targets. |

### 12. Analytics & Reporting

| Approach | Pattern Description | XM Cloud Compatibility | Pros | Cons | Best Fit Scenario |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Retain** | Use Sitecore Experience Analytics and xDB SQL/MongoDB reporting databases. | **Not Supported.** | Built into legacy XP Launchpad. | Unavailable in XM Cloud; historical xDB data does not migrate into XM Cloud. | Legacy XP solutions prior to migration. |
| **Modernize** | Standardize on **Google Analytics 4 (GA4)**, **Adobe Analytics**, or **Matomo** via front-end tag management (GTM). | **Supported.** | Industry-standard web analytics decoupled from CMS architecture. | Does not natively tie behavioral events back to CMS authoring insights without extra integration. | Standard marketing and content performance measurement. |
| **Replace** | Implement **Sitecore CDP** alongside **XM Cloud Analytics** for unified visitor tracking and content performance insights. | **Supported.** | Connects behavioral visitor data with personalization decisioning in a composable stack. | Higher licensing and data-governance implementation effort. | Organizations requiring unified behavioral tracking and real-time personalization. |

---

## Official Documentation to Validate

Before finalizing technology selections, validate current capabilities and limits in the official Sitecore Documentation portal (`doc.sitecore.com`) for:
1. **XM Cloud Deploy App & Git Integration:** `[Placeholder: Sitecore Documentation -> XM Cloud -> XM Cloud Deploy -> Supported Source Control Providers]`
2. **XM Cloud Forms Capabilities & Webhooks:** `[Placeholder: Sitecore Documentation -> XM Cloud -> XM Cloud Forms -> Integrating and Submitting Forms]`
3. **Experience Edge GraphQL Complexity & Rate Limits:** `[Placeholder: Sitecore Documentation -> XM Cloud -> Experience Edge -> Query Complexity and Rate Limits]`
4. **Sitecore Connect & Webhook Payload Specifications:** `[Placeholder: Sitecore Documentation -> Sitecore Connect / XM Cloud Webhooks]`
