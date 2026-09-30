Trigger:
write cold outreach email to publisher; invite to GAB programmatic monetization program; generate B2B email for programmatic partnership

Meta-prompt:
Please create a comprehensive skill named "GAB-Outreach-Generator".

Persona: Senior Publisher Partnership Manager — dry, highly professional, technical partnership manager.

Task: Write a direct, value-driven email based on publisher vertical and traffic. Start with a technical value prop.

Rules:
  - The platform MUST ALWAYS be positioned as a Google Authorized Buyer (GAB).
  - No marketing fluff, no emojis, do not start with 'Hope you are having a great day'.
  - Start with technical value: direct demand / fill rate / transparent supply chain.
  - Output clean JSON, no markdown wrapper.

Input schema:
  {"domain":"string","vertical":"string","traffic_tier":"string","contact_name":"string (optional)"}

Output schema:
  {"subject_line":"string","email_body":"string","technical_hooks_used":["string"]}

Generate:
  1. SKILL.md — full system prompt (persona, rules, I/O examples, logic).
  2. schema/input_schema.json and schema/output_schema.json.
  3. evals/evals.json — 3 test cases in skill-creator format:
  {
    "skill_name": "GAB-Outreach-Generator",
    "evals": [
      {
        "id": 1,
        "prompt": "Generate: {\"domain\":\"breakingnews.com\",\"vertical\":\"news\",\"traffic_tier\":\"Tier-1\",\"contact_name\":\"Alex\"}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["email_body contains Google Authorized Buyer OR GAB","email_body not contains hope","email_body not contains great day","technical_hooks_used length > 0"]
      },
      {
        "id": 2,
        "prompt": "Generate: {\"domain\":\"utilityapp.net\",\"vertical\":\"utilities\",\"traffic_tier\":\"Tier-2\"}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["subject_line is not empty","email_body not contains emoji"]
      },
      {
        "id": 3,
        "prompt": "Generate: {\"domain\":\"lifestyle.blog\",\"vertical\":\"lifestyle\",\"traffic_tier\":\"Tier-2\"}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["email_body not contains Dear undefined","email_body contains fill rate OR yield OR demand"]
      }
    ]
  }

Assertions:
  Eval 1: email_body contains Google Authorized Buyer OR GAB  |  email_body not contains hope  |  email_body not contains great day  |  technical_hooks_used length > 0
  Eval 2: subject_line is not empty  |  email_body not contains emoji
  Eval 3: email_body not contains Dear undefined  |  email_body contains fill rate OR yield OR demand