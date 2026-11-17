# MeetTrack — Vision Document

**Version:** 0.1 · **Date:** 2026-11-17 · **Status:** Draft for team review

This document proposes the product scope supplied to AI-DLC v1.0.1. It does not record completed team research, approvals, or implemented features.

## 1. Product vision and problem

MeetTrack is a local web application that helps student project teams turn text meeting notes into reviewed, persistent tasks. It extracts commitments and decisions, proposes owners and dates only when supported by the notes, and reconciles later discussions with existing work. A human reviewer controls what is saved to the task board. The intended result is a clear record of commitments and changes across successive meetings.

The initial problem hypothesis is that small course-project teams spend unnecessary effort transferring informal meeting notes into task lists. References such as “the interface work,” “that page,” and “the same thing from last week” make simple keyword extraction insufficient. Missing owners, tentative suggestions, revised deadlines, and repeated discussion can produce uncertainty or duplicate tasks. This hypothesis will be checked with classmates; no user interviews have been completed for this draft.

The product's central value is continuity from discussion to execution and the next meeting. LLMs interpret the language and propose changes; application rules and user review control the resulting task data.

## 2. Target users and operating context

| User | Context and needs | MVP interaction |
| --- | --- | --- |
| Course-project coordinator | Coordinates a small team and needs a consistent view of commitments | Creates a project, reviews suggestions, and checks work before the next meeting |
| Meeting recorder | Has informal notes that require organization and clarification | Pastes notes, checks source excerpts, and corrects or confirms extracted fields |
| Project teammate | Needs to know assigned work and understand later changes | Participates in shared review and views tasks filtered by member |

The initial setting is a course-project team of roughly 3–6 members. The MVP is operated by one person in one local browser session, with teammates reviewing through a shared screen when needed. Member records are task-assignment identities, not authenticated user accounts or permission roles.

Proposed pilot limits are 5,000 Unicode characters per meeting record and 50 current tasks per project. Existing tasks, including completed tasks, remain available for matching within that limit. The first evaluated workflow uses English notes and an English interface; Chinese and mixed-language quality are open questions rather than guaranteed capabilities.

## 3. End-to-end user journey

1. **Set up a project.** Enter its name and member roster.
2. **Record a meeting.** Enter a title, meeting date, and plain-text notes. Speaker labels are helpful but optional.
3. **Analyze the notes.** Receive source-linked decisions and candidate tasks. The system identifies missing fields and possible changes to existing tasks.
4. **Review suggestions.** Inspect the cited text and any before/after comparison; edit, approve, or reject each proposal. An uncertain match requires a user choice.
5. **Track work.** Use the saved task board to filter assignments, update status, and inspect due dates and provenance.
6. **Review the next meeting.** Analyze new notes against current tasks and confirm additions, reassignments, deadline changes, and completion reports.

### Illustrative two-meeting scenario

On November 18, a fictional team records: “Alex will finish the login page by November 21. Casey will finish the API by November 22. We still need someone to handle testing.” The application proposes two fully assigned tasks and one task with an unresolved owner and date. A reviewer confirms or edits them.

On November 20, the team records: “Jordan is taking over the login page, with a new deadline of November 23. Casey has finished the API.” The application proposes updating the existing login task and marking the API task complete. The testing task remains unchanged. Suggestions become effective only after review, and each accepted change retains the meeting and source passage supporting it.

## 4. MVP features in scope

| ID | Feature | Required behavior | Rationale |
| --- | --- | --- | --- |
| F1 | Project and roster management | Create and list projects; add or rename member records; keep each project's records separate | Establishes the context needed for assignment and matching |
| F2 | Text meeting records | Save a title, meeting date, and original notes; list and reopen previous meetings | Supplies durable input and source evidence |
| F3 | Structured extraction and review | Extract decisions and action items with evidence; show missing information; allow editing, approval, or rejection | Converts language into accountable work while retaining human control |
| F4 | Persistent task board | Show title, owner, due date, status, and source; filter by owner/status; mark overdue work; edit tasks manually | Supports execution after the meeting ends |
| F5 | Cross-meeting reconciliation | Propose additions or changes to same-project tasks; show before/after values; require confirmation for uncertain matches | Preserves continuity and limits duplicates |
| F6 | Change provenance and recovery | Retain accepted/rejected proposals and applied changes; preserve saved input when analysis fails; permit safe retry | Makes suggestions reviewable and errors recoverable |

The five user-facing modules comprise F1–F5. F6 is supporting behavior within review and tracking, not a separate dashboard or analytics subsystem.

## 5. Business rules and acceptance criteria

- Task statuses are **To do**, **In progress**, and **Done**. Overdue is computed from a known due date and unfinished status; it is not another status.
- Owner and due date may be empty. The interface labels these “Unassigned” and “Date not specified” and allows the reviewer to save them intentionally.
- A suggestion, rejected idea, or explicitly negated commitment must not be silently treated as an agreed task. Ambiguous cases are labeled for review.
- Relative date wording is preserved. Any normalized date is a proposal based on the meeting date; conflicting or ambiguous wording must be resolved by the user.
- Missing mention of a task in a later meeting does not imply completion, cancellation, reassignment, or deletion.
- The system must not guess an owner from an unlabeled “I will do it.” Only supported member identities may be proposed.
- Model-generated additions and updates remain proposals until approved. Manual task edits are deliberate user actions and are also recorded.
- Every extracted task/change includes a source segment and excerpt. An exact quote check establishes provenance, while user review and evaluation check whether the interpretation is correct.
- A suggested update may target only an existing task in the same project. Uncertain matches require the reviewer to select a target or choose a new task.
- Applying the same approved proposal again does not repeat the change or create another task. If a task changed after analysis, approval requires a fresh review of the current values.
- Original saved meeting notes are preserved. Corrections after analysis are submitted as a new meeting record so existing source references remain meaningful.
- Restarting the application preserves projects, meetings, tasks, and proposal history. A failed model request does not change existing tasks.

## 6. Features out of scope

| Excluded feature | Reason | Possible later phase |
| --- | --- | --- |
| Audio/video recording and transcription | Adds speech quality, file handling, and consent considerations beyond the core text workflow | Post-MVP |
| Zoom, Slack, email, calendar, or task-platform integration | Adds external authentication and synchronization dependencies | Post-MVP |
| Automatic notifications or sending messages | The MVP focuses on reviewed records and task tracking | Post-MVP |
| Separate accounts, role-based access, and concurrent editing | The pilot is a single-operator local application | Post-MVP |
| Cloud hosting and internet-accessible deployment | Local execution meets the course project's deployment need | Post-MVP |
| Automatic deletion or cancellation of tasks | Requires additional lifecycle and recovery design | Post-MVP |
| General chat, model training, and large document retrieval | Not necessary for the bounded meeting-to-task workflow | Post-MVP |
| PDF/image import, OCR, and rich-text editing | Plain-text input is sufficient to validate the product | Post-MVP |
| CSV export, agenda drafting, and analytics dashboards | Useful extensions after the end-to-end workflow works | Optional stretch |

At maturity, MeetTrack could support shared workspaces, meeting-service integrations, reminders, and agenda preparation. These aspirations do not expand the MVP acceptance criteria.

## 7. Success metrics and evaluation plan

All targets below are proposed acceptance goals, **not measured results**. Freeze the annotated evaluation set before final model/prompt evaluation and report sample counts, actual results, and limitations.

| Measure | Proposed target | How to evaluate |
| --- | --- | --- |
| Core workflow completion | A reviewer completes project setup and two successive meetings, including a reassignment, a deadline change, and a completion update | Scripted end-to-end demo with synthetic data |
| Action-item extraction | Precision and recall each at least 0.85 on the held-out action-item set | Manual semantic matching against annotated records; report both counts |
| Existing-task matching | At least 0.85 correct targets among unambiguous labeled updates; uncertain cases reported separately | Compare proposed task IDs with annotations |
| Unsupported owner/date claims | Zero in a dedicated set of missing-information and ambiguous-reference cases | Check the extracted fields against source evidence |
| Review and storage invariants | All tested cases preserve approval control, project isolation, persistence, and repeat-approval safety | Deterministic service/API tests |
| Pilot usability | At least 2 of 3 volunteer classmates can complete the scripted review workflow without step-by-step help | Observe a short usability session; record edits and difficulties |

Prepare approximately 15 synthetic or de-identified meeting records in five three-meeting sequences. Reserve three sequences for development and two sequences for held-out evaluation; include several action items per sequence and report the resulting task/update sample sizes. Include explicit commitments, missing fields, suggestions, negation, paraphrases, reassignments, date changes, completion, and uncertain references. Collect real notes only with participants' permission; public examples should be synthetic.

## 8. Risks and mitigations

| Risk | Planned mitigation |
| --- | --- |
| Plausible but unsupported extraction | Require evidence, retain empty fields, and provide explicit review before application |
| Duplicate tasks or incorrect task matching | Supply stable task IDs; show candidate matches and differences; permit rejection or manual target selection |
| Slow or inaccurate local model | Evaluate the selected model on a team laptop early; adjust the model/provider through the adapter only after team agreement |
| Review screens become too complex | Use one list of proposals with expandable evidence and before/after fields |
| Limited development time | Implement the five user-facing modules in order; defer all listed stretch features |
| Sensitive information in notes or commits | Use synthetic demonstration records; keep runtime notes, databases, and secrets out of version control |

## 9. Open questions for team review / Inception

| Question | Proposed starting point | Decision needed |
| --- | --- | --- |
| Which classmates will validate the problem and try the prototype? | Recruit three volunteers after the team agrees on scope | Assign recruitment and confirm participants |
| Is English-only evaluation sufficient for the first release? | English interface and primary evaluation notes | Confirm whether Chinese support is required |
| Does `qwen3:8b` meet quality and latency needs on available hardware? | Test ten short development examples locally | Record hardware, speed, quality, and final model choice |
| What does “next Friday” mean in the team's meeting context? | Preserve the wording and request explicit date confirmation | Agree on any automatic normalization rules |
| Are the 5,000-character and 50-task limits suitable? | Keep these as pilot limits | Check sample records and adjust before freezing requirements |
| Who operates the workflow, reviews each document, and owns evaluation? | One shared AI-DLC workflow with all members participating | Supply roster and assign responsibilities |

Resolve these questions through actual discussion and the AI-DLC workflow. Preserve the resulting answers and approvals in the workflow's own record.
