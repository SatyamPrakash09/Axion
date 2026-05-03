# 🔮 Axion — AI Research Agent with RAG, Vision & Voice

Axion is a multi-tool AI research agent that combines **Retrieval-Augmented Generation (RAG)** over local PDF documents with **real-time web search**, **computer vision**, **voice output**, and access to multiple external APIs — all orchestrated through a LangGraph agent running entirely on local LLMs via Ollama.

---

## ✨ Key Features

| Category | Capability |
|---|---|
| **RAG** | Ingest PDFs, chunk & embed them with HuggingFace, store in ChromaDB, query with semantic search |
| **Web Search** | Tavily advanced search for general web queries |
| **Vision** | Analyze images, capture webcam, detect objects, read text (OCR), compare images — powered by LLaVA 7B |
| **Voice** | Text-to-speech output using Kokoro TTS |
| **Jobs** | Real-time job/internship search via JSearch (RapidAPI) |
| **News** | Current news articles via Real-Time News Data (RapidAPI) |
| **Research** | Academic paper search via Semantic Scholar API |
| **Tech** | Hacker News top-story search |
| **Reference** | Wikipedia summary lookup |
| **Weather** | Live weather data via OpenWeatherMap |
| **Location** | Geocoding & place details via Nominatim (OpenStreetMap) |
| **Reasoning** | Deep multi-step reasoning over gathered context |

---

## 🏗️ Architecture

```
User Query
    │
    ▼
┌──────────────────────────────────┐
│        LangGraph Agent           │
│   (Qwen3:14b via Ollama)         │
│                                  │
│  System Prompt + Tool Router     │
│  Rolling Chat History (10 turns) │
└──────────┬───────────────────────┘
           │
     ┌─────┴─────┐
     ▼           ▼
 ┌────────┐  ┌────────────────────────────────────┐
 │ask_rag │  │        External Tools               │
 │(always │  │  search_web  · get_job · news_search│
 │ first) │  │  search_papers · hackernews_search   │
 │        │  │  wikipedia_search · weather_search   │
 │  PDF   │  │  location_search · analyze_image     │
 │ChromaDB│  │  capture_webcam · detect_objects     │
 │        │  │  compare_images · read_text_in_image │
 └────────┘  │  reason_deeply                      │
             └────────────────────────────────────┘
                        │
                        ▼
              ┌──────────────────┐
              │  Kokoro TTS      │
              │  Voice Response  │
              └──────────────────┘
```

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| **Language** | Python 3.10 |
| **Package Manager** | [uv](https://docs.astral.sh/uv/) |
| **Agent Framework** | LangChain + LangGraph |
| **Reasoning LLM** | Qwen3:14b (via Ollama) |
| **Vision LLM** | LLaVA:7b (via Ollama) |
| **Embeddings** | `sentence-transformers/all-MiniLM-L6-v2` (HuggingFace) |
| **Vector Store** | ChromaDB (persisted locally in `./rag_db`) |
| **PDF Loading** | PyPDF (via LangChain `PyPDFDirectoryLoader`) |
| **Text Splitting** | `RecursiveCharacterTextSplitter` (1000 chars, 200 overlap) |
| **Web Search** | Tavily Search API |
| **TTS** | Kokoro TTS + SoundDevice |
| **Vision** | OpenCV + base64 encoding → LLaVA |
| **External APIs** | RapidAPI (JSearch, Real-Time News), OpenWeatherMap, Nominatim, Semantic Scholar |

---

## 📁 Project Structure

```
rag_web_search/
├── main.ipynb           # Main notebook — agent setup, tools, and chat loop
├── files/               # Drop PDF documents here for RAG ingestion
│   ├── 1706.03762v7.pdf           # "Attention Is All You Need"
│   ├── Harry_Potter_Part1_Book.pdf
│   └── ... (7 PDFs tracked)
├── rag_db/              # ChromaDB persistent vector store
├── tracked_files.json   # MD5 hashes to detect new/changed PDFs
├── text.txt             # Sample text data (Renewable Energy report)
├── test.py              # Webcam connectivity test script
├── pyproject.toml       # Project metadata & dependencies
├── .env                 # API keys (not committed)
├── .gitignore
├── .python-version      # Python 3.10
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

- **Python 3.10+**
- **[uv](https://docs.astral.sh/uv/)** — fast Python package manager
- **[Ollama](https://ollama.com/)** — local LLM runtime
- A webcam (optional, for vision tools)

### 1. Clone & Install Dependencies

```bash
git clone "https://github.com/SatyamPrakash09/Axion.git"
cd rag_web_search
uv sync
```

### 2. Pull Required Ollama Models

```bash
ollama pull qwen3:14b
ollama pull llava:7b
```

### 3. Configure Environment Variables

Create a `.env` file in the project root:

```env
GEMINI_API_KEY=your_gemini_api_key
TAVILY_API_KEY=your_tavily_api_key
rapid_api_key=your_rapidapi_key
WEATHER_API_KEY=your_openweathermap_api_key
camera_index=0
```

| Variable | Source | Used For |
|---|---|---|
| `GEMINI_API_KEY` | [Google AI Studio](https://aistudio.google.com/) | (Optional) Gemini model fallback |
| `TAVILY_API_KEY` | [Tavily](https://tavily.com/) | Web search tool |
| `rapid_api_key` | [RapidAPI](https://rapidapi.com/) | Job search & News search |
| `WEATHER_API_KEY` | [OpenWeatherMap](https://openweathermap.org/api) | Weather tool |
| `camera_index` | Local device | Webcam device index (default `0`) |

### 4. Add PDFs for RAG

Drop PDF files into the `files/` directory. On the next run, the agent will automatically:

1. Detect new or modified PDFs via MD5 hash comparison
2. Load and split them into 1000-character chunks (200 overlap)
3. Embed with `all-MiniLM-L6-v2`
4. Store in the local ChromaDB vector store (`rag_db/`)

### 5. Run the Agent

Open `main.ipynb` in Jupyter / VS Code and run all cells. The final cell starts an interactive chat loop:

```
Axion is ready.
Vision model : llava:7b
Reasoning    : qwen3:14b
Type 'quit' to exit.

You: What does the Attention Is All You Need paper propose?
Axion: ...
```

---

## 🧰 Available Tools (15 Total)

### Document & Knowledge

| Tool | Description |
|---|---|
| `ask_rag` | Query the local PDF knowledge base (always called first) |
| `reason_deeply` | Multi-step reasoning over collected context |

### Web & Search

| Tool | Description |
|---|---|
| `search_web` | Tavily advanced internet search (20 results) |
| `wikipedia_search` | Wikipedia article summaries |
| `hackernews_search` | Search top 50 Hacker News stories by topic |
| `search_papers` | Semantic Scholar academic paper search |
| `news_search` | Real-time news articles (RapidAPI) |

### Vision

| Tool | Description |
|---|---|
| `analyze_image` | Describe/analyze an image file |
| `capture_webcam` | Capture and analyze a live webcam frame |
| `detect_objects` | Structured object inventory from an image |
| `compare_images` | Side-by-side comparison of two images |
| `read_text_in_image` | OCR — extract text from images |

### Utilities

| Tool | Description |
|---|---|
| `get_job` | Search internship/job listings (JSearch API) |
| `weather_search` | Current weather for any city (OpenWeatherMap) |
| `location_search` | Geocoding and place details (Nominatim) |

---

## 🔄 Agent Reasoning Flow

The agent follows a structured decision process for every query:

1. **Check Internal Knowledge** — Always calls `ask_rag` first to search local documents
2. **Enrich Externally** — Uses web/news/papers tools to verify or expand findings
3. **Match Intent to Tools** — Routes to specialized tools based on query type
4. **Cross-Verify** — For high-stakes claims, runs multiple independent tools
5. **Synthesize & Respond** — Combines all findings; uses `reason_deeply` for 3+ sources
6. **Voice Output** — Speaks the answer aloud using Kokoro TTS

---

## 💬 Example Queries

```
You: Search for machine learning internships in Bangalore
You: What's the weather in Tokyo?
You: Summarize the Attention Is All You Need paper
You: Analyze this image: C:/photos/chart.png
You: What can you see through the camera?
You: Find recent news about AI regulation
You: Search research papers on transformer architectures
```

---

## 📝 Notes

- **Incremental Ingestion** — Only new/modified PDFs are re-embedded (tracked via `tracked_files.json`)
- **Chat Memory** — Rolling window of the last 10 conversation turns (20 messages)
- **Fully Local LLMs** — Both reasoning and vision models run on-device via Ollama
- **Voice** — Kokoro TTS plays audio at 24kHz sample rate
- **Camera** — Set `camera_index` in `.env` to match your webcam device index

---

## 📄 License

This project is for educational and research purposes.
