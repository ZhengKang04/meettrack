# MeetTrack — Technical Environment Document

**Version:** 0.1 · **Date:** 2026-11-17 · **Status:** Proposed baseline for team review

This is an input specification for AI-DLC v1.0.1. The architecture and examples describe the planned Construction phase; they are not an implemented application or completed test results.

## 1. Language, package manager, and libraries

| Area | Required baseline / constraint |
| --- | --- |
| Runtime | Python **3.12.x** for the application; document the selected patch version during setup |
| Package manager | `uv`; use `pyproject.toml` and commit a resolved `uv.lock` during Construction |
| Backend | FastAPI with Uvicorn; one local application process |
| Pages | Jinja2 templates, HTML/CSS, and small amounts of plain JavaScript |
| Database | SQLite through SQLModel; enable foreign-key enforcement for every connection |
| Schema/validation | Pydantic v2; typed request, response, and model-output schemas |
| Model client | Official Ollama Python client behind an application adapter |
| Tests | pytest; FastAPI TestClient/httpx for API tests; isolated temporary SQLite databases |
| Code quality | Ruff formatting/linting; type hints on service and model-adapter functions |

Use a separate virtual environment. Select mutually compatible stable package releases, lock the exact resolved versions before Construction work is shared, and review dependency changes in Git. No package installation or lockfile resolution is claimed by this draft. Avoid adding a second backend language or frontend build system to the MVP.

## 2. Cloud provider, deployment, and product model

- **Deployment model:** Local only. Run the FastAPI server on `127.0.0.1`; access it through a browser on the same laptop. No cloud provider is required.
- **Database:** One local SQLite file under the ignored `data/` directory. Database content is durable across normal application restarts and is not published to GitHub.
- **LLM provider:** Local Ollama, normally at `http://127.0.0.1:11434`.
- **Initial model candidate:** `qwen3:8b`. Test quality and latency on the team's actual hardware before adopting it as the final model. Record the chosen tag/model digest and Ollama version for the evaluation run.
- **Fallback:** If local inference is impractical, the team may approve another local model or a hosted provider through the same adapter. Record the model, provider, data handling, cost, and any schema changes before using that alternative. No hosted account or paid service is assumed.
- **Configuration:** Environment variables such as `MEETTRACK_DATABASE_URL`, `MEETTRACK_OLLAMA_URL`, and `MEETTRACK_MODEL`; a future `.env.example` may contain safe examples only.

The application should remain usable for browsing and editing saved data when Ollama is unavailable. Analysis requests should show a clear failure and permit an intentional retry. A proposed per-call timeout is 120 seconds, with at most one bounded retry for transient failures or invalid output; benchmark and revise this during Inception/Construction.

## 3. Prohibited libraries, services, and approaches

| Prohibited for the MVP | Reason | Use instead |
| --- | --- | --- |
| LangChain, LangGraph, CrewAI, and AutoGen | A fixed two-step workflow does not require an orchestration framework | Small typed Python service functions and the Ollama client |
| React, Next.js, and additional frontend build tooling | Adds another toolchain to a form-and-review application | Jinja2, HTML/CSS, and plain JavaScript |
| Pinecone, Chroma, FAISS, and vector-database services | A bounded current-task list can be supplied directly to the model | Same-project SQLite queries and explicit context limits |
| Redis, Celery, and message-broker infrastructure | The local single-operator MVP can process one analysis request at a time | A bounded model call with visible in-progress state |
| PostgreSQL/MySQL and cloud database services | Adds setup and deployment work without a pilot requirement | SQLite with transactional writes |
| Speech-to-text SDKs, meeting-platform SDKs, and web-scraping libraries | Audio and external data ingestion are outside scope | Paste plain-text meeting notes |
| Automatic model download at application startup | Large downloads make startup unpredictable | Install the selected Ollama model explicitly before analysis |
| `eval`, `exec`, or executing model-generated SQL/shell/code | Model output must remain data and cannot define executable behavior | Pydantic schemas, field allowlists, and parameterized ORM operations |

Changes to these constraints require a documented team decision and a corresponding update to this input document.

## 4. Architecture and application boundaries

```text
Browser: project / meeting input / proposal review / task board
                         |
              FastAPI routes + Jinja2
                         |
        Application services and business validation
             |                           |
      SQLModel / SQLite           LLM provider adapter
                                       |
                                Local Ollama model
```

Routes parse requests and call services. Services enforce ownership-by-project, review state, valid changes, and transaction boundaries. The model adapter accepts typed inputs and returns typed candidate data; it has no database write capability. Database writes use SQLModel sessions owned by the request/service boundary. Templates display escaped text.

Planned application layout, to be created during Construction:

```text
app/
  main.py                 # application setup
  api/                    # project, meeting, proposal, and task routes
  services/               # analysis, approval, and task rules
  llm/                    # provider adapter and versioned prompts
  models/                 # database entities
  schemas/                # request/response/model-output schemas
  templates/
  static/
tests/
data/                     # runtime files; ignored by Git
docs/                     # team-provided input documents
aidlc-docs/               # artifacts produced by the actual AI-DLC workflow
```

Application code belongs at the repository root under `app/`, not inside `aidlc-docs/`.

### Core data entities

| Entity | Main fields / constraints |
| --- | --- |
| Project | ID, name, timestamps; supports a member roster |
| Member | ID, project ID, display name; member IDs resolve assignment identity |
| Meeting | ID, project ID, title, meeting date, immutable saved text, numbered source segments, analysis state |
| Decision | ID, meeting ID, decision text, source segment, reviewer state |
| Task | ID, project ID, title, optional owner ID, optional due date, status, version number, source references |
| Proposal | ID, meeting ID, action type, optional target task ID, explicit changed fields, source references, base task version, review state, applied task ID |
| TaskChange | Task ID, source meeting/proposal when applicable, before/after fields, timestamp; manual edits have an explicit manual origin |

Use UUIDs or another stable opaque ID representation consistently. Dates are ISO `YYYY-MM-DD`; event timestamps are stored in UTC. Display/overdue calculations use one configured project timezone, initially `America/New_York`. The prototype has no delete/cancel task operation.

Task status values are `todo`, `in_progress`, and `done`. Proposal states are `pending`, `approved`, and `rejected`. For update proposals, an omitted field means “leave unchanged”; an explicitly supplied `null` means “clear this field,” which is allowed only for nullable fields after reviewer confirmation. Every update has an allowlisted field patch.

## 5. Model workflow and validation

1. Save the meeting and stable numbered text segments before invoking the model. Enforce the 5,000-character and 50-current-task pilot limits rather than silently truncating input.
2. Supply the current meeting date, member IDs/names, and text as data. Extract candidate action items, decisions, source references, original date expressions, and unresolved questions using a JSON schema.
3. Supply those candidates and the current same-project task list, including completed tasks within the limit. Ask for new-task proposals or updates referring to existing stable task IDs. An uncertain match is explicitly flagged for the reviewer.
4. Validate schema, identifiers, status values, field lengths, source-quote presence, and project boundaries. Treat the model output as untrusted data. Source-string presence does not establish factual correctness; review and evaluation check interpretation.
5. Preserve uncertain owner/date fields as null. Present suggested date normalization and the original wording to the user; ambiguous relative dates require an explicit choice.
6. Store valid results as pending proposals and decisions. The model cannot approve changes or directly save tasks.
7. On approval, revalidate the edited proposal, member/project IDs, current task version, and review state inside one database transaction. Save the task and change record together. Repeat approval returns the previously applied result; a stale task version returns a conflict for fresh review.

Begin with low-temperature generation and a versioned prompt. Record model identifier, prompt version, request duration, and success/failure metadata. Store reviewed output as product data, but keep raw notes and model responses out of ordinary console logs. An analysis run creates one review batch; a retry reuses the run identifier or replaces only an unapplied failed batch. Reject overlapping analyses for the same meeting.

No embeddings, model fine-tuning, open-ended agent loop, or autonomous tool selection is required for this baseline. The LLM-powered extraction and cross-meeting semantic matching are the core integration.

## 6. Proposed API conventions

| Method and path | Responsibility |
| --- | --- |
| `POST /api/projects` | Create a project |
| `POST /api/projects/{project_id}/members` | Add a member |
| `POST /api/projects/{project_id}/meetings` | Save a meeting |
| `POST /api/projects/{project_id}/meetings/{meeting_id}/analyze` | Produce a pending review batch |
| `GET /api/projects/{project_id}/proposals` | List proposals and their evidence |
| `POST /api/projects/{project_id}/proposals/{proposal_id}/approve` | Apply an edited/confirmed proposal transactionally |
| `POST /api/projects/{project_id}/proposals/{proposal_id}/reject` | Reject a proposal without changing tasks |
| `GET /api/projects/{project_id}/tasks` | List/filter tasks |
| `PATCH /api/projects/{project_id}/tasks/{task_id}` | Perform a deliberate manual edit and record its origin |

Use Pydantic models at boundaries. Return 404 for absent or wrong-project records, 409 for stale/rejected/conflicting actions, 422 for invalid input, and a clear 503/504-style error for unavailable/timed-out inference. Error text shown to the user must not contain secrets or raw internal traces.

## 7. Security basics

| Area | MVP policy |
| --- | --- |
| Authentication / access | No application accounts in the single-operator pilot. Bind to loopback only and rely on the laptop's OS account. This design is not suitable for shared-network/public hosting without new authentication and authorization design. |
| Project isolation | Every member, meeting, task, and proposal lookup is scoped to the requested project; validate IDs again before applying changes. |
| Input/output validation | Enforce length limits and schemas; validate field allowlists and source references; parameterize database access and escape template text. |
| Model instructions | Delimit meeting text as untrusted content. Instructions written inside notes do not override the application's extraction rules or invoke external tools. |
| Browser writes | Validate same-origin requests and protect state-changing forms/requests against CSRF; do not allow wildcard CORS. |
| Secrets | Local environment variables or ignored `.env`; never commit tokens or include them in model prompts. Local Ollama requires no hosted API key. |
| Data handling | Use synthetic evaluation/demo data. Store real notes locally only with participants' permission; exclude databases, private fixtures, and logs from Git. |
| Transport / storage | HTTP is confined to loopback for this local pilot. SQLite is not application-encrypted; rely on OS file permissions and the laptop's storage protection. Revisit HTTPS and storage controls before any broader deployment. |

## 8. Testing and acceptance

| Test type | Tool / scope | Required checks |
| --- | --- | --- |
| Unit | pytest; deterministic functions | Missing fields, assignee validation, patch semantics, date/overdue rules, allowed states |
| Service/API integration | pytest + TestClient/httpx + temporary SQLite | Review gating, project isolation, atomic approval/history writes, stale-version conflicts, repeat-approval safety, restart persistence |
| Model failure handling | Fake provider with controlled responses | Invalid JSON/schema, unknown IDs, nonexistent quotes, timeouts, retry limits; unchanged tasks after failure |
| Model evaluation | Real selected model; fixed annotated records | Extraction precision/recall, task-target matching, unsupported fields, latency; report actual counts |
| User workflow | Scripted browser run and small volunteer pilot | Complete the two-meeting scenario and inspect reviewer corrections |

Automated functional tests use a fake provider so they do not require a running model or paid service. Real-model evaluation is a separate explicit run using the frozen development/held-out split defined in the Vision document. Proposed target: at least 80% statement coverage in service/validation modules, with every approval/isolation invariant covered regardless of the coverage percentage.

During Construction, the proposed shared checks are `uv run ruff check .`, `uv run ruff format --check .`, and `uv run pytest`. These commands are planned conventions; the application, dependency configuration, CI workflow, and tests do not exist at this input-draft stage.

## 9. Example code patterns

These short patterns satisfy the input-document requirement for an endpoint, a function, and a test. They are specifications for later implementation, not files already present in the application. Imports such as `app.db` and `app.services` refer to the planned layout.

### Endpoint: approve a reviewed proposal

```python
from typing import Annotated
from uuid import UUID

from fastapi import APIRouter, Depends, HTTPException
from sqlmodel import Session

from app.db import get_session
from app.schemas.proposals import ProposalApproval
from app.schemas.tasks import TaskRead
from app.services.proposals import (
    ProposalConflict,
    ProposalNotFound,
    approve_reviewed_proposal,
)

router = APIRouter(prefix="/api/projects")


@router.post(
    "/{project_id}/proposals/{proposal_id}/approve",
    response_model=TaskRead,
)
def approve_proposal(
    project_id: UUID,
    proposal_id: UUID,
    body: ProposalApproval,
    session: Annotated[Session, Depends(get_session)],
) -> TaskRead:
    try:
        # Service owns the transaction, project checks, and repeat-call rules.
        return approve_reviewed_proposal(session, project_id, proposal_id, body)
    except ProposalNotFound as exc:
        raise HTTPException(404, "Proposal not found") from exc
    except ProposalConflict as exc:
        raise HTTPException(409, "Review the latest task before confirming") from exc
```

`ProposalApproval` must contain the reviewed fields and expected proposal/task versions. Its schema must distinguish omitted update fields from explicitly cleared nullable fields. A local-app same-origin/CSRF dependency is to be applied consistently to write routes.

### Function: validate a proposed task owner

```python
from collections.abc import Collection
from uuid import UUID


def validate_assignee(
    assignee_id: UUID | None,
    project_member_ids: Collection[UUID],
) -> UUID | None:
    if assignee_id is None:
        return None  # An explicitly unassigned task is valid.
    if assignee_id not in project_member_ids:
        raise ValueError("Assignee must belong to the current project")
    return assignee_id
```

### Test: an owner from another project is rejected

```python
from uuid import UUID

import pytest

from app.services.validation import validate_assignee


def test_rejects_assignee_outside_project() -> None:
    current_member = UUID("00000000-0000-0000-0000-000000000001")
    other_project_member = UUID("00000000-0000-0000-0000-000000000002")

    with pytest.raises(ValueError, match="current project"):
        validate_assignee(other_project_member, {current_member})
```

## 10. AI-DLC boundaries and references

Use **AI-DLC v1.0.1** for development. The input documents live in `docs/`; actual Inception outputs and workflow records will be generated under `aidlc-docs/`. The team must answer clarifying questions and review each required approval gate. For the mid-semester milestone, complete Units Generation and stop before Construction. Do not mark planned artifacts, meetings, evaluations, or approvals as completed.

Primary references checked for this draft:

- [AI-DLC v1.0.1 input quickstart](https://github.com/awslabs/aidlc-workflows/blob/v1.0.1/docs/writing-inputs/inputs-quickstart.md)
- [FastAPI templates](https://fastapi.tiangolo.com/advanced/templates/)
- [FastAPI, SQLModel, and SQLite](https://fastapi.tiangolo.com/tutorial/sql-databases/)
- [Ollama structured outputs](https://docs.ollama.com/capabilities/structured-outputs)
- [Ollama qwen3:8b model](https://ollama.com/library/qwen3:8b)
- [Pydantic model validation](https://docs.pydantic.dev/latest/concepts/models/)
