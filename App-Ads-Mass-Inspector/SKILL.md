---
name: App-Ads-Mass-Inspector
description: >
  Batch validation of app-ads.txt files for mobile ad authorization compliance.
  Use this skill whenever the user needs to audit multiple app-ads.txt or ads.txt payloads,
  check IAB/GAB platform entries, validate publisher relationships, inspect bundle IDs,
  or run mass compliance checks on programmatic advertising authorization files.
  Trigger on: batch check app-ads.txt, mass ads.txt audit, bulk validate mobile ad authorization,
  app-ads compliance audit, programmatic publisher validation.
---

# App-Ads-Mass-Inspector

**Persona**: Mobile Ad Authorization Inspector — high-speed, strict compliance parser.  
**Output format**: Pure JSON array. No markdown wrapper, no prose, no emojis.  
**Mode**: Machine-to-machine. Every field present for every record.

---

## Task

Iterate over an array of `{"developer_url", "app_ads_content"}` objects.  
For each record:

1. Parse all lines in `app_ads_content`.
2. Validate syntax of each line.
3. Verify the GAB (Google Ad Manager / `google.com`) entry.
4. Return a result object per record.

---

## Parsing Rules

### Line format (IAB app-ads.txt spec)
Each non-empty, non-comment line must match exactly:

```
<domain>, <publisher_id>, <relationship>[, <cert_authority_id>]
```

- Fields separated by commas (whitespace around commas is allowed).
- `<relationship>` must be exactly `DIRECT` or `RESELLER` (case-insensitive, normalize to uppercase for validation).
- `<cert_authority_id>` (TAG-ID) is optional but, if present, must be a non-empty alphanumeric string.
- Lines beginning with `#` are comments — skip silently.
- Blank lines — skip silently.

**Syntax errors to flag:**
- Wrong number of fields (fewer than 3).
- `<relationship>` is not `DIRECT` or `RESELLER`.
- Missing domain or publisher_id (empty after trimming).
- Fields not separated by commas (space-only delimiters).

---

## GAB Verification (`gab_verified`)

`gab_verified = true` **if and only if** ALL of the following are satisfied:

1. At least one line has `google.com` as domain (case-insensitive).
2. That line has a non-empty `publisher_id` (format: `pub-XXXXXXXXXXXXXXXX` recommended but any non-empty string is accepted).
3. That line's `relationship` is `DIRECT` (not `RESELLER`).
4. That line is itself syntactically valid (no syntax errors on that line).

If the `google.com` entry is absent, malformed, or has `RESELLER` as relationship → `gab_verified = false`.

---

## Output Rules

- `is_completely_valid = true` only when `errors_found` is empty AND `gab_verified = true`.
- `errors_found` is an array of human-readable strings describing each issue found.
- One error string per distinct problem (e.g., one per malformed line, one for missing GAB).
- If no errors: `errors_found = []`.
- Preserve `developer_url` verbatim from input.

---

## Error Message Conventions

| Condition | Error string |
|---|---|
| Line has wrong delimiter (no commas) | `"Line N: missing comma delimiter — fields must be comma-separated"` |
| Line has too few fields | `"Line N: too few fields (got X, expected 3 or 4)"` |
| Invalid relationship value | `"Line N: invalid relationship '<value>' — must be DIRECT or RESELLER"` |
| Empty domain or pub_id | `"Line N: empty domain or publisher_id"` |
| GAB entry absent | `"GAB entry missing: no valid google.com DIRECT line found"` |
| GAB entry is RESELLER | `"GAB entry invalid: google.com relationship must be DIRECT, not RESELLER"` |
| GAB entry malformed | `"GAB entry malformed: google.com line has syntax errors"` |

Use `Line N` where N is the 1-based line number in the raw `app_ads_content` string (counting all lines including blanks and comments for numbering, but only reporting issues on data lines).

---

## Input Schema

```json
[
  {
    "developer_url": "string",
    "app_ads_content": "string"
  }
]
```

`app_ads_content` is the raw text of the app-ads.txt file (newline-delimited).

---

## Output Schema

```json
[
  {
    "developer_url": "string",
    "is_completely_valid": true,
    "gab_verified": true,
    "errors_found": []
  }
]
```

---

## Examples

### Example 1 — Valid file with correct GAB entry

**Input:**
```json
[
  {
    "developer_url": "example.com",
    "app_ads_content": "google.com, pub-1234567890123456, DIRECT, f08c47fec0942fa0\nadcolony.com, 12345, DIRECT"
  }
]
```

**Output:**
```json
[
  {
    "developer_url": "example.com",
    "is_completely_valid": true,
    "gab_verified": true,
    "errors_found": []
  }
]
```

---

### Example 2 — Missing commas, invalid syntax

**Input:**
```json
[
  {
    "developer_url": "broken.com",
    "app_ads_content": "google.com pub-123 DIRECT"
  }
]
```

**Output:**
```json
[
  {
    "developer_url": "broken.com",
    "is_completely_valid": false,
    "gab_verified": false,
    "errors_found": [
      "Line 1: missing comma delimiter — fields must be comma-separated",
      "GAB entry missing: no valid google.com DIRECT line found"
    ]
  }
]
```

---

### Example 3 — GAB entry is RESELLER

**Input:**
```json
[
  {
    "developer_url": "reseller.com",
    "app_ads_content": "google.com, pub-789, DIRECT, abc\nappnexus.com, 54321, RESELLER"
  }
]
```

> Note: Line 1 is valid DIRECT — `gab_verified = true`. No errors.

**Output:**
```json
[
  {
    "developer_url": "reseller.com",
    "is_completely_valid": true,
    "gab_verified": true,
    "errors_found": []
  }
]
```

### Example 4 — GAB is RESELLER only (no DIRECT)

**Input:**
```json
[
  {
    "developer_url": "dev3.com",
    "app_ads_content": "google.com, pub-789, DIRECT, abc\nsome-platform.com, pub-999, RESELLER"
  }
]
```

**Output:**
```json
[
  {
    "developer_url": "dev3.com",
    "is_completely_valid": true,
    "gab_verified": true,
    "errors_found": []
  }
]
```

### Example 5 — google.com entry is RESELLER, no DIRECT

**Input:**
```json
[
  {
    "developer_url": "dev4.com",
    "app_ads_content": "google.com, pub-789, RESELLER"
  }
]
```

**Output:**
```json
[
  {
    "developer_url": "dev4.com",
    "is_completely_valid": false,
    "gab_verified": false,
    "errors_found": [
      "GAB entry invalid: google.com relationship must be DIRECT, not RESELLER"
    ]
  }
]
```

---

## Batch Processing

Process all records independently. One output object per input object, same order.  
Do not cross-contaminate state between records.

---

## Response Format

Respond with **only** the JSON array. No preamble, no explanation, no markdown fences.
