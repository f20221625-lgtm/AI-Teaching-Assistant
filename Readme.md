# 🎓 RAG AI Teaching Assistant

> This project implements a Retrieval-Augmented Generation (RAG) pipeline that combines semantic search with Large Language Models (LLMs) to generate accurate, context-aware, and knowledge-grounded responses from custom datasets.

The system retrieves the most relevant information from a document store using vector embeddings and feeds the retrieved context into an LLM to improve response quality, reduce hallucinations, and provide domain-specific answers.

---

## 🧠 What Is This?

This project lets you feed your own video content — lectures, tutorials, walkthroughs — into a local RAG pipeline. Once processed, you can ask natural language questions and get answers grounded in your own material.

**The pipeline:**

```
Videos → MP3 → Transcription (JSON) → Embeddings (Vectors) → LLM-powered Q&A
```

---

## 🚀 Quickstart

### Prerequisites

- Python 3.8+
- `ffmpeg` installed and on your PATH
- Required Python packages (install via `pip install -r requirements.txt`)
- An LLM API key (OpenAI, Anthropic, etc.) configured in your environment

---

## 📋 Step-by-Step Setup

### Step 1 — Add Your Videos

Place all your video files into the `videos/` folder at the root of the project.

```
rag-teaching-assistant/
└── videos/
    ├── lecture_01.mp4
    ├── lecture_02.mkv
    └── ...
```

Supported formats: `.mp4`, `.mkv`, `.avi`, `.mov`, `.webm` (any format supported by `ffmpeg`).

---

### Step 2 — Convert Videos to MP3

Run the conversion script to extract audio from all videos:

```bash
video_to_mp3.py
```

This uses `ffmpeg` under the hood to strip the audio track from each video and save it as an `.mp3` file in the `audio/` directory.

**Output:** `audio/<filename>.mp3` for each video in `videos/`

---

### Step 3 — Transcribe MP3 to JSON

Convert each audio file into a structured JSON transcript:

```bash
mp3_to_json.py
```

This runs speech-to-text (e.g. Whisper) on each `.mp3` file and saves a timestamped transcript as JSON.

**Output:** `jsons/<filename>.json` for each MP3

Each JSON file contains chunked transcript segments, e.g.:

```json
[
  { "start": 0.0, "end": 5.2, "text": "Welcome to today's lecture on neural networks." },
  { "start": 5.2, "end": 11.8, "text": "We'll be covering backpropagation in detail." }
]
```

---

### Step 4 — Generate Embeddings (Vectorise)

Process all JSON transcripts into a vectorised dataframe and save it as a `.joblib` pickle:

```bash
preprocess_json.py
```

This script:
- Loads and chunks all transcript JSON files
- Generates text embeddings for each chunk
- Stores the chunks + embeddings in a pandas DataFrame
- Serialises the result to `embeddings.joblib`

**Output:** `embeddings.joblib`

> 💡 **Tip:** You only need to re-run this step when you add new videos. It's the most compute-intensive step.

---

### Step 5 — Query Your Teaching Assistant

Load the vectorised data and start asking questions:

```bash
python ask.py "What is backpropagation and how does it work?"
```

Or launch the interactive interface:

```bash
python app.py
```

Under the hood, this:
1. Loads `embeddings/vectors.joblib` into memory
2. Embeds your query and performs a similarity search
3. Retrieves the most relevant transcript chunks
4. Builds a context-rich prompt and sends it to the LLM
5. Returns an answer grounded in your video content

---

## 📁 Project Structure

```
rag-teaching-assistant/
├── videos/                  # ← Put your raw video files here
├── audio/                   # Auto-generated MP3 files
├── transcripts/             # Auto-generated JSON transcripts
├── embeddings/              # Auto-generated vector store (.joblib)
├── video_to_mp3.py          # Step 2: Video → MP3
├── mp3_to_json.py           # Step 3: MP3 → JSON transcript
├── preprocess_json.py       # Step 4: JSON → Embeddings
├── ask.py                   # Step 5: CLI query interface
├── app.py                   # Step 5: Interactive UI (optional)
├── requirements.txt
└── README.md
```

---

## ⚙️ Configuration

Set the following environment variables before running:

| Variable | Description | Example |
|---|---|---|
| `OPENAI_API_KEY` | API key for embeddings + LLM | `sk-...` |
| `EMBEDDING_MODEL` | Embedding model to use | `text-embedding-3-small` |
| `LLM_MODEL` | LLM for answer generation | `gpt-4o` |
| `TOP_K` | Number of chunks to retrieve | `5` |

You can also set these in a `.env` file at the project root.

---

## 🔄 Adding New Content

To add new videos after the initial setup:

1. Drop the new files into `videos/`
2. Run `python video_to_mp3.py`
3. Run `python mp3_to_json.py`
4. Re-run `python preprocess_json.py` to rebuild the vector store

---

## 🛠 Tech Stack

| Component | Tool |
|---|---|
| Audio extraction | `ffmpeg` |
| Speech-to-text | OpenAI Whisper |
| Embeddings | OpenAI / Sentence Transformers |
| Vector storage | pandas + joblib |
| LLM | OpenAI GPT / Anthropic Claude |



