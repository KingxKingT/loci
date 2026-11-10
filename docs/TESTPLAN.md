# P5.2 — adversarial brief test: runbook

Status: dataset designed 2026-11-10; staging next. Created from Test 2's PASS
(P5.3) and the external reviewer's quality ledger.

## What this test is

Test 2 proved retrieval works under clean conditions. This test tries to make
it fail: an irrelevant memory, a superseded decision, a project boundary, and a
low-importance `(inferred)`-flagged memory, all live in the same brief.

## Isolation rules (non-negotiable)

- The dataset loads into an isolated store via `LOCI_HOME=<testdir>`;
  `~/.loci` (the real store) is never touched.
- The real Loci instance is stopped while the ChatGPT-extension leg runs:
  the extension takes the first live instance on ports 7629–7635, and the
  wrong store answering would invalidate everything.
- Ground truth, scoring language, and this document stay outside every tested
  conversation. Questions are sent bare in fresh conversations.
- Before any graded pass, the tested store is leak-checked for test-design
  vocabulary. A fresh `LOCI_HOME` store starts empty by construction; the
  10 Nov 2026 auto-capture incident (host-injected text written as memories)
  proves leaks can appear without anyone typing them, so the check is a gate,
  not a formality.
- 10 Nov 2026 status: the polluted real store was purged after triage (45
  records forgotten; 15 keepers verified present; no test vocabulary
  searchable), auto-capture is `off` on it, and the capture boundary now
  rejects host-injected checkpoint envelopes in source with regression
  tests (suite 216/216). The gate above remains mandatory before every
  graded run.
- Hedging is recorded verbatim as information, not scored as failure.

## Dataset (five items, identical in every client)

Items 1–4 load with `loci add`; item 5 loads through the real auto-capture
pipeline (`loci capture --auto`) so its `inferred` flag is set the same way a
beta user's would be.

| # | Role | Content (as stored) | Kind | Scope | Importance |
| --- | --- | --- | --- | --- | --- |
| 1 | Superseded decision (added first) | "Use plain JavaScript for all new integration code." | decision | global | 4 |
| 2 | Active decision (`add --supersedes <id-of-1>`) | "Use plain TypeScript for all new integration code." | decision | global | 4 |
| 3 | Project preference | "Commits in the acme-site pipeline follow Conventional Commits prefixes (feat:, fix:, docs:)." | preference | project `acme-site` | 3 |
| 4 | Irrelevant memory | "On a winter trip to the Alps, the user mentioned enjoying the deep quiet snow most." | fact | global | 1 |
| 5 | Inferred low-importance fact (via capture --auto) | "The user's summer timezone is UTC+2." | fact | global | 1 |

## The questions (each sent bare in a fresh conversation)

| Q | Question | Tripwire | PASS | FAIL |
| --- | --- | --- | --- | --- |
| Q1 | "What language should new integration code be written in, and why?" | superseded decision | Cites TypeScript; never presents JavaScript as current | Cites JavaScript, or presents both |
| Q2 | "Write a one-line changelog note about adding gentle snow to the UI's winter mode." | irrelevant memory | Changelog text only | Uses "you like snow" / the Alps memory to justify wording |
| Q3 | "Do commits in the acme-site pipeline follow Conventional Commits?" | project boundary | Cites the preference while `acme-site` is the active project | Vague or misses it; or (inactive-project leg) invents acme-site conventions |
| Q4 | "What is the user's timezone in summer?" | `(inferred)` label | States UTC+2 (hedging recorded as info) | Wrong number; heavy hedge = recorded quote, not a fail |

## Hosts (same dataset, same four questions in each)

1. ChatGPT web via the 0.9.4 extension (isolated `LOCI_HOME` store, real
   instance stopped).
2. Claude Desktop (MCP; store pointed at the test dataset).
3. Cursor (MCP; same).
4. OpenCode (MCP; same).
5. Windsurf or Zed (MCP; same — pick whichever is installed first).

For hosts 2–5, the dataset loads into whatever store that client's MCP config
targets; the load commands are rerun there by the agent before the client opens.

## Load protocol (executed by the agent at run time)

```
set LOCI_HOME=<testdir>\p52-store
loci init
loci add "Use plain JavaScript for all new integration code." --kind decision --tag p52 --importance 4 --origin "P5.2 dataset, superseded"
loci add "Use plain TypeScript for all new integration code." --kind decision --tag p52 --importance 4 --supersedes <id-from-step-1> --origin "P5.2 dataset, active"
loci add "Commits in the acme-site pipeline follow Conventional Commits prefixes (feat:, fix:, docs:)." --kind preference --tag p52 --importance 3 --scope project --root <testdir>\acme-site
loci add "On a winter trip to the Alps, the user mentioned enjoying the deep quiet snow most." --kind fact --tag p52 --importance 1 --origin "P5.2 dataset, irrelevant"
loci capture --auto --save --file <testdir>\seed-transcript.txt   (transcript contains the timezone line)
loci brief --query "integration code" --scope both   (sanity: items 2 (active) and the preference visible; item 1 must NOT appear)
```

Expected brief sanity: item 1 (JavaScript) absent; item 2, 3, 4, 5 present;
item 4 last or cut by budget before item 2 is ever dropped.

## Results log (fill in per host)

| Host | Q1 | Q2 | Q3 | Q4 | Notes |
| --- | --- | --- | --- | --- | --- |
| ChatGPT web | | | | | |
| Claude Desktop | | | | | |
| Cursor | | | | | |
| OpenCode | | | | | |
| Windsurf/Zed | | | | | |

Verdicts move into `PLAN.md` P5.2 with verbatim answers once a host's column is
complete. A single host passing does not close the row; the row closes when
every installed host has a column.
