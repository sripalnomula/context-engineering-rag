# Advanced RAG Demo with Chroma

A local RAG demo that goes past the usual "chunk a PDF and ask it questions" example. It covers mixed document types, document-aware chunking, a persistent Chroma store, hybrid retrieval, metadata filters, parent-context expansion, cited answers and evaluation.

## Notebooks

| Notebook | What it does |
|---|---|
| `_rag_demo_core.ipynb` | Shared code: loaders, chunkers, OpenAI/Chroma wrappers, generation and metrics |
| `00_generate_and_inspect_documents.ipynb` | Generates the PDF and inspects the Markdown, CSV, HTML, JSON, TXT and PDF sources |
| `01_document_aware_chunking.ipynb` | Compares fixed, semantic, document-aware and parent-child chunking |
| `02_chroma_vector_store.ipynb` | Builds persistent Chroma collections and looks at stored embeddings/metadata |
| `03_advanced_rag_pipeline.ipynb` | Dense + BM25 hybrid retrieval, filters, RRF, context expansion, citations |
| `04_rag_evaluation.ipynb` | Retrieval and generation metrics, groundedness, failure analysis, strategy comparison |
| `05_context_engineering.ipynb` | Putting the full context together: instructions, history, user state, evidence, examples, tools, budget, injection defenses |

The company in the data, **Northstar Mobility**, is made up. Its policies, financials, incidents, tickets and products reference each other so retrieval isn't trivial, but the answers are fixed so evaluation stays repeatable.

## Setup

Use Python 3.11 or 3.12.

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
cp .env.example .env
# add your OPENAI_API_KEY to .env
jupyter lab
```

Run the notebooks in order. Each one runs `_rag_demo_core.ipynb` at the top, so you can also open any of them on its own once the dependencies are installed.

## Demo flow (20-30 min)

1. Show why one chunking rule doesn't work for prose, tables and structured records.
2. Compare chunk previews and the metadata that goes into Chroma.
3. Store vectors, query them, and apply metadata filters.
4. Compare dense, BM25 and fused (RRF) results.
5. Generate a cited answer and look at the exact evidence the model got.
6. Run the eval set and compare chunking/retrieval configs.
7. Finish with context engineering: retrieval is only one part of what the model sees.

## OpenAI config

Settings are loaded from `.env` with `python-dotenv`:

```dotenv
OPENAI_API_KEY=replace-with-your-openai-api-key
OPENAI_EMBEDDING_MODEL=text-embedding-3-small
OPENAI_CHAT_MODEL=gpt-5-mini
OPENAI_EMBEDDING_DIMENSIONS=
RAG_EVAL_USE_OPENAI=false
```

- `.env` is in `.gitignore`. Only commit `.env.example`.
- Embeddings come from OpenAI by default and are passed to Chroma explicitly for both indexing and queries.
- Collection names include the embedding model and dimensions, so vectors from different models never end up in the same collection.
- The final answer in notebook 03 uses the OpenAI Responses API.
- `RAG_EVAL_USE_OPENAI=false` skips the model call per eval question to save cost. Set it to `true` to evaluate real model answers.
- `RAG_EMBEDDER=hashing` switches to an offline hashing embedder for testing without an API key.

## Metrics

- Retrieval: Hit Rate@k, Recall@k, Precision@k, MRR, nDCG@k
- Context: evidence coverage and context precision, based on labeled evidence phrases
- Generation: token F1, answer relevance, citation validity, groundedness
- System: latency and context size

Groundedness here is a simple token-overlap check, not a real entailment model. Notebook 04 covers when you'd swap it for human labels or an LLM judge (and why that judge needs calibrating too).

## Layout

```text
data/raw/       source documents
data/eval/      labeled questions and evidence
notebooks/      shared core + demo notebooks
artifacts/      local Chroma DB and eval output (git-ignored)
```
