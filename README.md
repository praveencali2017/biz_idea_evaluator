# Biz Idea Evaluator

An AI-powered business idea evaluation tool built with [LangGraph](https://langchain-ai.github.io/langgraph/) and OpenAI.

Given a business idea, the evaluator asks clarifying questions until it understands the idea, then four advisors evaluate it in parallel from different perspectives, and a final node combines their findings into one report.

> **Status:** the full workflow runs end to end in the notebook. It is interactive (it prompts for input) and has no automated tests yet.

## How it works

The graph runs in three phases:

```
decide_node ──(needs info)──> ask_user_node ──┐
     ^                                        │
     └────────────────────────────────────────┘
     │
     └──(DONE)──> fanout ──> 4 advisors (parallel) ──> collect_and_report ──> END
```

1. **Clarify** — `decide_node` asks the LLM whether it has enough information. If not, the LLM asks one follow-up question, `ask_user_node` collects your answer, and the loop repeats. When the LLM replies `DONE`, `route` moves on.
2. **Fan out** — `fanout` is a no-op hub that starts all four advisors in the same step. Each advisor reviews the full clarifying conversation:

   | Node | Focus |
   | --- | --- |
   | `market_analyst_advisor` | market size, competition, target customers, timing |
   | `legal_advisor` | legal risk, regulatory compliance, intellectual property |
   | `technical_advisor` | feasibility, scalability, infrastructure |
   | `strategist_advisor` | positioning, competitive advantage, growth, partnerships |
3. **Synthesize** — `collect_and_report` waits for all four reports, merges them into a structured evaluation, and displays it as Markdown in the notebook.

Building blocks:

- **LLM** — one shared `ChatOpenAI` with `gpt-4o-mini` at `temperature=0` for repeatable evaluations.
- **State** — a LangGraph `State` (`TypedDict`) carries:
  - `idea` — the business idea under evaluation
  - `messages` — the clarifying conversation, merged with `add_messages`
  - `advisor_reports` — one report per advisor, merged with `operator.or_` so parallel advisors can each write their own key
  - `final_report` — the synthesized evaluation

### Known limitations

- `route` moves on only if the LLM's reply is exactly `DONE`. A reply like `DONE.` keeps the clarifying loop going.
- The graph is compiled without a checkpointer, so the `thread_id` passed to `graph.invoke` has no effect and runs are not persisted.

## Project structure

```
.
├── src/biz_idea_evaluator/
│   ├── __init__.py
│   └── idea_evaluator.ipynb   # the LangGraph agent notebook
├── .env                       # your API key (git-ignored, create it yourself)
├── .python-version
├── pyproject.toml
└── uv.lock
```

## Getting started

Requires Python 3.12+ and [uv](https://docs.astral.sh/uv/).

```bash
# Install dependencies
uv sync
```

Create a `.env` file in the project root with your OpenAI API key:

```
OPENAI_API_KEY=your-key-here
```

`.env` is git-ignored; never commit it.

### Run the notebook

In VS Code, open `src/biz_idea_evaluator/idea_evaluator.ipynb` and select the `.venv` interpreter as the kernel.

Or from the terminal (Jupyter isn't a project dependency, so `--with` pulls it in temporarily):

```bash
uv run --with jupyter jupyter lab src/biz_idea_evaluator/idea_evaluator.ipynb
```

Then run all cells. The last cell prompts you for your idea and then asks follow-up questions one at a time; answer each one in the input box. When the LLM has enough information, the four advisor reports and the final evaluation are produced, and the final evaluation is displayed in the notebook.

## Dependencies

- [langchain](https://python.langchain.com/) / [langchain-openai](https://python.langchain.com/docs/integrations/chat/openai/)
- [langgraph](https://langchain-ai.github.io/langgraph/)
- [pydantic](https://docs.pydantic.dev/)
- [python-dotenv](https://pypi.org/project/python-dotenv/)
- [ipython](https://ipython.readthedocs.io/)
