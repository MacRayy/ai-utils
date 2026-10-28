# The ExitPlanMode gate missed async verdicts because it keyed on the wrong id

> The §E enforcement hook claimed to "handle async/background verdicts," but in a real
> background session it blocked forever even after `plan-critic` returned `{"verdict":"approved"}`.
> The async branch tied approval to the dispatch's `tool_use` id — which the async verdict never
> carries.

**Date:** 2026-10-27

---

## The bug

`require-plan-critic.py` (§E in `agents-seed.md`) has two ways to detect an approved verdict:

1. **Synchronous** — the verdict arrives as the `tool_result` paired to the plan-critic
   dispatch by `tool_use_id`. Works in an interactive session.
2. **Async / background** — the dispatch's immediate `tool_result` is only an
   `"Async agent launched"` placeholder; the real verdict lands later as a **task-notification**.

The async branch accepted an `"approved"` string only if the same entry also carried a known
plan-critic **`tool_use` id**:

```python
and any(pid in full_text for pid in plan_critic_ids)   # pid = tool_use id
```

But a task-notification is keyed by the **runtime agent/task id** (e.g.
`agentId: a81b54a6028cacb53`), not the dispatch's `tool_use` id. The two are different strings, so
the match never fired — the approval was invisible and the gate blocked every ExitPlanMode. The
"stuck in plan mode" loop the module docstring warned about, caused by the module itself.

## The fix

Resolve the agent id from each plan-critic dispatch's async placeholder `tool_result`
(`agentId: …`) and accept a verdict that carries **either** token — the `tool_use` id **or** the
resolved agent id:

```python
AGENT_ID_RE = re.compile(r"agent[_ ]?id\D{0,4}([0-9a-f]{6,})", re.I)
# collect agent ids from the dispatch's placeholder result into plan_critic_tokens, then:
and any(tok in full_text for tok in plan_critic_tokens)
```

The security property is unchanged: an unrelated `"approved"` string carries neither token, so it
still can't satisfy the gate. Fail-open behavior is unchanged.

## Why it matters generally

Any hook that correlates a later notification back to an earlier tool dispatch must bridge
**tool_use id → runtime agent id** — the harness does not reuse one id for both. Matching on the
dispatch id alone silently works in interactive sessions (where the verdict is a paired
`tool_result`) and silently fails in background ones (where it is a free-standing notification).
Mirrors the same fix shipped to the private twin.

Verified with six transcript scenarios driven through the hook's real stdin path: async-approved
(via agent id) → allow; async-needs_revision → block; unrelated/approved-without-token → block;
resumed-agent-then-approved → allow; not-opted-in cwd → allow.
