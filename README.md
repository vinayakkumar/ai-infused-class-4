# Class 4 · RAG — Code Files

One file per code block in the HTML guide.

```
embeddings.py   ←→  Block 16  · cosine similarity demo
chunk.py        ←→  Block 17  · chunking snippet (teaching only)
index.py        ←→  Block 19  · step 1 — build vector store
rag.py          ←→  Block 19  · step 2 — retrieve + answer
rerank.py       ←→  Block 20  · reranking snippet (teaching only)
pdf_chat.py     ←→  Block 21  · shippable Gradio app
```

## Setup

```bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Create a `.env` file with your Groq API key (free at console.groq.com):

```
GROQ_API_KEY=your_key_here
```

## Run order

```bash
python embeddings.py        # no API key needed — local sentence-transformers
python index.py             # builds ./chroma_db  (run once)
python rag.py               # queries ./chroma_db via Groq
python pdf_chat.py          # Gradio app — upload any PDF and chat with it
```

## How pdf_chat.py retrieves answers

Uses a two-stage retrieval pipeline for better accuracy:

1. **Bi-encoder** (`all-MiniLM-L6-v2`) fetches 20 candidate chunks from Chroma via vector similarity.
2. **Cross-encoder reranker** (`ms-marco-MiniLM-L6-v2`) scores each (question, chunk) pair precisely and keeps the top 5.
3. Top 5 chunks are stuffed into the prompt and sent to **Groq** (`qwen/qwen3.8-27b`).

`chunk.py` and `rerank.py` are teaching snippets — not standalone scripts.

## Key Changes

### LLM provider: OpenAI → Groq
- `rag.py` and `pdf_chat.py` switched from `langchain_openai / ChatOpenAI / gpt-4o-mini` to `langchain_groq / ChatGroq / qwen/qwen3.8-27b`.
- Requires `GROQ_API_KEY` instead of `OPENAI_API_KEY`.

### requirements.txt — missing packages added
Three packages were missing and caused `ModuleNotFoundError` on a fresh install:

| Added | Needed by |
|---|---|
| `langchain-core` | `langchain_core.prompts` in `rag.py`, `pdf_chat.py` |
| `langchain-text-splitters` | `langchain_text_splitters` in `chunk.py`, `index.py`, `pdf_chat.py` |
| `numpy` | `numpy` in `embeddings.py` |

### pdf_chat.py — better retrieval (fixes missing answers)
Before: `similarity_search(k=4)` — only 4 chunks, wrong pages returned for many questions.

After: two-stage pipeline —
1. Fetch 20 candidates via vector similarity.
2. Rerank with CrossEncoder (`ms-marco-MiniLM-L6-v2`), keep top 5.

This fixes answers like "who is the author", "Introduction", etc. that were previously returning "I couldn't find that in the document."
