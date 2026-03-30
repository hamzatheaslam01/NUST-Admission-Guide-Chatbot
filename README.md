# NUST Admission Guide Chatbot

RAG-powered chatbot for NUST admissions with a modern web UI, local retrieval, and optional local LLM generation.

## Highlights

- Web app with chat interface, topic shortcuts, confidence chips, and latency display.
- Decision Mode for advice/comparison questions (for example: "NET or SAT?").
- Voice input (speech-to-text) and optional voice reply (text-to-speech) in supported browsers.
- Runtime Analytics dashboard (top-bar button) to visualize memory and cache usage.
- Intent-aware retrieval (for example, separates NET application vs NET rechecking).
- Cached FAISS index for fast startup and response.
- Local-first stack, no cloud API requirement for core functionality.

## What This Project Does

The chatbot answers admission-related questions using retrieval over `nust_data.txt` and then composes a response.

Processing flow:

1. User sends query from web UI.
2. `chatbot.py` detects intent (normal, decision, NET rechecking, etc.).
3. `rag.py` retrieves relevant chunks using hybrid scoring.
4. Answer is generated from retrieved context (with response sanitation).
5. UI renders formatted output with trust metadata.

## Project Structure

```text
app.py                 Flask server and API routes
chatbot.py             Answer pipeline, intent logic, decision mode
rag.py                 Retriever + FAISS index + scoring
nust_data.txt          Knowledge base used by RAG
templates/index.html   Main web UI
static/nust.png        Logo used in the UI
.cache/                Auto-generated retrieval cache
```

## Setup

### 1. Create and activate a virtual environment (recommended)

Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Prepare knowledge base

This project depends explicitly on `nust_data.txt`.

- Update `nust_data.txt` directly when you want to add or change knowledge.
- Keep topic blocks structured (see format below) for best retrieval quality.

### 4. Run the web app

```bash
python app.py
```

Open:

```text
http://localhost:5000
```

## Salient Features

### 1. Decision Mode (Advisor-style responses)

Triggers on phrases like:

- "should I"
- "which is better"
- "what should I choose"
- "NET or SAT"
- "recommend"
- "confused"

Behavior:

- Returns `[Decision Guide]` format with Option 1/2 Pros and Cons.
- Provides a recommendation with concise reasoning.
- Asks a clarification question if options are unclear.
- Uses only retrieved context.

### 2. Intent-aware disambiguation

Prevents common confusion in admissions queries:

- NET application vs NET rechecking are handled separately.
- Generic NET overview avoids architecture-only composition unless requested.

### 3. Voice controls in UI

- `Mic` icon button for speech-to-text input.
- `Voice On/Off` toggle to read bot answers aloud.
- Graceful fallback if browser does not support speech APIs.

### 4. Retrieval quality and performance

- Hybrid score:

```text
0.70 * semantic + 0.30 * lexical + topic boost
```

- MD5-based cache invalidation when `nust_data.txt` changes.
- Query embedding cache for repeated prompts.

### 5. Runtime Analytics (for constraints demo)

- Access from the top navigation bar via `Analytics`.
- Visualizes:
	- Process memory (RSS and peak)
	- Knowledge-base file size
	- Retriever cache size
	- Document/topic counts and query-cache size
	- App uptime and RAM target

## API Endpoints

- `GET /` : chat UI
- `POST /chat` : ask chatbot
- `GET /debug/ui` : template/debug metadata
- `GET /analytics` : runtime memory/cache metrics for dashboard

## Useful Chat Commands

- `/clear` clears conversation history.
- `/topics` lists known topics.
- `/faq` returns quick FAQ response.
- `/contact` returns contact information snippet.

## Knowledge Base Format

Each chunk in `nust_data.txt` should follow:

```text
[TOPIC: TOPIC NAME]
Source: SOURCE NAME

Fact line 1
Fact line 2
Fact line 3

---
```

Tips:

- Keep each chunk focused on one concept.
- Include clear definitional lines for major topics.
- Avoid mixing unrelated topics in one block.

## Troubleshooting

### UI changes are not visible

- Edit `templates/index.html` (not root `index.html`).
- Hard refresh browser (`Ctrl+F5`).
- Restart `python app.py` after backend changes.

### Logo not showing

- Ensure image exists at `static/nust.png`.
- Use `/static/nust.png` in template image paths.

### Answers are cut off or too short

- Ensure latest `chatbot.py` is running.
- Restart server after code changes.

### Wrong topic appears in answer

- Improve topic blocks in `nust_data.txt`.
- Rebuild retrieval cache by deleting `.cache/` and restarting.

### Analytics memory shows `0.00 MB`

- Restart the Flask server after pulling code updates.
- Ensure you are running `python app.py` from the project virtual environment.

## Notes

- This project is for guidance and demo use.
- Always verify final admission requirements from official NUST channels.
