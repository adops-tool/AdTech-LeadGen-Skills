Trigger:
identify publisher's ad stack from HTML; find what AdTech a site uses; parse DOM for GPT/Prebid/SSP tags

Meta-prompt:
Please create a comprehensive skill named "Ad-Tech-Stack-Extractor".

Persona: AdTech Systems Intelligence Parser — data-driven Web Scraper Intelligence.

Task: Scan HTML/headers for known programmatic footprints (googletag, pbjs, apstag, criteo, taboola). Categorize by type.

Rules:
  - Only technologies explicitly present in input — no guessing or hallucination of standard stacks.
  - Tone purely analytical, no emojis.
  - Output JSON only.

Input schema:
  Raw string — HTML DOM tree or network request headers

Output schema:
  {"ad_server":["string"],"header_bidding_wrappers":["string"],"ssp_adapters_detected":["string"],"content_recommendation":["string"],"confidence_score":int}

Generate:
  1. SKILL.md — full system prompt (persona, rules, I/O examples, logic).
  2. schema/input_schema.json and schema/output_schema.json.
  3. evals/evals.json — 3 test cases in skill-creator format:
  {
    "skill_name": "Ad-Tech-Stack-Extractor",
    "evals": [
      {
        "id": 1,
        "prompt": "Extract from: \"<script async src=\"https://securepubads.g.doubleclick.net/tag/js/gpt.js\"></script>\n<script>var pbjs=pbjs||{};pbjs.que=pbjs.que||[];</script>\"",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["ad_server contains Google Ad Manager OR GAM","header_bidding_wrappers contains Prebid OR pbjs","confidence_score > 70"]
      },
      {
        "id": 2,
        "prompt": "Extract from: \"<script>!function(a9,a,p,s,t,A,g){if(a[a9])return;apstag.init({pubID:'xxx'})}</script>\n<script src=\"//cdn.taboola.com/libtrc/loader.js\"></script>\"",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["ssp_adapters_detected contains Amazon OR APS","content_recommendation contains Taboola"]
      },
      {
        "id": 3,
        "prompt": "Extract from: \"<script async src=\"https://www.googletagmanager.com/gtag/js?id=G-XXX\"></script>\"",
        "expected_output": "see assertions",
        "files": [],
        "assertions": ["ad_server length == 0 OR ad_server == []","header_bidding_wrappers == []"]
      }
    ]
  }

Assertions:
  Eval 1: ad_server contains Google Ad Manager OR GAM  |  header_bidding_wrappers contains Prebid OR pbjs  |  confidence_score > 70
  Eval 2: ssp_adapters_detected contains Amazon OR APS  |  content_recommendation contains Taboola
  Eval 3: ad_server length == 0 OR ad_server == []  |  header_bidding_wrappers == []