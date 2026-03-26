# Subspace Data Dictionary

**Version:** 1.0 · **Last updated:** March 2026 · **Coverage:** 14,000+ companies (target: 100K+ by Q2 2026)

Every data point is derived from publicly observable network metadata. 33 deterministic health checks across 8 corporate cost centers, plus 42 enrichment sources.

---

## Company-Level Fields

| Field | Type | Description | Example |
|-------|------|-------------|---------|
| `domain` | string | Company website domain | `stripe.com` |
| `company_name` | string | Legal or common name | `Stripe` |
| `health_score` | integer (0-100) | Composite operational health score | `93` |
| `verdict` | enum | `healthy` / `at_risk` / `critical` | `healthy` |
| `confidence` | enum | `high` / `moderate` / `low` | `high` |
| `hiring_status` | enum | `actively_hiring` / `slow_hiring` / `not_hiring` | `actively_hiring` |
| `total_jobs` | integer | Active job listings tracked | `285` |
| `ats_provider` | string | Applicant tracking system | `greenhouse` |
| `sector` | string | Industry classification | `cloud_infra` |
| `last_activity` | ISO 8601 | Most recent job activity | `2026-03-25T00:00:00Z` |
| `last_checked` | ISO 8601 | Most recent health scan | `2026-03-25T00:00:00Z` |

---

## 8 Cost Center Scores

Each cost center returns:

| Field | Type | Description |
|-------|------|-------------|
| `score` | integer (0-100) or null | Weighted average of signals in this cost center |
| `tier` | enum | `Enterprise` (≥75) / `Growth` (≥50) / `Startup` (≥25) / `Inactive` (<25) |
| `budget_status` | enum | `Active` (≥75) / `Moderate` (≥50) / `Low` (≥25) / `Frozen` (<25) |
| `signals_detected` | string[] | Plain-English signal names (max 5) |

### Cost Center 1: IT & Infrastructure

**7 checks.** Tracks cloud spend, server infrastructure, and email security.

| Signal | Sentiment | Weight | What triggers it |
|--------|-----------|--------|-----------------|
| Verified SaaS Tooling | positive | 90 | DNS TXT records prove ownership of Atlassian, Google, Stripe, etc. |
| DMARC Fully Enforced | positive | 80 | DMARC policy set to `reject` — full email authentication |
| DMARC Quarantine | positive | 60 | DMARC policy set to `quarantine` |
| Enterprise CDN/WAF | positive | 75 | Akamai, Fastly, CloudFront, Azure CDN detected |
| Modern CDN | positive | 50 | Cloudflare, Vercel, Netlify detected |
| Web Application Firewall | positive | 70 | Imperva, Sucuri, DDoS-Guard WAF active |
| Enterprise Email Firewall | positive | 85 | Proofpoint, Mimecast, Barracuda in MX records |
| Enterprise Email Platform | positive | 40 | Google Workspace or Microsoft 365 via MX |
| Fast TTFB | positive | 55 | Server response ≤800ms |
| Slow TTFB | negative | 50 | Server response >1800ms — infrastructure degradation |
| TTFB Degrading | negative | 60 | TTFB increased >50% vs prior scan |
| Verified Vendor Spend | positive | 80 | SPF record authorizes email for CRM/ATS/marketing vendors |
| High Publish Velocity | positive | 70 | Active npm/PyPI package publishing |
| HSTS Active | positive | 35 | HTTP Strict Transport Security enforced |
| Mature Security Posture | positive | 55 | 4-5/5 security headers configured |

### Cost Center 2: Engineering & Product

**6 checks.** Tracks R&D velocity, developer tools, and product shipping cadence.

| Signal | Sentiment | Weight | What triggers it |
|--------|-----------|--------|-----------------|
| Enterprise Vendor Stack (CSP) | positive | 80 | 3+ enterprise vendors in Content Security Policy header |
| Payment Processing Active | positive | 60 | Stripe/Braintree/PayPal in CSP |
| Enterprise Sales Tooling | positive | 55 | Salesforce/Gong/Outreach in CSP |
| High Shipping Velocity | positive | 75 | 5+ changelog/release updates in 30 days |
| Active Product Development | positive | 55 | Changelog updated within 30 days |
| Product Development Stalled | negative | 60 | No changelog updates in 90+ days |
| Rich Internal Infrastructure | positive | 70 | 8+ subdomains resolve (api, docs, status, sso, etc.) |
| Mature Engineering Infrastructure | positive | 55 | API + docs + status page all active |
| SSO Present | positive | 50 | SSO subdomain resolves |
| Enterprise SSO Configured | positive | 75 | OpenID Connect `.well-known` returns valid JSON |
| Apple Pay Enabled | positive | 55 | Apple Pay merchant domain association |
| Security Disclosure Policy | positive | 40 | `security.txt` present |
| Active Mobile App | positive | 55 | App Store/Play Store app updated recently |
| Mobile App Deep Linking | positive | 50 | iOS + Android deep link config |
| Healthy Job Schema | positive | 65 | <20% expired `validThrough` in Schema.org JobPosting |
| Career Page Graveyard | negative | 75 | 80%+ of job schema dates expired |

### Cost Center 3: Sales & Marketing

**4 checks.** Tracks ad spend, international expansion, GTM motion, and market intelligence.

| Signal | Sentiment | Weight | What triggers it |
|--------|-----------|--------|-----------------|
| Multi-Platform Ad Spend | positive | 70 | 3+ ad platform pixels (Meta, Google, LinkedIn, etc.) |
| Active Digital Advertising | positive | 50 | 1-2 ad platforms detected |
| Multi-Network Ad Stack (GTM) | positive | 75 | 3+ ad networks found inside GTM container JS |
| Tag Manager Active | positive | 55 | Google Tag Manager, Adobe Launch, or Tealium |
| Active Performance Marketing | positive | 65 | Advanced conversion events (`fbq('track','Lead')`, etc.) |
| Heavy Campaign Infrastructure | positive | 60 | 3+ hidden landing page directories in robots.txt |
| Heavy Tracking Infrastructure | positive | 60 | 3+ tracking pixels/beacons detected |
| First-Party Analytics | positive | 50 | Self-hosted tracking endpoints on company domain |
| Global Market Presence | positive | 70 | 6+ hreflang language/region variants |
| European Market Expansion | positive | 50 | EU hreflang tags detected |
| Asia-Pacific Expansion | positive | 50 | APAC hreflang tags detected |
| New Market Expansion | positive | 65 | New hreflang regions added since last scan |
| Stable Website Structure | positive | 35 | <25% of core paths redirect |
| CDN Edge Routing | neutral | 15 | High redirect rate but same-domain (CDN, not reorg) |
| Major Website Restructuring | negative | 55 | >50% of core paths redirect cross-domain |
| Pricing Page Restructured | negative | 45 | `/pricing` redirected or 404 |
| Enrichment signals | varies | varies | 42 external sources (SEC, H-1B, patents, funding, news) |

### Cost Center 4: Legal & Compliance

**4 checks.** Tracks compliance frameworks, business registration, and regulatory posture.

| Signal | Sentiment | Weight | What triggers it |
|--------|-----------|--------|-----------------|
| Comprehensive Compliance Program | positive | 80 | 3+ compliance frameworks (SOC 2, ISO, HIPAA, etc.) |
| SOC 2 Certified | positive | 70 | SOC 2 keywords on trust/security page |
| HIPAA Compliant | positive | 60 | HIPAA keywords detected |
| Trust Page Present | positive | 35 | Trust or security page exists |
| Active H-1B Filer | positive | 30 | Certified LCA filings found |
| H-1B Filings Declining | negative | 60 | Year-over-year filing count dropped >20% |
| WARN Act Notice Filed | negative | 80 | WARN Act layoff notice detected |
| Verified Vendor Spend | positive | 80 | SPF authorizes enterprise vendor email |

### Cost Center 5: HR & Hiring

**9 checks.** Tracks job listing quality, hiring velocity, and ghost job detection.

| Signal | Sentiment | Weight | What triggers it |
|--------|-----------|--------|-----------------|
| Excellent Job Quality | positive | 40 | Average listing quality ≥80/100 |
| Good Job Quality | positive | 35 | Average listing quality ≥60/100 |
| Active Job Board | positive | 25 | Active listings with <10% stale |
| Posting Velocity Surge | positive | 35 | Posting rate >150% of baseline |
| Stable Hiring Rate | positive | 30 | Posting rate 80-150% of baseline |
| Ghost Job Acceleration | negative | 40 | Ghost job rate increasing over time |
| Healthy Description Quality | positive | 15 | Median description >3000 chars, low similarity |
| Normal Repost Rate | neutral | 10 | Repost churn <25% |
| Salary Transparency Declining | negative | 30 | Salary info removed from listings |

### Cost Center 6: Finance & RevOps

**1 check.** Tracks billing/payment infrastructure tier.

| Signal | Sentiment | Weight | What triggers it |
|--------|-----------|--------|-----------------|
| Enterprise Billing Platform | positive | 75 | Zuora, Chargebee, Recurly detected in CSP/HTML |
| Direct Bank API Integration | positive | 70 | Plaid, MX, Finicity in CSP/HTML |
| Enterprise Payment Processing | positive | 60 | Adyen, Braintree, Checkout.com detected |
| Growth-Stage Payments | positive | 40 | Stripe, PayPal detected |
| Startup-Tier Payments | neutral | 20 | Paddle, Gumroad, Lemon Squeezy |

### Cost Center 7: Telecom & Remote Work

**1 check.** Tracks enterprise UC contracts via DNS SRV records.

| Signal | Sentiment | Weight | What triggers it |
|--------|-----------|--------|-----------------|
| Enterprise UC Platform | positive | 70 | Teams, Zoom Phone, RingCentral, Cisco in SRV records |
| Microsoft Teams Federation | positive | 55 | `_sipfederationtls` SRV record pointing to Teams |
| Growth-Stage Telecom | positive | 40 | Dialpad, Twilio, Bandwidth detected |

### Cost Center 8: Corporate Development

**1 check.** Tracks M&A activity via infrastructure parent matching.

| Signal | Sentiment | Weight | What triggers it |
|--------|-----------|--------|-----------------|
| Possible Acquisition Detected | neutral | 70 | Same non-hosting parent controls both MX + NS |
| Independent Infrastructure | positive | 40 | Company manages its own email and DNS |
| Email Infrastructure Migration | neutral | 60 | MX parent changed since last scan |

---

## Boolean Convenience Flags (Clay-compatible)

| Field | Type | Description |
|-------|------|-------------|
| `is_spending_on_ads` | boolean | Ad pixels, GTM ad networks, campaign infrastructure, or tracking detected |
| `has_enterprise_sso` | boolean | OpenID Connect SSO configured |
| `has_enterprise_billing` | boolean | Enterprise billing platform (Zuora, Chargebee, Adyen) |
| `has_enterprise_email_security` | boolean | Enterprise email firewall (Proofpoint, Mimecast) |
| `has_enterprise_uc` | boolean | Enterprise UC platform (Teams, Zoom Phone) via SRV |
| `has_compliance_program` | boolean | SOC 2, HIPAA, or ISO 27001 compliance detected |
| `possible_acquisition` | boolean | MX+NS records suggest parent company absorption |
| `total_cost_centers_active` | integer | Number of cost centers with non-null scores |

---

## 42 Enrichment Sources

### RSS/News (15 broadcast sources)
Layoffs.fyi, GlobeNewswire, PR Newswire, Business Wire, SEC EDGAR RSS, SEC EDGAR 8-K, Hacker News, WARN Notices, Google News, TechCrunch, Crunchbase News, YC Blog, GeekWire, Wired Business, VentureBeat

### US Government (7 domain-targeted)
H-1B LCA filings, USAspending (federal contracts), SAM.gov, ProPublica 990 (nonprofits), USPTO Patents, GitHub Org activity, Delaware Corporations

### Financial (5 domain-targeted)
Yahoo Finance (stock data), SEC EDGAR Filings, Crunchbase Funding, PDL Headcount, Finnhub (financial metrics)

### Infrastructure (8 domain-targeted)
RDAP/WHOIS, DNS Records, Mozilla Observatory, HIBP Breaches, Shodan InternetDB, CrUX (Chrome Web Performance), CRT.sh (certificates — inactive), Finnhub

### International Registries (7 — selective activation)
UK Companies House, Australia ABN, Singapore ACRA, OpenCorporates, OpenSanctions, plus Canada/EU exchange data

---

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/company?domain={domain}` | GET | Nested JSON with cost center objects |
| `/api/v1/enrich?domain={domain}` | GET | Flat JSON for Clay column mapping |
| `/api/v1/docs` | GET | Full API documentation |

### Authentication
- **Explorer tier (free):** No API key required. 100 credits/month.
- **Outbound tier ($249/mo):** API key via `x-api-key` header. 2,500 credits/month.
- **Forensic tier ($899/mo):** API key. 15,000 credits/month. Priority rate limits.

### Rate Limits
60 requests per minute per IP.

---

## Coverage

| Source | Companies | Status |
|--------|-----------|--------|
| SEC EDGAR (US tickers) | 6,467 | Complete |
| Curated list | ~2,000 | In pipeline |
| TMX (Canada) | ~3,600 | Processing |
| Euronext (EU) | ~1,200 | Processing |
| GLEIF (Global LEIs) | ~1.1M available | Processing (US, GB, Asia) |
| Adzuna (job aggregator) | 13 countries | Active |
| **Current total** | **14,000+** | **Growing daily** |
| **Q2 target** | **100,000+** | |

---

## Update Frequency

| Data type | Frequency |
|-----------|-----------|
| Health checks (33 checks) | Per pipeline cycle (~daily per company) |
| Job listings | Scraped 2x daily |
| Enrichment sources (RSS/news) | Hourly |
| Enrichment sources (government) | Daily |
| Financial data (Finnhub, Yahoo) | Daily |
| CrUX web performance | Weekly (28-day rolling) |

---

**Live demo:** [thesubspace.io/trace](https://www.thesubspace.io/trace) — type any domain, watch 33 checks run in real-time.

**API docs:** [thesubspace.io/api/v1/docs](https://www.thesubspace.io/api/v1/docs)

**Open data sample:** [github.com/vvknyn/subspace-open-data](https://github.com/vvknyn/subspace-open-data)

**Contact:** hello@thesubspace.io
