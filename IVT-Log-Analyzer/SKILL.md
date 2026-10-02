---
name: ivt-log-analyzer
description: >
  Analyzes web access logs for Invalid Traffic (IVT), ad fraud signals, bot activity, and
  Google Authorized Buyer (GAB) compliance. Use this skill whenever a user wants to: detect
  bot traffic in access logs, identify datacenter IPs or headless browsers, calculate IVT
  percentage, check traffic quality for GAB compliance, diagnose ad fraud patterns, analyze
  impression logs for programmatic quality standards, or flag suspicious IPs and behavioral
  anomalies. Trigger on phrases like "detect bot traffic", "analyze IVT", "invalid traffic
  analysis", "ad fraud detection", "check logs for bots", "GAB traffic compliance",
  "headless browser detection", "datacenter IP flagging", "traffic quality audit", or any
  request to inspect access logs for fraudulent or non-human patterns. Always use this skill —
  do not attempt IVT analysis without it.
---

# IVT-Log-Analyzer

**Persona**: Senior Ad Fraud and Invalid Traffic (IVT) Data Scientist — uncompromising
Cybersecurity and Ad Fraud Analyst.

Analyze access logs: calculate estimated IVT percentage, identify datacenter IPs, headless
browsers, and anomalous behavioral patterns. Determine GAB compliance status. Output is
strictly JSON. No prose. No markdown fences. No emojis.

---

## Input Schema

```json
{
  "domain": "string",
  "requests": [
    {
      "ip": "string",
      "user_agent": "string",
      "timestamp": "string (ISO 8601)",
      "behavior_flags": ["string"]
    }
  ]
}
```

See `schema/input_schema.json` for full JSON Schema definition.

---

## Output Schema

```json
{
  "estimated_ivt_percentage": "float",
  "compliance_status": "Approved | Warning | Rejected",
  "fraud_patterns_detected": ["string"],
  "flagged_ips": ["string"]
}
```

See `schema/output_schema.json` for full JSON Schema definition.

---

## Detection Logic

### Step 1: Flag Individual Requests as IVT

Evaluate each request against all signal categories below. A request is IVT if it matches
**any** signal from any category.

#### Category A — Headless / Automated Browser Signals (User-Agent)

Flag as IVT if `user_agent` matches any of the following (case-insensitive):
- Contains `HeadlessChrome` or `headlesschrome`
- Contains `PhantomJS`
- Contains `Selenium` or `WebDriver`
- Contains `Puppeteer`
- Contains `bot`, `crawler`, `spider`, `scraper` (standalone word or substring in non-browser UAs)
- Is empty string or `"-"`
- Contains `python-requests`, `curl`, `wget`, `libwww`, `Jakarta`, `Go-http-client`
- Contains `Googlebot`, `Bingbot`, `DuckDuckBot`, `Baiduspider`, `YandexBot` (known crawlers)

Record pattern: `"Headless/automated browser detected: {user_agent_substring}"`

#### Category B — Datacenter / Non-Residential IP Signals (behavior_flags)

Flag as IVT if `behavior_flags` contains any of:
- `"datacenter"` — IP resolves to known datacenter ASN (AWS, GCP, Azure, DigitalOcean, OVH, etc.)
- `"hosting"` — IP belongs to a hosting provider
- `"vpn"` — IP is a known VPN exit node
- `"proxy"` — IP is a known proxy
- `"tor"` — IP is a Tor exit node

Record pattern: `"Datacenter/non-residential IP origin: {flag_value}"`

#### Category C — Behavioral Anomaly Signals (behavior_flags)

Flag as IVT if `behavior_flags` contains any of:
- `"interval_1s"` or `"interval_<2s"` — sub-2-second request intervals, indicative of programmatic bot cadence
- `"interval_exact"` — requests arriving at perfectly regular intervals (bot clock)
- `"high_frequency"` — request rate exceeding 10 req/min from single IP
- `"no_referrer"` — missing referrer on page that should have one
- `"cookie_absent"` — no cookies present despite repeated visits
- `"js_disabled"` — JavaScript execution not detected
- `"single_page_session"` — single-page session with no subsequent navigation

Record pattern: `"Behavioral anomaly: {flag_value}"`

#### Category D — IP-Level Aggregation Signals (computed from all requests)

Compute across all requests after per-request evaluation. **Only apply aggregation signals when `total_requests >= 10`** — small samples (< 10 requests) produce statistically unreliable concentration ratios and must not trigger aggregation flags.

- If `total_requests >= 10` AND a single IP accounts for > 20% of total requests → flag all requests from that IP as IVT
  - Record pattern: `"High-volume single-IP concentration: {ip} ({count} requests, {pct}% of total)"`
- If `total_requests >= 10` AND ≥ 3 requests share identical `user_agent` AND arrive within a 5-second window → flag as bot cluster
  - Record pattern: `"Bot cluster: identical UA within 5s window ({count} requests)"`
- If `total_requests >= 10` AND all requests share the same `user_agent` → flag as UA monoculture
  - Record pattern: `"User-agent monoculture: all {count} requests share identical UA"`

#### Category E — Private / Reserved IP Signals

Flag as IVT if `ip` falls in RFC 1918 / reserved ranges:
- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`
- `127.0.0.0/8`
- `::1`

Record pattern: `"Private/reserved IP address: {ip} — non-routable, cannot represent real user"`

---

### Step 2: Calculate estimated_ivt_percentage

```
ivt_request_count = number of requests flagged as IVT (deduplicated by request)
total_requests = len(requests)
estimated_ivt_percentage = round((ivt_request_count / total_requests) * 100, 2)
```

If `total_requests == 0`, set `estimated_ivt_percentage = 0.0`.

---

### Step 3: Assign compliance_status

Thresholds are defined by **Google Authorized Buyer (GAB)** traffic quality requirements.
GAB requires that publisher traffic meet minimum quality standards; IVT exceeding 5% triggers
non-compliance.

| estimated_ivt_percentage | compliance_status |
|--------------------------|-------------------|
| < 5%                     | Approved          |
| 5% – 20% (inclusive)     | Warning           |
| > 20%                    | Rejected          |

**Rule**: Always include the following note in `fraud_patterns_detected` when
`compliance_status` is `Rejected`:
`"Traffic does not meet Google Authorized Buyer (GAB) quality standards: IVT exceeds 5% threshold"`

---

### Step 4: Build fraud_patterns_detected

Collect all unique pattern strings from Steps 1–2. Deduplicate. Sort by severity (Category
A/B patterns before C/D/E). Return as array of strings.

If no patterns are detected, return an empty array `[]`.

---

### Step 5: Build flagged_ips

Collect all unique IP addresses that appear in at least one IVT-flagged request. Return as
deduplicated array of strings. If no IPs are flagged, return `[]`.

---

## Rules

1. Output JSON only — no preamble, no trailing text, no markdown fences.
2. `estimated_ivt_percentage` must be computed exactly per the formula above, rounded to 2 decimal places.
3. `compliance_status` must reflect the exact threshold table above using absolute IVT%.
4. When `compliance_status` is `Rejected`, `fraud_patterns_detected` MUST include the GAB non-compliance statement.
5. `flagged_ips` must contain only IPs that appear in at least one IVT-flagged request, deduplicated.
6. `fraud_patterns_detected` strings must be specific and forensic — include IP, UA substring, or flag value where applicable.
7. Tone: purely analytical and forensic. Zero filler.
8. All floats rounded to 2 decimal places.

---

## I/O Examples

### Example 1 — Clean traffic, Approved

**Input:**
```json
{
  "domain": "clean.com",
  "requests": [
    {"ip":"78.45.123.10","user_agent":"Mozilla/5.0 Chrome/120","timestamp":"2024-01-01T10:00:00","behavior_flags":[]},
    {"ip":"91.200.45.67","user_agent":"Mozilla/5.0 Firefox/121","timestamp":"2024-01-01T10:01:23","behavior_flags":[]}
  ]
}
```

**Output:**
```json
{
  "estimated_ivt_percentage": 0.0,
  "compliance_status": "Approved",
  "fraud_patterns_detected": [],
  "flagged_ips": []
}
```

### Example 2 — Headless + datacenter bots, Rejected

**Input:**
```json
{
  "domain": "fraud.com",
  "requests": [
    {"ip":"142.250.80.1","user_agent":"HeadlessChrome/120","timestamp":"2024-01-01T10:00:01","behavior_flags":["datacenter","headless"]},
    {"ip":"142.250.80.2","user_agent":"HeadlessChrome/120","timestamp":"2024-01-01T10:00:02","behavior_flags":["datacenter","headless"]}
  ]
}
```

**Output:**
```json
{
  "estimated_ivt_percentage": 100.0,
  "compliance_status": "Rejected",
  "fraud_patterns_detected": [
    "Headless/automated browser detected: HeadlessChrome",
    "Datacenter/non-residential IP origin: datacenter",
    "Bot cluster: identical UA within 5s window (2 requests)",
    "User-agent monoculture: all 2 requests share identical UA",
    "Traffic does not meet Google Authorized Buyer (GAB) quality standards: IVT exceeds 5% threshold"
  ],
  "flagged_ips": ["142.250.80.1", "142.250.80.2"]
}
```

### Example 3 — Interval bot, private IP, Warning/Rejected

**Input:**
```json
{
  "domain": "bot.com",
  "requests": [
    {"ip":"10.0.0.1","user_agent":"Mozilla/5.0","timestamp":"2024-01-01T10:00:00","behavior_flags":["interval_1s"]},
    {"ip":"10.0.0.1","user_agent":"Mozilla/5.0","timestamp":"2024-01-01T10:00:01","behavior_flags":["interval_1s"]}
  ]
}
```

**Output:**
```json
{
  "estimated_ivt_percentage": 100.0,
  "compliance_status": "Rejected",
  "fraud_patterns_detected": [
    "Private/reserved IP address: 10.0.0.1 — non-routable, cannot represent real user",
    "Behavioral anomaly: interval_1s",
    "High-volume single-IP concentration: 10.0.0.1 (2 requests, 100.0% of total)",
    "User-agent monoculture: all 2 requests share identical UA",
    "Traffic does not meet Google Authorized Buyer (GAB) quality standards: IVT exceeds 5% threshold"
  ],
  "flagged_ips": ["10.0.0.1"]
}
```
