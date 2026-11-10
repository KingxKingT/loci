# Loci gap audit

Status as of 10 November 2026 · version 0.9.4 built, tested, and packaged

## Open finding: auto-capture ingests host-injected text (10 Nov 2026)

The OpenCode auto-capture plugin trusts `role === "user"`. When the host
delivers a conversation-summary block in user role — which OpenCode does after
a compaction — the room's agent reasoning arrives as "user" text, and
`capture --auto` stores reasoning fragments as active memories. Observed live:
~45 junk records in a real store, including test-question quotes. The plugin
must dismiss host-injected summary blocks by shape, not only by role. Defect
fix candidate; tracked as a P5 finding in PLAN.md. Unchecked risk for every
OpenCode user until patched.

## What 0.9.4 changes

- **Loci starts with Windows by itself.** `loci autostart on` creates
  `Loci (background).lnk` in the user's Startup folder — no registry Run key,
  no hidden service; the user sees the file and can delete it any time. The
  shortcut runs `loci.cmd ui --no-open`, so the server comes up minimized with
  no browser window, and the extension stops being dead until Loci is launched
  by hand. `loci autostart off` removes exactly Loci's own shortcut, and
  `loci autostart status` prints the state and path. Tests use temp-dir
  overrides (`LOCI_STARTUP_DIR`, `LOCI_STARTUP_TARGET`) so the real Startup
  folder is never touched by the suite.
- **Inserted and copied briefs are framed as context, not as a question.** The
  extension appends one line — `(This is background from my own local memory
  store, Loci — context, not a question. Take it into account when I ask
  something next.)` — so the model does not answer "there's no question here".
  The popup copies the framed text and shows the bare block for reading.
  The contract is unchanged: the user clicks, the user sends.
- **The "not running" copy names the fix.** The extension flash and popup say
  to start Loci from the Desktop shortcut instead of a vague error.
- **Verification:** `node --test` reports 210 tests, 210 pass, 0 fail
  (6 new autostart tests, 1 new framing source-contract test). Artifacts and
  SHA-256 hashes are recorded in `SELLING.md`.

## What 0.9.3 changes

- **ChatGPT's web app is covered — by click, never by itself.** `integrations/browser`
  is a manifest V3 extension for `chatgpt.com`: a small **Loci ▸ insert brief** button
  puts the memory brief into the chat input when the user clicks it, and the user sends
  the message themselves. There is no page capture, no auto-send, and no silent
  injection; the popup also shows and copies the brief for any other site.
- **The server speaks to it through a deliberately narrow door.** `src/ui.js` gained two
  token-free, GET-only endpoints (`/api/extension/health` and `/api/extension/context`)
  that sit behind the existing loopback Host check, so DNS rebinding stays blocked; no
  CORS headers are written, so a hostile web page cannot read responses even if it
  could send the request. A local process calling 127.0.0.1 gains nothing it could not
  already read from `~/.loci` on disk. Every existing `/api/**` endpoint keeps its
  token and same-origin gates unchanged.
- **The launch ports are known, so the extension can find the server.** `loci ui` now
  tries ports 7629–7635 first and falls back to a random one only if all are busy; an
  explicit `--port` still wins and a running instance is still detected and reused.
- **The manifest is loopback-only.** Its sole host permission is
  `http://127.0.0.1/*`; the extension fetches, it never uploads, and there is no
  endpoint that writes memories from a browser on purpose.
- **Doctor prints the extension folder** so "Load unpacked" has a concrete path.
- **Verification:** `node --test` reports 203 tests, 203 pass, 0 fail, including 7
  new extension-surface tests (tokenless read, block parity, GET-only refusal,
  gating preserved, rebinding refusal, no-CORS, and manifest/package agreement).

## Historical notes — release by release

The section below is kept for history. Its version references describe the
state **when 0.9.2 shipped**, not today: 0.9.3 and 0.9.4 followed it, so that
"installed copy is still 0.9.0" note is at least two releases out of date. The
current installer is `install.bat` from `dist\loci-memory-0.9.4`; it upgrades
program files and never touches `~/.loci`. The scorecard paragraph below is
still current — **Shipped** means the 0.9.4 build in `dist/` does it today.

This is the honest scorecard for the gap list. **Shipped** means the 0.9.4
build in `dist/` does it today. **Partial** means a usable piece exists and the
missing part is named. **Not built** means Loci does not claim it.

**What 0.9.2 added (historical).** Automatic capture for OpenCode 2,
so a user no longer has to say "remember this" in that one host: an opt-in
plugin reads only the user's messages, writes only durable high-confidence
statements, and sends everything else to review. It still exposes **12 tools —
8 memory and 4 document**, and no memory files were touched by that upgrade.

| Gap | Status | What is actually there | What is not |
| --- | --- | --- | --- |
| Automatic / semi-automatic memory extraction | Partial | **OpenCode 2 auto-capture** (opt-in plugin): reads only the messages the user types, auto-saves durable statements at 0.8 confidence or above in the auto-safe kinds (person, preference, decision, other), marks each save `inferred` with the exact source sentence as origin, and keeps a per-session cursor so history is not replayed and turns are not duplicated. `loci auto off` is the single switch. Findings scoring 0.6–0.8, facts, and project notes go to the review queue; weaker or non-durable lines are dropped. **From a chat** paste, `loci capture`, and the `memory_capture` tool work as before. | No capture in ChatGPT web, Cursor, Claude Desktop, Windsurf, or Zed; no reading of assistant output; no LLM-quality extraction. Fact and project auto-writes are deliberately withheld. Rule-based extraction misses unusual phrasing and can occasionally over-save a durable-sounding sentence; both are visible and reversible. |
| Stronger semantic retrieval | Partial | Lexical BM25 with a two-level curated concept graph (around fifty equivalence groups, plus directly related groups at a lower weight), importance, recency, usage, pinned state, and project boost. When BM25 returns nothing, a trigram-coverage fallback (at least 60% of the query's three-letter fragments) catches typos and missing words; it is labelled "fuzzy lexical", not semantic. | No embeddings, no vector index, no paraphrase-only recall. Expansion is one hop and cannot follow a second edge. A rewrite that shares no tokens or trigrams with the question can still miss it. |
| Dynamic, context-aware briefs | Shipped (deterministic) | **What the AI sees** accepts a question and rebuilds the injected block against it, within the character budget. Documents enter only when they match. | The brief is template-compiled, not LLM-written. The no-question block is the stable handover baseline. |
| Deeper knowledge base | Partial | Files tab, CLI `loci docs`, and `doc_add` for assistant-side import; text/Markdown/CSV/JSON/HTML/source files, best-effort `.docx`, RTF, and text-based PDF. A scanned PDF or an image can be read with opt-in Windows OCR (20 MB, 25 pages) when normal extraction finds no text. Re-import matches by absolute path (or file name in the browser), hashes the body, updates in place, and keeps a hand-edited title and tags. Markdown headings are section boundaries; code and data files split at lines, not at `#` comments. Cited excerpts, no overlap, and a heading becomes the citation anchor. Imported files record size and modification time; listings and search mark `changedOnDisk` or `missing`, and the Files view re-checks that metadata every ten seconds while it is open. | No source-path watcher, no live re-index, and no OCR outside Windows. The timer refreshes badges only; it never rewrites a document on a disk change. Other views still check stale state when read. Local files only. |
| Memory hygiene and decay | Partial | Supersede (writes replacement first, then retires the old record; accepts one id or an array), retire, delete, active/superseded status, append-only history, explicit origin text. A second decision that overlaps an active one is **refused** unless `supersedes` or `force` is given, and near-identical paraphrases merge first. The same guard covers corrected facts and preferences: matching attribute before *is/are/was/were*, matching head noun after a preference verb, or a matching leading predicate with a different ending. A **Review queue** combines flagged memories with decisions and preferences unconfirmed for 180 days, shows why each row is there, and offers Confirm, edit, retire, or **Snooze 30 days** (per row or whole queue). | No automatic decay, no scheduled job, no automatic retirement. The aging marker is computed on read and changes nothing the AI sees. Conflict detection is lexical: a paraphrase with no shared subject can slip past, and a shared subject can still ask even when the two claims agree. |
| Project / context switching UX | Shipped | A header switch lists every project in `projects.json`, opens any folder through a native Windows picker or a pasted path, and returns to global memory. Opening a subfolder promotes it to the nearest project root. Global/project/both scope filters update with the switch. The last folder opened is stored in config and restored on the next launch when it still exists; an explicit `--root` always wins. | No per-project window, no recent-project ordering, and no automatic switch when a browser tab changes project. |
| Usefulness evaluation | Partial | **Was this right?** records `helpful` / `needs review` on each memory, keeps local counts, and the Review queue combines flags with review-due items. A row can be **Confirmed** once the user approves it; that records the basis as stated and removes the `(inferred)` label from the brief. Snoozing hides a row for 30 days without changing its text, basis, or stored flag. | No outcome metrics, no A/B evaluation of brief quality, no automatic ranking change from feedback. |
| Multi-device / private sync | Out of scope | The store is plain files under `~/.loci`; a user can copy it manually, and opt-in AES-256-GCM encryption protects a copied folder in passphrase mode. | No sync protocol and no conflict resolution across machines. Key-file encryption does not protect a copied folder because the key sits beside the store. |
| Notion / Drive / Slack connectors | Out of scope | Local file import covers the common "bring my notes" case. | Connectors require network access and a hosted auth surface, which conflicts with the local-only core. |
| Browser extension | Shipped | `integrations\browser` (shipped 0.9.3): a manual-insert-only content script on `chatgpt.com` puts the brief into the chat input when the user clicks; since 0.9.4 the inserted text carries a context-framing line. Loci can be started by the Desktop shortcut or autostart with Windows (0.9.4). | Insertion stays click-only by contract: no auto-send, no page capture, no silent injection. Only `chatgpt.com` is scripted; other sites use the popup's copy. Writes from a browser are not possible on purpose. |
| Storefront | Blocked | Sales page, zip, installer, and purchase-test script are ready. The page is deployed at `https://kingxkingt.github.io/loci/` as a `noindex` preview with absolute canonical and Open Graph URLs. | No Gumroad checkout URL yet, so the four buy links still point at `#price`; the preview is intentionally `noindex` and must not be shared until checkout is wired. |

## What 0.9.2 changes

- **Capture no longer depends on remembering to ask.** The OpenCode 2 plugin
  registers the host's `context` hook and, on each model request, sends only the
  unseen user messages to `loci capture --auto`. The user types normally; the
  memory appears.
- **The gate is confidence, not enthusiasm.** Writes require 0.8 or above and
  one of the auto-safe kinds. Facts, project notes, and anything less certain
  stay in review. Every automatic save is `inferred` with the source sentence
  preserved.
- **One switch, checked every turn.** The plugin asks the CLI whether capture
  is enabled; `loci auto off` stops it without a restart, and `loci auto on`
  resumes it. The cursor is not replayed.
- **The plugin is inspectable and removable.** One dependency-free `index.js`,
  installed by `loci plugin opencode on` into the OpenCode config folder. `off`
  removes only files Loci wrote; `status` prints the path and the last capture
  line. The installer offers the same choice once, with a yes/no prompt.
- **The build cannot ship without it.** `integrations/` is staged and the build
  fails if the plugin file is missing.
- **Verification:** `node --test` reports 184 tests, 184 pass, 0 fail,
  including 22 capture tests and 10 plugin tests. A real `opencode run` on
  2.0.31 saved a preference, did not duplicate it, honoured `auto off`, and
  captured only the new statement after it was turned back on.

## What 0.9.1 changes

- **A killed server leaves a record.** Every MCP stdio server writes a run
  record with a five-second heartbeat. A clean close is stored as `clean`; a
  crash is stored as `crashed` with its reason. A process terminated from
  outside cannot write anything, so the next startup or `loci doctor` run finds
  the last heartbeat, marks the run `unclean`, and estimates the end time from
  it. This is the case the old crash log could not see at all.
- **The parent pid separates the two failure shapes.** If the parent is also
  dead, the whole tree went down together (the AI app quit or was killed). If
  the parent is still alive, only the MCP process was killed. Windows reuses
  pids, so this is a hint, not proof.
- **`doctor` stays quiet when the latest run is fine.** It prints the run line,
  reason, and log path only when the most recent run is `unclean` or `crashed`;
  otherwise it prints one short `ok` line. An old kill followed by clean starts
  is history, not a live alarm.
- **A crash now puts its stack on stderr too.** The full trace is written with
  a `[loci] fatal:` prefix before the process exits, so a host that only shows
  the last stderr lines still has something to report. The stack also remains
  in `~/.loci/server.log`.
- **`resources/templates/list` answers with an empty list.** Loci has one fixed
  `loci://context` resource, not parameterised templates. The old
  method-not-found reply made clients log a warning on every connect.
- **The hooks are shared, not duplicated.** Crash handling stays with
  `crashlog.js`, stdin stays with readline, and lifecycle bookkeeping is told
  to leave both alone. An unhandled rejection is still logged without taking
  memory offline, and a clean stdin close records the run without forcing
  `process.exit`, so buffered stdout is not truncated on Windows.

## What 0.9.0 changes

- **A changed file shows up without pressing Refresh.** While the Files view is
  open and the page is visible, Loci asks `GET /api/documents/status` every ten
  seconds. The answer is metadata only — id, update time, `changedOnDisk`, and
  `missing` — so a large library is not re-chunked on a timer. Only the badge
  text changes; document contents are never rewritten.
- **OCR is a fallback, and only when asked.** `loci docs add --ocr`, `doc_add`
  with `ocr: true`, and the dialog checkbox all go through the same reader.
  Normal extraction runs first and wins, so a PDF with a real text layer or a
  `.docx` is imported exactly as before. OCR runs only for a scanned PDF or an
  image whose extraction produced no text.
- **Every limit is explicit.** 20 MB input, 25 PDF pages, 3 minutes, 8 MB of
  captured output, and 2 million characters. A file over a cap gets a "split the
  file" error, not truncated text.
- **No surface promises OCR it cannot do.** `GET /api/capabilities` probes
  `Windows.Media.Ocr` once per process and reports its languages; the Add-file
  dialog shows the checkbox only when the probe succeeds.
- 141/141 tests pass (109 core, 32 UI). New tests cover metadata-only status,
  both stale flags, scope on the composite list, extraction winning over OCR,
  OCR's file-type gate, the 20 MB input cap, and the capability shape.

## What 0.8.0 changes

- **Recall reaches one step further, without leaving the machine.** The concept
  graph grew to around fifty equivalence groups covering software work and
  everyday domains. Same-group terms weigh `0.4`; groups that are directly
  related weigh `0.18`, so *backend* can reach a stored *Postgres* note.
- **The graph cannot run away.** Expansion happens only on the query side,
  generated terms are never re-expanded, and the walk stops after one hop. A
  literal match still outranks an expanded one.
- **The assistant can propose memories without saving them.** `memory_capture`
  scans pasted text or a transcript, returns candidates with confidence, marks
  near-duplicates against active memories, and writes nothing. Only
  `memory_add` writes, and only after the user approves a line.
- **The assistant can import a document.** `doc_add` accepts a named local file
  or pasted text with an optional title and tags, uses the same reader as the
  window and CLI, and updates a re-imported path in place.
- The shipped surface is **12 tools — 8 memory and 4 document**.
- 135/135 tests pass. New tests cover the 12-tool handshake, candidate-only
  capture, an empty capture, relative-path import, pasted-text import, search of
  the imported document, and rejection of ambiguous `doc_add` input.
- The buyer ZIP was extracted to a fresh folder and run against an isolated
  `LOCI_HOME`. The handshake reported version 0.8.0 and **12 tools — 8 memory
  and 4 document**. `memory_capture` returned a preference candidate and
  `Nothing was saved`. `doc_add` imported a relative-path Markdown file,
  `doc_search` found its text, and a 400-character `memory_context` returned
  content after one memory was explicitly added. `doctor` exited 0 with the
  packaged runtime.

## What 0.7.0 changes

- **Encryption is opt-in and never the default.** `loci encrypt` writes a random
  256-bit key to `~/.loci/encryption.key` and rewrites every memory and document
  line as an AES-256-GCM envelope. `loci encrypt --passphrase` derives the key
  with scrypt from `LOCI_PASSPHRASE` (or `LOCI_PASSPHRASE_FILE`) and stores only
  a salt. `loci decrypt` rewrites the stores as plain text and removes the key
  file. A store can hold plain and encrypted lines at once, so an interrupted
  migration is still readable.
- **The failure is explicit, not silent.** A missing key or passphrase produces a
  named error instead of an empty store, and `loci doctor` reports encryption
  state, key availability, and the encrypted/plain line counts. If the key is
  unavailable, doctor stops with the reason rather than crashing on the first
  store read.
- **Redaction is separate and also opt-in.** `loci redact --enable` replaces
  recognised credentials with `[redacted:<rule>]` before a memory or document is
  written. `loci redact` lists existing matches without printing the secret, and
  `loci redact --apply` rewrites those records. Ten rules cover private keys,
  OpenAI, GitHub, AWS, Google, Slack, Stripe, JWTs, bearer tokens, and
  password/secret assignments.
- **Documents get the same treatment.** Title, text, and tags are redacted on add
  and update; a redacted document gets a fresh hash and character count.
- 127/127 tests pass (97 core, 30 UI). The suite covers envelope round-trips,
  refusing to open without the key, mixed plain/encrypted stores, migration in
  both directions, redaction on write, the redaction sweep, the CLI toggle, and
  passphrase gating.
- The packaged ZIP was verified from a fresh extraction and isolated
  `LOCI_HOME`: the MCP handshake lists 10 tools, a stored fact survives an
  encrypt/decrypt round-trip, plaintext does not appear on disk while encrypted,
  and an existing secret is swept by `loci redact --apply`.

## What 0.6.0 changes

- **Facts and preferences are corrected, not stacked.** `memory_add` now
  compares a new fact with active facts, and a new preference with active
  preferences. A fact is blocked when the attribute before *is/are/was/were*
  matches at a token Jaccard of 0.50 or more. A preference is blocked when both
  texts use a preference verb (*prefer, like, want, need, love, hate, avoid*),
  end in the same head noun, and differ. A leading-predicate rule catches the
  remaining cases with at least one shared non-filler token and Jaccard 0.30.
  Different attributes and different predicates are allowed, and claims of
  different kinds never block each other.
- **The blocked write explains the kind.** The result says "this fact", "this
  preference", or "this decision", lists the conflicting ids and texts, and
  shows both retry forms. The CLI exits 2 for all three; the window returns 409.
  `supersedes` retires the old claim, `force` keeps both on purpose.
- **The review queue has two sources and a way out.** It includes flagged
  memories plus decisions and preferences not confirmed for 180 days, and each
  row carries `reviewReason` (`flagged` or `stale`). Rows offer Confirm, edit,
  retire, and **Snooze 30 days**; the toolbar offers **Snooze all 30 days**.
- **Snoozing preserves the record.** It sets `snoozedUntil`, leaves the text,
  basis, and stored `needsReview` flag untouched, and only changes the decorated
  view. Confirming, flagging, or editing wakes the row. `days: 0` wakes it
  explicitly, and the API accepts 0 to 365 days.
- **The agent sees the same queue.** `memory_list({needsReview: true})` includes
  stale decisions and preferences and omits snoozed memories, so an assistant
  cannot ask about a row the user just put away.
- **Counts are separate.** `stats()` reports `needsReview`, `reviewDue`, and
  `snoozed`; the window's queue badge is flagged plus review-due, excluding
  snoozed.
- 118/118 tests pass (88 core, 30 UI). The 0.6.0 zip was built, installed from
  outside the install folder, and checked: installed version 0.6.0, `doctor`
  exit 0, and the MCP handshake reports 10 tools.

## What 0.5.1 changes

- **A small budget no longer claims the store is empty.** When `budgetChars`
  could not fit one section heading plus a line, `memory_context` returned
  nothing and the server answered "No memories stored yet." even though records
  existed. It now falls back to a compact handover with bare lines, and if not
  even one line fits it says how many memories are stored and what budget would
  include one. `budgetChars: 400` against a populated store returns the
  memories.
- **The README matches setup.** Setup wires Claude Desktop, Cursor, Windsurf,
  OpenCode, and Zed. The README no longer implies VS Code is automatic; it
  documents the `mcp.json` block for anyone who wants VS Code by hand.
- 105/105 tests pass (78 core, 27 UI). The zip was built, installed from
  outside the install folder, and checked: ten tools, `doctor` exit 0.

## What 0.5.0 changes

- **An overlapping decision is refused, not stored.** `memory_add` on a
  `decision` compares it with active decisions in the same scope. With at least
  one shared subject token and a Jaccard score of at least 0.30, nothing is written: the
  result lists up to three conflicting ids and both ways forward. Near-identical
  paraphrases (0.90) merge before the check, so rephrasing is not a conflict.
  Filler verbs such as "use", "prefer", and "choose" cannot be the only shared
  evidence.
- **`supersedes` takes one id or an array.** Every named claim is validated and
  retired; the replacement is appended first, so an interrupted write cannot
  lose both versions.
- **Bulk paths stay bulk.** Capture skips a blocked candidate and reports the
  count; the setup interview and JSON import pass `force: true` on purpose.
- **A fuzzy lexical fallback.** When BM25 finds nothing, trigram coverage at 60%
  catches typos and missing words. It is labelled "fuzzy lexical" and a normal
  BM25 hit always wins. There are still no embeddings.
- **Stale files are visible.** Imports record `sourceMtimeMs` and `sourceSize`;
  list and search results add `changedOnDisk` or `missing` when the source no
  longer matches. Nothing is watched or re-indexed in the background.
- **Citations name the heading.** A chunk under a Markdown heading renders as
  `“Title” — § Heading`; plain text keeps `part n of m`.
- **The last project reopens.** Opening or switching a folder writes
  `ui.lastProject`; a launch with no explicit root restores it if it still
  exists.
- **Aging is a prompt, not decay.** Decisions and preferences unconfirmed for
  180 days are marked "may be out of date" in the window and counted separately
  from manual review flags. `review.afterDays: 0` disables it. The stored record
  is not rewritten and the brief does not change.
- **Back up downloads everything.** The header button fetches `/api/export` with
  the per-run token (header-only), then saves one JSON file containing memories
  and documents.
- **The CLI explains the stop.** `loci add` on a blocked decision prints the ids
  and both retry forms, supports `--force`, and exits 2.
- **The window explains the stop.** The add form shows the conflicting claims,
  preselects them as the replacement, and changes Save to **Replace and save**;
  the second save retires them. The API returns 409 for this case.
- 103/103 tests passed when 0.5.0 was built (76 core, 27 UI).

## What 0.4.5 changes

- **A crash log instead of silence.** Uncaught exceptions and unhandled
  rejections append a timestamp, message, and stack to `~/.loci/server.log`,
  rotated at 256 KB. `loci doctor` prints the last error and the log path.
- **The window can confirm.** Review-queue rows offer **Confirm**, which calls
  `memory_confirm`: the basis becomes stated, `lastConfirmedAt` is recorded, and
  the flag clears. **Confirm all pending** does the same for the queue only.
- **`memory_confirm` says when to call it.** The description begins with the
  rule that the user must explicitly approve the memory in this conversation,
  so an assistant cannot self-confirm an inference.
- **`memory_list({needsReview: true})`** returns flagged memories and memories
  that were never confirmed, so an assistant can ask about them directly.
- **Pinned memories stop hijacking search.** They no longer enter every query's
  pool at zero relevance; a search keeps only hits above the configured floor.
- **The brief uses its budget.** It compiles from up to 32 memories and lets the
  2,000-character budget decide what fits, instead of truncating at 12 records.
  Inferred memories are labelled `(inferred)`; `source: document` records are
  not.
- **Conflict warnings ignore the "Decision:" prefix** before comparing texts.
- **Security:** the window rejects non-loopback `Host` headers (DNS rebinding
  could otherwise steal the token), refuses framing, and no longer accepts a
  `?token=` query parameter.
- **Accessibility:** the lists are no longer live regions (the count speaks
  instead), icon actions name the memory they act on, and the window honours
  `prefers-reduced-motion`.
- 89/89 tests pass. The zip is built and its source matches the repository; the
  installed copy is still 0.4.4.

## What 0.4.4 fixed

- **Upgrading no longer disconnects memory.** The old installer stopped every
  Node process running from `%LOCALAPPDATA%\Loci`, which included the AI app's
  live MCP server. Apps that do not retry a failed local server, such as
  OpenCode, stayed disconnected for the rest of the session.
- **Upgrades are staged, then swapped.** The download is copied to
  `%LOCALAPPDATA%\Loci.staging` first, so a failed copy cannot leave a
  half-old, half-new install. The old tree is replaced by rename, with rollback
  if the second rename fails.
- **Live connections are refused, not killed.** If a memory server is still
  running from the install folder, the installer names the holding app and
  stops instead of terminating it.
- **The restart reminder is unconditional.** It no longer appears only on the
  failure path.
- 72/72 tests pass. The upgrade path was tested both ways: refused with a live
  connection (naming `opencode`), and completed with the connection closed.

## What 0.4.3 added

- **Provenance you can see.** `memory_confirm` records `lastConfirmedAt` and
  marks the basis as stated. Search and list results show `created:`,
  `confirmed:`, and `source:` for every memory. Older records without those
  fields still load.
- **Search is read-only by default.** `memory_search` updates usage only when
  `trackUsage` is true.
- **Conflict warnings, not automatic retirement.** Adding a decision lists up
  to three similar active decisions in the same scope and asks you to supersede
  one explicitly. Nothing is retired automatically.
- **The full brief from both paths.** `memory_context` and the
  `loci://context` resource return the same compiled handover instead of a
  pinned-only subset.
- **Explicit budget failure.** A `budgetChars` value too small for the header
  and footer returns an error instead of claiming no memories are stored.
- **Delimited document excerpts.** `doc_search` fences each excerpt and escapes
  any closing marker in the source text.
- **Configured limits reach both entry points.** MCP and the CLI both pass
  `relevanceFloor`, `baselineLimit`, `documentLimit`, and
  `documentRelevanceFloor` through from config.
- 72/72 tests pass. The 0.4.3 zip is built, installed into
  `%LOCALAPPDATA%\Loci`, and checked: source, staged, and installed files match,
  `doctor` is healthy, and setup plus the MCP handshake report 10 tools.

## What 0.4.2 added

- **Re-import instead of duplicate.** `loci docs add` (and the Files tab) matches
  an existing document by absolute path, or by file name when the browser hides
  the path, hashes the new body with SHA-256, and updates the same document.
  Hand-edited titles and tags survive. The CLI reports added / updated /
  unchanged / skipped.
- **Review queue.** Each memory has **Was this right?** with **Looks right** and
  **Needs review**. The header toggle shows a count, flags the row, and filters
  the list to flagged memories. Feedback is local and never changes what the AI
  sees.
- **Section-aware chunking.** Markdown headings start new chunks, so a citation
  for "Refunds" does not trail into "Support". Fenced code blocks and tables
  stay atomic when they fit.
- **Source-aware `#` handling.** A `#` line is only a heading in a Markdown file.
  Python, shell, SQL, and plain-text comments chunk as lines, so they never
  become citation boundaries. Markdown imports keep their heading markers;
  other formatting is still flattened for search.
- 65/65 tests pass. The 0.4.2 zip is built, installed into `%LOCALAPPDATA%\Loci`,
  and checked: source, zip, and installed hashes match, `doctor` is healthy, and
  setup reports the MCP server with 9 tools.

## What 0.4.1 added

- **Project switch in the header.** It lists known projects, shows which folder
  is active, and offers **Browse…** plus a paste-a-path fallback.
- Any existing folder can become a project, not only a folder with
  `package.json`. A nested folder is promoted to the nearest project marker.
- Global and project scope selects, the add/edit scope chooser, the capture
  scope chooser, and the Files tab all follow the active project.
- The native folder picker is a short PowerShell WinForms dialog, so there is
  still no native module and nothing to install.
- The switch sits under the Loci name, so the header stays on one row at
  1200 px. Verified with a local browser at 1200×900.
- 57/57 tests pass. The 0.4.1 zip was built, installed into
  `%LOCALAPPDATA%\Loci`, and checked: source, zip, and installed hashes match,
  `doctor` is healthy, and setup reports the MCP server with 9 tools.

## What 0.4.0 added

- **From a chat** in the desktop window, plus `loci capture` in the CLI.
  Candidates are suggestions; the save button counts only checked rows.
- A question field in **What the AI sees**, so the dynamic brief is visible and
  testable without opening an AI client.
- The primary **Add memory** action moved to the header; the toolbar now fits
  on one row at 1200 px.
- 55/55 tests pass. The 0.4.0 package was rebuilt, installed to
  `%LOCALAPPDATA%\Loci`, and the installed MCP server reports 9 tools.

## Next, in order

The binding build order is in `PLAN.md`. P0 shipped in 0.5.1; P1.1 and P1.2
shipped in 0.6.0; P1.3 shipped in 0.7.0; P2.1 and P2.2 shipped in 0.8.0; P2.3
shipped in 0.9.0. 0.9.1 is a reliability patch on top of P2.3 and adds no new
capability. P3.1 shipped in 0.9.3; P4 (onboarding friction from the user's own
beta test) shipped in 0.9.4.

1. **A clean-VM purchase run.** The packaged-ZIP test still passed on a
   development machine; a genuinely fresh Windows VM has not run the installer
   yet. Do it before the storefront opens.

Checkout is still the user's call: the four `data-buy` links point at `#price`,
the preview stays `noindex`, and the storefront must not be shared until a real
checkout URL replaces them. Each rebuild changes the ZIP hash: `SELLING.md`'s
artifact sizes and hashes must be refreshed with it (done for 0.9.4).

## Claims to keep out of the storefront

- "Semantic search" or "understands meaning". Retrieval is lexical.
- "Captures your chats automatically". In OpenCode it reads what you type, gated
  at high confidence and subject to review; the browser extension captures nothing.
- "Syncs across devices". It does not.
- "Works with ChatGPT" without the scope. ChatGPT web works only through the
  manually loaded extension and only by click-to-insert; say both.
- "Connects to Notion, Drive, or Slack". It does not.
