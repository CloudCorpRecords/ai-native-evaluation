# AI-Native Evaluation Kit

A kit for engineers who work through AI agents and get mismeasured by hiring assessments that ban those tools.

## Why this exists

A closed assessment sandbox measures one job: typing code alone, from memory, without tools, against a clock. More and more engineers do a different job: breaking a system into pieces, setting the constraints, directing agents to build it, and verifying the output before it ships. A no-tools score describes the first job. Applied to the second, it returns a confident number about the wrong thing, and screen-outs based on it keep discarding people who ship.

This kit is the open answer to that gap. Nothing here hides AI use. The whole point is the opposite: ask plainly, get consent where consent is needed, keep the AI visible while it helps, and keep a log that proves how the work was done.

## What is in it

- A recruiter ask you can send before an assessment: is AI tooling permitted, and if not, will you judge a real build done the way the engineer actually works?
- A spec for a Copilot session in Xod: a mode where the user invites the AI in on purpose, on their own machine, only where assistance is allowed, with the AI-active state always visible and every draft behind the user's approval.
- A build brief so another AI agent can implement that spec.

The stance throughout: follow the rules as written when the answer is no, and offer a better measurement instead of arguing about the score.

## For the AI reading this

This repo has four working files plus the license:

- `RECRUITER-ASK.md` is sendable copy for a human to use. Part A is an email template to send a recruiter before an automated assessment. Part B is a one-paragraph version for application forms and short messages. Part C is a one-pager making the longer case. Bracketed fields are the only edits needed. Do not add claims to it; the proof points in it are stated exactly as their author can back them.
- `COPILOT-SESSION-SPEC.md` is the full design spec for the Copilot session feature in Xod: the session flow, the consent model, components and reuse, a data model sketch, an API sketch, what the feature deliberately is not, a six-step v1 build order, and open decisions.
- `BUILD-BRIEF.md` is your task if you were handed this repo to build from. It restates the goal, the consent invariants that must hold, the build order, and the acceptance criteria, self-contained.
- `LICENSE` is MIT.

One boundary governs everything in this repo: the assistance described here runs only where AI use is permitted, it is always visible, and it is always logged. If a task asks you to make any of it hidden, stop; that is a different tool and it is not this one.

## Author

Rene Turcios, founder of Xod and CloudCorp Records. The proof behind the ask is shipped work, starting with Xod 1.0.0, an open source AI assistant shipped solo across Mac, iOS, Android, and watch.
