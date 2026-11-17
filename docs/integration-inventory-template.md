# Sitecore XM Cloud Integration Inventory Template

Use this catalog to document every inbound, outbound, and bidirectional integration connected to your existing **Sitecore Experience Manager (XM)** or **Sitecore Experience Platform (XP)** solution. Every integration must be assigned a clear **XM Cloud Disposition** before migration estimation is finalized.

---

## Field Definitions & Allowed Values

- **Direction:** `Inbound` (external system pushes to Sitecore), `Outbound` (Sitecore pushes/queries external system), or `Bidirectional`.
- **Protocol / Transport:** `REST/JSON`, `GraphQL`, `SOAP/XML`, `SFTP/CSV`, `Direct SQL`, `Webhook`, `SDK/Client-Side JS`.
- **Authentication:** `OAuth 2.0 / OIDC`, `API Key`, `mTLS`, `Basic Auth`, `HMAC Signature`, `Managed Identity`, `Anonymous`.
- **Data Classification:** `Public`, `Internal`, `Confidential`, `PII (Personally Identifiable Information)`, `PCI (Payment Card Data)`.
- **XM Cloud Impact:**
  - **Low:** Decoupled client-side script or standard REST API easily called from Next.js.
  - **Medium:** Requires refactoring a C# pipeline/scheduled task into an XM Cloud Webhook + serverless function.
  - **High:** Relies on prohibited patterns (direct SQL writes, synchronous CD HTTP modules, or xConnect custom facets).
- **Disposition:**
  - **Retain:** Works in XM Cloud with configuration updates only (e.g., updating endpoint URLs).
  - **Modernize:** Re-architect using webhooks, serverless middleware, or GraphQL Authoring API.
  - **Replace:** Swap legacy integration for a SaaS-native connector or third-party embed.
  - **Retire:** Decommission unused or obsolete integration during migration.

---

## Integration Catalog

| ID | Integration Name | Source System | Target System | Business Purpose | Direction | Protocol | Authentication | Data Classification | Frequency & Volume | System Owner | Availability SLA | Failure Handling & Retry | XM Cloud Impact | Disposition | Target Architecture Notes |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `INT-001` | `[e.g., Product Catalog Sync]` | `[SAP ERP]` | `[Sitecore Master DB]` | `[Sync SKU titles, specs, and categories]` | `Inbound` | `[SOAP / C# Agent]` | `[Basic Auth]` | `[Public / Internal]` | `[Nightly; ~15,000 SKUs]` | `[ERP Team]` | `[99.5% (Batch)]` | `[Email alert; manual re-run]` | `High` | `Modernize` | `[Move to Azure Function calling XM Cloud GraphQL Authoring API, or serve dynamic specs via front-end API]` |
| `INT-002` | `[e.g., Lead Form Submission]` | `[Sitecore Forms]` | `[Salesforce CRM]` | `[Route contact & demo requests to sales]` | `Outbound` | `[REST/JSON]` | `[OAuth 2.0]` | `[PII]` | `[Real-time; ~400/day]` | `[Marketing Ops]` | `[99.9%]` | `[None (lost on timeout)]` | `High` | `Replace` | `[Replace with Salesforce Web-to-Lead / HubSpot embed or Next.js API route with dead-letter queue]` |
| `INT-003` | `[e.g., Store Locator Data]` | `[Retail Store API]` | `[Sitecore CD Controller]` | `[Display store hours and stock availability]` | `Outbound` | `[REST/JSON]` | `[API Key]` | `[Public]` | `[Real-time; ~50 req/sec peak]` | `[Digital Retail Team]` | `[99.95%]` | `[Fallback to cached JSON]` | `Medium` | `Modernize` | `[Call directly from Next.js API route / client with SWR caching; bypass CMS entirely]` |
| `INT-004` | `[e.g., Author SSO]` | `[Microsoft Entra ID]` | `[Sitecore Identity Server]` | `[Single sign-on for content editors]` | `Inbound` | `[OIDC]` | `[OAuth 2.0 / OIDC]` | `[Internal / PII]` | `[On login; ~80 authors]` | `[Enterprise IAM]` | `[99.9%]` | `[Fallback admin login]` | `Low` | `Retain` | `[Configure Enterprise SSO federation in Sitecore Cloud Portal]` |
| `INT-005` | `[e.g., Legacy Print Export]` | `[Sitecore Master DB]` | `[On-Prem File Share]` | `[Export weekly XML catalog for print team]` | `Outbound` | `[C# Scheduled Task / SMB]` | `[Windows Service Account]` | `[Internal]` | `[Weekly; 1 batch]` | `[Print Ops]` | `[99.0%]` | `[Manual log check]` | `High` | `Retire` | `[Print team confirmed legacy feed is deprecated; retire prior to cutover]` |

---

## Summary Counts by Disposition

| Disposition | Count | Percentage | Primary Remediation Owner |
| :--- | :---: | :---: | :--- |
| **Retain** | `[X]` | `[X%]` | `[Platform / DevOps Team]` |
| **Modernize** | `[X]` | `[X%]` | `[Backend / Integration Engineering]` |
| **Replace** | `[X]` | `[X%]` | `[Front-End / MarTech Engineering]` |
| **Retire** | `[X]` | `[X%]` | `[Product Owner / Business Analyst]` |
| **Total** | **`[X]`** | **`100%`** | — |

---

## Official Documentation to Validate

When designing target-state integration patterns, confirm current limits and specifications in the official Sitecore Documentation (`doc.sitecore.com`):
1. **XM Cloud Authoring & Management GraphQL API (Mutations & Bulk Operations):** `[Placeholder: Sitecore Documentation -> XM Cloud -> Developer Tools -> Authoring and Management API]`
2. **XM Cloud Webhooks (Event Types, Payload Signing, and Timeout Limits):** `[Placeholder: Sitecore Documentation -> XM Cloud -> Developer Tools -> Webhooks]`
3. **Sitecore Connect Integration Patterns:** `[Placeholder: Sitecore Documentation -> Sitecore Connect -> Connectors and Recipes]`
