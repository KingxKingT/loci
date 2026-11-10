# Loci build plan

Priority order is binding: finish P0, then P1, then P2, then P3. Do not start a
later phase while an earlier one is unfinished unless the user says so. This plan
is also stored in Loci itself (`memory_search "roadmap"`).

Status as of 6 November 2026. P0, P1, and P2 are complete; P3 is next.

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

## P5 — cross-assistant reliability test (planned, not started)

External review (GPT, 10 Nov 2026, second pass) converged with the plan here:
stop expanding the feature list; prove the existing features deliver
reliably across real AI assistants.

| # | Item | Status |
| --- | --- | --- |
| P5.1 | Clean-VM install and purchase run: a genuinely fresh Windows machine installs from the buyer ZIP, walks the interview, and a real checkout is exercised. Required before the storefront opens. | Pending (needs the user: checkout URL + clean machine) |
| P5.2 | Brief quality across hosts: create identical memory state, then compare the initial brief and a follow-up answer in Claude Desktop, Cursor, OpenCode, Windsurf or Zed, and ChatGPT web (extension). Record where the brief reads as context and the answers actually use it. | Planned |
| P5.3 | Re-test with 0.9.4 in ChatGPT web specifically: the framed insert, an ack, a follow-up question, and an answer that uses the brief — the flow the site demo describes. | Planned |

## Rules that apply to every phase

- Local-only core: no cloud sync, no hosted knowledge base, no telemetry.
- Windows 10/11, bundled runtime, no Node.js install for the buyer.
- Twelve MCP tools: eight memory, four document.
- Claims stay literal. No "semantic search". No "captures every chat" or "syncs
  across devices". ChatGPT web works only through the extension, and only by
  manual click-to-insert; state that scope wherever ChatGPT is mentioned.
- A rebuild of `src/` or `README.md` changes the ZIP hash: refresh `SELLING.md`.
