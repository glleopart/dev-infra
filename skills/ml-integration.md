---
name: ml-integration
description: >
  ML/DL/AI layer design and implementation skill. Trigger this skill when the
  user wants to add machine learning, deep learning, or AI features to any
  project — including: LLM-based features (chat, summarization, extraction,
  classification, recommendations), embeddings and semantic search, fine-tuning
  workflows, computer vision, tabular ML models, RAG pipelines, agent
  architectures, periodic automated reports, user behavior analysis, or
  personalized recommendations. Also trigger for phrases like "add AI to my
  app", "integrate an LLM", "build a recommender", "semantic search", "chatbot",
  "classification model", "anomaly detection", "automate this report with AI",
  or "train a model on my data". This skill always isolates AI features in a
  dedicated project version and never mixes them with UI or data model work.
---

# ML/AI Integration Skill

You design and implement AI/ML layers for software projects. You treat ML
as a software engineering problem: reproducible, testable, versioned, and
decoupled from the rest of the application.

---

## Step 1 — AI feature intake

Before designing anything, answer these questions:

1. **What decision or output does the AI need to produce?**
   (Classify a document? Rank items? Generate text? Extract entities? Detect anomalies?)

2. **What data is available?**
   - Volume: how many examples exist now? How many per month going forward?
   - Labels: labeled or unlabeled? Who labels new data?
   - Privacy: can data be sent to a third-party API (OpenAI, Anthropic, etc.)?

3. **What are the latency requirements?**
   - Batch (run once a day/week)? Near-real-time (< 5 seconds)? Real-time (< 200ms)?

4. **What is the failure mode?**
   - What happens if the model is wrong? Is a wrong answer worse than no answer?

5. **What is the budget?**
   - API cost per call vs. self-hosted inference cost

These answers determine the approach. Present the recommended approach to the
user before writing any code.

---

## Approach decision tree

```
Need to generate text or reason over it?
├── Yes → Use an LLM API (Anthropic, OpenAI, etc.)
│   ├── Need to search over your own docs/data first? → RAG pipeline
│   ├── Need structured extraction from text? → Structured output / function calling
│   ├── Need a back-and-forth conversation? → Stateful chat with history management
│   └── Need automated periodic reports? → Scheduled batch LLM calls
│
Need to rank, match, or find similar items?
├── Yes → Embeddings + vector search (no fine-tuning required for most use cases)
│   ├── Items are text → text-embedding-3-small or equivalent
│   ├── Items are images → CLIP or equivalent
│   └── Hybrid (metadata + semantic) → combine vector score with BM25
│
Need to classify or predict from structured/tabular data?
├── Small data (< 10k rows) → scikit-learn (XGBoost, RandomForest, LogisticRegression)
├── Medium data (10k–1M rows) → LightGBM or XGBoost with cross-validation
└── Large data (> 1M rows) → consider Spark MLlib or a managed platform
│
Need computer vision?
├── Classification / detection → fine-tune a pretrained model (ResNet, EfficientNet, YOLOv8)
├── No custom data → use a vision API (Google Vision, AWS Rekognition, GPT-4V)
└── Edge device deployment → consider TFLite or ONNX
│
Need to detect anomalies in time-series or logs?
└── Isolation Forest, LSTM autoencoder, or statistical baselines first
```

---

## Module structure for AI features

AI code must be **fully decoupled** from the rest of the application.
The rest of the app calls the AI layer; the AI layer never imports from the app.

```
project/
├── ai/                         # all ML/AI code lives here
│   ├── __init__.py
│   ├── config.py               # model names, thresholds, API keys loaded from env
│   ├── llm/
│   │   ├── client.py           # thin wrapper around Anthropic/OpenAI SDK
│   │   ├── prompts/            # all prompts as .md or .txt files (not hardcoded)
│   │   │   └── summarize.md
│   │   └── structured.py       # structured output helpers
│   ├── embeddings/
│   │   ├── embed.py            # embedding generation
│   │   ├── vector_store.py     # CRUD for vector DB (Chroma, Pinecone, pgvector)
│   │   └── search.py           # similarity search + hybrid search
│   ├── recommender/            # if applicable
│   │   ├── model.py
│   │   └── train.py
│   ├── batch/                  # scheduled/batch jobs
│   │   └── report_generator.py
│   └── utils/
│       ├── tokenizer.py        # count tokens before sending
│       └── retry.py            # exponential backoff for API calls
├── tests/
│   └── ai/                     # AI tests isolated from app tests
└── scripts/
    └── embed_documents.py      # one-off data ingestion scripts
```

---

## LLM integration patterns

### Pattern 1 — Simple completion
Use for: summarization, classification, extraction, single-turn Q&A.

```python
# ai/llm/client.py
import anthropic
import os
from tenacity import retry, stop_after_attempt, wait_exponential

client = anthropic.Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])

@retry(stop=stop_after_attempt(3), wait=wait_exponential(min=1, max=10))
def complete(prompt: str, system: str = "", max_tokens: int = 1024) -> str:
    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=max_tokens,
        system=system,
        messages=[{"role": "user", "content": prompt}]
    )
    return response.content[0].text
```

### Pattern 2 — Structured output (JSON extraction)
Use for: entity extraction, classification with confidence, form filling.

```python
# ai/llm/structured.py
import json
from .client import complete

def extract_structured(text: str, schema: dict, prompt_template: str) -> dict:
    """Extract structured data from text using a schema description."""
    system = (
        "You extract information from text and return ONLY valid JSON. "
        "Never add explanation. Never add markdown fences. "
        f"Return an object matching this schema: {json.dumps(schema)}"
    )
    prompt = prompt_template.format(text=text)
    raw = complete(prompt=prompt, system=system, max_tokens=512)
    try:
        return json.loads(raw)
    except json.JSONDecodeError:
        # Try stripping accidental markdown
        cleaned = raw.strip().strip("```json").strip("```").strip()
        return json.loads(cleaned)
```

### Pattern 3 — RAG (Retrieval-Augmented Generation)
Use for: document Q&A, search over internal knowledge bases.

```python
# ai/embeddings/search.py
from .embed import embed_text
from .vector_store import VectorStore

store = VectorStore()  # wraps Chroma / pgvector / Pinecone

def rag_query(question: str, top_k: int = 5) -> str:
    """Retrieve relevant chunks and generate an answer."""
    query_embedding = embed_text(question)
    chunks = store.similarity_search(query_embedding, top_k=top_k)
    context = "\n\n---\n\n".join(c["text"] for c in chunks)
    prompt = f"Context:\n{context}\n\nQuestion: {question}\n\nAnswer:"
    return complete(prompt=prompt, system="Answer using only the provided context.")
```

### Pattern 4 — Stateful chat
Use for: chatbots, multi-turn assistants.

```python
# ai/llm/chat.py
from .client import client

class ChatSession:
    def __init__(self, system: str = ""):
        self.history: list[dict] = []
        self.system = system

    def send(self, user_message: str, max_tokens: int = 1024) -> str:
        self.history.append({"role": "user", "content": user_message})
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=max_tokens,
            system=self.system,
            messages=self.history
        )
        assistant_message = response.content[0].text
        self.history.append({"role": "assistant", "content": assistant_message})
        return assistant_message

    def reset(self):
        self.history = []
```

### Pattern 5 — Scheduled batch report
Use for: weekly summaries, automated analytics reports, digest emails.

```python
# ai/batch/report_generator.py
from datetime import datetime, timedelta
from ..llm.client import complete
from ..llm.structured import extract_structured

def generate_weekly_report(data: list[dict]) -> str:
    """Generate a natural-language report from structured data."""
    data_summary = summarize_data(data)  # your domain function
    prompt = (
        f"Here is a summary of activity from the past 7 days:\n{data_summary}\n\n"
        "Write a concise executive summary (3–5 paragraphs). "
        "Highlight trends, anomalies, and recommended actions."
    )
    return complete(prompt=prompt, max_tokens=800)
```

---

## Embeddings and semantic search

### Choosing a vector store

| Store | Use when |
|-------|----------|
| pgvector (PostgreSQL extension) | Already using PostgreSQL; < 1M vectors |
| Chroma | Local dev or small self-hosted deployments |
| Pinecone | Managed, production, > 1M vectors |
| Qdrant | Self-hosted, production, good Python SDK |
| Weaviate | Multi-modal (text + image vectors) |

### Embedding model selection

| Model | Use when |
|-------|----------|
| `text-embedding-3-small` (OpenAI) | Low cost, good quality, English-heavy |
| `text-embedding-3-large` (OpenAI) | High accuracy needs |
| `all-MiniLM-L6-v2` (sentence-transformers) | Self-hosted, no API cost |
| `multilingual-e5-large` | Multi-language support |

---

## Tabular ML patterns

Use scikit-learn for most cases. Follow this sequence:

1. **Exploratory analysis** — `pandas-profiling` or `ydata-profiling`
2. **Baseline** — always establish a simple baseline (mean predictor, majority class)
3. **Feature engineering** — domain-specific transformations before model selection
4. **Model selection** — start with LogisticRegression/LinearRegression, then XGBoost
5. **Cross-validation** — k-fold, never hold-out only
6. **Calibration** — calibrate probabilities if using classification scores downstream
7. **Serialization** — save with `joblib.dump`; version the model file with a timestamp

```python
# Minimal training pipeline
import joblib
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.model_selection import cross_val_score

pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("model", GradientBoostingClassifier(n_estimators=100, random_state=42))
])
scores = cross_val_score(pipeline, X_train, y_train, cv=5, scoring="roc_auc")
print(f"CV AUC: {scores.mean():.3f} ± {scores.std():.3f}")
pipeline.fit(X_train, y_train)
joblib.dump(pipeline, f"models/classifier_{datetime.now():%Y%m%d}.pkl")
```

---

## AI-specific audit additions (for audit agents)

When the project includes an AI layer, audit agents should additionally check:

**Security:**
- Prompt injection vectors (user input concatenated directly into prompts)
- Token budget enforcement (no unbounded context window usage)
- API key rotation policy

**Quality:**
- Prompts are in files, not hardcoded strings
- Retry/backoff logic on all API calls
- Token counting before sending (avoid 400 errors on long inputs)
- Model name is a config variable (not hardcoded in 10 places)

**Parity:**
- AI responses are typed (structured output) where downstream code consumes fields
- Async AI calls don't block the request/response cycle (use background tasks or queues)

**Costs:**
- Estimated monthly cost at current usage is documented
- Token usage is logged (for cost tracking)
- Caching implemented for repeated identical queries

---

## Version placement rules

ML/AI features always get their own isolated version. Never bundle AI with:
- UI redesigns
- Database schema changes
- Auth changes

Recommended placement:
- Simple LLM call: can go in any version after the core CRUD layer is stable
- Embeddings + vector store: its own version (requires new infrastructure)
- Fine-tuning or custom model: its own version (requires training pipeline)
- Scheduled batch jobs: its own version or the hardening version
