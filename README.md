# Enterprise Structured Ingestion Firewall

A beginner-friendly AI project. You chat with an AI assistant in a React
web app, and for every message the backend:

1.  **Week 1 -- Understands it.** An LLM sorts the message into a fixed
    category (payment issue, order tracking, greeting...) and returns
    structured JSON. Pydantic checks that JSON before anything else uses
    it.
2.  **Week 2 -- Answers it.** A second LLM call uses that checked JSON
    to write a helpful reply in Markdown.
3.  **Week 3 -- Remembers it.** Every conversation is saved in a
    **SQLite** database, so follow-up questions ("what about my other
    order?") make sense and you can reopen old chats.

The "firewall" idea: the AI's output is not trusted until it passes a
strict schema. If the model returns a category that doesn't exist or a
missing field, the app rejects it.

## What you will learn

-   Building an API with **FastAPI**
-   Calling a cloud LLM (**Groq**) from Python
-   Getting **structured output** from an LLM with **Instructor**
-   **Validating** data with **Pydantic**
-   Saving chat history in **SQLite** (built into Python, no server
    needed)
-   Building a **React + Vite** frontend and connecting it to an API

## Key words (quick glossary)

  -----------------------------------------------------------------------
  Word                                Meaning in this project
  ----------------------------------- -----------------------------------
  **LLM**                             Large Language Model, the AI that
                                      reads and writes text. We use one
                                      hosted by Groq.

  **API**                             The backend URLs the frontend
                                      calls, e.g.Â `/api/analyze`.

  **FastAPI**                         The Python library that runs our
                                      API.

  **Pydantic**                        Checks that data has the right
                                      fields and allowed values.

  **Instructor**                      Makes the LLM reply in a shape that
                                      matches our Pydantic model.

  **Schema**                          The rules for what valid data looks
                                      like.

  **Intent**                          What the user wants,
                                      e.g.Â `payment_issue`.

  **Session ID**                      A unique ID for one conversation,
                                      so the backend knows which history
                                      to use.

  **SQLite**                          A database stored in a single file
                                      (`backend/data/chat.db`).

  **React / Vite**                    React builds the UI from
                                      components; Vite runs it in
                                      development and builds it for
                                      production.

  **`.env` file**                     A private file holding your API
                                      key. Never share or commit it.

  **venv**                            A private Python environment just
                                      for this project.
  -----------------------------------------------------------------------

## How a message flows through the app

``` text
You type in the React app (http://localhost:5173)
        |
        v
POST /api/analyze { message, session_id? } --> FastAPI backend (port 8000)
                                                  |
    1. Check the input                         schemas/ingestion.py
    2. WEEK 1: classify it                     services/llm_service.py       -- LLM call #1
         +-- Pydantic validates the JSON
    3. Estimate tokens                         utils/token_counter.py
    4. WEEK 3: load recent history             services/database_service.py -- SQLite
    5. WEEK 2: write a reply                   services/response_service.py -- LLM call #2
    6. WEEK 3: save both messages               services/database_service.py -- SQLite
                                                  |
        <-- JSON: intent details + response_markdown + session_id
        |
The chat shows the answer; the right panel shows intent, risk and priority
```

## Workshop output screenshots

The screenshots below show the outputs produced by the running
application during testing.

### Week 1 -- Enterprise Structured Ingestion Firewall

The message is classified as a payment issue. The application displays
the detected intent, risk profile, priority, action requirement, and
estimated input tokens.

![Week 1 -- Message classification
output](submission-screenshots/Week-1/week-1-classification.png)

### Week 2 -- Intent-Based AI Response Generator

The application uses the detected intent to generate a context-relevant,
Markdown-formatted response and displays it in the React chat interface.

![Week 2 -- AI response generation
output](submission-screenshots/week-2/week-2-response-generation.png)

### Week 3 -- Conversation Memory and Chat History

The first screenshot shows the conversation details provided by the user
and acknowledged by the assistant.

![Week 3 -- Conversation
start](submission-screenshots/week-3/week-3-conversation-start.png)

The second screenshot shows the follow-up question. The assistant
correctly recalls the name and laptop RAM details from the same
conversation.

![Week 3 -- Context-aware follow-up
response](submission-screenshots/week-3/week-3-follow-up-memory.png)

## What you need

-   **Python 3.10 or newer** (check with `python --version`)
-   **Node.js 18 or newer** (check with `node --version`):
    https://nodejs.org
-   A free **Groq API key**: https://console.groq.com/keys
-   Internet connection

## Run it (Windows PowerShell)

You need **two terminals**: one for the backend, one for the frontend.

### Step 1: Set up the backend (one time only)

``` powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

You'll know the venv is active when your prompt starts with `(.venv)`.

### Step 2: Add your API key (one time only)

``` powershell
copy .env.example .env
notepad .env
```

Replace the placeholder key with yours, then save:

``` env
GROQ_API_KEY=gsk_your_real_key_here
GROQ_MODEL=openai/gpt-oss-120b
```

No database setup is needed; the SQLite file is created automatically.

### Step 3: Start the backend (Terminal 1)

Still inside `backend`, with `(.venv)` showing:

``` powershell
uvicorn app.main:app --reload --port 8000
```

Check it at http://127.0.0.1:8000/health. You should see
`{"status":"ok",...,"database":"sqlite"}`. Leave this terminal running.

### Step 4: Start the frontend (Terminal 2)

Open a **new** terminal in the project root:

``` powershell
cd frontend-react
npm install        # first time only
npm run dev
```

### Step 5: Open the app

Go to http://localhost:5173 and click **Get Started**. The sidebar dot
is green when the backend is connected.

To stop the servers, press `Ctrl + C` in each terminal.

On macOS/Linux, use `source .venv/bin/activate` instead of
`Activate.ps1`, and `cp` instead of `copy`.

## Where is my chat history?

In `backend/data/chat.db`, created the first time the backend starts.
Ways to look at it:

-   In the browser: http://127.0.0.1:8000/api/conversations
-   In VS Code: install the **SQLite Viewer** extension and click the
    file
-   To start fresh: stop the backend and delete the `backend/data`
    folder

The database is in `.gitignore`, so your chats are never pushed to
GitHub.

## Common problems

  ------------------------------------------------------------------------------------------------------------------
  What you see                      Why                     Fix
  --------------------------------- ----------------------- --------------------------------------------------------
  Red dot / "Backend offline" in    Backend isn't running   Do Step 3
  the app                                                   

  `npm` is not recognized           Node.js isn't installed Install it from https://nodejs.org and reopen the
                                                            terminal

  `{"detail":"Not Found"}` at       Normal: there is no `/` Use `/docs` or `/health`
  http://127.0.0.1:8000/            route                   

  `502 The AI request failed`       Wrong key, or the model See
                                    name doesn't exist      [backend/README.md](backend/README.md#troubleshooting)

  `Activate.ps1 cannot be loaded`   PowerShell blocks       Run
                                    scripts                 `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`
                                                            once

  Port already in use               An old server is still  Press `Ctrl + C` in its terminal, then start again
                                    running                 
  ------------------------------------------------------------------------------------------------------------------

## Project map

``` text
enterprise-ingestion-firewall/
â”œâ”€â”€ backend/                         <-- Python API (see backend/README.md)
â”‚   â”œâ”€â”€ app/
â”‚   â”‚   â”œâ”€â”€ main.py                  <-- all API routes
â”‚   â”‚   â”œâ”€â”€ config.py                <-- reads your .env file
â”‚   â”‚   â”œâ”€â”€ schemas/
â”‚   â”‚   â”‚   â”œâ”€â”€ ingestion.py         <-- Week 1 data rules (input + intent result)
â”‚   â”‚   â”‚   â””â”€â”€ assistant.py         <-- Week 2 data rules (adds answer + session_id)
â”‚   â”‚   â”œâ”€â”€ services/
â”‚   â”‚   â”‚   â”œâ”€â”€ llm_service.py       <-- Week 1: classify the message
â”‚   â”‚   â”‚   â”œâ”€â”€ response_service.py  <-- Week 2: write the answer
â”‚   â”‚   â”‚   â””â”€â”€ database_service.py  <-- Week 3: save/load chats in SQLite
â”‚   â”‚   â””â”€â”€ utils/token_counter.py   <-- estimates token count
â”‚   â”œâ”€â”€ sql/schema.sql               <-- the database tables (reference)
â”‚   â”œâ”€â”€ tests/                       <-- pytest tests
â”‚   â”œâ”€â”€ .env.example                 <-- copy this to .env
â”‚   â””â”€â”€ requirements.txt
â”œâ”€â”€ frontend-react/                  <-- React + Vite app (see frontend-react/README.md)
â”œâ”€â”€ submission-screenshots/          <-- screenshots for Weeks 1, 2 and 3
â””â”€â”€ README.md                        <-- project overview and setup
```

## Good to know

-   Each message makes **two** LLM calls (classify, then answer).
-   The assistant uses the **last 12 messages** of a conversation as
    memory.
-   The app does **not** really look up orders or reset passwords; it
    only classifies and gives advice.
-   Token counts are **estimates**, not what Groq bills.
-   Schema validation checks the **shape** of data, not whether the AI's
    answer is true.

## Next reading

-   [backend/README.md](backend/README.md): each Python file, the API,
    the database, tests, troubleshooting
-   [frontend-react/README.md](frontend-react/README.md): how the React
    app is organized and connects to the API

