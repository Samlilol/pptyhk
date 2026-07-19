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

## Product Roadmap
Phase 1 - Single Agent, one click quick analysis, low latency
Phase 2 - Multi Agents analysis, comparsion
Phase 3 - Personalised property search

## Archtecture Diagram
Version 0
```mermaid
flowchart TD
    U([User query]) --> IG[Input guardrails]
    IG --> ORC{Orchestrator<br/>detect intent}
    ORC -.->|low confidence| ASK[Ask user to clarify]
    ASK -.-> ORC

    ORC --> QA[Quick agent]
    ORC --> GATE{Confirm depth<br/>show cost + latency}
    ORC --> SA[Search agent]
    ORC --> CA[Comparison agent]

    GATE -->|quick| QA
    GATE -->|deep| DA[Deep agent<br/>7 sub-agents in parallel]

    QA --> LV[Light validation]
    DA --> OG{Output guardrails}
    SA --> OG
    CA --> OG

    OG -->|fail| RT[Retry with feedback<br/>max 2]
    RT --> OG
    OG -.->|unresolvable| ASK
    OG -->|pass| REV[Reviewer<br/>LLM judge]

    LV --> SYN[Synthesizer<br/>report / shortlist / text]
    REV --> SYN
    SYN --> OUT([Response to user])

    classDef guard fill:#FAEEDA,stroke:#BA7517,color:#633806
    classDef agent fill:#EEEDFE,stroke:#534AB7,color:#3C3489
    classDef io fill:#F1EFE8,stroke:#5F5E5A,color:#2C2C2A
    class IG,GATE,OG,LV,RT guard
    class ORC,QA,DA,SA,CA,REV,SYN agent
    class U,ASK,OUT io
```
```mermaid
flowchart LR
    subgraph TOOLS[" Shared tool layer — callable by all agents and synthesizer "]
        direction LR
        DB[(Property DB<br/>+ MTR data)]
        WS[Web search<br/>news, 中原]
        MC[Mortgage<br/>calculator]
        RF[Report<br/>formatting]
        DB ~~~ WS ~~~ MC ~~~ RF
    end

    classDef tool fill:#E1F5EE,stroke:#0F6E56,color:#085041
    class DB,WS,MC,RF tool
    style TOOLS fill:#F4FBF8,stroke:#0F6E56,stroke-width:1px,stroke-dasharray:6 4,color:#085041
```
```mermaid
flowchart LR
    subgraph OBS[" Observability layer — every node emits spans to LangSmith / Langfuse "]
        direction TB
        ROOT[Root trace<br/>one per user query]
        ROOT --> S1[Orchestrator span<br/>intent, route, confidence]
        ROOT --> S2[Agent spans<br/>latency, tokens, cost]
        ROOT --> S3[Tool call spans<br/>cache hit or miss]
        ROOT --> S4[Guardrail + reviewer spans<br/>verdict, retries, fail reason]
        ROOT --> S5[Synthesizer span<br/>output mode, final response]
    end

    classDef obs fill:#E6F1FB,stroke:#185FA5,color:#0C447C
    class ROOT,S1,S2,S3,S4,S5 obs
    style OBS fill:#F2F8FD,stroke:#185FA5,stroke-width:1px,stroke-dasharray:6 4,color:#0C447C
```
