# Beca 18 RAG Chatbot

## Project Description & Research Question

This project builds a Retrieval-Augmented Generation (RAG) chatbot that answers questions about the Beca 18 scholarship program regulations exclusively from the official PDF document (Resolución Directoral Ejecutiva N.° 033-2026-MINEDU/VMGI-PRONABEC). The system refuses to answer off-topic questions.

**Research question:** Can a RAG system grounded in a single regulatory document reliably answer student queries about Beca 18 eligibility, modalities, stipends, obligations, and conditions for losing the scholarship — while correctly rejecting unrelated queries?

## Data

Place the official PDF in `data/` before running the notebook (not committed — see `.gitignore`):

| File | Source |
|------|--------|
| `data/beca18_reglamento.pdf` | Download from gob.pe/institucion/pronabec/normas-legales/7778068-033-2026-minedu-vmgi-pronabec |

## API Key Setup

1. Get a free Gemini API key from Google AI Studio.
2. Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```
3. Replace `your_api_key_here` with your actual key. **Never commit `.env`.**

## Dependencies & Installation

Python 3.9+ is required. Install all dependencies with:

```bash
pip install -r requirements.txt
```

Libraries used: `pypdf`, `tiktoken`, `langchain-text-splitters`, `google-genai`, `chromadb`, `ipywidgets`, `tqdm`, `python-dotenv`.

## How to Run the Notebook

1. Complete the Data and API Key setup steps above.
2. Open the notebook:
   ```bash
   jupyter notebook notebooks/beca18_rag_chatbot.ipynb
   ```
3. Run all cells top to bottom using **Kernel → Restart & Run All**.
4. The first run will index the PDF into ChromaDB (`chroma_db_beca18/`). Subsequent runs reuse the existing index automatically (idempotent).
5. The interactive chat UI appears at the end of the notebook (Step 7).

> **Google Colab:** The first cell installs all dependencies automatically. Upload `beca18_reglamento.pdf` to the `data/` folder and set `GEMINI_API_KEY` as a Colab secret or in a `.env` file.

## Pipeline Summary

| Step | Description |
|------|-------------|
| 0 | Load `.env`, initialize Gemini client, verify paths |
| 1 | Extract text page-by-page with `[PAGE N]` markers; clean whitespace; report character/word counts |
| 2 | Count tokens with `tiktoken` (cl100k_base); justify 400-token chunks; split with `RecursiveCharacterTextSplitter` + metadata |
| 3 | Implement `embed_documents()` and `embed_query()` with exponential backoff for rate limits |
| 4 | Create persistent ChromaDB collection (cosine distance); idempotent indexing |
| 5 | `semantic_search(question, k)` → returns text, metadata, distance; tested with sample query |
| 6 | `answer_with_context(question, k)` via `gemini-2.5-flash`; tested with 5 on-topic + 1 off-topic question |
| 7 | Interactive `ipywidgets` UI with k slider and source accordion |

## Key Findings

The RAG system demonstrates that a well-structured regulatory PDF can be effectively chunked and indexed for semantic retrieval, with the `RETRIEVAL_DOCUMENT` / `RETRIEVAL_QUERY` dual task-type distinction in `gemini-embedding-001` producing more precise results than generic embeddings. The strict system prompt successfully prevents hallucination by forcing the model to cite specific page numbers and return a fixed refusal message for off-topic queries. The persistent ChromaDB index eliminates redundant API calls on subsequent sessions, making the system practical for repeated use without consuming free-tier quota.
