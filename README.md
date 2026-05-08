# Agentic Memory

This repository contains notebook-based examples for building agents with long-term memory. It focuses on two related tracks:

- An email assistant that evolves from a baseline LangGraph agent into an assistant with semantic, episodic, and procedural memory.
- A memory-aware agent architecture backed by Oracle AI Database vector stores and SQL tables.

The examples are educational and experimental. Most of the runnable code lives in Jupyter notebooks, with small Python modules used for prompts, schemas, utilities, and memory infrastructure.

## Repository Layout

```text
.
├── email_assistant/
│   ├── baseline_no_memory/
│   │   └── agent_setup.ipynb
│   ├── semantic_memory/
│   │   ├── agent_setup.ipynb
│   │   ├── prompts.py
│   │   ├── schemas.py
│   │   └── utils.py
│   ├── semantic_episodic_memory/
│   │   ├── agent_setup.ipynb
│   │   ├── prompts.py
│   │   ├── schemas.py
│   │   └── utils.py
│   └── semantic_episodic_procedural_memory/
│       ├── lesson_5.ipynb
│       ├── prompts.py
│       ├── schemas.py
│       └── utils.py
└── memory_aware_agent/
    ├── agent_setup.ipynb
    ├── helper.py
    └── requirements.txt
```

## Email Assistant Lessons

The `email_assistant` notebooks build an executive-assistant style email workflow. The assistant triages incoming email into three categories:

- `ignore`: irrelevant messages, newsletters, spam, and broad announcements.
- `notify`: important information that does not require a direct response.
- `respond`: direct questions, meeting requests, critical issues, or anything needing a reply.

The baseline implementation uses LangGraph to route each email through a triage node and, when needed, into a response agent with tools such as `write_email`, `schedule_meeting`, and `check_calendar_availability`.

### 1. Baseline No Memory

Path: `email_assistant/baseline_no_memory/agent_setup.ipynb`

Builds the first version of the email assistant:

- Defines a prompt-driven triage router.
- Uses Pydantic schemas for structured email classification.
- Creates a LangGraph workflow with a router and response agent.
- Uses hard-coded rules and tool instructions.

### 2. Semantic Memory

Path: `email_assistant/semantic_memory/agent_setup.ipynb`

Adds memory tools with `langmem` and LangGraph's in-memory store:

- Stores useful facts from previous emails.
- Searches remembered context before responding.
- Maintains per-user memory namespaces.
- Shows how semantic recall changes future responses.

### 3. Semantic + Episodic Memory

Path: `email_assistant/semantic_episodic_memory/agent_setup.ipynb`

Adds example-based memory for human feedback:

- Stores prior email triage examples.
- Retrieves similar examples as few-shot context.
- Uses corrected routing decisions to improve future classifications.
- Demonstrates how feedback can change behavior for similar future emails.

### 4. Semantic + Episodic + Procedural Memory

Path: `email_assistant/semantic_episodic_procedural_memory/lesson_5.ipynb`

Adds procedural memory by updating the assistant's instructions:

- Treats prompts and triage rules as updateable memory.
- Uses feedback to revise agent instructions and routing rules.
- Demonstrates updates such as changing email signature behavior or ignoring a specific sender.
- Shows how persistent instructions can modify future agent behavior.

## Memory-Aware Agent

Path: `memory_aware_agent/agent_setup.ipynb`

This notebook explores a broader memory architecture for agents using Oracle AI Database. The core implementation is in `memory_aware_agent/helper.py`.

The `MemoryManager` provides a unified interface over several memory types:

- Conversational memory: recent thread messages stored in SQL.
- Tool log memory: raw tool calls, inputs, outputs, status, and errors.
- Knowledge base memory: reference documents stored in an Oracle vector store.
- Workflow memory: prior successful task trajectories.
- Toolbox memory: semantically searchable tool definitions.
- Entity memory: people, places, systems, and named objects extracted from text.
- Summary memory: compressed older context with expandable summary IDs.

The `Toolbox` class registers Python functions as retrievable tools. It can optionally use an LLM to augment tool descriptions and generate synthetic queries so the agent can find relevant tools by semantic search.

The notebook covers:

- Oracle database setup and connection helpers.
- Vector-store initialization with `langchain-oracledb`.
- Memory retrieval and context assembly.
- Dynamic tool selection from toolbox memory.
- An OpenAI chat loop that reads memory, selects tools, executes tool calls, and writes results back to memory.
- Context compression into summary memory when conversation history grows too large.

## Requirements

Each lesson folder includes its own `requirements.txt`. The email assistant lessons are pinned for Python 3.11 and use:

- `langchain`
- `langchain-openai`
- `langchain-anthropic`
- `langgraph`
- `langmem`
- `python-dotenv`

The Oracle-backed memory-aware agent additionally uses packages such as:

- `langchain-oracledb`
- `langchain-huggingface`
- `sentence-transformers`
- `langchain-community`
- `oracledb`
- `openai`
- `arxiv`
- `pymupdf`
- `tavily-python`

## Setup

Create and activate a Python 3.11 environment, then install the requirements for the lesson you want to run:

```bash
python3.11 -m venv .venv
source .venv/bin/activate
pip install -r email_assistant/semantic_memory/requirements.txt
```

For the Oracle-backed agent:

```bash
pip install -r memory_aware_agent/requirements.txt
```

Create a `.env` file with the API keys required by the notebooks you run:

```bash
OPENAI_API_KEY=your_openai_api_key
ANTHROPIC_API_KEY=your_anthropic_api_key
TAVILY_API_KEY=your_tavily_api_key
```

Only set the keys needed by the specific notebook.

## Oracle Database Notes

The `memory_aware_agent` notebook expects an Oracle database that supports vector search. The helper module includes:

- `setup_oracle_database()` for one-time user and tablespace setup.
- `connect_to_oracle()` for retrying database connections.
- `create_conversational_history_table()` and `create_tool_log_table()` for SQL memory tables.
- `cleanup_vector_memory()` and `list_vector_objects()` for vector index/table diagnostics.

Default sample connection values in the helper are local-development placeholders. Update usernames, passwords, DSNs, and table names for your environment before using them outside a local demo.

## Running the Notebooks

Start Jupyter from the repository root:

```bash
jupyter lab
```

Then open the relevant notebook:

- `email_assistant/baseline_no_memory/agent_setup.ipynb`
- `email_assistant/semantic_memory/agent_setup.ipynb`
- `email_assistant/semantic_episodic_memory/agent_setup.ipynb`
- `email_assistant/semantic_episodic_procedural_memory/lesson_5.ipynb`
- `memory_aware_agent/agent_setup.ipynb`

Run cells in order. The notebooks are designed as lessons, so later cells usually depend on variables, stores, tools, and graph objects created earlier.

## Key Concepts

- Semantic memory: facts and details retrieved by similarity search.
- Episodic memory: previous examples or experiences used as few-shot guidance.
- Procedural memory: durable instructions and rules that change how the agent behaves.
- Conversational memory: chronological message history for the current thread.
- Summary memory: compressed older context that can be expanded when needed.
- Toolbox memory: tool definitions embedded and retrieved dynamically for a query.

## Status

This is a learning repository rather than a packaged Python library. There is no single application entry point or automated test suite. Treat the notebooks as the source of truth for each lesson flow.
