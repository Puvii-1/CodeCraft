# CodeCraft — Multi-Agent Code Generator

A small multi-agent system built with **LangGraph** that turns a plain-English
request into working code files. Three agents collaborate in a loop:

- **Planner** — breaks the request into an ordered list of files/tasks
- **Coder** — writes the code for the current task
- **Reviewer** — checks syntax (`ast.parse`) and does an LLM code review;
  rejects with feedback if something's wrong
- **Advance** (orchestration node) — decides whether to move to the next
  task, retry the current one with the reviewer's feedback, or stop

```
START → planner → coder → reviewer → advance ─┐
                     ▲                          │
                     └────── retry/next task ────┘
                                                 │
                                    done/failed → END
```

## Setup

1. **Get a free Gemini API key**: https://aistudio.google.com/apikey

2. **Create a virtual environment** (from the `codecraft` folder):

   ```
   python -m venv venv
   venv\Scripts\activate          # Windows
   ```

3. **Install dependencies**:

   ```
   pip install -r requirements.txt
   ```

4. **Add your API key**:
   Copy `.env.example` to `.env` and paste your key in:

   ```
   copy .env.example .env
   ```

   Then edit `.env` so it reads:

   ```
   GOOGLE_API_KEY=your_actual_key
   ```

5. **Run the app**:

   ```
   streamlit run app.py
   ```

6. Try a request like:

   > "Build a command-line expense tracker with add, list, and total commands"

   Watch the sidebar log show each agent's decisions, and expand the
   generated files to see the code.

## Project structure

```
codecraft/
├── agents/
│   ├── state.py      # shared state schema (TypedDict)
│   ├── llm.py         # LLM client setup (swap models here)
│   ├── planner.py      # planner agent node
│   ├── coder.py         # coder agent node
│   ├── reviewer.py       # reviewer agent node
│   └── graph.py            # wires nodes into the LangGraph + retry loop
├── app.py                     # Streamlit UI
├── requirements.txt
└── .env.example
```

## How the retry loop works

Each file gets up to `max_iterations` attempts (default 3, adjustable in
the sidebar). If the Reviewer rejects a file, its feedback is passed back
to the Coder, which rewrites just that file. If it's still rejected after
the max attempts, CodeCraft moves on and reports it as failed in the log,
rather than hanging forever.

## Ideas to extend this (good for a "v2" commit or interview talking point)

- **Add a Tester agent**: actually execute the generated code (in a
  sandboxed subprocess) and feed real error output back to the Reviewer,
  instead of relying only on an LLM opinion.
- **Parallelize independent files**: right now tasks run sequentially;
  LangGraph supports fan-out/fan-in if files don't depend on each other.
- **Persist state with LangGraph checkpointing**: let a user pause and
  resume a generation session.
- **Swap the LLM**: `agents/llm.py` is the only file that needs to change
  to run this on OpenAI, Anthropic, or a local model instead of Gemini.

## Why this project is worth having in your portfolio

It mirrors, at small scale, exactly what an AI app-generation platform
does internally: plan → generate → review → refine. It's a good project
to bring up in conversations about your work at Nexus, since the
architecture pattern (not the exact stack) is the same one production
systems like theirs are built on.
