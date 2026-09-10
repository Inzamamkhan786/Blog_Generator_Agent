# 🤖 Blog Generator Agent (BWA)

An **autonomous, multi-agent AI system** that researches, outlines, writes, and visually illustrates high-quality technical blog posts — fully end-to-end, from a single topic input to a polished Markdown article with AI-generated diagrams.

---

## ✨ Features

- 🔍 **Intelligent Routing** — Automatically determines if web research is needed based on your topic
- 🌐 **Real-time Web Research** — Uses Tavily Search API to pull live, relevant, deduplicated evidence
- 🧠 **Structured Planning** — Orchestrator creates a full blog outline with 5–9 tasks, tone, audience, and word counts
- ⚡ **Parallel Section Writing** — Worker agents write all sections simultaneously using LangGraph fan-out
- 🖼️ **AI Image Generation** — Google Gemini generates custom technical diagrams embedded in the article
- 📥 **Downloadable Output** — Export the final Markdown file or a `.zip` bundle with images
- 🖥️ **Streamlit UI** — Clean, real-time web interface with live progress tracking

---

## 🏗️ Architecture & Workflow

The core engine is a **LangGraph `StateGraph`** that orchestrates the following pipeline:

```
[START]
   │
   ▼
[Router Node] ──── (Needs Research?) ────► [Yes] ──► [Research Node (Tavily)]
   │                                                        │
   └──────────────────── [No] ──────────────────────────────┘
                              │
                              ▼
                     [Orchestrator Node]
                              │
                      (Fan-out / Parallel)
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
   [Worker 1]            [Worker 2]            [Worker N]
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              │
                              ▼
                     [Reducer Subgraph]
                              │
             ┌────────────────┴────────────────┐
             ▼                                 ▼
     (merge_content) ──► (decide_images) ──► (generate_and_place_images)
                              │
                              ▼
                           [END]
```

### 🔵 Node Descriptions

| Node | Role | Description |
|------|------|-------------|
| **Router** | `router_node` | Classifies the topic into `closed_book`, `hybrid`, or `open_book` mode; generates targeted search queries |
| **Research** | `research_node` | Runs Tavily searches, parses results into `EvidenceItem` objects, deduplicates by URL, filters by recency |
| **Orchestrator** | `orchestrator_node` | Acts as a senior technical writer; creates a structured `Plan` with sections, goals, word counts, and constraints |
| **Workers** | `worker_node` | Parallel agents that each write one section of the blog, respecting word budgets, citation rules, and code requirements |
| **Reducer** | `reducer_subgraph` | Merges all sections → decides image placement → calls Gemini to generate and embed diagrams |

### 🟡 Research Modes

| Mode | When Used | Recency Window |
|------|-----------|----------------|
| `closed_book` | Evergreen concepts (e.g., "What is Recursion?") | None |
| `hybrid` | Core concepts + up-to-date examples/tools | Last 45 days |
| `open_book` | Time-sensitive news, weekly roundups | Last 7 days |

---

## 🗂️ Project Structure

```
blog-writing-agent/
│
├── bwa_backend.py                    # LangGraph state machine, nodes, Pydantic models
├── bwa_frontend.py                   # Streamlit UI, stream handler, Markdown preview, ZIP bundler
│
├── 1_bwa_basic.ipynb                 # Notebook: Basic agent setup walkthrough
├── 2_bwa_improved_prompting.ipynb    # Notebook: Advanced prompt engineering techniques
├── 3_bwa_research.ipynb              # Notebook: Tavily search integration
├── 4_bwa_research_fine_tuned.ipynb   # Notebook: Fine-tuning research extraction
├── 5_bwa_image.ipynb                 # Notebook: Gemini image generation experiments
│
├── images/                           # Output directory for AI-generated diagrams (PNG/JPEG)
├── .env                              # API keys — ⚠️ GIT IGNORED, never commit this
└── .gitignore                        # Ignores .env, __pycache__, etc.
```

---

## 🧩 Data Schemas

All components communicate through a unified **Pydantic + TypedDict** state:

| Schema | Purpose |
|--------|---------|
| `State` | Graph-wide state: topic, mode, queries, evidence, plan, sections, final markdown |
| `Task` | Single section spec: title, goal, bullets, word count, code/citation flags |
| `Plan` | Blog metadata: title, audience, tone, blog kind, list of tasks |
| `EvidenceItem` | Web result: title, URL, published date, snippet, source |
| `RouterDecision` | Routing output: mode, research flag, search queries |
| `ImageSpec` | Image metadata: placeholder, filename, alt text, caption, generation prompt |
| `GlobalImagePlan` | Image planning output: markdown with placeholders + list of `ImageSpec` |

---

## 🖥️ Frontend Interface (`bwa_frontend.py`)

Built with **Streamlit**, the UI provides:

- **📝 Topic Input** — Enter a blog topic and target date
- **⚡ Live Progress** — Real-time execution step tracking (current node, evidence count, tasks generated)
- **🧩 Plan Tab** — Structured outline, task breakdown table, and metadata
- **🔎 Evidence Tab** — Table of researched sources, URLs, and publication dates
- **📝 Markdown Tab** — Formatted article preview with inline images and download buttons (`.md` + `.zip`)
- **🖼️ Images Tab** — Gallery of all AI-generated diagrams with zip export
- **🧾 Logs Tab** — Full technical event stream log
- **📂 Sidebar** — Load previously generated blog files directly into the preview

---

## ⚙️ Setup & Installation

### 1. Clone the repository

```bash
git clone https://github.com/Inzamamkhan786/Blog_Generator_Agent.git
cd Blog_Generator_Agent/blog-writing-agent
```

### 2. Create a virtual environment (recommended)

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS / Linux
source .venv/bin/activate
```

### 3. Install dependencies

```bash
pip install langgraph langchain-openai langchain-community tavily-python google-genai streamlit pandas python-dotenv
```

### 4. Configure API keys

Create a `.env` file in the `blog-writing-agent/` directory:

```env
OPENAI_API_KEY=sk-...
TAVILY_API_KEY=tvly-...
GOOGLE_API_KEY=AIza...
```

> ⚠️ **Never commit your `.env` file.** It is already listed in `.gitignore`.

### 5. Run the application

```bash
streamlit run bwa_frontend.py
```

Open your browser at `http://localhost:8501`.

---

## 🔑 Required API Keys

| API Key | Used For | Get It |
|---------|----------|--------|
| `OPENAI_API_KEY` | GPT-4 routing, planning & section writing | [platform.openai.com](https://platform.openai.com) |
| `TAVILY_API_KEY` | Real-time web search & evidence collection | [tavily.com](https://tavily.com) |
| `GOOGLE_API_KEY` | Gemini image generation (`gemini-2.5-flash-image`) | [aistudio.google.com](https://aistudio.google.com) |

---

## 📓 Notebook Progression

The Jupyter notebooks document the **iterative development journey** of this agent:

| Notebook | Topic |
|----------|-------|
| `1_bwa_basic.ipynb` | Basic single-agent blog writer setup |
| `2_bwa_improved_prompting.ipynb` | Prompt engineering for better quality output |
| `3_bwa_research.ipynb` | Integrating Tavily for live web research |
| `4_bwa_research_fine_tuned.ipynb` | Fine-tuning evidence extraction and filtering |
| `5_bwa_image.ipynb` | Gemini image generation and placement experiments |

---

## 🛠️ Tech Stack

| Technology | Role |
|-----------|------|
| [LangGraph](https://langchain-ai.github.io/langgraph/) | State machine orchestration, fan-out parallelism |
| [LangChain](https://www.langchain.com/) | LLM abstraction, tools |
| [OpenAI GPT-4](https://openai.com/) | Routing, planning, writing |
| [Tavily Search](https://tavily.com/) | Real-time web evidence |
| [Google Gemini](https://deepmind.google/technologies/gemini/) | AI image generation |
| [Streamlit](https://streamlit.io/) | Web frontend UI |
| [Pydantic](https://docs.pydantic.dev/) | Schema validation |
| [Python-dotenv](https://pypi.org/project/python-dotenv/) | Environment variable management |

---

## 📄 Sample Output

Generated blog posts are saved as `.md` files alongside their AI-generated images in the `images/` folder. Example outputs included in this repo:

- `gta_6_latest_developments_and_industry_impact_-_july_2026_news_roundup.md`
- `gta_6_news_roundup_latest_developments_and_insights.md`

---

## 🤝 Contributing

Pull requests are welcome! For major changes, please open an issue first to discuss what you'd like to change.

---

## 📜 License

This project is open source. See the repository for license details.

---

*Built with ❤️ using LangGraph, OpenAI, Tavily, and Google Gemini.*
