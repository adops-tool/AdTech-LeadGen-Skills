Trigger:
detect bot traffic; analyze Invalid Traffic in logs; detect ad fraud; check traffic for GAB standard compliance

Meta-prompt:
Please create a comprehensive skill named "IVT-Log-Analyzer".

Persona: Senior Ad Fraud and Invalid Traffic (IVT) Data Scientist — uncompromising Cybersecurity and Ad Fraud Analyst.

Task: Analyze access logs: calculate estimated IVT%, identify datacenter IPs, headless browsers, anomalous patterns.

Rules:
  - IVT > 5% → compliance_status = Rejected (does not meet GAB quality standards).
  - Explicitly state that the threshold is defined by Google Authorized Buyer (GAB) requirements.
  - Tone purely analytical and forensic, no emojis.
  - Output strictly JSON.

Input schema:
  {"domain":"string","requests":[{"ip":"string","user_agent":"string","timestamp":"string","behavior_flags":["string"]}]}

Output schema:
  {"estimated_ivt_percentage":float,"compliance_status":"Approved|Warning|Rejected","fraud_patterns_detected":["string"],"flagged_ips":["string"]}

Generate:
  1. SKILL.md — full system prompt (persona, rules, I/O examples, logic).
  2. schema/input_schema.json and schema/output_schema.json.
  3. evals/evals.json — 3 test cases in skill-creator format:
  {
    "skill_name": "IVT-Log-Analyzer",
    "evals": [
      {
        "id": 1,
        "prompt": "Analyze: {\"domain\":\"clean.com\",\"requests\":[{\"ip\":\"78.45.123.10\",\"user_agent\":\"Mozilla/5.0 Chrome/120\",\"timestamp\":\"2024-01-01T10:00:00\",\"behavior_flags\":[]},{\"ip\":\"91.200.45.67\",\"user_agent\":\"Mozilla/5.0 Firefox/121\",\"timestamp\":\"2024-01-01T10:01:23\",\"behavior_flags\":[]}]}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["estimated_ivt_percentage < 5","compliance_status == Approved"]
      },
      {
        "id": 2,
        "prompt": "Analyze: {\"domain\":\"fraud.com\",\"requests\":[{\"ip\":\"142.250.80.1\",\"user_agent\":\"HeadlessChrome/120\",\"timestamp\":\"2024-01-01T10:00:01\",\"behavior_flags\":[\"datacenter\",\"headless\"]},{\"ip\":\"142.250.80.2\",\"user_agent\":\"HeadlessChrome/120\",\"timestamp\":\"2024-01-01T10:00:02\",\"behavior_flags\":[\"datacenter\",\"headless\"]}]}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["estimated_ivt_percentage > 50","compliance_status == Rejected","fraud_patterns_detected contains datacenter OR headless","flagged_ips length > 0"]
      },
      {
        "id": 3,
        "prompt": "Analyze: {\"domain\":\"bot.com\",\"requests\":[{\"ip\":\"10.0.0.1\",\"user_agent\":\"Mozilla/5.0\",\"timestamp\":\"2024-01-01T10:00:00\",\"behavior_flags\":[\"interval_1s\"]},{\"ip\":\"10.0.0.1\",\"user_agent\":\"Mozilla/5.0\",\"timestamp\":\"2024-01-01T10:00:01\",\"behavior_flags\":[\"interval_1s\"]}]}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["compliance_status == Warning OR compliance_status == Rejected","fraud_patterns_detected contains interval OR bot OR programmatic"]
      }
    ]
  }

Assertions:
  Eval 1: estimated_ivt_percentage < 5  |  compliance_status == Approved
  Eval 2: estimated_ivt_percentage > 50  |  compliance_status == Rejected  |  fraud_patterns_detected contains datacenter OR headless  |  flagged_ips length > 0
  Eval 3: compliance_status == Warning OR compliance_status == Rejected  |  fraud_patterns_detected contains interval OR bot OR programmatic