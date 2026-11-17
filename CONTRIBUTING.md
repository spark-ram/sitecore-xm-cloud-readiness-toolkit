# Contributing to Sitecore XM Cloud Readiness Toolkit

Thank you for helping keep this toolkit practical, accurate, and vendor-neutral for the Sitecore community.

## Contribution Principles

1. **Vendor Neutrality:** Avoid promoting specific commercial agencies, proprietary accelerators, or single-vendor lock-in where multiple valid composable approaches exist.
2. **Fact vs. Opinion Separation:** Clearly distinguish established Sitecore XM Cloud platform constraints from practitioner recommendations.
3. **No Invented URLs or Unverified Claims:** When referencing official Sitecore capabilities or limits, use descriptive placeholders in the `Official Documentation to Validate` section rather than brittle or unverified links.
4. **Confidentiality & Sanitization:** Never include real client names, proprietary project details, internal architecture diagrams, license keys, or non-public/NDA-protected product information.

## How to Submit Changes

1. **Fork the repository** and create a descriptive feature branch from `main`:
   ```bash
   git checkout -b docs/add-serialization-checks
   ```
2. **Make your edits** following the existing Markdown formatting conventions:
   - Spell out acronyms on first use in every standalone document (e.g., `Content Delivery (CD)`).
   - Keep tables readable and ensure every checklist item includes Evidence, RAG criteria, Migration Implication, and Recommended Next Action.
3. **Verify hygiene:** Check that no proprietary names, credentials, or broken relative links exist in your changes.
4. **Open a Pull Request (PR):** Provide a clear summary of what was added or updated and why it improves discovery or migration planning.

## Reporting Issues

If a Sitecore XM Cloud SaaS release changes an architectural constraint documented in this toolkit, please open an issue specifying:
- The file and section affected.
- What changed in the platform.
- Suggested wording updates.
