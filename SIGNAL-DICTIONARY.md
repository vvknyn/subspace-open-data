Now I have all 33 files. Here is the comprehensive data dictionary:

---

# Subspace Signal Data Dictionary
## Complete Inventory of All Signals Across 33 Health Checks / 8 Cost Centers

---

## 1. IT & Infrastructure (7 checks)

---

### Check: cdn-fingerprint -- CDN & WAF Fingerprint
**Cost Center:** IT & Infrastructure
**Detection Method:** HTTP HEAD request to `https://{domain}`, reads response headers for CDN/WAF provider signatures.

**How it works:**
Sends an HTTP HEAD request to the company homepage with a 10s timeout. Inspects the `Server`, `Via`, `X-Akamai-Transformed`, `cf-ray`, `x-fastly-request-id`, `x-amz-cf-id`, `x-vercel-id`, `x-nf-request-id`, `x-azure-ref` and other vendor-specific headers. Classifies detected CDN providers into enterprise tier (Akamai, Fastly, CloudFront, Google Cloud, Azure) or modern tier (Cloudflare, Vercel, Netlify). Also checks for WAF signatures (Sucuri, Imperva, DDoS-Guard).

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| cdn-enterprise | Enterprise CDN/WAF | positive | 75 | Enterprise-tier CDN detected (Akamai, Fastly, CloudFront, Google, Azure) | Company invests in premium infrastructure -- likely mid-market or enterprise |
| cdn-modern | Modern CDN | positive | 50 | Modern CDN detected (Cloudflare, Vercel, Netlify) | Company uses modern hosting platforms -- active engineering team |
| cdn-none-basic | Direct Hosting (No CDN) | neutral | 15 | No CDN detected but known server (nginx, Apache, etc.) | No CDN investment -- hosting directly, early-stage or cost-conscious |
| cdn-unknown | Unidentified Infrastructure | neutral | 10 | No CDN and no recognizable server header | Cannot determine hosting provider from headers |
| waf-detected | Web Application Firewall | positive | 40 | WAF signatures found (Sucuri, Imperva, DDoS-Guard, or Cloudflare WAF) | Active security investment -- indicates compliance or security mandate |
| cdn-downgrade-detected | CDN Tier Downgrade (Temporal) | negative | 60 | CDN tier dropped from prior scan (e.g., enterprise to modern to none) | Infrastructure cost-cutting -- company may be tightening budget |
| waf-removed | WAF Protection Removed (Temporal) | negative | 45 | WAF was present in prior scan but absent now | Security downgrade -- possible cost-cutting or infrastructure change |

---

### Check: server-identity -- Server & Edge Identity
**Cost Center:** IT & Infrastructure
**Detection Method:** HTTP HEAD request to homepage, reads Server, X-Powered-By, Via headers + 5 security headers.

**How it works:**
Sends an HTTP HEAD request to `https://{domain}`. Combines the Server, X-Powered-By, and Via headers into a single identity string, then classifies it against a database of known server tiers: WAF (Imperva, Sucuri, F5), enterprise (Akamai, CloudFront, Google, Azure), modern (Cloudflare, Vercel, Netlify, Fastly), and basic (Nginx, Apache). Also inventories 5 security headers: Strict-Transport-Security, X-Content-Type-Options, X-Frame-Options, Referrer-Policy, and Content-Security-Policy.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| server-waf-active | WAF Active | positive | 70 | Server header matches a WAF provider (Imperva, Sucuri, DDoS-Guard, F5) | Enterprise-grade security spending -- likely compliance-driven |
| server-security-mature | Mature Security Posture | positive | 55 | 4-5 of 5 security headers present | Well-configured security posture -- mature engineering/IT team |
| server-security-weak | Weak Security Posture | neutral | 20 | 0-1 of 5 security headers present | Basic security configuration -- early-stage or underfunded IT |
| server-hsts-active | HSTS Active | positive | 35 | Strict-Transport-Security header present | Enforced HTTPS -- basic security hygiene met |
| server-identity-changed | Server Identity Changed (Temporal) | neutral | 45 | Server header differs from prior scan | Platform migration or scaling event in progress |
| server-waf-added | WAF Deployed (Temporal) | positive | 70 | WAF detected now but not in prior scan | Security escalation -- company investing in protection |
| server-waf-removed | WAF Removed (Temporal) | negative | 60 | WAF was present in prior scan, now absent | Security downgrade -- cost-cutting or misconfiguration |

---

### Check: ttfb-tracking -- Time-to-First-Byte (TTFB)
**Cost Center:** IT & Infrastructure
**Detection Method:** 3 sequential HTTP GET requests to homepage, measures time until first bytes arrive.

**How it works:**
Makes 3 sequential GET requests to `https://{domain}` with 200ms delays between them, each with a 10s timeout. Records the elapsed time for each request, takes the median of 3 readings as the TTFB. Compares against prior scan median for temporal trend detection.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| ttfb-fast | Fast TTFB | positive | 55 | Median TTFB <= 800ms | Well-maintained infrastructure -- good CDN/cloud investment |
| ttfb-moderate | Moderate TTFB | neutral | 25 | Median TTFB 800-1800ms | Average server response -- no red flag |
| ttfb-slow | Slow TTFB | negative | 50 | Median TTFB > 1800ms | Possible infrastructure degradation or cost-cutting |
| ttfb-degrading | TTFB Degrading (Temporal) | negative | 60 | TTFB increased > 50% from prior scan | Infrastructure may be deteriorating -- resource constraints |
| ttfb-improving | TTFB Improving (Temporal) | positive | 40 | TTFB decreased > 30% from prior scan | Infrastructure investment detected |

---

### Check: email-security -- Email Security Maturity
**Cost Center:** IT & Infrastructure
**Detection Method:** DNS TXT lookup for `_dmarc.{domain}` + DNS MX lookup for `{domain}`.

**How it works:**
Runs two DNS queries in parallel: (1) resolves TXT records for `_dmarc.{domain}` to find the DMARC policy (none/quarantine/reject) and whether aggregate reporting (rua=) is enabled, and (2) resolves MX records to identify the email provider and check for enterprise email security gateways (Proofpoint, Mimecast, Barracuda, etc.).

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| dmarc-enforcing | DMARC Fully Enforced | positive | 80 | DMARC policy set to `p=reject` | Full email authentication -- enterprise-grade security team |
| dmarc-quarantine | DMARC Quarantine | positive | 60 | DMARC policy set to `p=quarantine` | Active email security management -- mid-stage DMARC rollout |
| dmarc-monitoring | DMARC Monitor-Only | neutral | 25 | DMARC policy set to `p=none` | Monitoring only -- company aware of DMARC but hasn't enforced yet |
| mx-enterprise-gateway | Enterprise Email Gateway | positive | 85 | MX records point to Proofpoint, Mimecast, Barracuda, FireEye, or Cisco Email Security | Significant headcount and security budget -- enterprise email protection |
| mx-business-email | Business Email Provider | positive | 40 | MX records point to Google Workspace or Microsoft 365 | Standard business email -- company is operational |
| mx-custom | Custom Email Infrastructure | neutral | 20 | MX records exist but don't match any known provider | Custom or self-hosted email -- indeterminate |
| email-no-data | No Email Security Data | neutral | 10 | No DMARC and no MX data found | Cannot assess email security posture |

---

### Check: saas-verification -- SaaS Verification
**Cost Center:** IT & Infrastructure
**Detection Method:** DNS TXT lookup for `{domain}`, scans for SaaS verification tokens.

**How it works:**
Resolves all DNS TXT records for the company domain. Scans the flat record list for verification tokens from 11 major SaaS providers: Atlassian, Google, Stripe, Meta Business, Apple Business Manager, HubSpot, Adobe Enterprise, DocuSign, Okta, Slack, and Zoom.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| saas-verified-tools | Verified SaaS Tooling | positive | 50 (90 if 3+) | Domain has verification tokens for 1+ SaaS providers | Company actively uses paid SaaS -- budget and operational maturity |
| saas-no-verification | No SaaS Verification | neutral | 20 | No recognized SaaS verification TXT records | May use SaaS without domain verification, or early-stage |
| saas-no-txt-records | No TXT Records | neutral | 10 | DNS ENODATA -- no TXT records at all | Domain has no TXT records -- very early-stage or parked |

---

### Check: package-registry -- Package Registry Velocity
**Cost Center:** IT & Infrastructure
**Detection Method:** HTTP GET to NPM registry org endpoint + package metadata + downloads API.

**How it works:**
Derives NPM organization name(s) from the domain (with a mapping table for known companies). Queries `registry.npmjs.org/-/org/{name}/package` to get org packages, then fetches metadata for up to 10 packages to compute release dates and download counts. Aggregates releases in last 90/180 days, weekly downloads, and days since last publish.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| pkg-high-velocity | High Publish Velocity | positive | 50-70 | 3+ releases across sampled packages in last 90 days (70 if 10+) | Active engineering team shipping open-source -- strong engineering culture |
| pkg-moderate-velocity | Moderate Publish Velocity | positive | 30 | 2+ releases in last 180 days but <3 in 90 days | Engineering team is active but not high-velocity |
| pkg-stale | Package Registry Stale | negative | 45 | No releases in 180+ days | Engineering output may be slowing or team reduced |
| pkg-low-velocity | Low Publish Velocity | neutral | 15 | Has packages but limited recent releases | Minimal open-source activity -- not necessarily bad |
| pkg-no-org | No NPM Organization | neutral | 5 | No NPM org found for derived name | Company may not publish open-source packages |
| pkg-data-unavailable | Package Registry Unavailable | neutral | 5 | Error fetching NPM data | Data source unavailable |

---

### Check: mx-maturation -- Email Infrastructure Maturity (MX)
**Cost Center:** IT & Infrastructure
**Detection Method:** DNS-over-HTTPS query to Google DNS (`dns.google/resolve?type=MX`) for the company domain.

**How it works:**
Queries Google DNS-over-HTTPS for MX records, parses the exchange hostnames, and classifies each against a database of known email providers in 4 tiers: enterprise_security (Proofpoint, Mimecast, Barracuda, Trellix, Symantec, Cisco IronPort), enterprise (Google Workspace, Microsoft 365, Amazon SES), standard (Zoho, Fastmail, Rackspace, Mailgun), and basic (GoDaddy, Rackspace Basic, Hover).

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| mx-enterprise-security | Enterprise Email Security Gateway | positive | 80 | Proofpoint, Mimecast, Barracuda, Trellix, Symantec, or Cisco IronPort MX | Significant cybersecurity budget -- enterprise compliance infrastructure |
| mx-enterprise-email | Enterprise Email Platform | positive | 40 | Google Workspace, Microsoft 365, or Amazon SES MX (no security gateway) | Standard enterprise email -- company is operational |
| mx-standard-email | Standard Email Provider | neutral | 20 | Zoho, Fastmail, Rackspace, or Mailgun MX | SMB-grade email -- smaller company or cost-conscious |
| mx-basic-email | Basic Email Hosting | neutral | 10 | GoDaddy, Rackspace Basic, or Hover MX | Early-stage or basic infrastructure |
| mx-none | No MX Records | negative | 40 | No MX records found | Email misconfigured or domain inactive |
| mx-upgrade | Email Infrastructure Upgrade (Temporal) | positive | 65 | MX tier improved from prior scan | Enterprise investment -- company upgrading email infrastructure |

---

## 2. Engineering & Product (6 checks)

---

### Check: csp-tech-stack -- CSP Vendor Inventory
**Cost Center:** Engineering & Product
**Detection Method:** HTTP HEAD then GET to homepage, parses Content-Security-Policy header for authorized third-party domains.

**How it works:**
First tries HEAD, then falls back to GET if no CSP header on HEAD. Parses the CSP header to extract all declared domains, then classifies each against a vendor database of 35+ vendors across categories: payments (Stripe, Braintree, PayPal), analytics (Segment, Mixpanel, Amplitude, Heap, FullStory, Hotjar), CRM/sales (Salesforce, HubSpot, Marketo, Pardot, Gong, Outreach, SalesLoft), support (Intercom, Zendesk, Drift, Crisp, Tawk.to), advertising, and infrastructure.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| csp-absent | No CSP Header | neutral | 15 | No Content-Security-Policy header found | Cannot inventory vendor stack via CSP -- many sites don't use CSP |
| csp-enterprise-stack | Enterprise Vendor Stack | positive | 80 | 3+ enterprise-tier vendors in CSP | Major vendor spend -- enterprise-level tool investment |
| csp-growth-stack | Active Vendor Stack | positive | 55 | 3+ total vendors but <3 enterprise | Active vendor stack -- growth-stage tool investment |
| csp-minimal-stack | Minimal Vendor Stack | neutral | 20 | Fewer than 3 recognized vendors | Minimal third-party tooling |
| csp-payments-active | Payment Processing Active | positive | 60 | Payment vendor (Stripe, Braintree, PayPal) in CSP | Monetization infrastructure in place -- revenue-generating product |
| csp-sales-tooling | Enterprise Sales Tooling | positive | 55 | CRM or sales tool (Salesforce, Gong, Outreach, SalesLoft) in CSP | Enterprise sales motion -- dedicated sales team |
| csp-vendor-removed | Enterprise Vendor Removed (Temporal) | negative | 65 | Enterprise vendor disappeared from CSP since prior scan | Possible cost-cutting -- vendor contract not renewed |

---

### Check: changelog-velocity -- Product Changelog Velocity
**Cost Center:** Engineering & Product
**Detection Method:** HTTP GET to 6 common changelog paths (`/changelog`, `/releases`, `/updates`, `/whats-new`, `/blog/changelog`, `/product-updates`), parses dates from page content.

**How it works:**
Probes 6 common changelog/release page URLs in parallel with 8s timeouts. Reads the first 200KB of each page that returns 200, extracts dates using 4 regex patterns (ISO dates, US long/short month formats, numeric). Deduplicates dates, counts updates in last 30 and 90 days, and computes days since last update.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| changelog-high-velocity | High Shipping Velocity | positive | 75 | 5+ updates in last 30 days | Rapid product development -- strong engineering team |
| changelog-active | Actively Shipping | positive | 55 | 1-4 updates in last 30 days | Product is being actively developed |
| changelog-slowing | Shipping Velocity Slowing | neutral | 30 | Last update 31-90 days ago | Product velocity declining -- possible resource constraints |
| changelog-stalled | Product Shipping Stalled | negative | 60 | Last update 90+ days ago | Engineering may be in maintenance mode -- possible team reduction |
| changelog-not-found | No Public Changelog | neutral | 10 | No changelog page found at common paths | Many companies don't have public changelogs |
| changelog-exists-no-dates | Changelog Without Dates | neutral | 20 | Page exists but no parseable dates | Page exists but may use non-standard date formats |

---

### Check: subdomain-signals -- Internal Infrastructure Subdomains
**Cost Center:** Engineering & Product
**Detection Method:** DNS A-record queries via Google DNS-over-HTTPS for 20 common subdomains (hr, careers, jobs, api, docs, status, staging, dev, git, sso, vpn, mail, remote, support, help, blog, shop, app, portal, dashboard).

**How it works:**
Probes 20 common subdomains in batches of 5 via Google DNS JSON API (`dns.google/resolve?type=A`). Each subdomain is categorized: HR (hr, careers, jobs), engineering (api, docs, status, staging, dev, git), IT/security (sso, vpn, mail, remote), and business (support, help, blog, shop, app, portal, dashboard). Counts resolved vs. not-resolved.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| infra-rich | Rich Internal Infrastructure | positive | 70 | 8+ subdomains resolved across multiple categories | Large organization with mature infrastructure -- enterprise-scale |
| infra-moderate | Moderate Infrastructure Footprint | positive | 45 | 4-7 subdomains resolved | Mid-size company with active infrastructure |
| infra-minimal | Minimal Subdomain Footprint | neutral | 20 | 1-3 subdomains resolved | Small company or consolidated infrastructure |
| infra-bare | No Common Subdomains Detected | neutral | 10 | Zero subdomains resolved | Very small company or all services on third-party platforms |
| engineering-infra-mature | Mature Engineering Infrastructure | positive | 55 | api + docs + status all resolve | Engineering team has API, docs, and status page -- platform company |
| sso-present | SSO Infrastructure Detected | positive | 50 | sso.{domain} resolves | Enterprise-grade identity management -- likely SOC 2 compliant |
| status-page-present | Public Status Page Active | positive | 40 | status.{domain} resolves | Operational transparency and maturity |

---

### Check: schema-job-decay -- Schema.org Job Posting Health
**Cost Center:** Engineering & Product
**Detection Method:** HTTP GET to 4 career page paths (`/careers`, `/jobs`, `/careers/jobs`, `/open-positions`), parses JSON-LD `JobPosting` schema.

**How it works:**
Probes 4 common career page URLs in parallel, reads first 500KB of each page. Extracts all `<script type="application/ld+json">` blocks, parses for `@type: "JobPosting"` objects. Inspects `validThrough` dates to compute decay rate (percentage of postings with expired dates).

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| schema-no-career-page | No Career Page Detected | neutral | 10 | No career page found at common paths | Company may use external ATS only |
| schema-no-jsonld | No Schema.org JobPosting Markup | neutral | 15 | Career page exists but no JSON-LD JobPosting | ATS not configured for Google Jobs -- less SEO-optimized hiring |
| schema-healthy | Healthy Job Schema | positive | 65 | <20% of postings expired | HR pipeline actively maintained -- job postings are current |
| schema-moderate-decay | Moderate Schema Decay | neutral | 40 | 20-50% of postings expired | Some postings stale -- HR may be behind on cleanup |
| schema-high-decay | Significant Schema Decay | negative | 60 | 50-80% of postings expired | HR may be neglecting the pipeline -- questionable hiring intent |
| schema-critical-decay | Career Page Graveyard | negative | 75 | 80%+ of postings expired | Career page is abandoned -- active hiring is unlikely |
| schema-missing-dates | Missing validThrough Dates | neutral | 30 | >50% of postings lack validThrough | ATS not properly configured -- cannot assess posting freshness |

---

### Check: app-store-pulse -- App Store Pulse
**Cost Center:** Engineering & Product
**Detection Method:** HTTP GET to iTunes Search API (`itunes.apple.com/search?entity=software`) with company name as search term.

**How it works:**
Derives a search term from the company name or domain. Queries the iTunes Search API for up to 5 matching software apps. Fuzzy-matches returned apps against the company name/domain by comparing seller name, artist name, and seller URL. Reports the most recently updated matched app's age.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| app-recently-updated | App Recently Updated | positive | 55 | Most recent matched app updated within 90 days | Active mobile engineering team -- product under development |
| app-stale | App Aging | neutral | 20 | Most recent matched app updated 91-365 days ago | Mobile product may be in maintenance mode |
| app-abandoned | App Abandoned | negative | 50 | Most recent matched app not updated in 365+ days | Mobile product may be abandoned -- engineering resource concern |
| app-not-found | No Apps Found | neutral | 5 | No matching apps or API error | Many B2B companies don't have iOS apps |

---

### Check: wellknown-discovery -- Well-Known URI Discovery
**Cost Center:** Engineering & Product
**Detection Method:** HTTP HEAD (then GET for SSO validation) to 10 `.well-known` paths on the company domain.

**How it works:**
Probes 10 standard `.well-known` URIs in parallel: OpenID Connect configuration, OAuth server, Apple Pay merchant association, security.txt, change-password URL, Android App Links, iOS Universal Links, WebFinger, host-meta, and Matrix server. For OpenID Connect specifically, validates with a GET and JSON parse to confirm it's real SSO (not a generic 200).

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| wellknown-sso | Enterprise SSO (OpenID Connect) | positive | 75 | Valid OpenID Connect configuration (JSON-validated) | Enterprise SSO -- selling to Fortune 500 clients requiring SSO |
| wellknown-apple-pay | Apple Pay Merchant Association | positive | 55 | Apple Pay merchant domain association file present | Consumer payment infrastructure -- B2C revenue stream |
| wellknown-security-txt | Security Disclosure Policy | positive | 40 | security.txt present | Security-mature organization with vulnerability disclosure process |
| wellknown-mobile-apps | Mobile App Deep Linking (iOS + Android) | positive | 50 | Both Apple App Site Association and Android Asset Links present | Active mobile product on both platforms |
| wellknown-mobile-partial | Mobile App Deep Linking (partial) | positive | 30 | Only iOS or only Android deep linking configured | Mobile product on one platform only |
| wellknown-rich | Rich .well-known Configuration | positive | 60 | 4+ capabilities found | Mature engineering organization with multiple platform integrations |
| wellknown-none | No .well-known Capabilities | neutral | 10 | Zero capabilities detected | No standard well-known URIs -- not necessarily negative |

---

## 3. Sales & Marketing (4 checks)

---

### Check: web-tech-stack -- Web Technology Stack Analysis
**Cost Center:** Sales & Marketing
**Detection Method:** HTTP GET to homepage (reads full HTML up to 500KB), scans for ad pixels, analytics, SaaS scripts, tracking pixels. Also fetches GTM container JS and robots.txt.

**How it works:**
Fetches the full homepage HTML. Scans for 6 ad pixel patterns (Meta, Google, LinkedIn, Twitter/X, Microsoft, TikTok), 8 analytics tools (Mixpanel, Amplitude, Segment, Heap, FullStory, Hotjar, PostHog, Plausible), and 13 SaaS vendor scripts (Intercom, Gong, Drift, Qualified, 6Sense, Clearbit, Marketo, Pardot, HubSpot, Crisp, Tawk.to, Tidio, LiveChat). Also detects tag managers (GTM, Adobe Launch, Tealium, Segment), tracking pixels, data layers, self-hosted tracking, GTM container ad networks, advanced conversion events (fbq/gtag/lintrk), and hidden campaign landing pages in robots.txt.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| ad-heavy-spend | Heavy Ad Spend | positive | 70 | 3+ ad platforms detected | Significant customer acquisition budget -- funded growth |
| ad-active | Active Advertising | positive | 50 | 1-2 ad platforms detected | Company is investing in paid acquisition |
| tag-manager-active | Tag Manager Active | positive | 55 | GTM, Adobe Launch, Tealium, or Segment detected | Multiple tracking tools downstream -- marketing ops team |
| tracking-heavy | Heavy Tracking Infrastructure | positive | 60 | 3+ tracking pixels/beacons | Active conversion measurement and audience targeting |
| tracking-active | Tracking Pixels Active | positive | 40 | 1-2 tracking pixels/beacons | Conversion tracking in place |
| self-hosted-tracking | First-Party Tracking | positive | 50 | Self-hosted tracking endpoints detected | Company operates its own analytics -- privacy-conscious and sophisticated |
| data-layer-active | Analytics Data Layer | positive | 35 | dataLayer or equivalent present (without tag manager) | Structured event tracking -- data-driven product team |
| gtm-multi-network | Multi-Network Ad Stack (GTM) | positive | 75 | 3+ ad networks found inside GTM container JavaScript | Broad paid acquisition program spanning multiple ad networks |
| gtm-ad-networks | Ad Networks in GTM | positive | 55 | 1-2 ad networks in GTM container | Ad network(s) managed through GTM |
| advanced-conversions | Active Performance Marketing | positive | 65 | Advanced conversion events (fbq track, gtag event, lintrk track) found | Optimized campaigns with attribution -- serious performance marketing |
| hidden-campaigns-heavy | Heavy Campaign Infrastructure | positive | 60 | 3+ campaign directories in robots.txt | Active paid acquisition program with dedicated landing pages |
| hidden-campaigns | Campaign Landing Pages Detected | positive | 40 | 1-2 campaign directories in robots.txt | Some campaign landing pages -- mild marketing investment |
| analytics-enterprise | Enterprise Analytics Stack | positive | 55 | Segment, Heap, or FullStory detected | Enterprise-tier analytics -- significant data investment |
| analytics-growth | Growth Analytics Stack | positive | 35 | Mixpanel, Amplitude, Hotjar, or PostHog detected | Growth-tier analytics -- data-driven product decisions |
| analytics-none | No Analytics Detected | neutral | 10 | No recognized analytics tools | No analytics detected -- may use server-side only |
| saas-premium-stack | Premium SaaS Stack | positive | 65 | Premium SaaS tools (Intercom, Gong, Drift, Qualified, 6Sense, Clearbit, Marketo, Pardot) | Active operational budget for premium tools |
| saas-budget-stack | Budget SaaS Stack | neutral | 20 | Only budget SaaS tools (Crisp, Tawk.to, Tidio) | Early-stage or cost-conscious |
| rich-tech-stack | Rich Third-Party Integration | positive | 30 | 10+ external script tags | Active product investment with many integrations |

---

### Check: hreflang-expansion -- International Expansion (Hreflang)
**Cost Center:** Sales & Marketing
**Detection Method:** HTTP GET to homepage, parses `<link rel="alternate" hreflang="...">` tags from HTML and HTTP Link headers.

**How it works:**
Fetches the first 200KB of the homepage HTML. Extracts hreflang tags from both HTML `<link>` tags and HTTP `Link` headers using two regex patterns to handle reversed attribute order. Maps each language/region code to a human-readable region name. Categorizes languages as European (de, fr, es, it, nl, sv, da, fi, pl) or Asia-Pacific (ja, ko, zh, th, vi, id, ms).

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| hreflang-none | No Hreflang Tags | neutral | 10 | No hreflang tags found | Single-market presence -- domestic-only |
| hreflang-moderate | Moderate International Presence | positive | 45 | 2-5 language/region variants | International presence -- active in multiple markets |
| hreflang-global | Global Presence (Hreflang) | positive | 70 | 6+ language/region variants | Significant investment in global expansion |
| hreflang-eu-expansion | European Market Expansion | positive | 50 | European language (de/fr/es/it/nl/sv/da/fi/pl) present | GDPR compliance and localization investment -- European GTM |
| hreflang-apac-expansion | Asia-Pacific Market Expansion | positive | 50 | APAC language (ja/ko/zh/th/vi/id/ms) present | Asia-Pacific market entry |
| hreflang-new-markets | New Market Expansion (Temporal) | positive | 65 | New regions added since prior scan | Active geographic expansion -- new markets launched |

---

### Check: redirect-cascade -- URL Redirect Cascade Detection
**Cost Center:** Sales & Marketing
**Detection Method:** HTTP GET with `redirect: "manual"` to 15 core paths (/product, /products, /pricing, /features, /solutions, /platform, /enterprise, /about, /blog, /docs, /api, /developers, /customers, /careers, /contact).

**How it works:**
Probes 15 core website paths with manual redirect handling (doesn't follow redirects) to detect 301/302/307/308 status codes. Computes redirect rate (percentage of paths that redirect), distinguishes same-domain CDN routing from cross-domain restructuring, and checks specific high-signal paths (/pricing, /careers).

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| redirect-cascade-high | High Redirect Cascade | negative | 55 | >50% of core paths permanently redirected (cross-domain) | Strategic pivot or reorg underway -- major URL restructuring |
| redirect-cdn-routing | CDN Edge Routing | neutral | 15 | >50% redirected but all same-domain | CDN/edge routing pattern -- not restructuring |
| redirect-cascade-moderate | Moderate Redirect Cascade | neutral | 30 | 25-50% of paths redirected | Website evolution in progress |
| redirect-stable | Stable URL Structure | positive | 35 | <25% of paths redirected | Stable infrastructure -- no major restructuring |
| redirect-many-404s | High 404 Rate on Core Paths | negative | 50 | >40% of core paths return 404 | Site degradation or content removal |
| redirect-pricing-gone | Pricing Page Redirected or Missing | negative | 45 | /pricing returns redirect or 404 (non-CDN) | Pricing model change or product pivot |
| redirect-careers-gone | Careers Page Missing | negative | 40 | /careers returns 404 | Hiring page taken down -- possible hiring freeze |
| redirect-spike | Redirect Rate Spike (Temporal) | negative | 60 | Redirect rate jumped >20pp from prior scan | Active restructuring in progress |

---

### Check: enrichment-signals -- External Enrichment Signals
**Cost Center:** Sales & Marketing (default; routes signals to their respective cost centers)
**Detection Method:** Reads pre-fetched enrichment data from `context.enrichmentData` (populated from `company_enrichment` table containing data from 32+ external sources).

**How it works:**
Iterates over all enrichment rows from external sources (RSS feeds, government databases, financial APIs, international registries, sanctions databases, etc.). Each row has a `data_type` and `payload`. Routes signals to appropriate cost center pillars. Computes post-loop aggregations: prior-layoff-pattern (serial cutter detection), funding-velocity, valuation-direction (down-round detection), and cross-references Form D + declining headcount.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| enrichment-layoff-recent-{source} | Recent Layoff (External Intel) | negative | 70 | Layoff event within 90 days | Company recently laid off employees -- active contraction |
| enrichment-layoff-historical-{source} | Historical Layoff (External Intel) | negative | 35 | Layoff event older than 90 days | Past layoff event -- may still affect culture/headcount |
| enrichment-hiring-freeze-{source} | Hiring Freeze (External Intel) | negative | 75 | Hiring freeze signal from external source | Company has publicly or implicitly paused hiring |
| enrichment-shutdown-{source} | Company Shutdown (External Intel) | critical | 95 | Shutdown/closing signal detected | Company is shutting down |
| enrichment-funding-{source} | Recent Funding (External Intel) | positive | 60 | Funding event within 90 days | Recently capitalized -- growth trajectory |
| enrichment-ipo-{source} | IPO Signal (External Intel) | positive | 50 | IPO/going public signal | Company pursuing public listing |
| enrichment-acquisition-{source} | Acquisition Signal (External Intel) | neutral | 40 | Acquisition/merger activity | Corporate development activity -- could go either way |
| enrichment-press-{source} | Press Activity (External Intel) | neutral | 30 | Press release, earnings, or executive change | Company active in media -- operational |
| enrichment-blog-{source} | VC/Accelerator Mention | positive | 35 | Featured by a VC/accelerator blog | VC/accelerator backing -- validation signal |
| enrichment-restructuring-{source} | Restructuring Filing (SEC 8-K) | critical/negative | 85/60 | SEC 8-K Item 2.05 restructuring (recent=critical, old=negative) | Formal restructuring -- government-verified cost reduction |
| enrichment-stock-{source} | Stock Data | varies | 20-55 | Stock price data available (negative if >15% below 200-day avg) | Public market valuation signal |
| enrichment-stock-below-sma200 | Stock Below 200-Day SMA | negative/neutral | 25-45 | Stock trading below 200-day SMA (negative if >20% below) | Market bearish on the company |
| enrichment-warn-{source} | WARN Act Filing | critical | 92 | WARN Act layoff notice from enrichment | Government-verified mass layoff -- highest confidence |
| enrichment-news-{source} | News Event | neutral | 20 | Google News mention | Company in the news -- awareness signal |
| enrichment-community-{source} | Community Discussion | neutral | 15 | HN discussion with >100 points | Active tech community discussion |
| enrichment-h1b-{source} | H-1B Visa Filings (DOL) | positive | 40 | Certified H-1B filings found | Active visa sponsorship -- hiring specialized talent |
| enrichment-gov-contract-{source} | Government Contracts | positive | 45 | Federal contract awards found in USAspending | Stable government revenue stream |
| enrichment-patents-{source} | Patent Activity (USPTO) | positive | 30 | Patents filed in last 2 years | Active R&D investment |
| enrichment-github-{source} | GitHub Activity | positive/neutral | 10-25 | Public repos found (positive if 2+ recently active) | Open-source engineering presence |
| enrichment-nonprofit-{source} | Nonprofit 990 Filing | neutral | 15 | IRS 990 filing found | Nonprofit entity -- different financial model |
| enrichment-osha-{source} | OSHA Violations | negative | 50 | OSHA inspections with penalties | Workplace safety issues -- liability and reputation risk |
| enrichment-financial-filing-{source} | Financial Filings | positive | 35 | EDINET/DART/other financial filings in last year | Publicly reporting company -- financial transparency |
| enrichment-entity-inactive-{source} | Entity Inactive | critical/negative | 60-85 | Business registry shows entity dissolved/cancelled/struck off | Company no longer legally active -- critical if dissolved |
| enrichment-entity-active-{source} | Entity Active | positive | 25 | Business registry confirms active status | Confirmed operational entity |
| enrichment-officer-exodus-{source} | Officer Exodus | negative | 65 | 3+ officer resignations in 180 days | Leadership instability -- board/exec turnover |
| enrichment-headcount-trajectory | Headcount Trajectory | varies | 15-60 | Job posting velocity trend over 6 months | Hiring momentum indicator from temporal job data |
| enrichment-posting-velocity-surge | Posting Velocity Surge | negative | 55-70 | Job posting volume surged >2x | Possible restructuring -- mass backfill after layoffs |
| enrichment-posting-velocity-collapse | Posting Velocity Collapse | negative | 65 | Job posting volume collapsed to <33% of prior | Hiring freeze confirmed via posting data |
| enrichment-pdl-growth | Headcount Growth (PDL) | varies | 15-65 | People Data Labs headcount growth/contraction data | LinkedIn-derived headcount trend |
| enrichment-pdl-high-churn | High Employee Churn (PDL) | negative | 55 | PDL churn rate >15% over 6 months | Significant employee turnover |
| enrichment-pdl-departures | Mass Departures (PDL) | critical | 80 | PDL departures >10% of workforce in 6 months | Mass exodus -- critical workforce stability issue |
| enrichment-pdl-low-tenure | Low Average Tenure (PDL) | neutral | 20 | Average employee tenure <1.5 years | Young workforce or high turnover |
| enrichment-sec-s1-filed | S-1 Filing (IPO/Direct Listing) | neutral | 35 | S-1 registration found in SEC EDGAR | IPO pursuit |
| enrichment-sec-restructuring | SEC 8-K Restructuring Filing | negative | 75 | SEC 8-K Item 2.05 filed | Formal restructuring activity |
| enrichment-sec-officer-change | SEC 8-K Officer Change | neutral | 25 | SEC 8-K Item 5.02 filed | Director/officer change |
| enrichment-sec-form-d | SEC Form D (Private Fundraising) | positive (flips to negative with layoffs) | 35 (55 if flipped) | Form D filing for non-public filer | Private fundraising -- but if paired with layoffs, indicates bridge/restructuring round |
| enrichment-sec-public-filer | SEC Public Filer | positive | 30 | Active 10-K/10-Q filer | Established public company |
| enrichment-sanctions-{source} | Sanctions/Enforcement Match | critical | 95 | Entity on sanctions/enforcement lists | Legal risk -- sanctions compliance issue |
| enrichment-watchlist-{source} | Watchlist Mention | negative | 40 | Entity appears in watchlist databases (non-sanctioned) | Compliance attention needed |
| enrichment-prior-layoff-pattern | Prior Layoff Pattern | critical/negative | 85 | 2+ distinct layoff events in 24 months (critical if 3+) | Serial cutter -- repeated layoff pattern |
| enrichment-funding-velocity | Funding Velocity | varies | 15-50 | Time since last funding event (positive <=12mo, negative >36mo) | Capital runway indicator |
| enrichment-valuation-down-round | Potential Down-Round | negative | 60 | Latest funding round raised >20% less than prior | Valuation markdown -- company struggling to raise at prior valuation |

---

## 4. Legal & Compliance (4 checks)

---

### Check: trust-center -- Trust & Compliance Center
**Cost Center:** Legal & Compliance
**Detection Method:** HTTP HEAD then GET to 5 trust/security page URLs: `trust.{domain}`, `security.{domain}`, `{domain}/trust`, `{domain}/security`, `{domain}/compliance`.

**How it works:**
Probes 5 trust/security page variants (2 subdomains + 3 paths) with HEAD for existence check, then GET to scan body for 12 compliance framework keywords: SOC 2, ISO 27001, ISO 27017, ISO 27018, HIPAA, GDPR, PCI DSS, FedRAMP, CCPA, CSA STAR, AICPA, and SOC Type I/II. Reads first 100KB of each page body.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| trust-center-comprehensive | Comprehensive Trust Center | positive | 80 | Trust page with 3+ compliance frameworks detected | Significant enterprise investment -- selling to regulated buyers |
| trust-center-basic | Basic Trust Center | positive | 60 | Trust page with 1-2 compliance frameworks | Enterprise-ready with basic compliance |
| trust-page-present | Trust Page Present | positive | 35 | Trust/security page exists but no compliance keywords detected | Security-conscious organization |
| no-trust-center | No Trust Center | neutral | 15 | No trust or security page found | May not target enterprise buyers |
| soc2-detected | SOC 2 Compliance | positive | 70 | SOC 2 or SOC Type I/II keyword detected | Enterprise sales-ready -- $50K-150K investment in SOC 2 audit |
| hipaa-detected | HIPAA Compliance | positive | 60 | HIPAA keyword detected | Healthcare enterprise vertical |

---

### Check: vendor-wallet -- Vendor Wallet
**Cost Center:** Legal & Compliance
**Detection Method:** DNS TXT lookup for `{domain}`, finds the SPF record (`v=spf1`), scans for vendor domain signatures.

**How it works:**
Resolves DNS TXT records and finds the SPF record (starts with `v=spf1`). Scans the SPF record for 14 known vendor signatures: ATS tools (Greenhouse, Lever, Workable, Ashby, Breezy HR), CRM (Salesforce, HubSpot), support (Zendesk, Freshdesk, Intercom), and infrastructure (Amazon SES, SendGrid, Mailgun, Mandrill).

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| vendor-spend-detected | Verified Vendor Spend | positive | 80 | 1+ known vendor signatures in SPF record | DNS-verified SaaS spending -- company actively uses paid tools |
| vendor-ats-verified | ATS Integration Verified | positive | 65 | ATS tool (Greenhouse, Lever, Workable, Ashby, Breezy) in SPF | Hiring tools verified via DNS -- company actively recruiting |
| vendor-no-major-tools | No Major Vendors in SPF | neutral | 30 | SPF record exists but no known vendor signatures | May use GSuite only or vendor detection missed |
| vendor-no-spf | No SPF Record | neutral | 10 | No SPF record found | Weak email security configuration |
| vendor-tools-removed | Vendor Tools Removed (Temporal) | negative | 50-70 | Vendors dropped from SPF since prior scan (70 if ATS removed) | Cost-cutting -- dropping paid tools, especially concerning if ATS dropped |
| vendor-tools-added | Vendor Tools Added (Temporal) | positive | 30 | New vendors added to SPF since prior scan (with no removals) | Company onboarding new tools -- investment signal |

---

### Check: h1b-lca -- H-1B / Foreign Labor
**Cost Center:** Legal & Compliance
**Detection Method:** HTTP GET to h1bdata.info, scrapes HTML table of certified LCA filings for current and prior year.

**How it works:**
Normalizes company name, queries h1bdata.info for the current year and prior year. Parses the HTML table using Cheerio to extract employer, job title, base salary, location, and submit date. Computes YoY filing count change, average salary, and top roles by frequency.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| h1b-active-filer | Active H-1B Filer | positive | 30-70 | Recent H-1B filings found (no strong YoY trend). Weight: 30 (<20), 50 (20-99), 70 (100+) | Active visa sponsorship -- importing specialized talent |
| h1b-declining | H-1B Filings Declining | negative | 60 | Filing volume dropped >30% YoY (with prior count >= 10) | Hiring contraction for specialized roles |
| h1b-growing | H-1B Filings Growing | positive | 50 | Filing volume grew >30% YoY (with recent count >= 10) | Expanding hiring for specialized roles |
| h1b-no-filings | No H-1B Filings | neutral | 5 | No filings found for the company | Company may not use H-1B program |
| h1b-data-unavailable | H-1B Data Unavailable | neutral | 5 | Error fetching data | Data source unavailable |

---

### Check: warn-notice -- WARN Notice Aggregator
**Cost Center:** Legal & Compliance
**Detection Method:** HTTP GET to California EDD and New York DOL WARN notice pages, scrapes HTML tables with Cheerio.

**How it works:**
Fetches California and New York WARN Act layoff notice filings in parallel. Parses HTML tables from both state government websites. Fuzzy-matches company names (case-insensitive, stripped of legal suffixes). Filters to last 180 days and reports the most recent match.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| warn-critical | WARN Act Layoff Filed | critical | 95 | WARN notice match within 90 days | Government-verified mass layoff -- highest confidence signal |
| warn-historical | Historical WARN Notice | negative | 70 | WARN notice match 91-180 days ago | Past mass layoff event -- may still impact company |
| warn-none | No WARN Notices | neutral | 5 | No matching WARN notices in CA or NY | No government layoff filings found |

---

## 5. HR & Hiring (9 checks)

---

### Check: job-forensics -- Job Forensics
**Cost Center:** HR & Hiring
**Detection Method:** Reads pre-scored `context.jobQualityData` (quality_score, ghost_score per listing from the pipeline scoring engine).

**How it works:**
Aggregates job-level quality and ghost scores across a company's recent job listings. Computes average quality score, average ghost score, ghost job prevalence (percentage of listings with ghost_score > 0.5), and a quality distribution (excellent/good/fair/poor tiers).

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| job-quality-excellent | Excellent Job Quality | positive | 55 | Avg quality score >= 0.75 | Well-written, specific job postings -- legitimate hiring intent |
| job-quality-good | Good Job Quality | positive | 35 | Avg quality score 0.55-0.74 | Decent job postings -- standard hiring |
| job-quality-poor | Poor Job Quality | negative | 55 | Avg quality score 0.30-0.54 | High boilerplate, low specificity -- questionable hiring intent |
| job-quality-critical | Critical Job Quality Failure | critical | 85 | Avg quality score < 0.30 | Potential ghost job farm -- extremely low-quality postings |
| job-ghost-prevalence-high | High Ghost Job Prevalence | critical | 80 | >= 70% of listings flagged as ghost (ghost_score > 0.5) | Company is posting mostly phantom listings |
| job-ghost-prevalence-moderate | Moderate Ghost Job Prevalence | negative | 60 | 50-69% of listings flagged as ghost | Concerning level of ghost job activity |
| job-quality-insufficient-data | Insufficient Job Data | neutral | 5 | Fewer than 3 jobs available | Not enough data to assess |

---

### Check: posting-velocity -- Posting Velocity Delta
**Cost Center:** HR & Hiring
**Detection Method:** Reads `context.jobListingsDetail` (first_seen_at/posted_at dates), buckets into 12 weekly cohorts.

**How it works:**
Buckets job listings by first_seen_at into 12 weekly cohorts. Computes recent posting rate (average per week, weeks 1-4) vs. baseline (average per week, weeks 5-12). The ratio of recent/baseline determines the signal.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| posting-velocity-collapse | Posting Velocity Collapse | critical | 85 | Recent posting rate < 33% of baseline | Hiring has effectively stopped -- possible hiring freeze or layoffs imminent |
| posting-velocity-decline | Posting Velocity Decline | negative | 60 | Recent posting rate < 66% of baseline | Hiring slowing down -- budget tightening |
| posting-velocity-surge | Posting Velocity Surge | positive | 45 | Recent posting rate > 200% of baseline | Active hiring expansion -- rapid growth or backfill |
| posting-velocity-stable | Posting Velocity Stable | positive | 30 | Rate within 66-200% of baseline | Normal hiring cadence |
| posting-velocity-insufficient-data | Insufficient Temporal Data | neutral | 5 | Fewer than 4 weeks of data | Not enough history to assess |

---

### Check: listing-flux -- Net Listing Flux
**Cost Center:** HR & Hiring
**Detection Method:** Reads `context.jobListingsDetail` current count + new listings (30d), compares against prior scan total from `context.priorCheckResults`.

**How it works:**
Counts current total listings and new listings (first_seen_at within 30 days). Compares against the prior scan's total count to estimate removals (prior_total + new_listings - current_total). Computes net flux and flux percentage.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| listing-flux-negative | Negative Listing Flux | critical | 80 | Net flux < -20% of prior total | Company removed significantly more jobs than it posted -- contraction |
| listing-flux-shrinking | Shrinking Job Board | negative | 55 | Net flux < 0 (but not < -20%) | More jobs removed than posted -- mild contraction |
| listing-flux-growing | Growing Job Board | positive | 30 | Net flux >= 0 | Board is growing -- active hiring |
| listing-flux-insufficient-data | Insufficient Flux Data | neutral | 5 | No prior listing count available | First scan -- cannot compute flux yet |

---

### Check: ghost-acceleration -- Ghost Score Acceleration
**Cost Center:** HR & Hiring
**Detection Method:** Reads `context.jobListingsDetail` (ghost_score per listing), compares average against prior scan from `context.priorCheckResults`.

**How it works:**
Filters listings with ghost scores, computes current average ghost score and percentage with ghost_score > 0.5. Compares against prior scan's average ghost score to compute delta (trend direction).

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| ghost-score-surging | Ghost Score Surging | critical | 75 | Avg ghost rose >0.15 AND >50% listings are high-ghost | Pre-layoff signal -- company posting phantom jobs to appear healthy |
| ghost-score-rising | Ghost Score Rising | negative | 50 | Avg ghost rose >0.05 | Listing quality degrading -- more ghost-like postings |
| ghost-score-improving | Ghost Score Improving | positive | 40 | Avg ghost dropped >0.05 | Listing quality improving -- more legitimate postings |
| ghost-acceleration-insufficient-data | Insufficient Ghost Data | neutral | 5 | <5 listings with ghost scores or no prior scan | Not enough data or history |

---

### Check: role-mix-shift -- Role Mix Shift
**Cost Center:** HR & Hiring
**Detection Method:** Reads `context.jobListingsDetail` (job titles), classifies into departments via keyword regex, compares mix against prior scan.

**How it works:**
Classifies each job title into one of 6 departments using keyword matching: engineering, HR, legal, finance, sales, or other. Computes the percentage mix for each department. Compares against prior scan's mix to detect shifts -- specifically watching for HR+legal+finance spikes combined with engineering drops (classic pre-layoff restructuring signal).

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| role-mix-restructuring | Role Mix Restructuring Signal | critical | 90 | HR+legal+finance >30% of listings AND engineering dropped >10pp | Classic pre-layoff restructuring -- hiring lawyers and HR while freezing engineering |
| role-mix-hr-spike | HR/Legal Hiring Spike | negative | 65 | HR+legal+finance hiring grew >10pp | Administrative hiring spike -- may indicate upcoming reorganization |
| role-mix-engineering-freeze | Engineering Hiring Freeze | negative | 70 | Engineering percentage dropped >15pp | Engineering hiring frozen -- product investment declining |
| role-mix-insufficient-data | Insufficient Role Data | neutral | 5 | Fewer than 5 listings | Not enough data to assess |

---

### Check: repost-churn -- Repost Churn Rate
**Cost Center:** HR & Hiring
**Detection Method:** Reads `context.jobListingsDetail` (titles, ats_created_at, posted_at), normalizes titles and computes duplicates + ATS recycling gap.

**How it works:**
Two complementary analyses: (A) Title-based churn: normalizes titles (strips seniority levels, location suffixes, parentheticals), groups by normalized title, computes percentage of unique titles that appear more than once. (B) ATS recycling gap: when ats_created_at is >90 days before posted_at, the listing was recycled from an old draft -- a "fake fresh" posting.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| repost-churn-extreme | Extreme Repost Churn | critical | 70 | Churn rate > 40% of unique titles reposted | Roles being repeatedly recycled -- pipeline theater, not real hiring |
| repost-churn-elevated | Elevated Repost Churn | negative | 45 | Churn rate > 20% | Some roles being recycled -- may indicate difficulty filling positions |
| repost-churn-normal | Normal Repost Rate | neutral | 10 | Churn rate < 20% | Within normal range |
| recycled-listings-high | High ATS Recycling | negative | 55 | >30% of listings recycled from ATS drafts >90 days old | Old requisitions republished as "new" -- evergreen reqs or phantom hiring |
| recycled-listings-moderate | Moderate ATS Recycling | negative | 30 | >15% of listings recycled | Some ATS recycling present |
| repost-churn-insufficient-data | Insufficient Listing Data | neutral | 5 | Fewer than 5 listings | Not enough data to assess |

---

### Check: description-staleness -- Description Entropy Drop
**Cost Center:** HR & Hiring
**Detection Method:** Reads `context.jobListingsDetail` (description_length, description_snippet), computes median length and Jaccard similarity across description pairs.

**How it works:**
Filters listings with descriptions, computes median description length. Compares length against prior scan for temporal trend. Measures content similarity via Jaccard word-set overlap on description snippets -- comparing sampled pairs to detect boilerplate reuse (when many descriptions are near-identical, content is templated).

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| description-going-dark | Descriptions Going Dark | critical | 65 | Median length dropped >50% OR >60% of pairs are near-identical | Boilerplate hiring or hiring freeze dressed up as active recruiting |
| description-degrading | Description Quality Degrading | negative | 40 | Median length dropped >25% OR >40% of pairs are near-identical | Descriptions getting shorter or more templated |
| description-healthy | Healthy Description Quality | positive | 15 | Varied descriptions, reasonable length | Well-written, unique descriptions -- legitimate hiring effort |
| description-staleness-insufficient-data | Insufficient Description Data | neutral | 5 | Fewer than 5 descriptions | Not enough data to assess |

---

### Check: salary-withdrawal -- Salary Disclosure Withdrawal
**Cost Center:** HR & Hiring
**Detection Method:** Reads `context.jobListingsDetail` (salary_min, salary_max), computes disclosure rate and compares against prior scan.

**How it works:**
Counts listings with salary information (salary_min or salary_max not null). Computes disclosure rate as a percentage. Compares against prior scan's disclosure rate to detect withdrawal trends.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| salary-withdrawal-severe | Severe Salary Withdrawal | critical | 60 | Disclosure rate dropped >20pp from prior scan | Company pulling back on transparency -- budget-tightening signal |
| salary-withdrawal-moderate | Moderate Salary Withdrawal | negative | 35 | Disclosure rate dropped >10pp | Salary transparency declining |
| salary-transparent | Salary Transparent | positive | 25 | Disclosure rate >50% | High salary transparency -- good employer brand signal |
| salary-withdrawal-insufficient-data | Insufficient Salary Data | neutral | 5 | Fewer than 5 listings | Not enough data to assess |

---

### Check: board-vitality -- Board Vitality
**Cost Center:** HR & Hiring
**Detection Method:** Reads `context.jobListingsDetail` (counts, first_seen_at/posted_at for age), compares against prior scan's count.

**How it works:**
Counts current total listings and stale listings (first_seen_at > 90 days old). Compares current count against prior scan's count to compute percentage change. Detects boards that are emptying out or going stale.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| board-going-dark | Board Going Dark | critical | 90 | Listing count dropped >70% from prior scan | Job board going dark -- company has stopped hiring or is shutting down |
| board-contracting | Board Contracting | negative | 55 | Listing count dropped >30% | Job board shrinking -- hiring pullback |
| board-stale | Stale Job Board | negative | 45 | >80% of listings are >90 days old (with 5+ listings) | Board may be abandoned -- listings left to rot |
| board-active | Active Job Board | positive | 25 | 5+ listings, <50% stale, not contracting >30% | Healthy job board -- active hiring |
| board-vitality-insufficient-data | No Board Data | neutral | 5 | No current listings and no prior scan | Cannot assess yet |

---

## 6. Finance & RevOps (1 check)

---

### Check: finance-infrastructure -- Finance & Billing Infrastructure
**Cost Center:** Finance & RevOps
**Detection Method:** HTTP GET to homepage (reads CSP header + first 300KB HTML), scans for financial vendor signatures.

**How it works:**
Fetches the homepage, reads the Content-Security-Policy header and HTML body. Scans both for 20+ financial vendor patterns across 4 tiers: enterprise billing (Zuora, Chargebee, Recurly, Maxio), enterprise payments (Adyen, Braintree, Worldpay, Checkout.com), growth payments (Stripe, PayPal, Square), startup payments (Paddle, Lemon Squeezy, Gumroad), banking APIs (Plaid, MX, Finicity, Yodlee), and revenue operations (Dealroom, Clari).

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| finance-enterprise-billing | Enterprise Billing Platform | positive | 75 | Zuora, Chargebee, Recurly, or Maxio detected | Multi-currency subscription management -- mature revenue operations |
| finance-banking-api | Bank API Integration | positive | 70 | Plaid, MX, Finicity, or Yodlee detected | Fintech or financial services operation |
| finance-enterprise-payments | Enterprise Payment Processing | positive | 60 | Adyen, Braintree, Worldpay, or Checkout.com detected | High-volume transaction infrastructure |
| finance-growth-payments | Growth Payment Processing | positive | 40 | Stripe, PayPal, or Square detected | Growth-stage payment processing |
| finance-startup-payments | Startup Payment Processing | neutral | 20 | Paddle, Lemon Squeezy, or Gumroad detected | Early-stage monetization |

---

## 7. Telecom & Remote Work (1 check)

---

### Check: telecom-srv -- Enterprise Telecom (SRV Records)
**Cost Center:** Telecom & Remote Work
**Detection Method:** DNS SRV queries via Google DNS-over-HTTPS for 5 VoIP/SIP service records: `_sip._tls`, `_sip._tcp`, `_sip._udp`, `_sipfederationtls._tcp`, `_h323cs._tcp`.

**How it works:**
Queries 5 DNS SRV record types in parallel via Google DNS-over-HTTPS. Parses SRV record targets (priority, weight, port, target hostname) and classifies against 12 telecom provider patterns: enterprise UC (Microsoft Teams, Zoom Phone, RingCentral, Cisco Webex, Vonage, 8x8, Genesys), growth (Dialpad, Twilio, Bandwidth), and basic (Asterisk/FreePBX, 3CX).

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| telecom-enterprise-uc | Enterprise UC Platform | positive | 70 | Enterprise provider (Teams, Zoom, RingCentral, Webex, Vonage, 8x8, Genesys) detected | Significant telecom contract -- distributed workforce infrastructure |
| telecom-teams-federation | Microsoft Teams SIP Federation | positive | 55 | `_sipfederationtls._tcp` SRV record resolves | Enterprise Teams deployment with federation -- serious Microsoft investment |
| telecom-growth | Growth-Stage Telecom | positive | 40 | Growth provider (Dialpad, Twilio, Bandwidth) detected | Modern telecom -- growth-stage communication infrastructure |
| telecom-basic | Basic PBX System | neutral | 15 | Basic provider (Asterisk/FreePBX, 3CX) detected | Legacy or self-hosted PBX |
| telecom-rich-infra | Rich Telecom Infrastructure | positive | 50 | 3+ SRV records across multiple protocols | Multiple telecom protocols -- complex communication needs |

---

## 8. Corporate Development (1 check)

---

### Check: ma-detection -- M&A Infrastructure Detection
**Cost Center:** Corporate Development
**Detection Method:** DNS MX + NS queries via Google DNS-over-HTTPS, compares hostnames against known enterprise parent company infrastructure.

**How it works:**
Queries MX (type 15) and NS (type 2) records via Google DNS-over-HTTPS. Matches all returned hostnames against 26 enterprise signatures in 3 categories: big tech (Google, Microsoft, Amazon/AWS, Apple), enterprise SaaS (Salesforce, Cisco, Oracle, SAP, IBM, Broadcom/VMware, Adobe, HubSpot, Twilio, Atlassian, Intuit, ServiceNow), and hosting/DNS (Cloudflare, AWS Route53, Azure DNS, GoDaddy, Namecheap, NS1). Hosting signatures are excluded from M&A detection. If the same non-hosting parent controls both MX and NS, flags a possible acquisition.

**Signals:**

| Signal ID | Name | Sentiment | Weight | Trigger | Business Meaning |
|-----------|------|-----------|--------|---------|-----------------|
| ma-possible-acquisition | Possible Acquisition Detected | neutral | 70 | Same non-hosting parent controls both MX and NS | Infrastructure suggests corporate absorption -- possible acquisition |
| ma-independent | Independent Infrastructure | positive | 40 | No enterprise parent on either MX or NS | Company manages its own infrastructure -- truly independent |
| ma-mx-migration | Email Infrastructure Migration (Temporal) | neutral | 60 | MX parent changed from prior scan | Email infrastructure moved -- possible acquisition or reorganization |
| ma-ns-migration | DNS Infrastructure Migration (Temporal) | neutral | 55 | NS parent changed from prior scan | DNS infrastructure moved -- possible acquisition or migration |

---

## Summary Statistics

- **Total checks:** 33
- **Total unique signal IDs:** ~155+ (including enrichment-signals dynamic IDs)
- **Cost centers:** 8 (IT & Infrastructure, Engineering & Product, Sales & Marketing, Legal & Compliance, HR & Hiring, Finance & RevOps, Telecom & Remote Work, Corporate Development)
- **Detection methods used:** HTTP HEAD, HTTP GET, DNS TXT, DNS MX, DNS NS, DNS SRV, DNS A (via Google DoH), iTunes Search API, NPM Registry API, h1bdata.info scraping, CA/NY WARN pages scraping, pre-computed pipeline data
- **Temporal signals:** 18 (signals that compare current scan against prior scan to detect changes over time)