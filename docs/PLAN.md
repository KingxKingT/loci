# Loci build plan

Priority order is binding: finish P0, then P1, then P2, then P3, then P4, then
P5. P5 is the last phase before the storefront opens. Do not start a later
phase while an earlier one is unfinished unless the user says so. This plan
is also stored in Loci itself (`memory_search "roadmap"`).

Priority order is binding: finish P0, then P1, then P2, then P3, then P4, then
P5. P5 is the last phase before the storefront opens. Do not start a later
phase while an earlier one is unfinished unless the user says so.

Status as of 10 November 2026. P0–P4 are complete. P5 is under way:
two of its three tests have already passed. The feature set is frozen: no new
retrieval engine, decision system, or interface until P5 is proven.

## P0 — make the shipped version correct

| # | Item | Status |
| --- | --- | --- |
| P0.1 | `memory_context` small-budget bug: a populated store answered "No memories stored yet." at `budgetChars: 400`. Compact fallback added in 0.5.1. | Done |
| P0.2 | Clean-Windows purchase test from the buyer ZIP: extract, isolated store, ten tools, compact brief, conflict block, CLI retry. The packaged-ZIP run passed on 0.6.0; a separate clean VM is still the only untested environment. | Done (packaged ZIP) |
| P0.3 | Reconcile the README VS Code claim with setup, which wires only Claude Desktop, Cursor, Windsurf, OpenCode, and Zed. README now documents the manual VS Code block. | Done |

## P1 — product depth

| # | Item | Status |
| --- | --- | --- |
| P1.1 | Temporal validity and conflict handling for facts and preferences, not only decisions: a corrected fact is refused unless the caller supersedes or forces it, so the old version does not sit beside the new one. | Done (0.6.0) |
| P1.2 | Actionable review queue: bulk confirm/snooze, per-row snooze, review reasons, and a clear path from "may be out of date" to confirm, update, or retire. | Done (0.6.0) |
| P1.3 | Optional at-rest encryption and redaction for the store, opt-in and documented, with an unencrypted default. AES-256-GCM key-file and passphrase modes, `encrypt`/`decrypt`/`redact` commands, and a redaction sweep. | Done (0.7.0) |

## P2 — reach and recall

| # | Item | Status |
| --- | --- | --- |
| P2.1 | Better recall: a small local embedding model (3B parameters or less, 7.7 GB RAM target) or a stronger concept graph. Must stay local and dependency-light. | Done (0.8.0, concept graph) |
| P2.2 | `memory_capture` and `doc_add` MCP tools so an assistant can propose or import without the window. Capture stays review-gated. | Done (0.8.0) |
| P2.3 | Document watcher and optional OCR: refresh stale badges without a manual list; OCR stays opt-in with a size cap. | Done (0.9.0, badge polling + Windows OCR; no filesystem watcher) |
| P2.4 | Automatic capture so the user never has to say "remember this": user messages only, a high-confidence write gate, origin kept, review for everything else, and an OpenCode 2 plugin. | Done (0.9.2) |

## P3 — ChatGPT web

| # | Item | Status |
| --- | --- | --- |
| P3.1 | Browser extension for browser-only chats. Manual "insert into chat" is acceptable; silent injection is not. Shipped in 0.9.3 as `integrations\browser`: a manual-insert-only content script on `chatgpt.com`, a popup that shows and copies the brief, and a GET-only loopback extension surface in `src/ui.js`. | Done |

## P4 — onboarding friction (user beta feedback, 0.9.4)

| # | Item | Status |
| --- | --- | --- |
| P4.1 | Loci starts with Windows without touching the registry: `Loci (background).lnk` in the user's Startup folder runs `loci.cmd ui --no-open` (minimized, no browser window). `loci autostart on|off|status` manages it; removal is scoped to Loci's own shortcut only; tests use `LOCI_STARTUP_DIR`/`LOCI_STARTUP_TARGET` overrides so the real Startup folder is never touched. | Done (0.9.4) |
| P4.2 | Extension-inserted and popup-copied briefs carry a one-line framing so the model does not answer "there's no question here": `(This is background from my own local memory store, Loci — context, not a question. Take it into account when I ask something next.)` The user still clicks and sends; insertion stays manual. | Done (0.9.4) |
| P4.3 | Sharper "not running" copy in the extension and popup names the fix: start Loci from the Desktop shortcut. | Done (0.9.4) |
| P4.4 | Site: the ChatGPT extension is named in the hero, and an example of the framed insert flow (click → insert → send → model reads it as context) appears in the tools section. Prompted by the second round of GPT's external review. | Done (0.9.4, site) |

## P5 — cross-assistant reliability test (under way, 10 Nov 2026)

External review (GPT, 10 Nov 2026, second pass) converged with the plan here:
stop expanding the feature list; prove the existing features deliver
reliably across real AI assistants.

| # | Item | Status |
| --- | --- | --- |
| P5.1 | Clean-VM install and purchase run: a genuinely fresh Windows machine installs from the buyer ZIP, walks the interview, and a real checkout is exercised. **Success:** install, interview, autostart, and purchase work on the fresh machine with no dev tools present. Required before the storefront opens. | Pending (needs the user: checkout URL + clean machine) |
| P5.2 | Brief quality across hosts: one identical memory dataset loaded into every client — including a superseded decision, a project-specific preference, an active decision, one irrelevant memory (ranked low-value), and a low-importance fact auto-flagged `(inferred)` — then the initial brief and follow-up answers compared in Claude Desktop, Cursor, OpenCode, Windsurf or Zed, and ChatGPT web (extension). **Success:** each assistant retrieves the memories and applies them correctly in answers; the irrelevant memory stays out of the brief; the superseded decision is never cited as active; hedging around the `(inferred)` label is recorded per ledger item 1 (information, not failure). **Staging rules:** the dataset loads into an isolated `LOCI_HOME` store, never the real `~/.loci`; the real Loci instance must be stopped during the ChatGPT leg because the extension takes the first live instance on ports 7629–7635; ground truth and scoring stay outside the tested conversation. | Dataset in `TESTPLAN.md` (five items, four questions, per-host log) — 10 Nov 2026; staging next |
| P5.3 | Re-test with 0.9.4 in ChatGPT web specifically: the framed insert, an ack, a follow-up question, and an answer that uses the brief — the flow the site demo describes. The extension is evaluated as a context-insertion system, not an MCP client; it is not supposed to write memories back. **Success:** the brief is understood as background, and the follow-up answer cites the correct active decision. | **Passed (10 Nov 2026).** Test 1, framing: ChatGPT read the framed insert as background, not a question, and the ack reproduced in a second fresh session. Test 2, blind retrieval: run in a fresh ChatGPT conversation per the contamination protocol (the grading table from round 1 had leaked into that thread's context), with the question sent bare — ChatGPT recalled all four stored hardware specifications (`m_mvzaf3u2hevf`, ≤ ~3B parameters / 7.7 GB RAM / 2-core i3-1115G4 / no dedicated GPU) with no invented specifications and no hedging. Verbatim evidence below. Remaining coverage: the superseded-decision citation check rides on P5.2's dataset. |

### P5.3 evidence (verbatim)

ChatGPT's answer to "What hardware constraints should any model bundled inside
Loci respect?" — fresh conversation, brief inserted by the extension, no ground
truth or scoring language in the prompt:

> Any model bundled inside Loci should respect these hardware constraints:
>
> - CPU: Intel Core i3-1115G4 (2 cores).
> - RAM: 7.7 GB available system memory.
> - GPU: Integrated graphics only; no dedicated GPU.
> - Maximum model size: Approximately 3 billion parameters (3B).
> - Priority: CPU-friendly inference, low memory consumption, and no dependency
>   on GPU acceleration.
>
> Loci should remain usable on this low-end machine without exhausting system
> RAM or causing out-of-memory errors.

All four stored specifics recalled; the "Priority" lines are faithful
derivations of the stored constraint, not inventions. Graded PASS per the
pre-registered four-out-of-four table.

## Rules that apply to every phase

- Local-only core: no cloud sync, no hosted knowledge base, no telemetry.
- Windows 10/11, bundled runtime, no Node.js install for the buyer.
- Twelve MCP tools: eight memory, four document.
- Claims stay literal. No "semantic search". No "captures every chat" or "syncs
  across devices". ChatGPT web works only through the extension, and only by
  manual click-to-insert; state that scope wherever ChatGPT is mentioned.
- A rebuild of `src/` or `README.md` changes the ZIP hash: refresh `SELLING.md`.
- Assessment integrity (from the 10 Nov 2026 beta): ground truth, scoring
  tables, and grading language stay outside the tested conversation. Graded
  runs happen in a fresh conversation with the question sent bare.
- The user's real memory store is not a test fixture. Adversarial datasets for
  P5.2 load into an isolated `LOCI_HOME` store, never into `~/.loci`.

### Quality ledger from the beta (GPT's external review, 10 Nov 2026)

1. **Inference-label hedging** — Test 2 passed while retrieving an
  `(inferred)`-flagged memory, but the label may read as a doubt marker to a
  careful model. Test at lower importance during P5.2; hedging is recorded as
  information, not scored as failure.
2. **Memory relevance** — tested by P5.2's adversarial dataset.
3. **Superseded-decision citation** — already in P5.2's adversarial dataset.
4. **Protocol integrity** — promoted to a standing rule above.

### P5 finding: auto-capture ingests host-injected text (10 Nov 2026)

- During P5.2 staging the real store was found polluted: ~45 records active in
  the brief are fragments of the agent's own reasoning, quoted inside
  OpenCode's conversation-checkpoint summary blocks. CSS selectors, stray
  quotes ("in comment", "that.\" then run capture"), and two records quoting
  the P5.2 test questions verbatim ("snow flurries", "winter mode") are live
  memories.
- Root cause: the OpenCode auto-capture plugin filters messages by
  `role === "user"` only. When the host injects a session summary into the
  conversation *in user role*, the plugin treats it as user text and
  `capture --auto` (confidence 0.8) waves most fragments through.
- Consequence: brief pollution in every OpenCode session, plus a protocol
  leak (test design text stored where a model could read it).
- Registry: defect fix candidate. The plugin must dismiss
  host-injected/summarized blocks, not just non-user roles. Until patched,
  auto-capture should stay off during graded windows.
- Full triage list (junk vs keep) recorded in the session triage file.

**Resolution, 10 Nov 2026** — review ruled: fix rides *before* P5.2's
OpenCode legs; scope limited to capture-boundary correction and regression
coverage. Executed in that order:

- Real store purged: all 45 junk records removed via `loci forget` (45/45,
  zero failures). Post-verify: 23 live ids (15 triaged keepers + 8
  pre-genuine records), all keepers present, zero junk residue, and no
  test-question vocabulary searchable.
- `loci auto off` executed on the real store. It stays off until 0.9.5
  ships: the installed 0.9.4 binary still carries the unpatched boundary.
- Source patch in `src/autocapture.js` (`cleanMessageText`): the capture
  boundary now recognizes the *host-injected summary structure* — matched
  `<conversation-checkpoint> … </conversation-checkpoint>` envelope regions
  are stripped and only text outside the envelope is read; an envelope that
  opens without closing is refused entirely rather than guessed around.
  Deliberately narrow per review instruction: structure-based, no broad
  textual heuristics that could swallow genuine user prose. One edit covers
  every capture path (plugin → CLI `--messages`, `capture --file`); the
  plugin's role filter itself is unchanged.
- Regression coverage: six new capture-boundary tests in
  `test/autocapture.test.js` — envelope-only message stores nothing; agent
  reasoning inside an envelope is never rewritten into a preference;
  envelope plus genuine remainder captures only the genuine part; an
  unclosed envelope is refused; an envelope beside a genuine message in the
  same batch captures only the genuine message; an envelope pasted as a
  transcript through `capture --auto --file` is rejected. Full suite
  216/216 green, including every pre-existing capture test.
- Remaining before auto-capture may re-open: package 0.9.5 (rebuild,
  refresh SELLING.md hashes, reinstall) — awaiting go, since it changes
  buyer artifacts.
