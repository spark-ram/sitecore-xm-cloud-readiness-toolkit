# Sitecore XM Cloud Readiness Toolkit

A vendor-neutral, reusable discovery and architecture assessment toolkit for engineering teams, solution architects, and delivery leads evaluating a migration from **Sitecore Experience Manager (XM)** or **Sitecore Experience Platform (XP)** to **Sitecore XM Cloud**.

---

## Purpose

Moving from self-hosted or Platform-as-a-Service (PaaS) Sitecore XM/XP topologies to Sitecore XM Cloud is **not an in-place version upgrade**. XM Cloud is a Software-as-a-Service (SaaS), headless-first Content Management System (CMS) that separates content management from front-end rendering, eliminates traditional Content Delivery (CD) servers, and replaces server-side customization patterns with composable, API-driven architectures.

This toolkit provides structured templates, evaluation rubrics, and decision frameworks to help teams:
- Inventory legacy dependencies, custom code, and third-party integrations.
- Quantify migration complexity and architectural debt before committing to timelines.
- Identify capabilities that require replacement or re-platforming (such as xDB/xConnect analytics, in-process pipelines, or legacy MVC renderings).
- Produce standardized discovery deliverables for technical and executive stakeholders.

## Intended Audience

- **Sitecore Solution & Enterprise Architects:** Designing target-state composable topologies and evaluating build-versus-buy trade-offs.
- **Technical Leads & Senior Developers:** Auditing existing .NET solutions, custom pipelines, serialization setups, and front-end rendering layers.
- **Delivery Managers & Program Leads:** Estimating migration waves, scoping workstreams, and managing technical risk.

## What This Toolkit Does and Does Not Do

| What This Toolkit Does | What This Toolkit Does Not Do |
| :--- | :--- |
| Provides a 35-point architectural readiness checklist across 15 technical domains. | Does not run automated static code analysis against your Visual Studio solutions. |
| Offers structured templates for assessments, integration inventories, and risk registers. | Does not replace official Sitecore Migration Audit tools or Sitecore product documentation. |
| Establishes a repeatable Red/Amber/Green (RAG) scoring model for discovery workshops. | Does not prescribe a single front-end framework or third-party vendor stack. |
| Distinguishes established XM Cloud architectural constraints from practitioner recommendations. | Does not guarantee licensing terms, SaaS entitlements, or regional hosting availability. |

## Repository Structure

```text
sitecore-xm-cloud-readiness-toolkit/
├── CHANGELOG.md                            # Release history and version notes
├── CONTRIBUTING.md                         # Guidelines for community pull requests and feedback
├── LICENSE                                 # MIT open-source license
├── README.md                               # Toolkit overview, scoring guide, and usage instructions
├── docs/
│   ├── architecture-decision-matrix.md     # Retain / Modernize / Replace decision framework
│   ├── integration-inventory-template.md   # Standardized system-to-system integration catalog
│   ├── migration-assessment-template.md    # Executive and technical discovery report template
│   ├── risk-register-template.md           # Migration risk tracking and mitigation register
│   └── xm-cloud-readiness-checklist.md     # 35-item domain-by-domain readiness questionnaire
└── examples/
    └── sample-retail-site-assessment.md    # Worked example for a fictional multilingual retailer
```

## Quick-Start Steps

1. **Clone the repository** into your discovery workspace or internal documentation repository:
   ```bash
   git clone https://github.com/your-org/sitecore-xm-cloud-readiness-toolkit.git
   cd sitecore-xm-cloud-readiness-toolkit
   ```
2. **Create a working copy** of the templates inside a client- or project-specific folder (e.g., `assessments/project-alpha/`).
3. **Conduct discovery workshops** using [`docs/xm-cloud-readiness-checklist.md`](docs/xm-cloud-readiness-checklist.md) alongside your existing codebase, Azure/on-premises infrastructure diagrams, and Sitecore configuration showconfig exports.
4. **Catalog all external dependencies** in [`docs/integration-inventory-template.md`](docs/integration-inventory-template.md).
5. **Resolve architectural trade-offs** using [`docs/architecture-decision-matrix.md`](docs/architecture-decision-matrix.md).
6. **Compile findings, waves, and risks** into [`docs/migration-assessment-template.md`](docs/migration-assessment-template.md) and [`docs/risk-register-template.md`](docs/risk-register-template.md). Review [`examples/sample-retail-site-assessment.md`](examples/sample-retail-site-assessment.md) as a reference baseline.

## How to Score an Assessment

Each item in [`docs/xm-cloud-readiness-checklist.md`](docs/xm-cloud-readiness-checklist.md) is evaluated using a **Red / Amber / Green (RAG)** rubric:

- **Green (0 points — Ready / Low Friction):** The capability aligns with XM Cloud SaaS constraints or requires minimal refactoring (e.g., existing Sitecore JavaScript Rendering SDKs [JSS] using GraphQL/Layout Service, standard item serialization).
- **Amber (1 point — Refactor / Re-architect):** The capability is unsupported as-is in XM Cloud, but a clear, well-established replacement or refactoring path exists (e.g., migrating Solr search indexes to a composable search provider, converting MVC renderings to headless components).
- **Red (2 points — High Complexity / Blocker):** Heavy reliance on deprecated XP-only features, invasive core pipeline overrides, or tightly coupled legacy monoliths requiring significant re-engineering before or during migration (e.g., deep xDB/xConnect custom facets, custom Content Delivery HTTP modules, heavily customized Sitecore Forms with server-side submit actions).

### Interpreting Aggregate Scores

Calculate the **Readiness Index** across all 35 checklist items (maximum possible friction score = 70):

| Total Friction Score | Overall Rating | Typical Migration Profile |
| :--- | :--- | :--- |
| **0 – 18** | **Green (Streamlined)** | Modern XM topology (e.g., Sitecore 10.x) already using headless/JSS or clean SXA structures with minimal custom .NET pipeline overrides. |
| **19 – 42** | **Amber (Moderate Transformation)** | Hybrid XM/XP solution requiring front-end rebuild to headless, extraction of custom server-side business logic to external APIs, and search/analytics replacement. |
| **43 – 70** | **Red (Major Re-platforming)** | Legacy XP solution (e.g., Sitecore 8.x/9.x) with heavy MVC/WebForms debt, deep xDB personalization dependencies, and complex ERP/CRM synchronous coupling. |

*Note: A single critical Red blocker (such as an unreplaceable regulatory compliance workflow or an unsupported third-party module) can dictate wave sequencing regardless of the overall numerical score.*

## Disclaimer & Fact Validation Policy

- **Fact vs. Practice:** Throughout this repository, **Verified Product Architecture Facts** (core SaaS boundaries of Sitecore XM Cloud) are explicitly separated from **Practitioner Recommendations** (field-tested patterns that may vary by team maturity, budget, and timeline).
- **Rapid SaaS Evolution:** Because Sitecore XM Cloud is a continuously updated SaaS platform, feature availability and limits evolve frequently. Every document includes an **Official Documentation to Validate** section listing specific topics to verify against current Sitecore documentation prior to finalizing architecture decisions.
- **No Warranty:** This toolkit is provided "as is" under the MIT License without warranty of any kind. Always validate licensing, entitlements, and architectural constraints with Sitecore or an accredited Sitecore solution partner.

## Contributing

Contributions, issue reports, and refinements from the Sitecore engineering community are welcome. Please read [`CONTRIBUTING.md`](CONTRIBUTING.md) before submitting a pull request.

## Roadmap

- **v1.0.0:** Initial release with 35-point readiness checklist, architecture decision matrix, discovery templates, risk register, and fictional retail worked example.
- **v1.1.0 (Planned):** Add a machine-readable CSV/JSON export schema for the readiness checklist and integration inventory.
- **v1.2.0 (Planned):** Add discovery workshop facilitation guide and stakeholder interview question bank.
- **v1.3.0 (Planned):** Add reference architecture diagrams (Mermaid.js) illustrating common composable patterns (webhooks, edge caching, externalized search).
