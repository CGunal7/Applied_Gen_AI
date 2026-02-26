## Architecture Overview
```
User Query
    │
    ▼
┌─────────────────────────────────┐
│   LangChain Agent               │
│   (Tool-Calling Agent)          │
│   + ConversationMemory (k=5)    │
│   + User Preference Memory      │
└──────────┬──────────────────────┘
           │
     ┌─────┴─────┐
     │   Tool     │
     │   Router   │
     └─────┬─────┘
           │
    ┌──────┼──────┬──────────┬──────────┬──────────┐
    ▼      ▼      ▼          ▼          ▼          ▼
┌──────┐┌──────┐┌────────┐┌─────────┐┌─────────┐┌──────────┐
│Discov││Ingest││Retriev ││Reasoning││Report   ││Comparison│
│ery   ││Tool  ││/Filter ││/Aggreg  ││Generate ││Tool      │
│Tool  ││      ││Tool    ││Tool     ││Tool     ││(NEW)     │
└──────┘└──────┘└───┬────┘└────┬────┘└────┬────┘└────┬─────┘
                    │          │          │          │
                    ▼          ▼          ▼          ▼
             ┌───────────┐  ┌──────┐  ┌──────┐  ┌──────┐
             │  FAISS    │  │Multi │  │ LLM  │  │Multi │
             │  Vector   │  │Query │  │Synth │  │Query │
             │  Store    │  │Pred  │  │Chain │  │Comp  │
             └───────────┘  └──────┘  └──────┘  └──────┘
```

## Pipeline Stages & Tools

| # | Tool Name | Pipeline Stage | Description |
|---|-----------|---------------|-------------|
| 1 | `dataset_discovery_tool` | Discovery | Lists available data: years, teams, match count |
| 2 | `data_ingestion_tool` | Ingestion | Loads raw stats for a specific team, personalizes for favorite team |
| 3 | `retrieval_or_filter_tool` | Retrieval | FAISS similarity search — returns top 6 relevant documents |
| 4 | `reasoning_or_aggregation_tool` | Reasoning | STAR-method match prediction using multi-query retrieval (3 searches) |
| 5 | `report_generation_tool` | Synthesis | Structured report generation via LLM chain |
| 6 | `comparison_tool` | Analysis | Side-by-side team comparison with statistical edge indicators |

## Prompt Engineering

### RAG Chain — Chain-of-Thought + Source Citations
The Q&A system prompt enforces a 3-step reasoning process:
1. **THINK** — Identify which context documents are relevant
2. **REASON** — Connect facts logically
3. **ANSWER** — Respond with citations

Every response includes a `📎 Sources` section listing exactly which data was used (e.g., "Tournament data: 2014 World Cup"). This prevents hallucination and proves grounding.

**Design decisions:**
- `"Use ONLY the provided context"` — prevents the LLM from using training data, ensuring reproducible answers
- Chain-of-thought format — forces the model to show reasoning, not just conclusions
- Source citations — every answer is traceable back to the dataset

### Prediction Chain — STAR Framework
Match predictions follow the STAR analysis method:
- **Situation** — Team profiles and head-to-head history (establishes facts)
- **Task** — What each team needs to do to win (shows analytical depth)
- **Action** — How the match unfolds based on historical patterns (data-driven reasoning)
- **Result** — Predicted score with confidence level and citations

**Key innovation:** Multi-query retrieval — instead of one search, the prediction runs 3 separate searches (team1 stats, team2 stats, head-to-head record) and combines the results. This ensures the LLM has complete context for both teams.

### Agent System Prompt
The agent prompt includes a **Tool Selection Guide** that maps question patterns to tools:
- "Who won..." → `retrieval_or_filter_tool`
- "Predict..." → `reasoning_or_aggregation_tool`
- "Compare..." → `comparison_tool`

This reduces tool selection errors and improves response accuracy.

## Memory & State Persistence

### Conversation Memory
- **Type:** `ConversationBufferWindowMemory` with `k=5`
- **Purpose:** Retains the last 5 question-answer exchanges
- **Enables:** Follow-up questions like "How many goals were scored in that tournament?" after asking about the 2014 World Cup
- **Implementation:** Memory is injected via `MessagesPlaceholder("chat_history")` in every prompt

### User Preference Memory
- **Type:** Python dictionary (`user_preferences`)
- **Purpose:** Remembers user's favorite team across the session
- **Enables:** Personalized responses — e.g., "Here are the stats for your favorite team, Brazil!"
- **Persists:** Two relevant user preferences (favorite team, preferred analysis style)

## Limitations & Responsible AI

### Data Limitations
- Historical data covers **1930–2014 World Cups only** (excludes 2018, 2022)
- Player data limited to participation records (no individual goals/assists in dataset)
- Team names may vary across eras (e.g., "Germany FR" vs "Germany")

### Prediction Limitations
- Predictions based on **historical patterns only** — no current rosters, injuries, or form
- Confidence levels (Low/Medium/High) reflect data availability, not prediction accuracy
- Small sample sizes for some head-to-head matchups may reduce prediction reliability

### Responsible AI Practices
- Every prediction includes a mandatory disclaimer
- All answers cite their data sources
- The system explicitly states when data is insufficient rather than guessing
- Outputs are labeled as **educational only, not professional sports analytics**
- The chatbot will not provide gambling or betting advice