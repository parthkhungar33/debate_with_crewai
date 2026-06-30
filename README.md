# 🗣️ Debate Crew — A CrewAI Example

A small, self-contained [CrewAI](https://crewai.com) project that stages a one-on-one debate between AI agents. You give it a **motion** (a statement to argue about), and the crew:

1. Argues **in favour** of the motion,
2. Argues **against** the motion, and
3. Has an impartial **judge** decide which side was more convincing.

It's a great first project for understanding how multi-agent AI systems work, because the whole flow is tiny, readable, and easy to tweak.

---

## 🤔 What is CrewAI?

**CrewAI is a framework for building multi-agent AI systems** — instead of sending a single prompt to a single LLM, you orchestrate a *crew* of specialised AI agents that each have a role and collaborate to complete a larger task.

Think of it like assembling a team of people: you wouldn't ask one person to research, write, fact-check, and approve a report all at once. You'd give each person a clear role and hand work off between them. CrewAI lets you do exactly that with LLMs.

### What is it used for?

- **Research assistants** — one agent searches, another summarises, another writes the report.
- **Content pipelines** — outline → draft → edit → SEO-optimise.
- **Analysis & decision-making** — gather data, weigh options, recommend an action.
- **Automation** — multi-step workflows that need reasoning at each step.

### Core concepts (everything in this project)

| Concept | What it is | In this project |
|---------|-----------|-----------------|
| **Agent** | An AI "worker" with a role, a goal, and a backstory. Backed by an LLM. | `debater` and `judge` |
| **Task** | A unit of work assigned to an agent, with a description and an expected output. | `propose`, `oppose`, `decide` |
| **Crew** | The team of agents + tasks, plus how they run. | The `Debate` crew |
| **Process** | How tasks are executed — `sequential` (one after another) or `hierarchical`. | `sequential` |

A key idea: **agents and tasks are defined in plain YAML files** (`config/agents.yaml` and `config/tasks.yaml`), not buried in code. This keeps the *behaviour* (prompts, roles, goals) separate from the *wiring* (Python), so you can change what the agents do without touching the logic.

> 📘 New to the CrewAI API? The included [`AGENTS.md`](AGENTS.md) is an auto-generated, version-accurate reference that AI coding assistants (Claude Code, Cursor, Copilot, etc.) can use to write correct CrewAI code.

---

## 🧩 How this project uses CrewAI

The whole debate is driven by **2 agents** and **3 tasks** running sequentially.

### The agents — [`src/debate/config/agents.yaml`](src/debate/config/agents.yaml)

- **`debater`** — "A compelling debater" whose goal is to win the judge over with concise, convincing arguments. The *same* debater agent argues both sides.
- **`judge`** — "A fair judge" who weighs the arguments on their merits alone, ignoring personal opinion.

### The tasks — [`src/debate/config/tasks.yaml`](src/debate/config/tasks.yaml)

| Task | Agent | What it does | Saves to |
|------|-------|--------------|----------|
| `propose` | `debater` | Builds the strongest case **for** the motion | `output/propose.md` |
| `oppose` | `debater` | Builds the strongest case **against** the motion | `output/oppose.md` |
| `decide` | `judge` | Reads both arguments and picks a winner, with reasoning | `output/decide.md` |

### The wiring — [`src/debate/crew.py`](src/debate/crew.py)

The `Debate` class uses CrewAI's decorators to assemble everything:

```python
@CrewBase
class Debate():
    @agent
    def debater(self) -> Agent: ...   # reads config from agents.yaml

    @agent
    def judge(self) -> Agent: ...

    @task
    def propose(self) -> Task: ...     # reads config from tasks.yaml
    @task
    def oppose(self) -> Task: ...
    @task
    def decide(self) -> Task: ...

    @crew
    def crew(self) -> Crew:
        return Crew(
            agents=self.agents,        # auto-collected from @agent methods
            tasks=self.tasks,          # auto-collected from @task methods
            process=Process.sequential # propose → oppose → decide
        )
```

### The flow

```
        ┌──────────────┐      ┌─────────────┐      ┌─────────────┐
motion ▶│   propose    │ ───▶ │   oppose    │ ───▶ │   decide    │ ▶ winner
        │  (debater)   │      │  (debater)  │      │   (judge)   │
        └──────────────┘      └─────────────┘      └─────────────┘
         output/propose.md     output/oppose.md     output/decide.md
```

The motion is passed in as the `{motion}` variable, which CrewAI automatically interpolates into the agent and task prompts at runtime — see [`src/debate/main.py`](src/debate/main.py).

---

## 📁 Project structure

```
debate/
├── README.md
├── AGENTS.md                  # CrewAI reference for AI coding assistants
├── pyproject.toml             # dependencies & CLI scripts
├── .env.example               # copy to .env and add your API key
├── knowledge/
│   └── user_preference.txt    # example knowledge source
├── output/                    # generated debate results land here
└── src/debate/
    ├── main.py                # entry points (run, train, test, replay)
    ├── crew.py                # the Debate crew definition
    ├── config/
    │   ├── agents.yaml         # WHO the agents are
    │   └── tasks.yaml          # WHAT they do
    └── tools/
        └── custom_tool.py      # example custom tool (unused, for reference)
```

---

## 🚀 Getting started

### Prerequisites

- Python `>=3.10, <3.14`
- An LLM API key (OpenAI by default — see [Using a different model](#-using-a-different-model) to switch providers)
- [UV](https://docs.astral.sh/uv/) for dependency management

### 1. Install UV (if you don't have it)

```bash
pip install uv
```

### 2. Install dependencies

```bash
crewai install
```

(or `uv sync`)

### 3. Add your API key

Copy the example env file and fill in your key:

```bash
cp .env.example .env
```

Then edit `.env`:

```
OPENAI_API_KEY=sk-...your-key-here...
```

### 4. Run the debate

```bash
crewai run
```

You'll be prompted to enter a motion, for example:

```
Enter the motion: Remote work is better than working in an office
```

The crew runs the three tasks in order and writes the results to the `output/` folder. The judge's verdict is printed at the end of the run.

---

## 📄 Example output

Running it with the motion **"Anthropic is better than OpenAI"** produced (abridged):

- **`output/propose.md`** — argues Anthropic is better built for trust, reliability, and responsible use.
- **`output/oppose.md`** — argues OpenAI leads on capability, ecosystem, adoption, and impact.
- **`output/decide.md`** — the judge sided with the opposition, reasoning that the pro side leaned on subjective values (safety, trust) while the opposition grounded its case in broader, more concrete criteria.

The files committed in `output/` are sample results from a previous run — they'll be overwritten the next time you run the crew.

---

## 🔄 Using a different model

The model is set per-agent in [`agents.yaml`](src/debate/config/agents.yaml) via the `llm` field. CrewAI supports OpenAI, Anthropic, Google, Ollama (local models), and many others.

```yaml
debater:
  role: A compelling debater
  # ...
  llm: openai/gpt-4o-mini              # OpenAI
  # llm: anthropic/claude-sonnet-4-6   # Anthropic
  # llm: gemini/gemini-2.5-flash       # Google
  # llm: ollama/llama3                 # local via Ollama
```

Just make sure the matching API key is in your `.env` (e.g. `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`). See the [CrewAI LLM docs](https://docs.crewai.com/concepts/llms) for the full list of supported providers.

---

## 🛠️ Customising the crew

- **Change the debaters' personalities** → edit roles/goals/backstories in `config/agents.yaml`
- **Change what they argue or how they're judged** → edit descriptions in `config/tasks.yaml`
- **Add tools** (web search, calculators, etc.) → see `tools/custom_tool.py` and pass them to an agent in `crew.py`
- **Add more rounds or rebuttals** → add new tasks in `tasks.yaml` and `crew.py`

You rarely need to touch `crew.py` or `main.py` for behaviour changes — most of the magic lives in the YAML.

---

## 📚 Other entry points

Defined in [`pyproject.toml`](pyproject.toml) and [`main.py`](src/debate/main.py):

| Command | What it does |
|---------|--------------|
| `crewai run` | Run the debate |
| `uv run train <n> <file>` | Train the crew over N iterations |
| `uv run test <n> <eval_llm>` | Test the crew and evaluate results |
| `uv run replay <task_id>` | Replay execution from a specific task |

---

## 🔗 Learn more

- [CrewAI Documentation](https://docs.crewai.com)
- [CrewAI GitHub](https://github.com/crewAIInc/crewAI)
- This project is based on the CrewAI section of [Ed Donner's "Agentic AI" course](https://github.com/ed-donner/agents).

---

## 📝 License

Released under the [MIT License](LICENSE).
