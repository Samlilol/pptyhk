# pptyhk
## What 
In one sencentence: AI-Driven Hong Kong Property Risk Checker for 1st Time Home Buyer

Website: https://pptyhk.vercel.app

## Who and why
The primary user is a first-time Hong Kong private-market buyer who is actively considering buying their first flat.

They usually have enough savings to start looking seriously, but limited experience judging whether a property is a good long-term decision. They may understand the headline price and mortgage amount, but they are less confident about resale liquidity, future upgrade potential, district quality, transport reality, and whether the property is fairly priced against better alternatives.

This buyer is not casually browsing. They are somewhere between "I think I should buy soon" and "I may buy this specific option." The decision feels high-stakes because a bad first purchase can damage years of savings and make future upgrading harder.

Hence, first-time Hong Kong buyers can find property information, but they struggle to make a high-quality and confidence decision on a specific property candidate.

They would love to know whether the candidate is financially safe, fairly priced, resaleable, and compatible with their future upgrade path.

## Explain
Discovery
Validated demand from real people:

Friends are actively evaluating properties for their first home — some for investment, some ahead of marriage
One friend pays HK$500/month just to have an agent hunt for the right properties
YouTube channels and discussion threads on real estate consistently drive high traffic
Buying a home is a life-changing, high-stakes decision — the cost of getting it wrong is enormous
The core gap: buyers face massive information asymmetry against developers and sales agents, while the research process is fragmented across dozens of sources.

### Goals
Derisk the buying process — close the information gap created by developer marketing and agent sales pitches
Lower cognitive load — especially for busy professionals who can't spend weeks researching a single property
Easier and safer.

### The Current Buyer Journey (and Where It Breaks)
Most buyers go through six stages. Each one has friction:

Stage	What buyers do today
1. Awareness	YouTube videos, friends' advice, news, property portals, estate agent posters
2. Financial check	Mortgage calculators, narrowing price range and shortlist
3. Searching	Visiting show flats, hiring agents, browsing portals
4. Evaluation	Aggregating info from online research, agents, friends, forums, social media
5. Negotiation	Price discussion, final mortgage confirmation
6. Close	Signing and completion
Stage 4 is where buyers suffer most. They are:

Manually collecting information scattered across the internet, friends, family, and agents
Easily misled by sales pitches ("5-minute walk to the MTR" that takes 15 minutes in rain)
Unsure how to compare properties objectively — which is actually better, safer, more defensible in the current economic climate

### Solution: A Three-Phase Agent Platform
Phase 1 — Free: Research Agent
Quickly organise all public information about a specific property into a structured, fact-checked report.

Factual data: transaction prices per sq ft, building age, developer, nearest MTR stations
Web search synthesis: news, forum sentiment, developer reputation
Quick analysis: evaluation, potential risks, recommended next steps
Phase 2 — Paid: Deep Analysis Agent
Multi-angle analysis going beyond what any single source provides:

同區平均呎價比較 (district average price comparison)
實際步行去站要多久 (actual walking time to station, not developer estimate)
途經有沒有蓋 (is the route to the station covered/sheltered?)
租值分析 (rental yield analysis)
防守力評估 (capital defensibility in a downturn)
Comparison across multiple shortlisted properties
Phase 3 — TBC: Search & Filter Agent
Given a buyer's context (budget, life stage, location preference, financial profile), proactively surface and rank matching properties with alternatives.

### Architecture
Design Philosophy
Built on LangGraph with the same dependency injection pattern used in the refund agent: each specialist is a Runnable, so Phase 1 deterministic specialists can be swapped for LLM-backed agents in Phase 2 without changing the graph topology.

Graph Flow
User Input (URL or estate name)
          │
          ▼
   resolve_identity          ← RVD estate master lookup → canonical name, address, completion year
          │
          ▼
   plan_research             ← generates topics to search (developer, news, forum, prices)
          │
          ▼
   collect_evidence          ← three parallel tool calls:
     ├── station_lookup_tool      MTR CSV → nearest stations + walking estimate
     ├── transaction_tool         House730/Centaline CSVs → price/sqft, recent transactions
     └── web_search_tool          Tavily/SerpAPI → developer reputation, news, forums
          │
          ▼
   compile_evidence_pack     ← merge structured + web evidence, tag each field with provenance
          │
          ▼
   analyse                   ← LLM: evaluation, potential risks, next steps, confidence level
          │
          ▼
   revision_check            ← guard: are all required fields present? any unverified claims?
          │
     ┌────┴────┐
     │         │
  [pass]   [fail, attempt < 2]
     │         │
     │    collect_evidence   ← retry with gap-filling instructions
     │
     ▼
generate_report              ← structured PropertyReport with provenance + warnings
          │
          ▼
   Output: Free Report

### Data Sources
Estate Master
Source: RVD Names of Buildings + Address Lookup Service
Used for: canonical estate identity, display name (ZH), address, completion year
Provenance key: grounding.provenance.estateMaster
RVD XML: http://www.rvd.gov.hk/datagovhk/bnb-u.xml (HK/Kowloon), http://www.rvd.gov.hk/datagovhk/bnb-nt.xml (NT)
ALS bulk data: https://www.als.gov.hk/data/ALS-GeoJSON.zip

Station Master
Source: MTR official open data
Used for: station names (EN/ZH), line names, proximity calculation
Provenance key: grounding.provenance.stationMaster
Direct CSV: https://opendata.mtr.com.hk/data/mtr_lines_and_stations.csv

Pricing / Transaction Data
Current Phase 1 source: normalised rows captured from House730 and Centaline (120 rows for TKO/TKL launch)
Long-term target: Land Registry IRIS / Memorial Day Book (MDB) for authoritative, timely private transaction records
Provenance key: grounding.provenance.pricingSummary
IRIS: https://www.iris.gov.hk/
MDB files: https://www.landreg.gov.hk/en/services/services_b_7.htm

Note: data.gov.hk provides monthly aggregate statistics and Housing Authority scheme transactions, but does not provide an open feed of private residential unit transactions at estate level. The canonical source for private estate pricing remains Land Registry IRIS/MDB.

Web Search (Unstructured)
Developer reputation, news coverage, forum sentiment (e.g. HKDiscuss)
Retrieved at query time via Tavily / SerpAPI
Clearly labelled as unstructured in the evidence pack
Evidence Pack Output Fields
Each field is tagged with its provenance source so the UI can display appropriate trust indicators:
