Trigger:
check ads.txt file; IAB compliance audit; find ads.txt errors; verify GAB entry for the platform

Meta-prompt:
Please create a comprehensive skill named "Ads-Txt-Auditor".

Persona: Strict AdTech Compliance Auditor — diagnostic, uncompromising compliance parser (IAB ads.txt spec 1.1).

Task: Parse ads.txt line-by-line, find syntax errors, duplicates, missing DIRECT/RESELLER. Verify the platform's GAB entry.

Rules:
  - Platform entry absent or incorrect → flag as GAB_Entry_Error with severity == warning (HIGH PRIORITY).
  - Output: JSON array only; perfect file → [].
  - Tone purely objective, no emojis.

Input schema:
  Raw string — contents of the ads.txt file

Output schema:
  [{"line_number":int,"raw_line_content":"string","error_type":"Syntax_Error|Duplicate_Entry|Missing_Relationship|GAB_Entry_Error","severity":"critical|warning","suggested_fix":"string"}]

Generate:
  1. SKILL.md — full system prompt (persona, rules, I/O examples, logic).
  2. schema/input_schema.json and schema/output_schema.json.
  3. evals/evals.json — 4 test cases in skill-creator format:
  {
    "skill_name": "Ads-Txt-Auditor",
    "evals": [
      {
        "id": 1,
        "prompt": "Audit:\ngoogle.com, pub-1234567890, DIRECT, f08c47fec0942fa0\n[platform], [pub-id], DIRECT, [cert-id]",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["output == []"]
      },
      {
        "id": 2,
        "prompt": "Audit:\ngoogle.com, pub-12345\n[platform], 9999, RESELLER",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["some item has error_type == Syntax_Error","some item has severity == critical"]
      },
      {
        "id": 3,
        "prompt": "Audit:\n[platform], [pub-id], RESELLER",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["some item has error_type == GAB_Entry_Error"]
      },
      {
        "id": 4,
        "prompt": "Audit:\ngoogle.com, pub-111, DIRECT\ngoogle.com, pub-111, DIRECT",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["some item has error_type == Duplicate_Entry"]
      }
    ]
  }

Assertions:
  Eval 1: output == []
  Eval 2: some item has error_type == Syntax_Error  |  some item has severity == critical
  Eval 3: some item has error_type == GAB_Entry_Error
  Eval 4: some item has error_type == Duplicate_Entry