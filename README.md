# MeetTrack

**A meeting action-item tracker for student project teams.**

INFO5940 · Fall 2026 · Class Project

**Status:** Initial input-document draft for team review. Application implementation and the formal AI-DLC Inception workflow have not started. This repository is not yet a complete milestone submission.

## Team

| Full name | Cornell NetID | GitHub username |
| --- | --- | --- |
| To be supplied by the team | To be supplied by the team | To be supplied by the team |

Replace this row with one row for **every** team member before submission. The repository owner's account is not a substitute for the team roster.

## Project overview

Student project teams often leave meetings with commitments scattered across informal notes. A discussion may assign a task, revise a deadline, or report completion without using consistent wording. Manually transferring these changes into a task list takes time, and later meetings can create duplicate or contradictory tasks. MeetTrack will help small course-project teams turn meeting notes into a persistent, reviewable record of who will do what and when. Its intended users are a meeting coordinator or recorder operating the application, with teammates participating in review and checking their assigned work.

LLMs are appropriate because the input is unstructured language rather than a predictable form. The product must distinguish a commitment from a suggestion, connect paraphrases to an existing task, and identify missing or ambiguous information. For example, a later note saying “the login screen is now Alex's responsibility” should propose a change to the existing login task. MeetTrack will attach source excerpts to suggestions and leave unsupported owners or dates unresolved. These behaviors make language understanding central to the product's value.

The planned LLM integration has two bounded steps: extract decisions and candidate action items from the current meeting; then compare those items with the same project's existing tasks to propose additions or updates. The initial model/provider candidate is **`qwen3:8b` served locally by Ollama**, subject to testing on the team's hardware. Responses will use a JSON schema and be checked with Pydantic and application rules. The application will handle persistence, date checks, project isolation, and transaction safety. A user must review and approve a suggestion before it changes the task board. The MVP uses this fixed workflow; autonomous tool selection is a possible later extension.

The MVP will support creating projects and member rosters, entering text meeting notes, reviewing extracted decisions and task suggestions, maintaining a task board, and reconciling changes from a later meeting. Tasks will have an owner, optional due date, status, and traceable source. A single local operator will use the application during shared team sessions; named members do not require separate accounts. Initial scope limits are 5,000 characters per meeting and 50 current tasks per project. Audio transcription, external-platform synchronization, notifications, cloud hosting, and simultaneous editing are outside the MVP. This creates a complete two-meeting workflow while keeping the final implementation achievable.

The proposed implementation uses Python 3.12, FastAPI, Jinja2, SQLite/SQLModel, and an isolated LLM adapter. The final product will run on a team laptop and retain data across restarts. Evaluation will combine deterministic tests for approval and persistence with a small, manually annotated set of synthetic meeting records for extraction and task matching. The mid-semester milestone will establish requirements, user stories, application design, and units of work through **AI-DLC v1.0.1**. Application construction, validation, and the recorded product demonstration belong to the final-project stage.

Our collaboration plan is to hold **two team meetings per week**, using an in-person session when practical and a video call otherwise. We will coordinate meeting times and everyday questions in one team chat, and use GitHub Issues to track work, owners, and review requests. One designated operator will run a single shared AI-DLC workflow while all members help answer questions, review documents, and approve or request changes. Before starting Inception, we will dedicate a full meeting to discussing the problem, MVP, and input documents. Important documents will receive whole-team review; implementation work will be shared through branches and pull requests when useful. Operator and review responsibilities will be assigned once the complete roster is supplied.

## Documents

- [Vision document](docs/vision.md): users, goals, scope, workflow, acceptance criteria, and open questions.
- [Technical environment document](docs/technical-environment.md): stack, architecture, constraints, security, testing, and code-pattern examples.

These are input documents, not AI-DLC-generated Inception artifacts. Their defaults are proposals for team review; questions still requiring a team decision are listed explicitly.

## Development tools and workflow

| Item | Selection / current status |
| --- | --- |
| AI coding agent | OpenAI Codex; list any additional agents used by teammates |
| Required development methodology | AI-DLC **v1.0.1**, not the current main-branch version |
| Product model/provider | Planned: `qwen3:8b` / local Ollama; not yet benchmarked |
| AI-DLC rule installation | Pending; install the official v1.0.1 rules and commit them |
| Inception and approvals | Not started; no approvals or generated artifacts are claimed |
| Application / runtime setup | Planned for Construction; no application start command exists yet |

For Codex, the assignment places the official core rules in `AGENTS.md` and supporting rules in `.aidlc-rule-details/`. Use the [v1.0.1 release](https://github.com/awslabs/aidlc-workflows/releases/tag/v1.0.1) and its [setup instructions](https://github.com/awslabs/aidlc-workflows/blob/v1.0.1/README.md).

After the team reviews both input documents and installs the rules, start a shared workflow with a prompt such as:

> Using AI-DLC v1.0.1, read `docs/vision.md` and `docs/technical-environment.md`, then begin Inception. Ask the team clarifying questions and request approval at each required gate. Complete the required Inception artifacts through Units Generation, and stop before Construction.

Keep the workflow-generated `aidlc-docs/audit.md` and `aidlc-docs/aidlc-state.md` complete and committed. Review them at each gate and ask the agent to correct omissions through the workflow. They must reflect actual answers, decisions, and approvals.

## Milestone completion checklist

Due **November 24, 2026, 11:59 PM America/New_York**. One submission per team; 150 points, worth 15% of the course grade.

- [ ] List every member's full name and NetID, and add collaborators to GitHub.
- [ ] Review and agree on the README, Vision, and Technical Environment documents.
- [ ] Install, commit, and verify the official AI-DLC v1.0.1 rules.
- [ ] Run one shared Inception workflow and answer every required question.
- [ ] Produce requirements and completed requirement-verification questions.
- [ ] Produce user stories, personas, and the execution plan.
- [ ] Produce application-design files and all three units-of-work files.
- [ ] Check and commit the actual audit and state records.
- [ ] Approve Units Generation and stop before Construction.
- [ ] Push all work, verify the repository opens without signing in, and submit its URL on Canvas.

The final-project deadline shown in the assignment is **December 16, 2026**. The final stage includes Construction, a product demo video, a recorded presentation, the repository, and the internal team evaluation.
