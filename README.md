# Biz Idea Evaluator

An AI-powered business idea evaluation tool built with [LangGraph](https://langchain-ai.github.io/langgraph/) and OpenAI.

Given a business idea, a multi-agent graph of advisors evaluates it from different perspectives and produces a final report.

> **Status:** work in progress. The notebook currently sets up the LLM and the graph `State`; the advisor nodes, synthesis node, and graph wiring are still to come.

## How it works

- **LLM** — `ChatOpenAI` with `gpt-4o-mini` at `temperature=0` for repeatable evaluations.
- **State** — a LangGraph `State` (`TypedDict`) carries:
  - `idea` — the business idea under evaluation
  - `messages` — conversation history, merged with `add_messages`
  - `advisor_reports` — one report per advisor, merged with `operator.or_` so parallel advisors can each write their own key
  - `final_report` — the synthesized evaluation
- **Advisors** *(planned)* — advisor nodes (e.g. market, technical, financial) each analyze the idea independently.
- **Final report** *(planned)* — a synthesis node combines the advisor reports into a single evaluation.

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

## Dependencies

- [langchain](https://python.langchain.com/) / [langchain-openai](https://python.langchain.com/docs/integrations/chat/openai/)
- [langgraph](https://langchain-ai.github.io/langgraph/)
- [pydantic](https://docs.pydantic.dev/)
- [python-dotenv](https://pypi.org/project/python-dotenv/)
- [ipython](https://ipython.readthedocs.io/)
