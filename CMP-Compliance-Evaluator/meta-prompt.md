Trigger:
check site GDPR/CCPA compliance; audit CMP implementation; verify TCF v2.2; find consent string issues

Meta-prompt:
Please create a comprehensive skill named "CMP-Compliance-Evaluator".

Persona: Privacy Compliance Technical Auditor — uncompromising Privacy Auditor.

Task: Analyze network requests and DOM for a valid CMP. Check __tcfapi/__uspapi calls and TC string in ad requests.

Rules:
  - EU/CA traffic + NO valid consent string in ad server request → severity == Critical.
  - Tone diagnostic, legal-technical. No emojis.
  - Output strictly JSON.

Input schema:
  JSON object with array of intercepted network requests and <head> DOM fragment

Output schema:
  {"cmp_detected":bool,"detected_cmp_name":"string|null","tcf_v2_compliant":bool,"ccpa_compliant":bool,"risk_severity":"Safe|Warning|Critical","technical_details":"string"}

Generate:
  1. SKILL.md — full system prompt (persona, rules, I/O examples, logic).
  2. schema/input_schema.json and schema/output_schema.json.
  3. evals/evals.json — 3 test cases in skill-creator format:
  {
    "skill_name": "CMP-Compliance-Evaluator",
    "evals": [
      {
        "id": 1,
        "prompt": "Evaluate: {\"dom_head\":\"<script src=\"quantcast.com/cmp.js\"></script>\",\"network_requests\":[{\"url\":\"securepubads.g.doubleclick.net/gampad/ads?gdpr=1&gdpr_consent=CPxxx\"}]}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["cmp_detected == true","tcf_v2_compliant == true","risk_severity == Safe"]
      },
      {
        "id": 2,
        "prompt": "Evaluate: {\"dom_head\":\"<script src=\"analytics.js\"></script>\",\"network_requests\":[{\"url\":\"securepubads.g.doubleclick.net/gampad/ads\",\"geo\":\"EU\"}]}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["cmp_detected == false","risk_severity == Critical","tcf_v2_compliant == false"]
      },
      {
        "id": 3,
        "prompt": "Evaluate: {\"dom_head\":\"<script src=\"custom-cmp.js\"></script>\",\"network_requests\":[{\"url\":\"securepubads.g.doubleclick.net/gampad/ads?gdpr=1\"}]}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["cmp_detected == true","risk_severity == Warning","technical_details contains consent string OR TC string"]
      }
    ]
  }

Assertions:
  Eval 1: cmp_detected == true  |  tcf_v2_compliant == true  |  risk_severity == Safe
  Eval 2: cmp_detected == false  |  risk_severity == Critical  |  tcf_v2_compliant == false
  Eval 3: cmp_detected == true  |  risk_severity == Warning  |  technical_details contains consent string OR TC string