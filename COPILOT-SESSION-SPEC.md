# Xod Copilot Session: Build Spec

Status: spec only, nothing built. Written 2026-11-07.
Directive (Rene, 2026-11-07): build a real way for Xod to sit in and help during work where AI help is allowed, openly, instead of Rene facing tool-restricted evaluations with no supported path. The immediate pressure is hiring assessments that ban AI outright; this spec is the consented-use answer to that pressure, not a workaround for the bans.

Working name: Copilot session. Rene can rename it; the name is load bearing in one way only, it must sound like what it is, a session the user opens on purpose.

## What this is

A Copilot session is a mode of Xod the user starts explicitly, on their own machine, under their own accounts, for work where AI assistance is permitted. While it runs:

- Xod can read the problem, brief, or page the user chooses to share with it.
- Xod and the user discuss approaches, by chat in v1 and by voice in v2.
- Xod drafts code and text into a staging pane inside Xod, never directly into an editor, a form, or a web page.
- Each draft reaches the user's workspace only through the user's approval of the actual diff, and the user applies it and submits everything themselves.

The human stays the driver at every step. Xod is the copilot: it reads, reasons, and drafts. It never holds the wheel, and it never touches a submit button.

## Where a session may be used

The start flow asks the user to pick the setting, and the setting is recorded on the session. Supported settings:

1. Take-home projects whose instructions permit AI tools.
2. Live interviews or pairing sessions where the interviewer has said AI assistance is allowed.
3. Practice and mock interviews, including self-run rehearsal against past problems.
4. Ordinary pair work on the user's own projects, where no third party's rules apply at all.

If the setting is an assessment or interview (settings 1 to 3), the start flow also requires a permission basis before the session opens: a short free-text record of who allowed AI use and in what words (a line from the task instructions, a recruiter's email, the interviewer's spoken yes at the start of the call). No basis recorded, no session. This is not a legal gate; it is a design statement that consent is part of the feature, stored with the work it covers.

## Consent is a first-class state

Three properties hold for the whole life of a session:

1. Visible. While a session is active, Xod shows a persistent AI-active state: a banner in the desktop app, a state line in the phone clients, and an entry in the capability manifest (see Components). There is no way to run a session with the indicator off. If the user screen-shares, anyone watching can see that a copilot session is running.
2. Recorded. The session keeps a complete local log: what was shared with Xod, the discussion, every draft, every approval or rejection, and what the user applied. The log is kept by default and is exportable in one action, so the user can hand it to an evaluator and show exactly how AI was used. Openness is the product: the log is the proof that nothing was hidden.
3. Bounded. A session ends when the user ends it or after an idle timeout the user sets. Ending a session closes Xod's access to anything shared during it; shared material is not retained as general context for later chats unless the user keeps it deliberately.

## The session flow

1. Start. The user opens a Copilot session from the desktop app (v1) and picks the setting. For assessment settings, they record the permission basis. Xod shows the AI-active state from this moment.
2. Share. The user shares the work with Xod in one of three ways: paste or type the problem into the session; attach files (a take-home repo, a brief, starter code); or point Xod at a page in their own browser, read through the existing browser-ops route, on a page the user has open. Xod reads only what is shared. A session has no ambient screen access and no background watching.
3. Discuss. Xod restates the problem, proposes approaches with tradeoffs, and suggests a test plan. The user picks the direction. This is ordinary Xod chat, scoped to the session, with the session log capturing it.
4. Draft. When the user asks for code, Xod writes it into the session staging pane as a diff against the shared files, or as a new file shown in full. Nothing is written to disk outside the session area at this stage.
5. Approve. The user reviews the actual diff in the approval surface and approves or rejects it, per draft. Approvals must mean something: the approval card shows the full change, approval is per draft and never bulk, and there is no auto-approve setting inside a Copilot session, even if the user has one elsewhere in Xod. This follows Rene's standing design rule that a rubber-stamp approval is worse than none.
6. Apply. On approval, the draft is written to the user's workspace files. From there the user runs it, tests it, edits it, and pastes or submits it wherever it belongs, themselves. Xod does not operate the destination: it does not type into assessment pages, fill forms, or press submit, in any setting.
7. Close and export. The user ends the session. Xod offers the session log export (Markdown, with drafts and approvals inline) and, for interview settings, offers to send a short disclosure note with the log through agent mail: what tool was used, in what mode, with the log attached. Sending is the user's choice and the user's tap.

## Components and reuse

V1 invents as little as possible. It is an assembly of pieces Xod already has, plus one new service.

- New: app/services/copilot_session.py. Owns the session record, the state machine (idle, active, ended), the staging area (a per-session directory under the data dir), and the log writer. One active session at a time in v1.
- Capability manifest (app/services/capability_manifest.py, shipped in PR #44): gains a copilot section. While a session is active it reports the session's setting, start time, and permission basis summary, so any client or agent answering "is AI helping right now, and under what agreement" answers from the manifest, never from a guess. When no session is active it says so.
- Approvals: the existing approval and needs-you machinery carries the draft approvals. Draft cards render in the same surface the user already trusts, with the diff as the card body. The session service supplies the drafts; the approval path is unchanged.
- Browser-ops: page sharing reuses the existing browser read route against the user's own browser. Read only in v1: the session can fetch the text of a page the user names, and cannot click, type, or navigate inside a session.
- Worker and chat: the discussion and drafting run through the existing chat and worker paths, with the session record injected as context (setting, shared artifacts, staging paths) the same way the capability manifest is injected today.
- Agent mail: the disclosure note and log delivery reuse agent mail and its existing approval rules for sends. Nothing about a session sends mail on its own.
- Family rules store: one new shared rule ships with the feature, copilot-consent-first: a Copilot session runs only where AI assistance is permitted, its active state is always visible, and its log is always kept. Written once in family-rules.json so every agent in the family inherits the boundary.
- Phone clients: v1 shows the AI-active state and the session log; starting a session is desktop only in v1, joining the state view from a phone is enough.

## Data model sketch

Session record (JSON, one file per session under the data dir):

- id, created_at, ended_at, state
- setting: one of takehome_permitted, interview_consented, practice, own_work
- permission_basis: free text, required for the first three settings, empty allowed only for own_work
- shared_artifacts: list of {kind: paste, file, page; label; path or url; shared_at}
- drafts: list of {id, created_at, target_file, diff_summary, status: staged, approved, rejected, applied}
- log_path: the session log file

API surface (sketch, beside the existing session and capability endpoints):

- POST /api/copilot/start {setting, permission_basis} returns the session record; 400 if an assessment setting has no basis.
- POST /api/copilot/share {kind, content or path or url} adds to the session.
- POST /api/copilot/draft {instruction} runs the worker against the shared artifacts and stages a draft.
- POST /api/copilot/drafts/{id}/approve and /reject, through the existing approval surface.
- POST /api/copilot/end returns the log path and the export.

## What this deliberately is NOT

- Not hidden. There is no stealth mode, no way to mute the AI-active state, and no design goal of making the assistance hard to notice. If a setting would require hiding the AI, the feature is the wrong tool for that setting.
- Not proctoring evasion. The session takes no position against any platform's detection and builds nothing to defeat it: no injected browser extensions for assessment pages, no synthetic typing, no second-device answer channel, no screen-share cloaking. Xod will not help conceal use, and the always-kept log exists partly so concealment has no foothold.
- Not for prohibited assessments. Where the rules ban AI assistance, the answer is the session stays closed. The supported path for those processes is the recruiter ask (see the companion kit in the job-search workspace): ask for permission, or ask to be measured on a real build. This feature is what Rene points to when permission is granted, proof that "let me use my tools" comes with visibility and a log, not with a trick.
- Not autonomous. The user starts it, directs it, approves every draft, applies the work, and submits. A session cannot run a task while the user is away, and nothing in it weakens Xod's existing approval rules anywhere else.

## V1 build order

1. Session service and record: state machine, staging directory, log writer, idle timeout. Smoke tests in the repo's style: start requires a basis for assessment settings, staging writes stay inside the session area, end closes access.
2. Start and share UI on desktop: setting picker, basis field, paste and file sharing. AI-active banner wired to session state.
3. Draft and approval loop: worker drafting into staging, diff rendering in the approval surface, per-draft approve and reject, apply to workspace on approval.
4. Manifest section and log export: copilot block in the capability manifest, one-action Markdown export, phone clients showing state.
5. Page sharing through browser-ops (read only), and the agent mail disclosure send.
6. Family rule added to the shared rules store; docs and changelog entry.

V2 candidates, unordered: voice discussion reusing the call presence audio stack, once that spec's v1 exists; phone-side session start; multi-draft comparison view; a practice mode pack with timed mock interviews that grades against the session log.

## Open decisions for Rene

1. The name. Copilot session is descriptive and a little plain; Rene may want it inside the Xod vocabulary.
2. Log retention default: keep forever, or auto-archive after a set period. The spec assumes kept, because the log is the openness proof.
3. Whether the permission basis field should also accept an attached artifact (a screenshot or email export of the permission) in v1, or stay free text until v2.
