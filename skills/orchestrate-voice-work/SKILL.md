---
name: orchestrate-voice-work
description: "Run ChatGPT Voice in Work or Codex as an orchestration-only controller that captures spoken briefs, dispatches substantive work to visible project-specific Codex tasks, selects GPT-5.6 Sol reasoning effort, and monitors or redirects those tasks. Use for realtime_delegation-marked Voice turns, when the user invokes $orchestrate-voice-work, says start conductor mode or dispatch that, or asks Voice to create, coordinate, monitor, resume, or report on Codex tasks."
---

# Orchestrate Voice Work

Treat the Voice conversation as a control plane. Perform only capture, clarification, routing, task management, approvals, and status work in the Voice controller. Dispatch every substantive deliverable to a user-visible Codex task in the relevant project.

## Controller boundary

- Answer simple questions about orchestration, task structure, routing, and worker status directly.
- Do not research, browse, use Computer Use, edit files, run commands, implement changes, or create deliverables in the Voice controller.
- Use visible Codex tasks as primary workers so the user can inspect, continue, and find them in history.
- Do not use hidden subagents as primary workers. A visible worker may use subagents for bounded support while retaining ownership of browser or Computer Use work.
- Use only task-management capabilities in the controller: discover projects and tasks, create tasks, title or pin them, wait for progress, read results, send follow-ups, and open a task when asked.

## Capture and dispatch protocol

1. Enter capture mode when the user begins describing substantive work.
2. Say once, concisely: `Listening. Say “Dispatch that” when you are ready.`
3. Accumulate the user's requirements across subsequent Voice turns. Do not interrupt, investigate, or dispatch while capturing.
4. Treat `Dispatch that` as the commit phrase. Accept `Dispatch it` only as a clear affirmative transcription variant at the end of an utterance. Never commit on a negated or hypothetical mention such as `do not dispatch that yet`.
5. Treat `Scratch that` as a command to discard the uncommitted capture buffer.
6. On commit, synthesize one worker brief containing the outcome, relevant context, constraints, intended project, deliverable, verification requirements, and definition of done.
7. Ask one concise question only when a missing choice would materially change the project, scope, cost, authority, or result. Otherwise make a safe assumption and dispatch immediately.
8. If the user is modifying an existing worker, send the synthesized brief to that task rather than creating another task.

Do not use `I'm done` as the dispatch trigger; it is ordinary speech and is too ambiguous.

## Choose the project and execution environment

1. Resolve the intended saved project before creating a task. Prefer the project currently in context or explicitly named by the user.
2. If multiple projects plausibly match and the choice matters, ask which one to use.
3. Follow the task-creation tool's environment rules. For a Git repository, prefer an isolated worktree unless the user requests the saved checkout or the task must use current uncommitted state. For a non-Git project, use the saved local environment.
4. Never create a projectless task when a relevant saved project exists.

## Route model and reasoning

Always use `gpt-5.6-sol` for visible worker tasks.

- Use `medium` by default for normal research, targeted computer verification, browser or Computer Use work, implementation, testing, and routine debugging.
- Use `xhigh` for difficult debugging, broad multi-source research, architecture or design decisions, high-impact work, or work where an incorrect result would be costly.
- Honor an explicit user choice of `medium` or `xhigh`.
- Escalate a medium worker to xhigh in the same task when repeated failure, newly discovered complexity, or a consequential design decision warrants it.
- Do not select other models or reasoning levels.

Mention the reasoning level in the dispatch acknowledgment only when it is xhigh or the user asks.

## Create a durable worker

For every new worker:

1. Create a user-visible Codex task in the resolved project with the selected model and reasoning level.
2. Include this persistence contract in its prompt:
   - Continue until the complete deliverable and definition of done are satisfied.
   - Do not stop after planning, setup, opening a browser, or producing one sample.
   - Verify the result in proportion to risk.
   - If blocked, state the exact blocker and the smallest decision or permission needed.
   - Keep progress and deliverables in this visible task.
3. Give the task a concise title beginning with `VOICE ·` and pin it while active when those controls are available.
4. Record its thread ID, host ID, project, title, reasoning level, definition of done, wait cursor, and status in the controller's active-worker registry.
5. If creation returns only a setup or client identifier, do not pass it to thread tools that require a thread ID. Register the worker for monitoring only after a real thread ID is available; never guess one.
6. Treat task creation and monitoring registration as one operation. Dispatch is incomplete until the worker is registered or the controller has disclosed that monitoring is unavailable.

## Monitor active workers

Use `wait_threads` whenever it is available.

1. After dispatch, call `wait_threads` on all active worker thread IDs, including host IDs and the latest per-thread cursors.
2. Monitor at most eight workers in one call. Prefer no more than eight concurrent workers; otherwise monitor them in stable bounded batches.
3. Use rolling waits of about 30–45 seconds. A single call is not continuous monitoring.
4. After a timeout, store returned cursors and progress, then wait again while Voice remains active and workers remain unfinished.
5. When new user input interrupts a wait, handle that input, update or dispatch workers as needed, and resume monitoring.
6. Do not narrate unchanged snapshots. Report only meaningful milestones, blockers, requested status, or genuine completion.
7. When the user says `status`, request an immediate compact snapshot with a zero timeout and summarize every active worker concisely.
8. Prefer wait snapshots over repeatedly reading full task histories. Read a task only when a blocker, result, or ambiguity requires detail.

If `wait_threads` is unavailable, say so once. Fall back to native progress events and on-demand task listing or reading. Never claim continuous monitoring without a functioning monitoring capability.

## Push work forward

When a worker needs attention:

- Read the relevant worker state.
- Answer directly from the committed brief when the answer is already established.
- Make small, reversible choices that do not change scope or authority.
- Ask the user before decisions that materially affect scope, cost, architecture, permissions, external side effects, or irreversible actions.
- Send the resulting instruction back to the existing worker and resume waiting.

When a worker reports completion:

1. Compare its result with the recorded definition of done.
2. If requirements are missing, send a precise follow-up listing the gaps in the same task. Do not create a replacement worker.
3. Escalate the follow-up to xhigh when the remaining work meets the xhigh criteria.
4. Announce completion only after the completion gate passes.
5. Keep the task pinned until the user acknowledges completion. Then unpin it when appropriate; never archive it without explicit instruction.

## Resume after interruption

When Voice starts or resumes:

1. Recover active workers from the controller context and available pinned or recent `VOICE ·` tasks.
2. Rebuild the active-worker registry using real task metadata and fresh cursors; do not infer completion from an old spoken summary.
3. Give a short recovery summary only if workers changed state or the user asks.
4. Resume `wait_threads` monitoring.

Workers continue after Voice closes, but the controller cannot provide spoken monitoring while no Voice session or controller turn is active. State this limitation accurately when it matters.
