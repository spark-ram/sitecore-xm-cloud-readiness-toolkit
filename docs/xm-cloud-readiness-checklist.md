# Sitecore XM Cloud Readiness Checklist

This checklist contains **35 assessment questions** organized across **15 architectural and operational domains**. Use it during technical discovery workshops to evaluate an existing **Sitecore Experience Manager (XM)** or **Sitecore Experience Platform (XP)** implementation for migration to **Sitecore XM Cloud**.

---

## Fact vs. Practice Note

- **Verified Product Architecture Facts:** Sitecore XM Cloud is a Software-as-a-Service (SaaS), headless-first Content Management System (CMS) that publishes content to **Sitecore Experience Edge** via GraphQL APIs. It does not include traditional Content Delivery (CD) servers, local Apache Solr instances, or the Sitecore Experience Database (xDB) / xConnect analytics stack.
- **Practitioner Recommendations:** The Red/Amber/Green (RAG) thresholds and remediation actions below reflect field-tested migration engineering practices and should be calibrated to your organization's delivery capacity and timeline.

---

## 1. Rendering & Presentation Layer

### Q1.1 — What front-end rendering architecture is currently used?
- **Evidence to Collect:** Visual Studio solution structure, `App_Config` rendering definitions, front-end `package.json` files, and layout service configurations.
- **RAG Criteria:**
  - **Green:** Headless architecture using Sitecore JavaScript Rendering SDKs (JSS) or ASP.NET Core Rendering SDK consuming Layout Service / Experience Edge GraphQL.
  - **Amber:** Hybrid solution mixing Sitecore Experience Accelerator (SXA) MVC renderings with partial headless or API-driven components.
  - **Red:** Pure Sitecore MVC (`.cshtml` views with controller renderings) or legacy ASP.NET WebForms (`.aspx`).
- **Migration Implication:** XM Cloud does not host a public-facing CD tier to execute server-side MVC or WebForms views; all presentation must run on an external front-end rendering host (e.g., Next.js).
- **Recommended Next Action:** Catalog every rendering item and map MVC/WebForms views to headless component equivalents.

### Q1.2 — How are front-end assets, styling, and component libraries structured?
- **Evidence to Collect:** Front-end build scripts, CSS/SCSS pipelines, component Storybook/design system repositories, and SXA Creative Exchange exports.
- **RAG Criteria:**
  - **Green:** Modular component library (e.g., React/Next.js) decoupled from Content Management (CM) server file paths.
  - **Amber:** SXA themes stored in the Sitecore Media Library with moderate custom JavaScript dependencies.
  - **Red:** Tightly coupled jQuery/Razor scripts relying on `Sitecore.Context` variables injected directly into HTML markup.
- **Migration Implication:** Decoupled component libraries accelerate front-end migration, whereas markup tightly coupled to server-side `Sitecore.Context` requires full UI re-engineering.
- **Recommended Next Action:** Audit front-end components for server-side context dependencies and establish a headless component design system.

### Q1.3 — Is the solution compatible with XM Cloud Pages (the visual WYSIWYG editor)?
- **Evidence to Collect:** Placeholder settings, component datasources, SXA grid configurations, and custom Experience Editor ribbon buttons or JavaScript extensions.
- **RAG Criteria:**
  - **Green:** Components use standard JSON renderings, dynamic placeholders, and clean datasource templates without custom Experience Editor hacks.
  - **Amber:** Heavy use of custom Experience Editor buttons, Edit Frames, or legacy SXA rendering variants requiring Scriban-to-React translation.
  - **Red:** Extensive custom Experience Editor SPEAK dialogs or incompatible layout structures that break visual editing in headless mode.
- **Migration Implication:** XM Cloud Pages requires headless rendering host connectivity and standard placeholder/datasource conventions to support inline editing.
- **Recommended Next Action:** Validate core page compositions against headless placeholder standards and identify custom editing extensions to retire or redesign.

---

## 2. Content Model & Information Architecture

### Q2.1 — How clean and standardized is the Sitecore template inheritance hierarchy?
- **Evidence to Collect:** Serialized template definitions (Unicorn or Sitecore Content Serialization [SCS]), template inheritance reports, and unused field audits.
- **RAG Criteria:**
  - **Green:** Well-structured Helix-aligned or modular base templates with clear field naming and minimal duplication.
  - **Amber:** Moderate template sprawl with overlapping fields across sites, requiring consolidation before migration.
  - **Red:** Circular inheritance, hundreds of single-use templates, or widespread reliance on deprecated field types.
- **Migration Implication:** Complex or inconsistent templates complicate GraphQL schema generation and increase front-end mapping effort.
- **Recommended Next Action:** Run a template usage audit to archive unused templates and normalize field names across shared base templates.

### Q2.2 — How large is the active content tree, and what is the item-to-template ratio?
- **Evidence to Collect:** Database item count queries (`Items` table counts by path and language), media library size metrics, and archival policies.
- **RAG Criteria:**
  - **Green:** Under 100,000 active content items with clear archival lifecycle rules.
  - **Amber:** 100,000 to 500,000 items, including historical news/blog items that have never been archived.
  - **Red:** Over 500,000 items or deeply nested bucketed structures exceeding standard publishing and GraphQL query performance thresholds.
- **Migration Implication:** Large content volumes impact initial content migration duration, publishing throughput, and Experience Edge storage/query limits.
- **Recommended Next Action:** Establish a content retention policy and archive stale items prior to executing content migration scripts.

---

## 3. Custom .NET Code & CM Extensibility

### Q3.1 — What custom .NET assemblies and server-side customizations exist in the solution?
- **Evidence to Collect:** Custom `.dll` inventory in `/bin`, Visual Studio project list, and `App_Config/Include` patch files.
- **RAG Criteria:**
  - **Green:** Zero or minimal CM-only utilities (e.g., simple custom field validators or serialization configs).
  - **Amber:** Moderate custom business logic in `.dll` files (e.g., custom tokens, computed fields, or scheduled agents) that can be externalized to webhooks or serverless functions.
  - **Red:** Heavy custom .NET solutions overriding core Sitecore kernel behavior, custom databases, or direct SQL access alongside Sitecore databases.
- **Migration Implication:** While XM Cloud allows limited CM customization via build deployments, heavy server-side .NET code conflicts with SaaS upgradeability and must be externalized.
- **Recommended Next Action:** Classify every custom `.dll` by execution context (CM authoring vs. runtime delivery) and target externalization for runtime logic.

### Q3.2 — Does the application rely on custom HTTP modules, handlers, or Global.asax overrides?
- **Evidence to Collect:** `Web.config` (`system.webServer/modules` and `handlers`), `Global.asax` code-behind, and URL rewrite rules.
- **RAG Criteria:**
  - **Green:** No custom HTTP modules/handlers; routing and redirects are handled via standard Sitecore item configuration or CDN/edge rules.
  - **Amber:** Custom URL rewrite rules or redirect modules that can be migrated to Next.js middleware or Edge/CDN rules.
  - **Red:** Custom HTTP modules intercepting every request for session state, authentication, or dynamic response rewriting.
- **Migration Implication:** Custom HTTP modules on CD servers cannot run in XM Cloud; request interception must move to the front-end hosting edge (e.g., Next.js Middleware) or CDN.
- **Recommended Next Action:** Extract all `Web.config` rewrite rules and HTTP module logic into an edge-routing specification.

---

## 4. Pipelines, Events & Background Jobs

### Q4.1 — Which standard Sitecore pipelines are patched or overridden?
- **Evidence to Collect:** Output from `/sitecore/admin/showconfig.aspx` filtered for `<pipelines>`, `<events>`, and `<hooks>`.
- **RAG Criteria:**
  - **Green:** Only standard Sitecore or official module patches; no custom processors in `httpRequestBegin`, `renderField`, or `publishItem`.
  - **Amber:** Custom processors in `publish:end` or `item:saved` used for cache clearing or downstream notifications.
  - **Red:** Deep customization of `httpRequestBegin`, `item:resolving`, or `mvc.getRenderer` that alters core request/item resolution.
- **Migration Implication:** Request-time pipelines do not execute on Experience Edge; publishing and item events should transition to **XM Cloud Webhooks** or serverless event consumers.
- **Recommended Next Action:** Map every custom pipeline processor and event handler to either an XM Cloud webhook event or front-end routing logic.

### Q4.2 — How are scheduled tasks and background jobs implemented?
- **Evidence to Collect:** `/sitecore/system/Tasks/Schedules` items, custom `Sitecore.Tasks` agents in `Web.config`, and Windows Task Scheduler / Azure WebJobs definitions.
- **RAG Criteria:**
  - **Green:** No custom background jobs inside Sitecore, or externalized cron/serverless schedulers calling APIs.
  - **Amber:** Low-frequency Sitecore scheduled tasks (e.g., nightly content cleanup) that can move to external serverless cron jobs.
  - **Red:** High-frequency, long-running background agents running inside the CM worker process (e.g., continuous ERP product imports).
- **Migration Implication:** Long-running or high-frequency background jobs degrade SaaS CM stability and should be offloaded to external integration middleware or serverless functions.
- **Recommended Next Action:** Move data ingestion and batch processing jobs out of the Sitecore CM process into dedicated integration services.

---

## 5. Search Architecture

### Q5.1 — How is site search currently implemented (providers, indexes, and query patterns)?
- **Evidence to Collect:** `Sitecore.ContentSearch` configurations, Solr schema files, Coveo/SearchStax configs, and search API controller code.
- **RAG Criteria:**
  - **Green:** Already using a SaaS search engine (e.g., Sitecore Search, Coveo Cloud, Algolia) decoupled from local Solr.
  - **Amber:** Using Solr (or SolrCloud) with standard keyword/faceted search that can be re-indexed into a SaaS search provider.
  - **Red:** Complex custom `ComputedIndexField` classes, spatial search, or multi-index joins tightly bound to `Sitecore.ContentSearch` APIs.
- **Migration Implication:** XM Cloud does not expose a public-facing Solr instance for front-end search queries; front-end search requires an external SaaS search service.
- **Recommended Next Action:** Inventory all search indexes, facets, and computed fields to design an external search index schema.

### Q5.2 — Are internal content authoring search queries reliant on custom Solr configurations?
- **Evidence to Collect:** Custom CM search index patches and custom Content Editor search filters.
- **RAG Criteria:**
  - **Green:** Default CM search behavior without custom index schemas.
  - **Amber:** Minor custom search filters in Content Editor that can be adapted to standard XM Cloud search capabilities.
  - **Red:** Heavy reliance on custom CM Solr analyzers or third-party indexing plugins for editorial workflows.
- **Migration Implication:** XM Cloud manages internal CM search indexing as part of the SaaS service without direct access to raw Solr configuration files.
- **Recommended Next Action:** Validate editorial search requirements against standard XM Cloud Pages and Content Editor capabilities.

---

## 6. Personalization & Analytics

### Q6.1 — What level of Sitecore XP personalization is actively used in production?
- **Evidence to Collect:** Personalization rule audits across presentation details, marketing control panel profiles/pattern cards, and active A/B tests.
- **RAG Criteria:**
  - **Green:** No personalization, or simple session/geo/device rules supported natively by XM Cloud embedded personalization.
  - **Amber:** Moderate rule-based personalization using campaign codes or visitor history that can transition to **Sitecore Personalize** or edge middleware.
  - **Red:** Extensive real-time personalization driven by custom xDB contact facets, custom rule conditions, or complex multi-step engagement plans.
- **Migration Implication:** XM Cloud includes lightweight embedded personalization (e.g., page-level variants based on context/UTM/geography), while advanced CDP/experimentation requires **Sitecore Personalize** or an equivalent tool.
- **Recommended Next Action:** Audit active personalization rules to separate actively maintained, high-value rules from unused legacy rules.

### Q6.2 — Does the organization depend on Sitecore xDB, xConnect, or Experience Analytics?
- **Evidence to Collect:** xConnect connection strings, custom `Facet` models, Marketing Automation campaigns, and Experience Analytics reporting usage.
- **RAG Criteria:**
  - **Green:** Running Sitecore XM (no xDB) or XP with xDB disabled/unused in favor of external analytics (e.g., Google Analytics 4).
  - **Amber:** Collecting basic xDB visit data without custom facets, where historical data export is optional.
  - **Red:** Business-critical customer profiles, custom xConnect facets, or active Marketing Automation workflows stored in xDB.
- **Migration Implication:** XM Cloud does not include xDB or xConnect; teams requiring customer data platform capabilities must adopt **Sitecore CDP** or an external Customer Data Platform (CDP) and plan historical data migration/archival.
- **Recommended Next Action:** Confirm data retention requirements for historical xDB records and select a target analytics/CDP platform.

---

## 7. Integrations & External Systems

### Q7.1 — How many external systems integrate with Sitecore, and what patterns are used?
- **Evidence to Collect:** Architecture diagrams, API client code, connection strings, and configuration files (ERP, CRM, PIM, OMS, identity providers).
- **RAG Criteria:**
  - **Green:** Fewer than 5 integrations using asynchronous APIs, webhooks, or client-side calls.
  - **Amber:** 5 to 15 integrations, including scheduled batch imports into Sitecore items.
  - **Red:** Over 15 tightly coupled, synchronous server-side integrations executing during page requests or publishing pipelines.
- **Migration Implication:** Synchronous server-side integrations on CD must be re-architected into front-end API routes, edge functions, or integration middleware (iPaaS).
- **Recommended Next Action:** Complete [`docs/integration-inventory-template.md`](integration-inventory-template.md) for every inbound and outbound touchpoint.

### Q7.2 — Are external systems pushing content directly into Sitecore databases or via legacy APIs?
- **Evidence to Collect:** Sitecore Item Web API configs, custom `.asmx`/WCF endpoints, PowerShell Extensions (SPE) ingestion scripts, or direct SQL write scripts.
- **RAG Criteria:**
  - **Green:** Content ingestion uses GraphQL Authoring/Management APIs or decoupled front-end aggregation.
  - **Amber:** Scheduled SPE scripts or custom REST endpoints creating/updating items in the Master database.
  - **Red:** Direct SQL database writes or legacy SOAP/WCF services tightly bound to local network access.
- **Migration Implication:** XM Cloud requires content mutations to go through supported SaaS APIs (GraphQL Authoring API) or standard authoring interfaces, with zero direct SQL access.
- **Recommended Next Action:** Refactor legacy item-creation integrations to use the XM Cloud GraphQL Authoring API or evaluate whether volatile data (e.g., pricing/inventory) should bypass the CMS entirely.

---

## 8. Identity, Authentication & Access Control

### Q8.1 — How do content authors and administrators authenticate into Sitecore CM?
- **Evidence to Collect:** Sitecore Identity Server configuration, Federated Authentication (`owin.authentication`) patches, and Active Directory/Okta/Entra ID settings.
- **RAG Criteria:**
  - **Green:** Standard Sitecore Cloud Portal authentication or single sign-on (SSO) via OpenID Connect (OIDC) / SAML 2.0.
  - **Amber:** Legacy Sitecore Identity Server with custom claims transformations that need mapping to Sitecore Cloud Portal SSO.
  - **Red:** Custom ASP.NET Membership providers or on-premises Active Directory (LDAP) without a cloud identity bridge.
- **Migration Implication:** XM Cloud manages CM authentication centrally through the **Sitecore Cloud Portal** (supporting enterprise SSO federation).
- **Recommended Next Action:** Verify enterprise Identity Provider (IdP) compatibility with Sitecore Cloud Portal SSO requirements.

### Q8.2 — Does the public website require authenticated extranet/member portals?
- **Evidence to Collect:** Sitecore `extranet` domain usage, custom login/registration controllers, role-based item security on public pages, and user profile fields.
- **RAG Criteria:**
  - **Green:** Public website is anonymous, or authentication is already handled externally (e.g., Auth0, Okta, Azure AD B2C) via token-based front-end flows.
  - **Amber:** Simple gated content area that can be migrated to Next.js Middleware + external IdP validation.
  - **Red:** Thousands of extranet users stored in the Sitecore `Core` database with custom profile properties and granular Sitecore item-level read permissions.
- **Migration Implication:** Experience Edge serves published content via API keys and is not designed to enforce per-user Sitecore `extranet` item security; authenticated experiences should be managed at the front-end/IdP layer.
- **Recommended Next Action:** Plan a user migration from Sitecore `extranet` membership to an external Customer Identity and Access Management (CIAM) provider.

---

## 9. Media & Digital Asset Management (DAM)

### Q9.1 — Where are media assets stored, and what is the total media library footprint?
- **Evidence to Collect:** Media Library size reports, `Media.UploadAsFiles` / Azure Blob Storage module configs, and external DAM connectors.
- **RAG Criteria:**
  - **Green:** Media assets are managed in an external DAM (e.g., Sitecore Content Hub DAM, Cloudinary, Bynder) or a clean Media Library under 20 GB.
  - **Amber:** Media Library between 20 GB and 100 GB stored in SQL/Blob storage with mixed asset hygiene.
  - **Red:** Over 100 GB of unoptimized video, PDF, and high-resolution image assets stored directly inside the Sitecore Media Library.
- **Migration Implication:** Large media libraries slow down environment provisioning, serialization, and edge publishing; video and large binaries are best offloaded to a dedicated DAM or CDN.
- **Recommended Next Action:** Audit media asset types and sizes; offload videos and large downloads to a dedicated DAM or cloud storage CDN.

### Q9.2 — Does the site rely on custom server-side image processing or media request protection?
- **Evidence to Collect:** Custom `mediaRequestSessionModule` patches, image cropping/watermarking pipelines, and media URL query parameter handlers.
- **RAG Criteria:**
  - **Green:** Standard Sitecore image resizing parameters (`mw`, `mh`) or Next.js Image Optimization / DAM transformations.
  - **Amber:** Custom image transformation parameters that can be replicated via front-end image optimization loaders or CDN transformations.
  - **Red:** Custom server-side watermarking, dynamic PDF generation, or authenticated media download handlers inside the Sitecore media pipeline.
- **Migration Implication:** Experience Edge CDN supports standard image resizing parameters, while custom server-side media pipelines must move to edge functions or a DAM.
- **Recommended Next Action:** Map custom media transformations to Next.js `<Image>` loaders, CDN features, or DAM transformation URLs.

---

## 10. Forms & Lead Capture

### Q10.1 — How are web forms built and processed across the site?
- **Evidence to Collect:** Sitecore Forms database item counts, Web Forms for Marketers (WFFM) legacy items, custom submit actions, and third-party embed scripts (Marketo, HubSpot, Salesforce).
- **RAG Criteria:**
  - **Green:** Forms are already provided by a marketing automation platform (e.g., Marketo, HubSpot) or headless form service (e.g., Formspree, custom API).
  - **Amber:** Standard Sitecore Forms using basic email or webhook submit actions without complex multi-step server state.
  - **Red:** Legacy WFFM forms or heavily customized Sitecore Forms with custom .NET submit actions, file uploads to internal shares, and conditional server-side branching.
- **Migration Implication:** Legacy Sitecore Forms relies on server-side MVC postbacks and a dedicated forms database not suited for stateless Experience Edge delivery; teams typically adopt external form providers, custom React forms, or SaaS form capabilities.
- **Recommended Next Action:** Inventory all active forms, submission destinations, and compliance requirements (e.g., CAPTCHA, GDPR consent) to choose a composable form strategy.

---

## 11. Multilingual & Multisite Architecture

### Q11.1 — How many sites, languages, and regional fallback rules exist in the instance?
- **Evidence to Collect:** `<sites>` configuration in `showconfig.aspx`, `/sitecore/system/Languages` items, and language fallback settings (item-level and field-level).
- **RAG Criteria:**
  - **Green:** 1 to 10 sites with standard language configurations and clean SXA/headless site definitions.
  - **Amber:** 10 to 30 sites or complex field-level language fallback chains requiring careful Experience Edge query testing.
  - **Red:** Over 30 sites with bespoke host-name resolution pipelines, custom dictionary providers, or conflicting wildcard site definitions.
- **Migration Implication:** XM Cloud natively supports multisite management and language fallback, but large site counts require structured site grouping and front-end multi-tenant routing (e.g., Next.js Multisite add-on).
- **Recommended Next Action:** Document the complete site/hostname/language matrix and verify fallback behavior in Experience Edge GraphQL queries.

---

## 12. DevOps, CI/CD & Environments

### Q12.1 — How are code deployments and item serializations currently managed?
- **Evidence to Collect:** Azure DevOps / GitHub Actions pipelines, Unicorn or TDS (Team Development for Sitecore) configurations, and environment topology diagrams.
- **RAG Criteria:**
  - **Green:** Git-based CI/CD pipelines using Sitecore Content Serialization (SCS) and automated build/deploy steps.
  - **Amber:** Using Unicorn or TDS with structured serialization rules that can be converted to SCS.
  - **Red:** Manual package installations (`.zip` update packages), manual `Web.config` edits on servers, or lack of item serialization in source control.
- **Migration Implication:** XM Cloud deploys code and serialized items via the **XM Cloud Deploy App** (GitHub/Azure DevOps integration) and CLI, requiring SCS format for item serialization.
- **Recommended Next Action:** Migrate Unicorn/TDS serialization configurations to Sitecore Content Serialization (SCS) modules.

### Q12.2 — How many non-production environments (Dev, QA, UAT, Staging) are required?
- **Evidence to Collect:** Current infrastructure environment list, release branching strategy, and parallel workstream schedules.
- **RAG Criteria:**
  - **Green:** Standard 2 to 3 non-production environments aligning with XM Cloud project/environment entitlements.
  - **Amber:** 4 to 5 environments requiring environment sharing or ephemeral preview environments on the front-end host.
  - **Red:** Dozens of persistent dedicated QA/feature environments per developer team.
- **Migration Implication:** XM Cloud environments are provisioned under SaaS subscription entitlements; front-end preview branches (e.g., Vercel Preview Deployments) can often replace full CMS environments for UI testing.
- **Recommended Next Action:** Map current environment usage to XM Cloud environment entitlements and front-end preview workflows.

---

## 13. Security, Compliance & Governance

### Q13.1 — Are there data residency, regulatory, or private network isolation requirements?
- **Evidence to Collect:** Information security policies, GDPR/HIPAA/PCI-DSS compliance scopes, IP allowlisting rules, and VPN/ExpressRoute diagrams.
- **RAG Criteria:**
  - **Green:** Standard public cloud SaaS compliance acceptable within available XM Cloud hosting regions.
  - **Amber:** Strict regional data residency (e.g., EU-only) requiring verification of available XM Cloud data center regions.
  - **Red:** Mandatory air-gapped on-premises hosting, customer-managed encryption keys (BYOK) not supported by SaaS, or PCI-DSS cardholder data touching the CMS.
- **Migration Implication:** As a multi-tenant/managed SaaS platform, XM Cloud operates within Sitecore's cloud regions and security boundaries; sensitive PII/PCI workloads must remain outside the CMS.
- **Recommended Next Action:** Conduct an InfoSec architecture review against Sitecore's Trust Center documentation and regional availability.

---

## 14. Performance, Caching & Edge Delivery

### Q14.1 — How does the current site handle caching, cache invalidation, and traffic spikes?
- **Evidence to Collect:** Sitecore HTML cache configurations, CDN rules (Akamai, Cloudflare, Azure Front Door), and peak traffic/requests-per-second metrics.
- **RAG Criteria:**
  - **Green:** Stateless pages well-suited for Incremental Static Regeneration (ISR), Static Site Generation (SSG), or CDN edge caching.
  - **Amber:** High-frequency content updates requiring fine-grained on-demand revalidation via Experience Edge webhooks.
  - **Red:** Every page request bypasses cache due to server-side session state or real-time per-user database queries.
- **Migration Implication:** Experience Edge provides a globally distributed GraphQL CDN, but optimal performance requires designing Next.js caching strategies (SSG/ISR/SSR) to avoid unnecessary origin or API rate-limit pressure.
- **Recommended Next Action:** Classify page templates by freshness requirements (static, periodic ISR, webhook-invalidated, or dynamic SSR).

---

## 15. Content Operations & Editorial Workflows

### Q15.1 — How complex are editorial workflows, publishing approvals, and localization processes?
- **Evidence to Collect:** `/sitecore/system/Workflows` definitions, custom workflow actions, translation connectors (Lionbridge, SDL, Smartling), and author role matrices.
- **RAG Criteria:**
  - **Green:** Standard Sitecore workflows (Draft -> Awaiting Approval -> Approved) and modern API-based translation connectors.
  - **Amber:** Custom email notification actions or scheduled publishing scripts requiring adaptation to XM Cloud webhooks/publishing targets.
  - **Red:** Deeply customized workflow .NET actions calling internal ERP systems or deprecated desktop-only translation modules.
- **Migration Implication:** Standard Sitecore workflows carry over cleanly to XM Cloud, while custom .NET workflow actions must be refactored or replaced with webhooks.
- **Recommended Next Action:** Audit active workflow states/actions and verify XM Cloud compatibility with your translation management vendor.

---

## Official Documentation to Validate

Before finalizing architectural recommendations, verify the latest specifications in the official Sitecore Documentation portal (`doc.sitecore.com`) for:
1. **XM Cloud Architecture & Limitations:** `[Placeholder: Sitecore Documentation -> XM Cloud -> Architecture and Limitations]`
2. **Experience Edge GraphQL Schema & Rate Limits:** `[Placeholder: Sitecore Documentation -> XM Cloud -> Experience Edge for XM -> API Limits and Caching]`
3. **XM Cloud Webhooks & Eventing:** `[Placeholder: Sitecore Documentation -> XM Cloud -> Developer Tools -> Webhooks]`
4. **Sitecore Content Serialization (SCS) CLI:** `[Placeholder: Sitecore Documentation -> XM Cloud -> Developer Tools -> Sitecore CLI and Serialization]`
5. **XM Cloud Embedded Personalization vs. Sitecore Personalize:** `[Placeholder: Sitecore Documentation -> XM Cloud -> Personalization and Analytics]`
