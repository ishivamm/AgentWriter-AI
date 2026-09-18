# AgentWriter-AI

AgentWriter-AI is a local technical article generator built with FastAPI and LangGraph. It turns a topic into a structured blog draft by deciding whether web research is needed, planning the article, writing section-by-section, and then merging the final markdown output.

This project is designed as a simple end-to-end writing workflow for developers who want to generate technical content from a single prompt in a browser UI.

## What the app does

- Accepts a technical topic from a web form
- Routes the request into a closed-book, hybrid, or open-book mode
- Uses Tavily to collect supporting evidence when research is needed
- Creates a section plan with goals, bullets, and target word counts
- Writes each section as a separate markdown block
- Merges the sections into a final blog article
- Optionally inserts image placeholders and generates diagram assets with Gemini
- Saves final markdown output under the outputs directory and generated images under images

## Architecture

The project is split into two main files:

- `app.py` — FastAPI app, Jinja templates, static assets, and the streaming UI
- `backend.py` — LangGraph workflow, state schema, router, research node, planner, section workers, reducer, and image generation pipeline

## Tech stack

- Python
- FastAPI
- Jinja2
- LangGraph
- LangChain
- Tavily search
- NVIDIA AI endpoint via `langchain_openai`
- Google GenAI for image generation
- PostgreSQL optional checkpointer support

## Project structure

```text
.
├── app.py                # FastAPI server and SSE streaming endpoint
├── backend.py            # LangGraph blog-writing workflow
├── requirements.txt      # Python package dependencies
├── LICENSE               # Project license
├── README.md             # Project documentation
├── outputs/              # Generated article output files
├── images/               # Generated or fallback image files
├── static/               # CSS and JavaScript for the UI
├── templates/            # HTML templates
└── .env                  # Local environment variables (not committed)
```

## Local setup

### 1. Clone the repository

```bash
git clone <your-repo-url>
cd AgentWriter-AI
```

### 2. Create and activate a virtual environment

```bash
python -m venv .venv
```

On macOS/Linux:

```bash
source .venv/bin/activate
```

On Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure environment variables

Create a `.env` file in the project root with the keys needed by the workflow:

```env
DATABASE_URL=postgresql://<user>:<password>@<host>:<port>/<database>
TAVILY_API_KEY=your_tavily_key
NVIDIA_API_KEY=your_nvidia_key
GOOGLE_API_KEY=your_google_ai_key
```

Notes:

- `DATABASE_URL` is optional. If it is missing, the app falls back to an in-memory LangGraph checkpointer.
- `TAVILY_API_KEY` is used for research-backed writing.
- `NVIDIA_API_KEY` is used by the active model client in `backend.py`.
- `GOOGLE_API_KEY` is required only if image generation is enabled.

### 5. Run the app

```bash
uvicorn app:app --reload
```

Then open the browser at:

```text
http://127.0.0.1:8000
```

## How the workflow works

1. The user submits a topic.
2. The router chooses whether the request is closed-book, hybrid, or open-book.
3. If needed, the research node calls Tavily to gather source material.
4. The orchestrator creates an article plan with section titles, goals, bullets, and target word counts.
5. Worker nodes generate each section in markdown.
6. The reducer combines the sections into a final article.
7. The app streams progress to the browser while the workflow runs.

## Output behavior

- Final markdown is saved under `outputs/<run_id>/blog.md`
- Generated article images are saved under `images/`
- A final blog draft is also written to the repository root when image generation is not requested

## Notes

- The app is designed to work as a local content-generation prototype rather than a production publishing platform.
- The current implementation prioritizes fast iteration and live browser feedback over full-scale production orchestration.
- PostgreSQL support is available for resumable execution, but the app still works with an in-memory fallback.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.