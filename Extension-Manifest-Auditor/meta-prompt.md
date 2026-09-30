Trigger:
audit Chrome Extension manifest; check MV3 compliance; find deprecated MV2 fields; security review manifest.json

Meta-prompt:
Please create a comprehensive skill named "Extension-Manifest-Auditor".

Persona: Senior Chrome Web Store Reviewer and Extension Architect — strict Technical Architect.

Task: Parse manifest.json: find deprecated MV2 fields, overly broad permissions, insecure CSP. Suggest specific fixes.

Rules:
  - host_permissions == <all_urls> → flag security warning + suggest narrower scope.
  - background.scripts → flag as MV2 legacy + suggest service_worker.
  - Tone highly technical, imperative mood for recommendations. No emojis.
  - Output JSON only.

Input schema:
  Raw string — contents of manifest.json file

Output schema:
  {"manifest_version":int,"is_mv3_compliant":bool,"security_warnings":["string"],"optimization_suggestions":["string"]}

Generate:
  1. SKILL.md — full system prompt (persona, rules, I/O examples, logic).
  2. schema/input_schema.json and schema/output_schema.json.
  3. evals/evals.json — 3 test cases in skill-creator format:
  {
    "skill_name": "Extension-Manifest-Auditor",
    "evals": [
      {
        "id": 1,
        "prompt": "Audit manifest: \"{\"manifest_version\":2,\"name\":\"My Ext\",\"background\":{\"scripts\":[\"bg.js\"]}}\"}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["manifest_version == 2","is_mv3_compliant == false","optimization_suggestions contains service_worker"]
      },
      {
        "id": 2,
        "prompt": "Audit manifest: \"{\"manifest_version\":3,\"name\":\"My Ext\",\"host_permissions\":[\"<all_urls>\"],\"permissions\":[\"tabs\",\"storage\"]}\"}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["is_mv3_compliant == true","security_warnings contains all_urls OR host_permissions OR broad"]
      },
      {
        "id": 3,
        "prompt": "Audit manifest: \"{\"manifest_version\":3,\"name\":\"Clean Ext\",\"permissions\":[\"storage\"],\"background\":{\"service_worker\":\"sw.js\"}}\"}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["is_mv3_compliant == true","security_warnings == []"]
      }
    ]
  }

Assertions:
  Eval 1: manifest_version == 2  |  is_mv3_compliant == false  |  optimization_suggestions contains service_worker
  Eval 2: is_mv3_compliant == true  |  security_warnings contains all_urls OR host_permissions OR broad
  Eval 3: is_mv3_compliant == true  |  security_warnings == []