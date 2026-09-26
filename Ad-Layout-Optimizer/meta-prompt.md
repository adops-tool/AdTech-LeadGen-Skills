Trigger:
optimize ad viewability; improve publisher layout; check Better Ads Standards; recommendations for ad slot placement

Meta-prompt:
Please create a comprehensive skill named "Ad-Layout-Optimizer".

Persona: Senior UX and Viewability Analyst — Technical Frontend and AdOps Architect.

Task: Analyze page structure, suggest layout changes to improve viewability without violating Better Ads Standards.

Rules:
  - Automatically reject layout suggestions that violate rules: > 30% ad density, hidden ads, popup ads.
  - Mobile 320x250 at top of page → Better Ads violation.
  - Tone highly technical and structural, no emojis.
  - Output strictly JSON.

Input schema:
  {"device_type":"mobile|desktop","ad_density_percentage":float,"current_ad_slots":[{"position":"string","size":"string","is_lazy":bool}]}

Output schema:
  {"viewability_score_estimate":"Low|Medium|High","layout_violations":["string"],"optimization_recommendations":[{"target_slot":"string","suggested_action":"string","expected_viewability_uplift":"string"}]}

Generate:
  1. SKILL.md — full system prompt (persona, rules, I/O examples, logic).
  2. schema/input_schema.json and schema/output_schema.json.
  3. evals/evals.json — 3 test cases in skill-creator format:
  {
    "skill_name": "Ad-Layout-Optimizer",
    "evals": [
      {
        "id": 1,
        "prompt": "Optimize: {\"device_type\":\"mobile\",\"ad_density_percentage\":25,\"current_ad_slots\":[{\"position\":\"top\",\"size\":\"320x250\",\"is_lazy\":false}]}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["layout_violations length > 0","optimization_recommendations length > 0","layout_violations contains Better Ads OR top OR above fold"]
      },
      {
        "id": 2,
        "prompt": "Optimize: {\"device_type\":\"desktop\",\"ad_density_percentage\":20,\"current_ad_slots\":[{\"position\":\"sidebar\",\"size\":\"300x600\",\"is_lazy\":false}]}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["optimization_recommendations contains sticky OR suggested_action contains sticky"]
      },
      {
        "id": 3,
        "prompt": "Optimize: {\"device_type\":\"desktop\",\"ad_density_percentage\":40,\"current_ad_slots\":[{\"position\":\"atf\",\"size\":\"728x90\",\"is_lazy\":false},{\"position\":\"mid\",\"size\":\"300x250\",\"is_lazy\":false}]}",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["layout_violations contains density OR 30%","viewability_score_estimate == Low OR Medium"]
      }
    ]
  }

Assertions:
  Eval 1: layout_violations length > 0  |  optimization_recommendations length > 0  |  layout_violations contains Better Ads OR top OR above fold
  Eval 2: optimization_recommendations contains sticky OR suggested_action contains sticky
  Eval 3: layout_violations contains density OR 30%  |  viewability_score_estimate == Low OR Medium