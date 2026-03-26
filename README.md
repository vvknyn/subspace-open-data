# Subspace Open Data — Silent Enterprise 500

**500 companies with enterprise-grade infrastructure, detected from public network metadata.**

Every company in this dataset was identified by Subspace's 33-check deterministic engine scanning publicly observable digital exhaust — DNS records, HTTP headers, Content Security Policy, SEC filings, and 38 other sources. No scraping. No AI hallucination. Just network plumbing.

## What's in this repo

| File | Format | Description |
|------|--------|-------------|
| `silent-enterprise-500-2026-03-26.csv` | CSV | 500 companies with 8 cost center scores, tiers, budget status, and top signals |
| `deep-traces-100-2026-03-26.json` | JSON | 100 companies with full signal-level breakdowns (10-37 signals each) |

## The "Silent Enterprise" Filter

These aren't random companies. Every company in this dataset passed two gates:

1. **Health score ≥ 60** — scored by 33 deterministic health checks
2. **2+ enterprise-tier cost centers** — at least two budget areas scored ≥ 75/100

This means every company on this list is actively spending on enterprise infrastructure — even if they haven't announced a funding round, IPO, or expansion.

## 8 Corporate Cost Centers

Subspace maps company health to corporate P&L line items:

| Cost Center | What it measures | How we detect it |
|-------------|-----------------|------------------|
| **IT & Infrastructure** | Cloud spend, CDN, WAF, email security | HTTP headers, DNS MX, TTFB, SSL |
| **Engineering & Product** | R&D velocity, shipping cadence, SSO | CSP headers, changelog, subdomains, .well-known |
| **Sales & Marketing** | Ad spend, international expansion | Tracking pixels, GTM container, hreflang, robots.txt |
| **Legal & Compliance** | SOC 2, ISO 27001, HIPAA | Trust center pages, H-1B filings, WARN Act |
| **HR & Hiring** | Job quality, ghost detection, velocity | ATS boards, Schema.org, posting patterns |
| **Finance & RevOps** | Billing tier, payment processors | CSP/HTML for Stripe vs Zuora vs Adyen |
| **Telecom & Remote Work** | Enterprise UC contracts | DNS SRV records for Teams, Zoom, RingCentral |
| **Corporate Development** | M&A activity | MX + NS parent company matching |

## CSV Columns

```
domain                      — Company website domain
company_name                — Legal/common name
health_score                — 0-100 composite score (higher = healthier)
verdict                     — actively_hiring / at_risk / ghost
sector                      — Industry classification
total_signals               — Number of signals detected

it_infrastructure_score     — 0-100 IT budget score
it_infrastructure_tier      — Enterprise / Growth / Startup / Inactive
it_infrastructure_budget    — Active / Moderate / Low / Frozen
it_infrastructure_signals   — Detected signals (e.g., "Verified SaaS Tooling | DMARC Fully Enforced")

engineering_product_score   — 0-100 R&D velocity score
engineering_product_tier    — Enterprise / Growth / Startup / Inactive
engineering_signals         — Detected signals

sales_marketing_score       — 0-100 GTM investment score
sales_marketing_tier        — Enterprise / Growth / Startup / Inactive
sales_marketing_signals     — Detected signals

legal_compliance_score      — 0-100 compliance score
legal_compliance_tier       — Enterprise / Growth / Startup / Inactive
legal_signals               — Detected signals

hr_hiring_score             — 0-100 hiring health (null if no ATS data)
hr_hiring_signals           — Detected signals

finance_revops_score        — 0-100 billing infrastructure
telecom_remote_score        — 0-100 UC infrastructure
corporate_dev_score         — 0-100 M&A signals

last_checked                — Date of last scan
trace_url                   — Live trace URL (try it yourself)
```

## Deep Trace Format (JSON)

Each company in `deep-traces-100.json` includes the full signal breakdown:

```json
{
  "domain": "docker.com",
  "company_name": "Docker",
  "health_score": 93,
  "verdict": "at_risk",
  "total_signals": 26,
  "cost_centers": {
    "it_infrastructure": {
      "label": "IT & Infrastructure",
      "score": 100,
      "tier": "Enterprise",
      "budget_status": "Active",
      "signals_detected": [
        {
          "signal": "Verified SaaS Tooling",
          "sentiment": "positive",
          "weight": 90,
          "description": "Domain has verified ownership for: Atlassian, Google Workspace, Stripe..."
        },
        {
          "signal": "DMARC Quarantine",
          "sentiment": "positive",
          "weight": 60,
          "description": "DMARC policy set to quarantine — active email security management"
        }
      ]
    }
  }
}
```

## How to use this data

**For SDRs and outbound teams:**
- Filter the CSV by `it_infrastructure_tier = Enterprise` to find companies with active cloud budgets
- Sort by `sales_marketing_score` descending to find companies actively spending on growth
- Use `hr_hiring_signals` to verify if they're actually hiring (not ghost posting)

**For data analysts:**
- Cross-reference against your CRM to score existing accounts
- Compare `engineering_product_score` across competitors in a sector
- Track companies with `legal_compliance_tier = Enterprise` for SOC 2/compliance sales

**For Clay users:**
- Import domains into a Clay table
- Add Subspace HTTP enrichment: `GET https://www.thesubspace.io/api/v1/enrich?domain={{domain}}`
- Get live, real-time scores + boolean filters (`is_spending_on_ads`, `has_enterprise_sso`, etc.)
- [2-minute Clay setup guide](https://www.thesubspace.io/integrations/clay)

## Try it live

Type any domain and watch 33 checks run in real-time:

**[thesubspace.io/trace](https://www.thesubspace.io/trace)**

## Get the full dataset via API

This data is a static 500-domain snapshot generated on 2026-03-26.

For live telemetry on 14,000+ domains (and growing), use the Subspace API:

```bash
curl "https://www.thesubspace.io/api/v1/enrich?domain=stripe.com"
```

Free during beta. 100 credits/month. No API key required.

**[API Documentation](https://www.thesubspace.io/api/v1/docs)** · **[Pricing](https://www.thesubspace.io/pricing)** · **[Clay Integration](https://www.thesubspace.io/integrations/clay)**

## Methodology

Every signal is derived from publicly observable network metadata. No surveys, no LinkedIn scraping, no AI inference. The engine reads what companies are forced to emit simply by operating on the internet.

Full scoring methodology: **[thesubspace.io/research](https://www.thesubspace.io/research)**

---

Built by [Subspace](https://www.thesubspace.io) · Data updated daily · [Contact](mailto:hello@thesubspace.io)
