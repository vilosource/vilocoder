# AI Web Dev PoC – Agent Contracts & Templates

This package gives you everything to start wiring a text‑based agent team (LangGraph + LangChain + OpenRouter) for a **local‑git, React/TS + Bootstrap + Django/Postgres** project. Copy/paste files into your repo as-is.

---

## Repo layout (suggested)

```
project-root/
├─ frontend/
├─ backend/
├─ agents/
│  ├─ contracts/
│  └─ prompts/
├─ scripts/
├─ docs/
│  ├─ stories/
│  ├─ rfc/
│  └─ templates/
├─ config/
│  └─ openrouter/
└─ src/graph/
```

---

## docs/templates/STORY\_TEMPLATE.md

```md
# Story: <Concise title>

## Goal
What changes for the user?

## Scope
What’s in / out (bullets)?

## Acceptance Criteria (G/W/T)
- Given … When … Then …
- …

## Constraints
(e.g., must use Bootstrap components, no new deps, perf budget <200ms TTFB)

## Notes
(e.g., links to designs, similar features)
```

---

## docs/templates/RFC\_TEMPLATE.md (Architect)

```md
# RFC: <Story title>

## Context & Objective
Why we’re doing this. 2–4 sentences.

## UI Map
- Routes/pages
- Key components
- State flows

## API Spec
- Endpoint: /api/<resource> (GET|POST|PUT|DELETE)
- Request/Response JSON (typed)
- Errors
- Auth/permissions

## Data Model (Django)
- Models, fields, relations
- Indexes/constraints/migrations

## Non‑functionals
Perf, a11y, i18n, basic security

## Tasks
- FE:
- BE:
- Tests:
- Infra (if any):

## Acceptance Criteria (copy from story)

## Risks & Mitigations
- Risk → Mitigation
```

---

## docs/templates/REVIEW\_CHECKLIST.md (Reviewer)

```md
# MR Review Checklist

## General
- [ ] Lint/format/typecheck pass
- [ ] Tests added/updated and green
- [ ] Coverage ≥ thresholds or justified

## Frontend
- [ ] Routes/components match RFC
- [ ] Loading/empty/error states present
- [ ] Basic a11y (labels, focus, keyboard)

## Backend
- [ ] Models/migrations correct & reversible
- [ ] API contracts & error handling explicit
- [ ] Auth/permissions enforced

## Security & Safety
- [ ] No secrets committed
- [ ] Dependency diffs reviewed

## Docs
- [ ] RFC/MR notes updated
- [ ] CHANGELOG entry if user‑visible
```

---

## agents/contracts – JSON Schemas

### `planner_contract.schema.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "PlannerOutput",
  "type": "object",
  "required": ["story_id", "rfc_title", "acceptance_criteria", "tasks", "api", "data_model"],
  "properties": {
    "story_id": {"type": "string"},
    "rfc_title": {"type": "string"},
    "acceptance_criteria": {"type": "array", "items": {"type": "string"}},
    "ui": {
      "type": "object",
      "properties": {
        "routes": {"type": "array", "items": {"type": "string"}},
        "components": {"type": "array", "items": {"type": "string"}},
        "state_notes": {"type": "string"}
      }
    },
    "api": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["path", "method", "request", "response", "errors", "auth"],
        "properties": {
          "path": {"type": "string"},
          "method": {"type": "string", "enum": ["GET", "POST", "PUT", "PATCH", "DELETE"]},
          "request": {"type": "object"},
          "response": {"type": "object"},
          "errors": {"type": "array", "items": {"type": "string"}},
          "auth": {"type": "string"}
        }
      }
    },
    "data_model": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["model", "fields"],
        "properties": {
          "model": {"type": "string"},
          "fields": {"type": "array", "items": {"type": "string"}},
          "relations": {"type": "array", "items": {"type": "string"}},
          "indexes": {"type": "array", "items": {"type": "string"}}
        }
      }
    },
    "tasks": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["id", "title", "area"],
        "properties": {
          "id": {"type": "string"},
          "title": {"type": "string"},
          "area": {"type": "string", "enum": ["frontend", "backend", "tests", "infra"]},
          "depends_on": {"type": "array", "items": {"type": "string"}}
        }
      }
    },
    "risks": {"type": "array", "items": {"type": "string"}}
  }
}
```

### `frontend_contract.schema.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "FrontendOutput",
  "type": "object",
  "required": ["branch", "changes", "tests"],
  "properties": {
    "branch": {"type": "string"},
    "changes": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["path", "diff"],
        "properties": {
          "path": {"type": "string"},
          "diff": {"type": "string", "description": "unified diff"}
        }
      }
    },
    "tests": {"type": "array", "items": {"type": "string"}},
    "notes": {"type": "string"}
  }
}
```

### `backend_contract.schema.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "BackendOutput",
  "type": "object",
  "required": ["branch", "changes", "migrations", "tests"],
  "properties": {
    "branch": {"type": "string"},
    "changes": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["path", "diff"],
        "properties": {
          "path": {"type": "string"},
          "diff": {"type": "string"}
        }
      }
    },
    "migrations": {"type": "array", "items": {"type": "string"}},
    "tests": {"type": "array", "items": {"type": "string"}},
    "notes": {"type": "string"}
  }
}
```

### `test_contract.schema.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "TestOutput",
  "type": "object",
  "required": ["summary", "reports", "status"],
  "properties": {
    "summary": {"type": "string"},
    "reports": {"type": "array", "items": {"type": "string"}},
    "status": {"type": "string", "enum": ["pass", "fail"]},
    "fix_suggestions": {"type": "array", "items": {"type": "string"}}
  }
}
```

### `review_contract.schema.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "ReviewOutput",
  "type": "object",
  "required": ["approved", "findings"],
  "properties": {
    "approved": {"type": "boolean"},
    "findings": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["path", "line", "severity", "note"],
        "properties": {
          "path": {"type": "string"},
          "line": {"type": "integer"},
          "severity": {"type": "string", "enum": ["info", "warn", "error"]},
          "note": {"type": "string"}
        }
      }
    },
    "required_changes": {"type": "array", "items": {"type": "string"}}
  }
}
```

### `release_contract.schema.json`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "ReleaseOutput",
  "type": "object",
  "required": ["version", "tag", "changelog"],
  "properties": {
    "version": {"type": "string"},
    "tag": {"type": "string"},
    "changelog": {"type": "string"}
  }
}
```

---

## config/openrouter/routing.json (example)

```json
{
  "default": "openrouter/auto",
  "nodes": {
    "planner": {"model": "openrouter/auto", "fallbacks": ["anthropic/claude-3.7-sonnet", "openai/gpt-4.1"]},
    "frontend": {"model": "qwen/qwen2.5-coder", "fallbacks": ["openai/gpt-4.1-mini"]},
    "backend": {"model": "codestral/codestral-latest", "fallbacks": ["deepseek/deepseek-coder"]},
    "tester": {"model": "openrouter/auto"},
    "reviewer": {"model": "anthropic/claude-3.7-sonnet", "fallbacks": ["openai/gpt-4.1"]},
    "release": {"model": "openrouter/auto"}
  }
}
```

> Configure via `OPENROUTER_API_KEY` env var. You can hard‑pin models here and override per run.

---

## agents/prompts (summaries)

### `planner.prompt.txt`

* Role: Architect/Planner. Produce RFC + tasks per `planner_contract.schema.json`.
* Keep scope minimal; reuse Bootstrap; avoid new deps.
* Respect constraints from story.

### `frontend.prompt.txt`

* Role: Frontend Dev. Implement only files you list in `changes[]` as unified diffs.
* Provide tests. No unrelated edits. Follow RFC exactly.

### `backend.prompt.txt`

* Role: Backend Dev. Django (DRF). Models/serializers/viewsets; reversible migrations; auth/permissions.

### `tester.prompt.txt`

* Role: SDET. Add/update unit + light e2e (Playwright) covering acceptance criteria.

### `reviewer.prompt.txt`

* Role: Reviewer. Apply `REVIEW_CHECKLIST.md`. Gate approval.

### `release.prompt.txt`

* Role: Release Agent. Propose semver bump + CHANGELOG from commits + RFC.

---

## src/graph/graph\_skeleton.py (minimal wiring example)

```python
# Pseudocode-level example to wire nodes with LangGraph + LangChain
from typing import TypedDict, List, Literal
from langgraph.graph import StateGraph, END
from langgraph.checkpoint.sqlite import SqliteSaver
from langchain_openai import ChatOpenAI  # replace with OpenRouter client wrapper

class State(TypedDict):
    story_md: str
    plan: dict
    fe_out: dict
    be_out: dict
    test_out: dict
    review_out: dict
    release_out: dict

checkpointer = SqliteSaver.from_conn_string("sqlite:///graph.db")

# — Node fns (stubs) —

def planner_node(state: State) -> State:
    # call LLM with planner.prompt + story_md, validate against planner_contract
    plan = {"rfc_title": "...", "tasks": [...]}  # validate
    return {**state, "plan": plan}


def frontend_node(state: State) -> State:
    # generate diffs under /frontend as per plan; return changes[]
    return {**state, "fe_out": {"branch": "feature/x", "changes": []}}


def backend_node(state: State) -> State:
    return {**state, "be_out": {"branch": "feature/x", "changes": [], "migrations": []}}


def test_node(state: State) -> State:
    return {**state, "test_out": {"status": "pass", "summary": "ok", "reports": []}}


def review_node(state: State) -> State:
    return {**state, "review_out": {"approved": True, "findings": []}}


def gate_node(state: State) -> State:
    # human-in-the-loop breakpoint, e.g., confirm migrations
    # printf prompt to terminal and wait for y/n
    return state


def release_node(state: State) -> State:
    return {**state, "release_out": {"version": "0.1.0", "tag": "v0.1.0", "changelog": "..."}}

# — Graph —
workflow = StateGraph(State)
workflow.add_node("planner", planner_node)
workflow.add_node("frontend", frontend_node)
workflow.add_node("backend", backend_node)
workflow.add_node("tests", test_node)
workflow.add_node("review", review_node)
workflow.add_node("gate", gate_node)
workflow.add_node("release", release_node)

workflow.set_entry_point("planner")
workflow.add_edge("planner", "frontend")
workflow.add_edge("planner", "backend")
workflow.add_edge("frontend", "tests")
workflow.add_edge("backend", "tests")
workflow.add_edge("tests", "review")
workflow.add_edge("review", "gate")
workflow.add_edge("gate", "release")
workflow.add_edge("release", END)

app = workflow.compile(checkpointer=checkpointer)

# run: app.invoke({"story_md": open("docs/stories/your_story.md").read()})
```

---

## scripts/local\_git\_flow\.md (commands the Orchestrator can call)

```sh
# create branch
git checkout -b feature/<story-id>

# apply unified diffs from agents
patch -p0 < /tmp/agent.diff

# run local checks
npm --prefix frontend ci && npm --prefix frontend test -- --ci
pytest -q backend

# commit/merge
git add -A && git commit -m "feat: <story-id> implement"
# (simple strategy for PoC)
git checkout main && git merge --no-ff feature/<story-id>

# tag release
ver=v0.1.0; git tag -a "$ver" -m "Release $ver" && git push --tags
```

---

## Next steps

1. Drop these files into your repo. 2) Wire real LLM calls + schema validators for each node. 3) Implement the **terminal REPL** that feeds `story_md` and handles the **gate** prompt.
