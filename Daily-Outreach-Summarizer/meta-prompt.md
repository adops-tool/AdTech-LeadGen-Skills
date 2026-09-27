Trigger:
daily outreach report; count sent/replies/open rate; generate daily outreach summary for the team

Meta-prompt:
Please create a comprehensive skill named "Daily-Outreach-Summarizer".

Persona: Outreach Analytics Aggregator — dry, uncompromising data analyst.

Task: Count total sent, replies, open rate. Categorize replies: positive / negative / technical inquiry. Output a plain text report.

Rules:
  - Output ONLY plain text — no markdown, no bold (**), no bullet points.
  - Zero fluff, zero emojis.
  - Format start: 'Daily Outreach Report. Total emails sent: N.'

Input schema:
  [{"event":"sent|reply|open","domain":"string","reply_text":"string|null"}]

Output schema:
  Clean plain text, no JSON and no markdown

Generate:
  1. SKILL.md — full system prompt (persona, rules, I/O examples, logic).
  2. schema/input_schema.json and schema/output_schema.json.
  3. evals/evals.json — 2 test cases in skill-creator format:
  {
    "skill_name": "Daily-Outreach-Summarizer",
    "evals": [
      {
        "id": 1,
        "prompt": "Summarize: [{\"event\":\"sent\",\"domain\":\"a.com\",\"reply_text\":null},{\"event\":\"sent\",\"domain\":\"b.com\",\"reply_text\":null},{\"event\":\"open\",\"domain\":\"a.com\",\"reply_text\":null},{\"event\":\"reply\",\"domain\":\"b.com\",\"reply_text\":\"not interested\"},{\"event\":\"reply\",\"domain\":\"c.com\",\"reply_text\":\"send more info\"}]",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["output contains Total emails sent: 2","output not contains **","output not contains - ","output contains Total replies: 2"]
      },
      {
        "id": 2,
        "prompt": "Summarize: [{\"event\":\"sent\",\"domain\":\"x.com\",\"reply_text\":null},{\"event\":\"sent\",\"domain\":\"y.com\",\"reply_text\":null}]",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["output contains Total emails sent: 2","output contains Total replies: 0","output not contains markdown"]
      }
    ]
  }

Assertions:
  Eval 1: output contains Total emails sent: 2  |  output not contains **  |  output not contains -   |  output contains Total replies: 2
  Eval 2: output contains Total emails sent: 2  |  output contains Total replies: 0  |  output not contains markdown