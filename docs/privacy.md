---
layout: default
title: Privacy Policy — AI PR Review + Rule Learning
---

# Privacy Policy

**Product:** AI PR Review + Rule Learning (Azure DevOps Extension)  
**Publisher:** AISRE  
**Effective date:** 2026-04-18  
**Last updated:** 2026-04-18

---

## 1. Who We Are

This Privacy Policy applies to the **AI PR Review + Rule Learning** extension for Azure DevOps ("Extension") and related services published by **AISRE** ("we", "us", "our").

**Contact:**  
E-mail: [privacy@aisre.dev](mailto:privacy@aisre.dev)  
Support: [github.com/aisre-labs/pr-review/issues](https://github.com/aisre-labs/pr-review/issues)

---

## 2. Architecture: Your Data Never Leaves Your Tenant

The Extension is built on a **zero-data-egress principle**. All source code, pull request diffs, review comments, and developer data are processed exclusively within **your own Azure environment** and are never transmitted to AISRE or any third-party service controlled by AISRE.

Concretely:

| Data | Where it goes | Does AISRE see it? |
|---|---|---|
| Source code, PR diffs | Your Azure OpenAI deployment only (BYOK) | **No** |
| PR review comments | Your Azure DevOps only | **No** |
| Learned rules & configuration | Your ADO Extension Data Service | **No** |
| Azure OpenAI API key | Your ADO Extension Data Service | **No** |
| Pipeline execution logs | Your Azure Pipelines agents | **No** |

This architecture is intentional and is the core security value proposition of the Extension.

---

## 3. Data We Collect

### 3.1 Free Tier (VS Marketplace)

When using the Extension in its free tier, **AISRE collects no personal data whatsoever.** The Extension operates entirely within your Azure DevOps organization.

### 3.2 Paid Tier (Azure Marketplace SaaS — upcoming)

When you purchase a subscription through Microsoft Azure Marketplace, AISRE receives the following data from Microsoft's SaaS Fulfillment platform for the sole purpose of license validation:

| Data | Purpose | Retention |
|---|---|---|
| Azure Active Directory Tenant ID | Binding license to your organization | Subscription duration + 90 days |
| Azure Marketplace Subscription ID | Subscription lifecycle management | Subscription duration + 90 days |
| Subscription plan and status | Feature gating (free trial vs. pro) | Subscription duration |

**We do not receive:** payment card data, billing address, individual user identities, or any code or PR content. Microsoft processes all payment information independently under [Microsoft's Privacy Statement](https://privacy.microsoft.com/en-us/privacystatement).

### 3.3 Support and Bug Reports

If you contact us through GitHub Issues or email, we may receive your GitHub username and the content of your message. This is used solely to respond to your request and is retained for up to 2 years.

---

## 4. Legal Basis (GDPR)

For users in the European Union and EEA, our legal bases for processing personal data are:

| Processing activity | Legal basis |
|---|---|
| License validation (paid tier) | Art. 6(1)(b) — contractual necessity |
| Responding to support requests | Art. 6(1)(f) — legitimate interest |
| Legal record retention | Art. 6(1)(c) — legal obligation |

---

## 5. Data Sharing

We do not sell, rent, or share personal data. The only third party that receives any data is:

- **Microsoft Corporation** — as the Azure Marketplace billing intermediary. Microsoft acts as an independent data controller for payment and subscription management.

We do not use advertising networks, analytics platforms, or any telemetry service that would receive information about Extension users or their code.

---

## 6. Data Retention

| Data | Retention |
|---|---|
| Subscription metadata (paid tier) | Subscription duration + 90 days after cancellation |
| Support communications | 2 years from last contact |
| Extension Data Service content | Fully controlled by you; deleted when you uninstall the Extension or remove your ADO organization |

---

## 7. Your Rights (GDPR)

If you are located in the EU, you have the right to:

- **Access** (Art. 15) — request a copy of personal data we hold about you  
- **Rectification** (Art. 16) — request correction of inaccurate data  
- **Erasure** (Art. 17) — request deletion ("right to be forgotten")  
- **Restriction** (Art. 18) — request that we restrict processing  
- **Portability** (Art. 20) — receive your data in machine-readable format  
- **Object** (Art. 21) — object to processing based on legitimate interest  

To exercise these rights, contact [privacy@aisre.dev](mailto:privacy@aisre.dev). We will respond within **30 calendar days**.

You also have the right to lodge a complaint with the Polish data protection authority:  
**UODO** (Urząd Ochrony Danych Osobowych) — [uodo.gov.pl](https://uodo.gov.pl)

---

## 8. Security

We apply appropriate technical and organizational measures to protect the limited personal data we process:

- Subscription metadata is stored in Azure Table Storage with encryption at rest and in transit (TLS 1.2+).
- Access to production systems is restricted to authorized personnel via multi-factor authentication.
- We do not store API keys, source code, diffs, or PR comment content.

---

## 9. Children

The Extension is a professional developer tool not directed at children under 16. We do not knowingly collect data from minors.

---

## 10. Changes to This Policy

We may update this policy to reflect product changes or legal requirements. We will update the "Last updated" date at the top of this page for all changes. For material changes that affect the paid tier, we will provide notice via the Marketplace listing changelog.

---

## 11. Intellectual Property

The Extension — including all source code, compiled artifacts, AI prompts, rule extraction algorithms, confidence scoring logic, and documentation — is the exclusive proprietary property of **AISRE** and is protected under copyright law and international intellectual property treaties.

**The AI prompts embedded in the Extension constitute proprietary trade secrets of AISRE.** They represent significant research and development investment and are the core value differentiator of the product.

The following acts are strictly prohibited without prior written authorization from AISRE:

- Copying, modifying, or creating derivative works of the Extension
- Reverse engineering, decompiling, or disassembling the Extension
- Extracting, reproducing, or redistributing the AI prompts or scoring algorithms
- Sublicensing, selling, or otherwise commercializing the Extension or its components
- Using the Extension to build a competing product

Use of the Extension is governed by the **AISRE Proprietary Software License** distributed with the Extension. Violation of these terms may result in civil and criminal liability under applicable Polish and international copyright law.

For licensing inquiries: [license@aisre.dev](mailto:license@aisre.dev)

---

*© 2026 AISRE. All rights reserved.*
