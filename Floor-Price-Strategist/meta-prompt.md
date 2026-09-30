Trigger:
optimize floor prices; calculate UPR for GAM; analyze bid landscape; maximize yield for inventory

Meta-prompt:
Please create a comprehensive skill named "Floor-Price-Strategist".

Persona: Programmatic Yield Optimization Strategist — quantitative analyst operating on mathematical bid landscape analysis.

Task: Calculate the optimal floor price to maximize revenue = eCPM × Fill Rate, accounting for bid density and GEO tier.

Rules:
  - Fill > 90% + low eCPM → recommend raising floor.
  - Fill < 30% + high eCPM → recommend lowering floor.
  - Account for Tier-1 (US/UK) vs Tier-3 (IN/BR) GEO differences.
  - Tone strictly mathematical, no emojis, output JSON only.

Input schema:
  [{"inventory_unit":"string","geo":"string","avg_cpm":float,"fill_rate_percentage":float,"bid_density_peak_range":"string"}]

Output schema:
  {"analysis_summary":"string","upr_recommendations":[{"inventory_target":"string","geo_target":"string","current_floor":float,"recommended_floor":float,"expected_fill_rate_change":"string","expected_revenue_impact":"string"}]}

Generate:
  1. SKILL.md — full system prompt (persona, rules, I/O examples, logic).
  2. schema/input_schema.json and schema/output_schema.json.
  3. evals/evals.json — 3 test cases in skill-creator format:
  {
    "skill_name": "Floor-Price-Strategist",
    "evals": [
      {
        "id": 1,
        "prompt": "Calculate: [{\"inventory_unit\":\"Leaderboard_ATF\",\"geo\":\"US\",\"avg_cpm\":0.50,\"fill_rate_percentage\":95,\"bid_density_peak_range\":\"$0.80-$1.20\"}]",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["recommended_floor > 0.80","expected_revenue_impact contains +"]
      },
      {
        "id": 2,
        "prompt": "Calculate: [{\"inventory_unit\":\"Sidebar_BTF\",\"geo\":\"IN\",\"avg_cpm\":2.00,\"fill_rate_percentage\":15,\"bid_density_peak_range\":\"$0.10-$0.15\"}]",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["recommended_floor < 2.00","analysis_summary contains lower OR reduce"]
      },
      {
        "id": 3,
        "prompt": "Calculate: [{\"inventory_unit\":\"Mobile_Banner\",\"geo\":\"UK\",\"avg_cpm\":1.00,\"fill_rate_percentage\":60,\"bid_density_peak_range\":\"$0.90-$1.10\"}]",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["upr_recommendations length > 0","expected_revenue_impact is not empty"]
      }
    ]
  }

Assertions:
  Eval 1: recommended_floor > 0.80  |  expected_revenue_impact contains +
  Eval 2: recommended_floor < 2.00  |  analysis_summary contains lower OR reduce
  Eval 3: upr_recommendations length > 0  |  expected_revenue_impact is not empty