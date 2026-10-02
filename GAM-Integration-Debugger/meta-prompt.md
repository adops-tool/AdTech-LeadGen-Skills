Trigger:
publisher sent GPT code with rendering issues; blank ad slots; GAM integration errors; debugging googletag

Meta-prompt:
Please create a comprehensive skill named "GAM-Integration-Debugger".

Persona: Level 3 Google Ad Manager Technical Support Engineer.

Task: Parse a GPT JavaScript snippet and console errors, find the root cause, return a fixed code block.

Rules:
  - Output strictly analytical, no emojis.
  - MUST provide fixed code — description without code is unacceptable.
  - Check: googletag.defineSlot, pubads().enableSingleRequest(), missing googletag.display(), div ID mismatch.

Input schema:
  {"issue_description":"string","html_js_snippet":"string","console_errors":"string (optional)"}

Output schema:
  {"root_cause_summary":"string","severity":"critical|warning","original_problematic_lines":["string"],"suggested_code_fix":"string"}

Generate:
  1. SKILL.md — full system prompt (persona, rules, I/O examples, logic).
  2. schema/input_schema.json and schema/output_schema.json.
  3. evals/evals.json — 3 test cases in skill-creator format:
  {
    "skill_name": "GAM-Integration-Debugger",
    "evals": [
      {
        "id": 1,
        "prompt": "Debug: {\"issue_description\":\"ads not rendering\",\"html_js_snippet\":\"googletag.enableServices();\ngoogletag.defineSlot('/1234/banner',[728,90],'div-1').addService(googletag.pubads());\",\"console_errors\":\"\"}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["root_cause_summary contains enableServices OR out-of-order","severity == critical","suggested_code_fix is not empty"]
      },
      {
        "id": 2,
        "prompt": "Debug: {\"issue_description\":\"blank slot\",\"html_js_snippet\":\"<div id='div-banner-1'></div>\ngoogletag.display('div-banner-2');\",\"console_errors\":\"slot not found\"}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["root_cause_summary contains ID OR mismatch","suggested_code_fix contains div-banner-1"]
      },
      {
        "id": 3,
        "prompt": "Debug: {\"issue_description\":\"slow load\",\"html_js_snippet\":\"<script src='gpt.js'></script>\",\"console_errors\":\"\"}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["root_cause_summary contains async OR synchronous","suggested_code_fix contains async"]
      }
    ]
  }

Assertions:
  Eval 1: root_cause_summary contains enableServices OR out-of-order  |  severity == critical  |  suggested_code_fix is not empty
  Eval 2: root_cause_summary contains ID OR mismatch  |  suggested_code_fix contains div-banner-1
  Eval 3: root_cause_summary contains async OR synchronous  |  suggested_code_fix contains async