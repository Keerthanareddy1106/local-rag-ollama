# local-rag-ollama

A minimal, fully offline RAG (Retrieval-Augmented Generation) pipeline in about 40 lines of Python. It answers questions about your Markdown documents using [Ollama](https://ollama.com), NumPy and cosine similarity, with no vector database and no frameworks.

The included demo runs on summary notes of Morgan Stanley's *2026 Global Economic Outlook*.

## How it works

1. **Load:** reads every `.md` file in `docs/` and splits it into chunks on blank lines.
2. **Embed:** converts each chunk into a vector with `nomic-embed-text` (via Ollama).
3. **Retrieve:** embeds your question and picks the top 3 chunks by cosine similarity (NumPy).
4. **Generate:** sends those chunks and your question to `qwen3:4b-instruct`, which is told to answer **only** from the context, or say "I don't know".

## Tech stack

- Python
- [Ollama](https://ollama.com) (local embeddings and chat)
- NumPy
- Requests
- [uv](https://docs.astral.sh/uv/) for dependency management

## Prerequisites

- [Ollama](https://ollama.com/download) installed and running (default: `http://localhost:11434`)
- [uv](https://docs.astral.sh/uv/getting-started/installation/) installed
- Python 3.14+ (as pinned in `pyproject.toml`)

## Setup

```bash
# 1. Clone the repo
git clone https://github.com/YOUR-USERNAME/local-rag-ollama.git
cd local-rag-ollama

# 2. Pull the models
ollama pull nomic-embed-text
ollama pull qwen3:4b-instruct

# 3. Install dependencies
uv sync
```

## Usage

Put your Markdown files in the `docs/` folder (the demo file is already there), then run:

```bash
uv run rag.py
```

The script indexes your documents, then opens a prompt:

```
Indexed 42 chunks

Ask: What is Morgan Stanley's 2026 U.S. GDP forecast?
```

Press `Ctrl+C` to exit.

## Sample questions for the demo

The demo document can answer:

- What is Morgan Stanley's 2026 U.S. GDP forecast?
- What path do they expect for the Federal Reserve?
- What are the three alternative scenarios?
- Which regions are expected to see inflation below target?
- What happens in the mild U.S. recession scenario?

Try a question the document doesn't cover, such as *"What will Bitcoin do in 2026?"*. The model should reply **"I don't know"**, which shows the context-only guardrail working.

## Project structure

```
local-rag-ollama/
├── docs/
│   └── morgan-stanley-2026-global-economic-outlook-summary.md
├── rag.py
├── pyproject.toml
├── uv.lock
├── .python-version
└── README.md
```

## Configuration

Edit the constants at the top of `rag.py`:

| Setting | Default | Purpose |
|---|---|---|
| `OLLAMA` | `http://localhost:11434` | Ollama server URL |
| `EMBED_MODEL` | `nomic-embed-text` | Embedding model |
| `CHAT_MODEL` | `qwen3:4b-instruct` | Answer-generation model |

You can also change `k` in `retrieve()` to pull more or fewer chunks per question.

## Limitations

- **Simple chunking:** documents are split on blank lines, so headings can become tiny standalone chunks. Splitting by heading, or attaching each heading to the paragraph below it, would improve retrieval.
- **In-memory index:** embeddings are recomputed on every start and are not saved to disk.
- **No chat history:** each question is answered independently.
- **Small local model:** answer quality depends on the model you choose.

## Attribution and disclaimer

The demo document in `docs/` is my own summary notes of Morgan Stanley Research's [2026 Economic Outlook: Moderate Growth With a Range of Possibilities](https://www.morganstanley.com/insights/articles/global-economic-outlook-2026) (published November 19, 2025). It is not a copy of the article. For the full text, charts and disclosures, please read the original.

The forecasts are a snapshot from the publication date and are **not current figures or financial advice**. This project is a technical demo, not affiliated with or endorsed by Morgan Stanley.
