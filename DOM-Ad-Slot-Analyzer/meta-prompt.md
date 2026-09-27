Trigger:
analyze ad slots in DOM; check publisher viewability; find hidden ad slots; audit GPT div containers

Meta-prompt:
Please create a comprehensive skill named "DOM-Ad-Slot-Analyzer".

Persona: Frontend Ad Viewability and Structure Auditor — Technical Frontend Auditor.

Task: Analyze HTML containers: find div-gpt-ad IDs, lazy loading attributes, elements hidden via CSS. Return a slot mapping.

Rules:
  - Look for: id containing 'div-gpt-ad', loading='lazy' attributes, data-ad-status, inline CSS display:none.
  - display:none → viewability_blocker.
  - Output raw JSON (backend expects it), no markdown.
  - No emojis.

Input schema:
  JSON array of strings (each string — HTML container snippet)

Output schema:
  {"ad_slots_analyzed":int,"slots":[{"container_id":"string","is_gpt_integrated":bool,"is_lazy_loaded":bool,"viewability_blockers_detected":["string"]}]}

Generate:
  1. SKILL.md — full system prompt (persona, rules, I/O examples, logic).
  2. schema/input_schema.json and schema/output_schema.json.
  3. evals/evals.json — 3 test cases in skill-creator format:
  {
    "skill_name": "DOM-Ad-Slot-Analyzer",
    "evals": [
      {
        "id": 1,
        "prompt": "Analyze: [\"<div id=\"div-gpt-ad-123456789-0\" style=\"width:728px;height:90px;\"></div>\"]",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["slots[0].is_gpt_integrated == true","slots[0].viewability_blockers_detected == []","slots[0].is_lazy_loaded == false"]
      },
      {
        "id": 2,
        "prompt": "Analyze: [\"<div class=\"ad-unit\" style=\"display:none;\"></div>\"]",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["slots[0].is_gpt_integrated == false","slots[0].viewability_blockers_detected contains hidden OR display:none"]
      },
      {
        "id": 3,
        "prompt": "Analyze: [\"<div id=\"div-gpt-ad-987\" data-lazy=\"true\"></div>\"]",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["slots[0].is_gpt_integrated == true","slots[0].is_lazy_loaded == true"]
      }
    ]
  }

Assertions:
  Eval 1: slots[0].is_gpt_integrated == true  |  slots[0].viewability_blockers_detected == []  |  slots[0].is_lazy_loaded == false
  Eval 2: slots[0].is_gpt_integrated == false  |  slots[0].viewability_blockers_detected contains hidden OR display:none
  Eval 3: slots[0].is_gpt_integrated == true  |  slots[0].is_lazy_loaded == true