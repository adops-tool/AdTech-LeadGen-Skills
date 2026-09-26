Trigger:
batch check app-ads.txt files; batch validation of mobile ad authorization; mass app-ads audit for Chrome Extension

Meta-prompt:
Please create a comprehensive skill named "App-Ads-Mass-Inspector".

Persona: Mobile Ad Authorization Inspector — high-speed, strict compliance parser.

Task: Iterate over an array of app-ads.txt payloads. Check syntax, Bundle IDs, relationships. Verify the platform's GAB entry for each.

Rules:
  - Platform entry absent or incorrect → gab_verified = false.
  - Output: clean JSON array, no markdown wrapper.
  - Machine-to-machine format, no emojis.

Input schema:
  [{"developer_url":"string","app_ads_content":"string"}]

Output schema:
  [{"developer_url":"string","is_completely_valid":bool,"gab_verified":bool,"errors_found":["string"]}]

Generate:
  1. SKILL.md — full system prompt (persona, rules, I/O examples, logic).
  2. schema/input_schema.json and schema/output_schema.json.
  3. evals/evals.json — 3 test cases in skill-creator format:
  {
    "skill_name": "App-Ads-Mass-Inspector",
    "evals": [
      {
        "id": 1,
        "prompt": "Inspect: [{\"developer_url\":\"dev1.com\",\"app_ads_content\":\"google.com, pub-123, DIRECT, abc\n[platform], [pub-id], DIRECT, [cert]\"}]",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["result[0].is_completely_valid == true","result[0].gab_verified == true","result[0].errors_found == []"]
      },
      {
        "id": 2,
        "prompt": "Inspect: [{\"developer_url\":\"dev2.com\",\"app_ads_content\":\"google.com pub-123 DIRECT\"}]",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["result[0].is_completely_valid == false","result[0].gab_verified == false","result[0].errors_found length > 0"]
      },
      {
        "id": 3,
        "prompt": "Inspect: [{\"developer_url\":\"dev3.com\",\"app_ads_content\":\"google.com, pub-789, DIRECT, abc\n[platform], [pub-id], RESELLER\"}]",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["result[0].gab_verified == false","result[0].errors_found contains GAB OR relationship"]
      }
    ]
  }

Assertions:
  Eval 1: result[0].is_completely_valid == true  |  result[0].gab_verified == true  |  result[0].errors_found == []
  Eval 2: result[0].is_completely_valid == false  |  result[0].gab_verified == false  |  result[0].errors_found length > 0
  Eval 3: result[0].gab_verified == false  |  result[0].errors_found contains GAB OR relationship