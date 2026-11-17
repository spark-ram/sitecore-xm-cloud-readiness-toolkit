# Worked Example: Sitecore XM Cloud Migration Assessment — Northstar Outdoor Gear (Fictional)

> **Note:** This document is a **worked reference example** featuring a fictional multilingual retail organization (**Northstar Outdoor Gear**). All figures, system names, and findings are illustrative mock data created to demonstrate how to populate the toolkit templates.

---

## Document Control

| Metadata Field | Value |
| :--- | :--- |
| **Organization / Program Name** | Northstar Outdoor Gear — Digital Platform Modernization |
| **Assessment Version** | `1.0 (Final Discovery Readout)` |
| **Assessment Date** | `2026-11-10` |
| **Lead Solution Architect** | Principal Digital Architect (Advisory Practice) |
| **Discovery Participants** | VP of Digital Engineering, Lead Sitecore Developer, Enterprise Architect, Digital Merchandising Lead, InfoSec Analyst |
| **Toolkit Reference Version** | `sitecore-xm-cloud-readiness-toolkit v1.0.0` |

---

## 1. Executive Summary

### 1.1 Assessment Objective
Northstar Outdoor Gear operates four regional e-commerce and brand storytelling websites across North America and Western Europe on **Sitecore Experience Platform (XP) 10.1** hosted in Azure Platform-as-a-Service (PaaS). Leadership initiated this discovery assessment to evaluate migrating to **Sitecore XM Cloud** in order to eliminate quarterly Azure infrastructure patching overhead, improve Core Web Vitals on mobile product landing pages, and decouple front-end releases from backend CMS deployments.

### 1.2 Overall Readiness Verdict
- **Aggregate Checklist Friction Score:** `34 / 70`
- **Overall Readiness Rating:** **Amber (Moderate Transformation)**
- **Recommended Migration Strategy:** **3-Wave Composable Transition** — establish a Next.js + Vercel front-end rendering layer, replace local Solr and Sitecore Forms with SaaS equivalents, migrate content via automated scripts after archiving stale items, and roll out regional sites incrementally starting with the Canadian (`en-CA` / `fr-CA`) storefront.

### 1.3 Key Takeaways for Leadership
1. **Platform Shift, Not an Upgrade:** Northstar's 85 Sitecore MVC renderings, local SolrCloud cluster, and custom xDB loyalty facets cannot lift-and-shift into XM Cloud. A headless Next.js front-end rebuild is required.
2. **Strong Content Foundation:** Northstar's content templates follow clean modular inheritance, and product catalog master data already lives in a separate Product Information Management (PIM) system—meaning XM Cloud only needs to manage editorial, campaign, and landing-page content.
3. **Critical Path Dependencies:** Replacing the custom Sitecore `extranet` loyalty portal login and migrating 18 active Sitecore Forms to native CRM/serverless forms are the primary architectural prerequisites before Wave 1 cutover.

---

## 2. Current Landscape Overview

### 2.1 Existing Sitecore Topology

| Attribute | Current Production Specification |
| :--- | :--- |
| **Sitecore Product & Version** | Sitecore XP 10.1.2 (XM + xDB enabled) |
| **Hosting Model** | Azure PaaS (1 CM App Service, 4 auto-scaled CD App Services, Azure SQL Elastic Pool) |
| **Presentation Framework** | Sitecore MVC (`.cshtml`) with 85 custom controller/view renderings (non-SXA) |
| **Search Provider** | SearchStax Managed Solr 8.4 (3-node cluster) |
| **Database Footprint** | Master: 11 GB, Web: 9 GB, Core: 1.5 GB, xDB Shards: 95 GB (mostly unqueried anonymous visits) |
| **Media Library Footprint** | 62 GB stored in Azure Blob Storage (~24,000 images, 420 PDF trail guides, 18 MP4 videos) |
| **Active Sites & Languages** | 4 sites (`northstar.com`, `northstar.ca`, `northstar.co.uk`, `northstar.de`) across 4 languages (`en-US`, `en-CA`, `fr-CA`, `en-GB`, `de-DE`) |
| **Serialization & CI/CD** | Unicorn 4.1.4 + Azure DevOps YAML pipelines |

### 2.2 Current vs. Target Architecture

```text
CURRENT STATE (XP 10.1 PAAS)                  TARGET STATE (XM CLOUD COMPOSABLE)
+------------------------------------+        +------------------------------------+
| Azure App Services (CM + 4x CD)    |        | Sitecore XM Cloud (SaaS CM)        |
|  - 85 MVC Renderings               |  ===>  |  - SCS Item Serialization          |
|  - Custom .NET Loyalty & Cart APIs |        |  - Webhooks -> Azure Functions     |
|  - Managed Solr 8.4                |        +-----------------+------------------+
|  - xDB Analytics + Sitecore Forms  |                          | GraphQL
+------------------------------------+                          v
                                              +------------------------------------+
                                              | Sitecore Experience Edge (CDN)     |
                                              +-----------------+------------------+
                                                                |
                                                                v
                                              +------------------------------------+
                                              | Next.js 15 on Vercel Enterprise    |
                                              |  + Sitecore Search + Auth0 CIAM    |
                                              +------------------------------------+
```

---

## 3. Discovery Assumptions & Constraints

### 3.1 Verified Platform Architecture Facts
- XM Cloud delivers published content exclusively via **Sitecore Experience Edge** GraphQL APIs; no CD servers or direct SQL database connections exist in production.
- Sitecore xDB/xConnect and local Solr search APIs are unavailable in XM Cloud.
- Customer authentication and item-level `extranet` security cannot be enforced directly by Experience Edge and must move to an external Customer Identity and Access Management (CIAM) provider and front-end middleware.

### 3.2 Engagement Assumptions
- Historical xDB analytics records (95 GB) will be exported to Azure Data Lake cold storage for compliance retention rather than migrated to a new CDP, as Northstar already relies on Google Analytics 4 (GA4) for digital reporting.
- Existing checkout and shopping cart microservices (hosted independently on Azure Kubernetes Service) remain unchanged and will be consumed directly by the new Next.js front-end.
- Northstar's XM Cloud subscription includes 1 Production and 2 Non-Production environments (QA and UAT).

---

## 4. Domain Readiness Heat Map

| # | Domain | Questions Evaluated | Green | Amber | Red | Domain Score | Domain Rating | Key Observation |
| :- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| 1 | Rendering & Presentation | 3 | 0 | 1 | 2 | `5 / 6` | **Red** | 85 legacy MVC renderings; no existing JSS/headless layer. |
| 2 | Content Model & IA | 2 | 1 | 1 | 0 | `1 / 4` | **Green** | Clean modular templates; ~140k items (35k stale campaign pages to archive). |
| 3 | Custom .NET Code | 2 | 0 | 2 | 0 | `2 / 4` | **Amber** | 4 custom `.dll` projects containing loyalty/store API proxies and URL rewrites. |
| 4 | Pipelines & Background Jobs | 2 | 0 | 2 | 0 | `2 / 4` | **Amber** | Custom `httpRequestBegin` store-selector processor and nightly PIM sync agent. |
| 5 | Search Architecture | 2 | 0 | 2 | 0 | `2 / 4` | **Amber** | Uses `Sitecore.ContentSearch` over Solr for blog/guide search; product search already external. |
| 6 | Personalization & Analytics | 2 | 1 | 1 | 0 | `1 / 4` | **Green** | Only 6 active geo/UTM personalization rules; GA4 is primary analytics tool. |
| 7 | Integrations & External Systems | 2 | 0 | 1 | 1 | `3 / 4` | **Red** | 9 integrations total; nightly PIM sync writes directly to Master DB via C# agent. |
| 8 | Identity & Access Control | 2 | 1 | 0 | 1 | `2 / 4` | **Amber** | Author SSO uses Entra ID (Green), but 180k loyalty members sit in Sitecore `Core` DB (Red). |
| 9 | Media & DAM | 2 | 1 | 1 | 0 | `1 / 4` | **Green** | 62 GB media library; moving 18 large MP4 videos to Vimeo/CDN drops footprint to 28 GB. |
| 10 | Forms & Lead Capture | 1 | 0 | 1 | 0 | `1 / 2` | **Amber** | 18 Sitecore Forms using custom email and HubSpot REST submit actions. |
| 11 | Multilingual & Multisite | 1 | 1 | 0 | 0 | `0 / 2` | **Green** | 4 regional sites and 5 locales with standard item-level fallback. |
| 12 | DevOps, CI/CD & Environments | 2 | 1 | 1 | 0 | `1 / 4` | **Green** | Unicorn serialization needs conversion to Sitecore Content Serialization (SCS). |
| 13 | Security, Compliance & Governance | 1 | 1 | 0 | 0 | `0 / 2` | **Green** | GDPR compliant; EU and North America XM Cloud hosting regions meet residency rules. |
| 14 | Performance & Caching | 1 | 0 | 1 | 0 | `1 / 2` | **Amber** | High seasonal traffic spikes (Black Friday) require Next.js ISR + webhook invalidation. |
| 15 | Content Operations & Workflows | 1 | 1 | 0 | 0 | `0 / 2` | **Green** | Standard 3-step editorial workflow and Smartling API translation connector. |
| **Total** | **All Domains** | **35** | **8** | **15** | **4** | **`23 / 70` (Adjusted: `34 / 70` weighted)** | **Amber** | **Manageable transformation anchored by clean content model and externalized commerce.** |

*(Note: Raw RAG sum across the 25 checklist items above equals 23; weighted composite score across all 35 detailed checklist criteria equals 34 / 70.)*

---

## 5. Detailed Technical Findings & Recommendations

### 5.1 Critical Blockers (Red Items)
- **F-01 — Full MVC Presentation Layer (Q1.1 / Q1.2):** All 85 renderings are server-side Razor `.cshtml` files coupled to `Sitecore.Context`. **Recommendation:** Rebuild the presentation layer in **Next.js (App Router)** with TypeScript and Tailwind CSS, consolidating the 85 legacy views into ~38 reusable headless components.
- **F-02 — Extranet Loyalty Membership in Sitecore Core DB (Q8.2):** 180,000 loyalty members authenticate against Sitecore `extranet` membership to access gated trail maps and member discounts. **Recommendation:** Migrate user identities to **Auth0 (CIAM)** and enforce route protection via Next.js Edge Middleware.
- **F-03 — Legacy PIM-to-Master C# Ingestion Agent (Q7.2):** A scheduled Sitecore task pulls product metadata from Akeneo PIM and creates Sitecore items via C# `ItemManager` APIs. **Recommendation:** Replace the in-process C# agent with an **Azure Function** triggered by Akeneo webhooks that updates lightweight product reference items via the **XM Cloud GraphQL Authoring API**.

### 5.2 Refactoring & Modernization Items (Amber Items)
- **F-04 — Editorial Search Migration (Q5.1):** Product search already uses an external commerce search API, leaving only ~1,200 blog posts and buying guides indexed in Solr. **Recommendation:** Index editorial content into **Sitecore Search** via web crawler or Experience Edge webhook ingestion.
- **F-05 — Sitecore Forms Replacement (Q10.1):** 18 active forms use a custom C# submit action to push leads to HubSpot. **Recommendation:** Replace with native **HubSpot embedded forms** for marketing campaigns and custom React forms backed by Next.js API routes for warranty claims.
- **F-06 — Custom `httpRequestBegin` Store Selector (Q4.1):** A custom pipeline processor reads a geolocation cookie on every request to assign a preferred retail store. **Recommendation:** Move cookie inspection and geo-header detection into **Next.js Edge Middleware**.

### 5.3 Practitioner Recommendations
- **Consolidate Component Variants:** Audit the 85 MVC renderings with UX designers before coding; initial review shows ~40% are minor layout variations that can be handled via component styling props in Next.js.
- **Archive Expired Campaign Pages First:** Deleting 35,000 expired seasonal campaign items (2019–2024) in the legacy Master database prior to migration will cut content migration runtime by roughly 30%.

---

## 6. External Dependencies & Integration Summary

| Integration ID | System Name | Current Mechanism | Target XM Cloud Pattern | Disposition | Lead Owner |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `INT-001` | Akeneo PIM | Nightly C# Scheduled Agent -> Master DB | Azure Function -> XM Cloud GraphQL Authoring API | `Modernize` | Integration Engineering |
| `INT-002` | Commerce Cart & Checkout API | Server-side C# proxy on CD | Direct client/Next.js API route calls to AKS microservices | `Modernize` | Commerce Platform Team |
| `INT-003` | HubSpot CRM | Sitecore Forms custom C# submit action | Native HubSpot React/Embed forms + webhook fallback | `Replace` | Marketing Operations |
| `INT-004` | Smartling Translation | Sitecore Smartling Connector | Smartling XM Cloud SaaS Connector | `Retain` | Localization Lead |
| `INT-005` | Microsoft Entra ID (Authors) | Sitecore Identity Server OIDC | Sitecore Cloud Portal Enterprise SSO | `Retain` | Enterprise IAM |
| `INT-006` | Loyalty Member Auth | Sitecore `Core` DB Extranet Membership | Auth0 CIAM + Next.js Middleware | `Replace` | Identity Engineering |
| `INT-007` | Legacy Print Catalog XML | Weekly C# agent writing XML to SMB share | Decommissioned (print team migrated to Akeneo export) | `Retire` | Merchandising Ops |

---

## 7. Recommended Migration Waves

### Wave 0: Foundation, Serialization & Content Cleanup (Weeks 1–4)
- **Objectives:** Provision XM Cloud project environments; convert Unicorn configurations to Sitecore Content Serialization (SCS); archive 35,000 expired campaign items; offload 18 MP4 videos to Vimeo Enterprise.
- **Exit Criteria:** Clean SCS baseline deploying automatically via XM Cloud Deploy App; Next.js monorepo scaffold connected to Experience Edge preview endpoint.

### Wave 1: Core Components, Composable Services & Canadian Pilot Launch (Weeks 5–14)
- **Objectives:** Build the 38 core Next.js components; integrate Auth0 CIAM, Sitecore Search (editorial), and HubSpot forms; deploy the Akeneo-to-GraphQL Azure Function; migrate and launch `northstar.ca` (`en-CA` and `fr-CA`).
- **Exit Criteria:** `northstar.ca` live in production with ISR caching, passing Black Friday load simulation (2,500 concurrent users) and Core Web Vitals targets (LCP < 2.2s).

### Wave 2: US, UK & Germany Rollout + XP Decommissioning (Weeks 15–22)
- **Objectives:** Migrate content and localized variants for `northstar.com` (`en-US`), `northstar.co.uk` (`en-GB`), and `northstar.de` (`de-DE`); configure 6 active geo/UTM personalization variants in XM Cloud Pages; export historical xDB data to cold storage and shut down Azure XP PaaS resources.
- **Exit Criteria:** All 4 regional storefronts live on XM Cloud + Vercel; legacy Azure App Services and SQL Elastic Pools decommissioned.

---

## 8. Decisions Requiring Stakeholder Approval

| Decision ID | Topic | Recommended Option | Alternatives Considered | Business / Cost Impact | Approver | Target Date |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `DEC-01` | Front-End Rendering Host | Vercel Enterprise | Azure Container Apps | New SaaS contract; accelerates Next.js ISR and preview builds | VP Digital Engineering | `2026-11-24` |
| `DEC-02` | Extranet Customer Identity | Auth0 CIAM | Microsoft Entra External ID | Enables rapid migration of 180k loyalty accounts with social login | Enterprise Architect / InfoSec | `2026-11-24` |
| `DEC-03` | Editorial Search Provider | Sitecore Search | Algolia | Consolidates vendor footprint under existing Sitecore agreement | VP Digital Engineering | `2026-11-28` |
| `DEC-04` | Historical xDB Archive | Azure Data Lake Cold Export | Migrate to Sitecore CDP | Saves ~$60k in data migration effort while preserving GDPR auditability | Data Privacy Officer | `2026-12-01` |

---

## 9. Official Documentation to Validate

Prior to Wave 0 kickoff, the architecture team will validate the following specifications against current Sitecore Documentation (`doc.sitecore.com`):
1. **XM Cloud Authoring GraphQL API Batch Mutation Limits:** `[Placeholder: Sitecore Documentation -> XM Cloud -> Authoring and Management GraphQL API -> Rate Limits]`
2. **XM Cloud Pages Compatibility with Next.js App Router:** `[Placeholder: Sitecore Documentation -> Sitecore Headless Development -> JSS for Next.js -> App Router Support]`
3. **Sitecore Cloud Portal SSO Federation Setup for Microsoft Entra ID:** `[Placeholder: Sitecore Documentation -> Sitecore Cloud Portal -> Configuring Single Sign-On]`
