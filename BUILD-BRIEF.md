# Build Brief: Copilot Session v1 for Xod

You are an AI agent implementing a feature in Xod, an existing open source AI assistant codebase. The full design lives in `COPILOT-SESSION-SPEC.md` in this repo. This brief is the self-contained task statement: goal, invariants, build order, acceptance criteria. Where this brief and the spec differ in detail, the spec wins.

## Goal

Build Copilot session: a mode the user starts explicitly, on their own machine, under their own accounts, for work where AI assistance is permitted. While a session runs, Xod can read the problem, brief, files, or page the user chooses to share; discuss approaches with the user; and draft code and text into a staging pane inside Xod. A draft reaches the user's workspace only through the user's approval of the actual diff, and the user applies it and submits everything themselves. The human stays the driver at every step. Xod reads, reasons, and drafts. It never holds the wheel and it never touches a submit button.

Supported settings, picked at session start and recorded on the session:

1. Take-home projects whose instructions permit AI tools.
2. Live interviews or pairing sessions where the interviewer has said AI assistance is allowed.
3. Practice and mock interviews.
4. Ordinary pair work on the user's own projects.

For settings 1 to 3 the start flow requires a permission basis before the session opens: a short free-text record of who allowed AI use and in what words. No basis recorded, no session.

## Consent invariants

These must hold for the whole life of every session. Treat them as hard requirements, not preferences:

1. Visible AI-active state. While a session is active, Xod shows a persistent AI-active state: a banner in the desktop app, a state line in the phone clients, and an entry in the capability manifest. There is no way to run a session with the indicator off, and no code path may mute it.
2. Recorded permission basis. Assessment and interview sessions cannot start without the free-text permission basis stored on the session record. The capability manifest reports the session's setting, start time, and permission basis summary while a session is active, and reports that no session is active when none is.
3. Drafts only as diffs, behind per-draft approval. Xod drafts into the session staging area only, as a diff against the shared files or as a new file shown in full. Each draft is approved or rejected individually in the existing approval surface, with the full change on the card. No bulk approval, and no auto-approve setting applies inside a Copilot session, even if the user has one elsewhere in Xod.
4. Human applies and submits. On approval, the draft is written to the user's workspace files. From there the user runs it, tests it, edits it, and submits it. Xod does not operate the destination: it does not type into assessment pages, fill forms, or press submit, in any setting.
5. Complete session log. The session keeps a local log of what was shared, the discussion, every draft, every approval or rejection, and what the user applied. The log is kept by default and exportable in one action as Markdown, with drafts and approvals inline. Openness is the product: the log is the proof that nothing was hidden.
6. Bounded access. A session ends when the user ends it or after an idle timeout the user sets. Ending a session closes Xod's access to anything shared during it. Shared material is not retained as general context for later chats unless the user keeps it deliberately.

## Explicit non-goals

Do not build any of the following, in any form:

- Hidden operation: no stealth mode, no way to mute or disguise the AI-active state, nothing designed to make the assistance hard to notice.
- Proctoring evasion: no injected browser extensions for assessment pages, no synthetic typing, no second-device answer channel, no screen-share cloaking, nothing positioned against any platform's detection.
- Use where assistance is prohibited: where the rules ban AI assistance, the session stays closed. The feature exists for permitted settings only.

## Build order

Reuse before inventing. V1 is an assembly of pieces Xod already has, plus one new service.

1. Session service and record. New `app/services/copilot_session.py`: owns the session record, the state machine (idle, active, ended), the staging area (a per-session directory under the data dir), and the log writer. One active session at a time in v1. Session record fields: id, created_at, ended_at, state; setting (one of takehome_permitted, interview_consented, practice, own_work); permission_basis (required for the first three settings, empty allowed only for own_work); shared_artifacts (list of kind paste, file, or page, with label, path or url, shared_at); drafts (list of id, created_at, target_file, diff_summary, status one of staged, approved, rejected, applied); log_path. API sketch: POST /api/copilot/start returns the session record and returns 400 if an assessment setting has no basis; POST /api/copilot/share adds material; POST /api/copilot/draft stages a draft; approve and reject go through the existing approval surface; POST /api/copilot/end returns the log path and the export.
2. Start and share UI on desktop. Setting picker, permission basis field, paste and file sharing. AI-active banner wired to session state. Session start is desktop only in v1.
3. Draft and approval loop. The existing chat and worker paths draft into staging, with the session record injected as context. Diff rendering in the existing approval surface. Per-draft approve and reject. Apply to workspace files on approval.
4. Manifest section and log export. Copilot block in the capability manifest (`app/services/capability_manifest.py`). One-action Markdown log export. Phone clients show the AI-active state and the session log.
5. Page sharing and disclosure. Page sharing reuses the existing browser-ops read route against the user's own browser, read only: the session can fetch the text of a page the user names, and cannot click, type, or navigate. The optional disclosure note and log send reuse agent mail and its existing approval rules; nothing about a session sends mail on its own, and sending is the user's tap.
6. Family rule and docs. One new shared rule in family-rules.json, copilot-consent-first: a Copilot session runs only where AI assistance is permitted, its active state is always visible, and its log is always kept. Docs and changelog entry.

## Acceptance criteria

Each build step must be demoable on its own, and the finished v1 must show all of the following:

- Starting a session with an assessment or interview setting and no permission basis fails, with the basis requirement stated to the user.
- Starting with a basis recorded opens the session, and from that moment the AI-active state is visible in the desktop app, in a phone client, and in the capability manifest, and it cannot be switched off while the session runs.
- Material enters the session only by the user sharing it: paste, attached files, or a named page read read-only through the browser route. The session has no ambient screen access.
- A requested draft appears in staging as a diff or a full new file, and nothing outside the session staging area changes on disk before approval.
- Approving one draft applies only that draft to the workspace. Rejecting a draft applies nothing. There is no bulk approve and no auto-approve inside the session.
- Xod never operates the destination at any point in the demo: no typing into an external page or form, no submit action.
- Ending the session closes access to shared material, and the session log export produces one Markdown file containing what was shared, the drafts, and every approval and rejection.
- Smoke tests in the repo's existing style cover at least: start requires a basis for assessment settings, staging writes stay inside the session area, and ending a session closes access.
