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

A **single HTML file** you can drop on GitHub Pages that turns any PDF into a conversation. It runs a real large language model *and* a semantic search index entirely inside your browser tab using WebAssembly and WebGPU — so you can interrogate a 500-page contract, textbook, or research paper without a single byte being sent to a server.

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
               │  embedded on your GPU (all-MiniLM-L6-v2)
               ▼
        ┌──────────────┐        ┌─────────────────────────┐
        │ Vector Index │ ◀────▶ │  IndexedDB cache         │
        └──────┬───────┘        │  (embed once, reuse fast)│
               │                └─────────────────────────┘
   your question embedded, top-K most similar chunks retrieved
               ▼
        ┌──────────────┐
        │  Llama-3.2   │   (WebLLM, runs on your GPU)
        │   answers    │   grounded in retrieved chunks + page cites
        └──────────────┘
```

It's **Retrieval-Augmented Generation (RAG)**, fully local. The language model never sees the whole document — only the handful of chunks relevant to your question. That's the trick that lets a small in-browser model reason over a huge PDF.

---

## 🚀 Features

| | |
|---|---|
| 🔒 **Totally private** | The LLM and the search index both run locally. Your document and questions never touch a network. |
| 📁 **Single file** | The entire app is one `index.html`. Host it anywhere static — GitHub Pages, an S3 bucket, a USB stick. |
| 📑 **Cited answers** | Every response shows the page numbers it drew from — hover a chip to see the source text. |
| 💬 **Real chat** | Multi-turn conversation with history, so follow-up questions just work. |
| ⚡ **GPU-accelerated** | Embeddings build on WebGPU (5–10× faster than CPU), with automatic WASM fallback. |
| 🌊 **Streaming index** | Pages extract, chunk, and embed in an overlapping pipeline with live progress — the tab never freezes. |
| 💾 **Smart caching** | Each PDF is embedded once and stored in IndexedDB. Re-open the same file and it loads **instantly**. |

---

## 🏎️ Built for big documents

Large PDFs are where naive "stuff the text into the prompt" approaches fall apart. This one is engineered for them:

- **WebGPU embeddings** — the index builds on your graphics card when available, freeing the CPU and slashing indexing time. Falls back to WASM/q8 automatically.
- **Overlapping pipeline** — PDF parsing (in a worker thread) runs concurrently with GPU embedding, page by page, so a 500-page doc indexes smoothly with `page 312/500` progress instead of a frozen tab.
- **Persistent cache** — documents are keyed by a content hash and stored locally. The second time you open a file, indexing is skipped entirely.
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

Then click **📄 Load PDF**, wait for the status dot to turn green, and start chatting.

> ⏳ **First load** downloads the language model (~2 GB) and caches it. Subsequent loads are fast. A **WebGPU-capable browser (Chrome/Edge)** is recommended.

---

## 🧩 Tech stack

| Layer | Tech |
|---|---|
| In-browser LLM | [WebLLM](https://github.com/mlc-ai/web-llm) · `Llama-3.2-3B-Instruct` |
| Embeddings | [Transformers.js](https://github.com/huggingface/transformers.js) · `all-MiniLM-L6-v2` |
| PDF parsing | [PDF.js](https://github.com/mozilla/pdf.js) |
| Vector store + cache | Plain JS cosine search + IndexedDB |
| Build step | **None.** It's one HTML file. |

---

## ⚠️ Limits & notes

- **Scanned PDFs** (image-only, no text layer) have no extractable text — no OCR yet.
- **WebGPU required** for the LLM. First load is heavy but cached afterwards.
- The local-cache hash uses `crypto.subtle`, which needs a secure origin (works on `localhost` and GitHub Pages; gracefully skips caching on plain `file://`).

---

<div align="center">

Built as a single file, because the best web app is one you can read top to bottom. 🤍

</div>
