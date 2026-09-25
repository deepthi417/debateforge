# DebateForge

**Multi-agent adversarial reasoning system that stress-tests decisions through structured debate, not consensus.**

[Live Demo](#) · [Architecture](#architecture) · [Evaluation](#evaluation) · [Setup](#setup)

---

## Why DebateForge

Most "AI agent" demos show agents *cooperating* on a task — one agent hands off to the next until a job gets done. But real decision-making isn't cooperative. A thesis that only gets one AI's opinion (e.g. "is this a good idea?" to a single LLM) is fragile — it tends to agree with whatever framing you gave it.

DebateForge instead runs a structured, multi-round debate between adversarial personas — a Skeptic, a RAG-grounded Domain Expert, and a Devil's Advocate — moderated and synthesized into a final verdict. A thesis that survives this process is far more load-bearing than one validated by a single model turn.

**Example use cases:** technical decisions ("should we migrate to microservices?"), business ideas ("subscription box for plant care — good idea?"), policy claims, personal decisions with real tradeoffs.

---

## Architecture

```
                     ┌──────────────┐
      Thesis ───────▶│  Moderator   │
                      │ (sets agenda,│
                      │ routes turns)│
                      └──────┬───────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
        ┌──────────┐  ┌─────────────┐ ┌───────────────┐
        │ Skeptic  │  │Domain Expert │ │Devil's Advocate│
        │          │  │  (RAG-       │ │                │
        │          │  │  grounded)   │ │                │
        └────┬─────┘  └──────┬───────┘ └───────┬────────┘
             │                │                 │
             └────────────────┼─────────────────┘
                               ▼
                    Shared Debate Transcript
                   (LangGraph state, N rounds)
                               │
                               ▼
                       ┌───────────────┐
                       │  Synthesizer   │
                       │ (final verdict)│
                       └───────────────┘
```

### Agent roles

| Agent | Role | Grounding |
|---|---|---|
| **Moderator** | Parses the thesis, selects relevant personas, sets round count, routes turns | — |
| **Skeptic** | Attacks the thesis, surfaces failure modes and hidden assumptions | Free-form reasoning |
| **Domain Expert** | Retrieves precedent/evidence and grounds claims in real sources | RAG (hybrid search + rerank) |
| **Devil's Advocate** | Argues the strongest possible *opposite* case, even if implausible, to stress-test edges | Free-form reasoning |
| **Synthesizer** | Reads the full transcript, produces structured pros/cons/risk verdict | Reads full debate state |

Orchestration is built as a **LangGraph state machine**: each agent reads the full prior transcript before responding, so arguments build on each other across rounds instead of being independent, disconnected takes.

---

## Key design decisions

- **Round-based debate, not single-pass.** [Fill in after eval: e.g. "Round-efficiency eval showed X% of new points emerge by round 2, with diminishing returns by round 3 — settled on N rounds as the default."]
- **Only the Domain Expert is RAG-grounded.** The Skeptic and Devil's Advocate are intentionally free-form — the goal is broad, creative argument generation, while factual claims from the Domain Expert are held to a citation standard (same discipline as [VulnLens](#)).
- **Shared transcript state, not isolated agent calls.** Each persona reads what came before, so the debate is genuinely adversarial rather than parallel monologues.
- [Add any tradeoff you hit during build — e.g. persona convergence, prompt drift across rounds, retrieval corpus scope]

---

## Evaluation

Built a test set of ~10–15 theses across categories (technical decisions, business ideas, policy claims). Measured:

| Metric | Result |
|---|---|
| Citation match rate (Domain Expert) | TBD |
| Hallucination rate (Domain Expert) | TBD |
| Argument diversity (embedding distance between personas per round) | TBD |
| Convergence quality (does verdict reflect all raised points?) | TBD |
| Round efficiency (new points per round) | TBD |

*Methodology notes: [describe how each metric was computed, e.g. LLM-judged vs. human-rated, embedding model used for diversity scoring]*

---

## Demo

**Live app:** [Streamlit Community Cloud link]

**Sample debate:**

> **Thesis:** "We should switch our backend to microservices."
>
> [Paste a real transcript excerpt + final verdict card once you have one — this is more convincing than any description]

![Debate transcript screenshot](docs/screenshot-transcript.png)
![Verdict card screenshot](docs/screenshot-verdict.png)

---

## Tech Stack

- **Orchestration:** LangChain, LangGraph
- **LLM:** [Groq / your provider — model name]
- **Backend:** FastAPI (`/debate`, `/status/{id}`, `/health`) with SSE streaming for live turn-by-turn output
- **Frontend:** Streamlit — thesis input, live transcript panel, verdict card
- **Retrieval:** [vector store] with hybrid search + reranking
- **Deployment:** Docker Compose (local/full stack), Streamlit Community Cloud (public demo)

---

## Setup

### Local (Python)

```bash
git clone https://github.com/deepthi417/debateforge.git
cd debateforge
python -m venv venv
venv\Scripts\activate      # Windows
pip install -r requirements.txt
cp .env.example .env        # add your LLM API key
python -m api.main
```

### Docker

```bash
docker compose up --build
```

Frontend: `http://localhost:8501` · API: `http://localhost:8000`

### Environment variables

| Variable | Description |
|---|---|
| `GROQ_API_KEY` (or provider equivalent) | LLM API key for agent generation |
| `[VECTOR_DB_CONFIG]` | Retrieval backend config for Domain Expert RAG |

---

## Project Structure

```
debateforge/
├── agents/           # Persona prompt templates + agent logic
│   ├── moderator.py
│   ├── skeptic.py
│   ├── domain_expert.py
│   ├── devils_advocate.py
│   └── synthesizer.py
├── graph/            # LangGraph state machine + transcript schema
│   └── debate_graph.py
├── rag/              # Retrieval pipeline for Domain Expert grounding
├── api/               # FastAPI app (main.py, routes)
├── frontend/          # Streamlit app
├── eval/              # Eval set + scoring scripts
├── docs/              # Screenshots, architecture diagram source
├── docker-compose.yml
├── Dockerfile.api
├── Dockerfile.frontend
└── requirements.txt
```

---

## Limitations & Future Work

- Personas can converge toward agreement on adversarially weak or under-specified theses — needs a "forced disagreement" guardrail
- No persistent debate history across sessions yet
- [Add any other honest limitation once built]

---

## License

MIT
