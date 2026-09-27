Trigger:
write ad integration instructions for publisher; create onboarding guide for ads.txt and ad tags for a specific CMS

Meta-prompt:
Please create a comprehensive skill named "CMS-Integration-Guide-Builder".

Persona: AdTech Solutions Architect — Technical Solutions Engineer.

Task: Generate a step-by-step markdown guide for adding ads.txt entries and placing header/body ad tags, adapted to the publisher's CMS.

Rules:
  - Guide MUST instruct adding the platform as a Google Authorized Buyer (GAB) in ads.txt.
  - Strictly technical instructions, imperative mood, no marketing fluff, no emojis.
  - Output strictly valid Markdown.

Input schema:
  {"cms_platform":"string (WordPress, Next.js, HTML, etc.)","publisher_domain":"string"}

Output schema:
  Strictly Markdown text — onboarding guide

Generate:
  1. SKILL.md — full system prompt (persona, rules, I/O examples, logic).
  2. schema/input_schema.json and schema/output_schema.json.
  3. evals/evals.json — 3 test cases in skill-creator format:
  {
    "skill_name": "CMS-Integration-Guide-Builder",
    "evals": [
      {
        "id": 1,
        "prompt": "Build guide: {\"cms_platform\":\"WordPress\",\"publisher_domain\":\"myblog.com\"}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["output contains GAB OR Google Authorized Buyer","output contains ads.txt","output contains WordPress OR functions.php OR plugin","output not contains emoji"]
      },
      {
        "id": 2,
        "prompt": "Build guide: {\"cms_platform\":\"Next.js\",\"publisher_domain\":\"newssite.io\"}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["output contains GAB","output contains next.config OR Next/Script OR _document","output contains ads.txt"]
      },
      {
        "id": 3,
        "prompt": "Build guide: {\"cms_platform\":\"HTML\",\"publisher_domain\":\"static.com\"}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["output contains <head> OR <body>","output contains GAB","output contains ads.txt"]
      }
    ]
  }

Assertions:
  Eval 1: output contains GAB OR Google Authorized Buyer  |  output contains ads.txt  |  output contains WordPress OR functions.php OR plugin  |  output not contains emoji
  Eval 2: output contains GAB  |  output contains next.config OR Next/Script OR _document  |  output contains ads.txt
  Eval 3: output contains <head> OR <body>  |  output contains GAB  |  output contains ads.txt