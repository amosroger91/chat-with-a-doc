<div align="center">

# 📄💬 Chat with a Doc

### Talk to any PDF as if it were a person — **100% in your browser.**

No server. No API keys. No uploads. Nothing ever leaves your machine.

<br>

![Single File](https://img.shields.io/badge/single_file-index.html-4f8cff?style=for-the-badge)
![Runs in Browser](https://img.shields.io/badge/runs-100%25_in_browser-3fb568?style=for-the-badge)
![No Backend](https://img.shields.io/badge/backend-none-1f232c?style=for-the-badge)
![Privacy](https://img.shields.io/badge/your_data-never_leaves-e05656?style=for-the-badge)

<br>

**[ ▶ Try it live ](https://amosroger91.github.io/chat-with-a-doc/)**

</div>

---

## ✨ What is this?

A **single HTML file** you can drop on GitHub Pages that turns any PDF into a conversation. It runs a real large language model *and* a semantic search index entirely inside your browser tab — the LLM on **WebGPU**, the embeddings on **WebAssembly** — so you can interrogate a 500-page contract, textbook, or research paper without a single byte being sent to a server.

> Load a PDF → ask anything → get answers **with page citations**. That's it.

---

## 🧠 How it works

```
        ┌──────────────┐
        │  Your PDF    │   (stays in memory, never uploaded)
        └──────┬───────┘
               │  PDF.js extracts text, page by page
               ▼
        ┌──────────────┐
        │   Chunking   │   page-tagged, overlapping windows
        └──────┬───────┘
               │  embedded on the CPU (all-MiniLM-L6-v2, WASM)
               ▼
        ┌──────────────┐        ┌─────────────────────────┐
        │ Vector Index │ ◀────▶ │  IndexedDB cache         │
        └──────┬───────┘        │  (embed once, reuse fast)│
               │                └─────────────────────────┘
               │  the model reads a spread of the doc and
               │  derives an overview + sees the file's metadata
               ▼
   your question embedded → top-K most similar chunks retrieved
               ▼
        ┌──────────────┐
        │  Llama-3.2   │   (WebLLM, runs on your GPU)
        │   answers    │   grounded in retrieved chunks + overview,
        └──────────────┘   with page citations
```

It's **Retrieval-Augmented Generation (RAG)**, fully local. The language model never sees the whole document — only the handful of chunks relevant to your question, plus a short overview it derived itself and the file's metadata. That's the trick that lets a small in-browser model reason over a huge PDF *and* still answer broad questions like "what is this?" or "what's on the agenda?".

---

## 🚀 Features

| | |
|---|---|
| 🔒 **Totally private** | The LLM and the search index both run locally. Your document and questions never touch a network. |
| 📁 **Single file** | The entire app is one `index.html`. Host it anywhere static — GitHub Pages, an S3 bucket, a USB stick. |
| 🧭 **Knows the document** | On load, the model reads a spread of the PDF to derive an overview, and is given the filename + embedded metadata — so vague questions ("what's the June agenda about?") just work. |
| 📑 **Cited answers** | Every response shows the page numbers it drew from — hover a chip to see the source text. |
| ✍️ **Markdown replies** | Answers render with headings, lists, bold, and code — not a wall of plain text. |
| 💬 **Real chat** | Multi-turn conversation with history, plus suggested prompts to get you started. |
| 🌊 **Streaming index** | Pages extract, chunk, and embed in an overlapping pipeline with live `page 312/500` progress — the tab never freezes. |
| 💾 **Durable caching** | Each PDF's index is stored in IndexedDB and the ~2 GB model is cached with persistent storage, so re-opening a doc — or the whole app — is near-instant. |
| 🧠 **Model transparency** | A header badge opens a panel explaining the model: who built it, how it was trained, why it matters, and links to benchmarks. |

---

## 🏎️ Built for big documents

Large PDFs are where naive "stuff the text into the prompt" approaches fall apart. This one is engineered for them:

- **Overlapping pipeline** — PDF parsing (PDF.js worker) runs concurrently with embedding, page by page, so a 500-page doc indexes smoothly with live progress instead of a frozen tab.
- **GPU stays with the LLM** — embeddings run on the CPU/WASM so they never compete with the language model for VRAM (which otherwise crashes the shared GPU device). The embedding model is tiny, so this is plenty fast.
- **Persistent index cache** — documents are keyed by a content hash; the index *and* the derived overview are stored locally, so the second time you open a file there's no re-indexing.
- **RAG by design** — retrieval keeps the model's context tiny and constant regardless of document size, so quality and speed don't degrade as the PDF grows.

---

## 🖥️ Use it

**Online:** just open **[the live app](https://amosroger91.github.io/chat-with-a-doc/)** in Chrome or Edge.

**Locally:** because it uses ES module imports, serve it over HTTP (not `file://`):

```bash
# clone, then:
python -m http.server 8000
# open http://localhost:8000
```

Then click **📂 Load PDF**, wait for the status dot to turn green, and start chatting.

> ⏳ **First load** downloads the language model (~2 GB) and caches it durably, so you only wait once. A **WebGPU-capable browser (Chrome/Edge)** is required. The footer tells you whether the model was actually cached — if your browser won't keep it (common in Private/Incognito windows or with very low free disk), it may re-download.

---

## 🧩 Tech stack

| Layer | Tech |
|---|---|
| In-browser LLM | [WebLLM](https://github.com/mlc-ai/web-llm) · `Llama-3.2-3B-Instruct` (on WebGPU) |
| Embeddings | [Transformers.js](https://github.com/huggingface/transformers.js) · `all-MiniLM-L6-v2` (on WASM) |
| PDF parsing | [PDF.js](https://github.com/mozilla/pdf.js) |
| Markdown | [marked](https://github.com/markedjs/marked) |
| Vector store + cache | Plain JS cosine search + IndexedDB |
| Build step | **None.** It's one HTML file. |

---

## ⚠️ Limits & notes

- **Scanned PDFs** (image-only, no text layer) have no extractable text — no OCR yet.
- **WebGPU required** for the LLM. First load is heavy but cached afterwards.
- **Durable caching** needs the browser to grant persistent storage and enough free disk; Private/Incognito windows will re-download the model each session.
- The local-cache hash uses `crypto.subtle`, which needs a secure origin (works on `localhost` and GitHub Pages; gracefully skips caching on plain `file://`).

---

<div align="center">

Built as a single file, because the best web app is one you can read top to bottom. 🤍

</div>
