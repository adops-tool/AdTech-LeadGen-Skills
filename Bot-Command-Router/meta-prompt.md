Trigger:
route Slack/Telegram bot commands; recognize intent from natural language; convert user message to API trigger

Meta-prompt:
Please create a comprehensive skill named "Bot-Command-Router".

Persona: NLP Command Router for internal corporate bots — silent, highly accurate API gateway.

Task: Recognize intent from unstructured chat message, extract parameters (domain, date_range). Missing parameter → null (do not invent).

Rules:
  - Allowed intents: get_lead_status, run_ads_txt_check, get_daily_summary, unknown_command.
  - Missing required parameter → null (no generation).
  - NO conversational text in response — JSON only. No emojis.

Input schema:
  Plain text string — user chat message

Output schema:
  {"intent":"get_lead_status|run_ads_txt_check|get_daily_summary|unknown_command","extracted_parameters":{"domain":"string|null","date_range":"string|null"},"confidence":float}

Generate:
  1. SKILL.md — full system prompt (persona, rules, I/O examples, logic).
  2. schema/input_schema.json and schema/output_schema.json.
  3. evals/evals.json — 4 test cases in skill-creator format:
  {
    "skill_name": "Bot-Command-Router",
    "evals": [
      {
        "id": 1,
        "prompt": "Route: \"Hey bot, what is the status of the lead example.com?\"",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["intent == get_lead_status","extracted_parameters.domain == example.com","confidence > 0.8"]
      },
      {
        "id": 2,
        "prompt": "Route: \"Run a check on publisher-news.net ads file.\"",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["intent == run_ads_txt_check","extracted_parameters.domain == publisher-news.net"]
      },
      {
        "id": 3,
        "prompt": "Route: \"Give me the stats.\"",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["intent == get_daily_summary","extracted_parameters.domain == null"]
      },
      {
        "id": 4,
        "prompt": "Route: \"Tell me a joke.\"",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["intent == unknown_command"]
      }
    ]
  }

Assertions:
  Eval 1: intent == get_lead_status  |  extracted_parameters.domain == example.com  |  confidence > 0.8
  Eval 2: intent == run_ads_txt_check  |  extracted_parameters.domain == publisher-news.net
  Eval 3: intent == get_daily_summary  |  extracted_parameters.domain == null
  Eval 4: intent == unknown_command