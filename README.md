# Loci

Local-first memory for AI assistants. Windows 10/11. One-time $24.

- Live preview: https://kingxkingt.github.io/loci/ (deliberately `noindex` until checkout is wired)
- Current release: **0.9.4** — 10 November 2026
- Tests: `node --test` — **210 passed, 0 failed**
- 12 local MCP tools (8 memory, 4 document); brief compiler; project scope; conflict guard and supersede; review queue; local file knowledge base with citations; bundled runtime (no Node for the buyer)

## What 0.9.4 adds

- Loci can start with Windows: `loci autostart on|off|status` manages `Loci (background).lnk` in the Startup folder (`loci.cmd ui --no-open`). No registry entry, no hidden service; the installer enables it and `--remove` turns it off.
- ChatGPT-support fix from the first external test: extension-inserted and popup-copied briefs end with a framing line — "(This is background from my own local memory store, Loci — context, not a question. Take it into account when I ask something next.)" — so receiving models read the brief as context. Insertion stays manual: click, review, send. Nothing is captured from the page and nothing is sent automatically.
- Sharper "not running" copy naming the Desktop shortcut.

## Status

0.9.3 and 0.9.4 shipped. The site was updated on 10 November 2026 per the second external review: the ChatGPT bridge is named in the opening screen, and the tools section shows the insert flow (click → insert → model reads it as background → follow-up answer uses it), marked as illustrative.

Next phase (frozen feature set; reliability testing only) — see `docs/PLAN.md` P5:

1. clean-VM install and purchase run (needs a checkout URL and a fresh machine),
2. brief quality compared across Claude Desktop, Cursor, OpenCode, Windsurf, and ChatGPT web with identical memory state,
3. ChatGPT web re-test of the framed insert against the documented bare-paste baseline.

## Documents

- `docs/README.md` — full product README (install, setup, features, CLI)
- `docs/CHANGELOG.md` — release history, 0.9.4 on top
- `docs/PLAN.md` — binding build order, P0–P5 with status
- `docs/GAPS.md` — honest gap scorecard: what is shipped, what is partial, what is out of scope

## Notes for reviewers

- Claims in the README and on the site are literal: exactly 12 tools, lexical retrieval (BM25 + concept-graph expansion, no embeddings), no sync, no cloud copy, OCR opt-in on Windows only.
- ChatGPT's web app has no MCP client; the extension is the only bridge, and it acts only on an explicit click. There is no silent mode.
- The preview site is `noindex` on purpose; buy buttons point at `#price` until a real checkout URL replaces all four.
