# Changelog

All notable changes to Loci are recorded here.

## [0.9.5] - 10 November 2026

### Added

- `loci wake` — one command that does the Desktop shortcut's job again: if
  the server is already alive it says so and exits quietly; if it is dead it
  starts it with the same visible window. This is the OS entry point for the
  `loci://` URL scheme, which lets the browser extension start Loci (a web
  page cannot launch programs; a custom scheme registered in HKCU is the one
  browser-blessed path, and Chrome shows its own "Open Loci?" confirmation
  the first time).
- `loci wake register` / `loci wake unregister` write or remove the
  `loci://` handler in `HKCU\Software\Classes\loci` only — no administrator
  rights, nothing else in the registry is touched. `reg.exe` is driven
  through argument lists, never a quoted command string.
- The popup shows a **Start Loci** button while the local server is down;
  clicking it fires `loci://wake`, then waits on the same endpoint the
  extension already uses and refreshes the brief when Loci is up.
- The in-page button on chatgpt.com, clicked while Loci is down, starts
  Loci the same way and asks for one more click to insert the brief.
- The installer runs `loci wake register` during installation and says so;
  `install.bat --remove` unregisters the handler.

### Changed

- Package and extension version bumped to 0.9.5. Verification: `node --test`
  reports 225 tests, 225 pass, 0 fail (9 new wake/protocol tests exercise a
  temporary store and a fake registry executor, so the real registry is
  never touched by the suite).

## [0.9.4] - 10 November 2026

### Added

- Autostart on Windows, without the registry: `loci autostart on` creates
  `Loci (background).lnk` in the user's Startup folder. The shortcut runs
  `loci.cmd ui --no-open`, so the server starts minimized with no browser
  window, and the extension finds it immediately after boot instead of staying
  dead until Loci is launched by hand. `loci autostart off` removes it again
  (only Loci's own shortcut file), and `loci autostart status` prints the
  current state and path. `loci doctor` reports the autostart state. Tests
  exercise a temporary Startup folder through the `LOCI_STARTUP_DIR` and
  `LOCI_STARTUP_TARGET` overrides, so the real Startup folder is never touched
  by the suite.
- The installer runs `loci autostart on` during installation and echoes what
  it created; `install.bat --remove` turns autostart off before disconnecting.
- A chat-framing line protects inserted and copied briefs from the
  "there's no question here" reply: background.js appends
  `(This is background from my own local memory store, Loci — context, not a
  question. Take it into account when I ask something next.)` as
  `blockForChat`. The content script inserts the framed text; the popup shows
  the bare block for reading and copies the framed text. The user still clicks
  and sends — insertion stays manual, and no mode runs without a click.

### Changed

- The extension's "Loci isn't running" flash and the popup's not-running
  message now say exactly where to start Loci: the Desktop shortcut. The popup
  falls back to a non-clipboard copy when the clipboard API is unavailable.
- Package and extension version bumped to 0.9.4. Verification: `node --test`
  reports 210 tests, 210 pass, 0 fail (6 new autostart tests, 1 new framing
  source-contract test).

## [0.9.3] - 9 November 2026

### Added

- The browser extension (`integrations/browser`), covering ChatGPT's web app —
  the one client that cannot spawn a local MCP server. A **Loci ▸ insert brief**
  button on `chatgpt.com` puts the memory brief into the chat input when the
  user clicks it; the user sends the message themselves. The manual-insert-only
  contract is the feature: no page capture, no auto-send, no silent injection,
  and no mode that runs without a click. A popup shows and copies the brief.
- `GET /api/extension/health` and `GET /api/extension/context` in `src/ui.js`,
  the token-free read surface the extension fetches. Both are GET-only, sit
  behind the existing loopback Host check (DNS rebinding stays blocked), and
  write no CORS headers, so a hostile web page cannot read responses. Every
  pre-existing `/api/**` endpoint keeps its token and same-origin gates.
- Known launch ports: `loci ui` tries 7629–7635 first (falling back to a random
  port only if all are busy), so the extension can find the server without any
  configuration. An explicit `--port` still wins; a running instance is still
  detected and reused, never disturbed.
- `loci doctor` prints the extension folder, so "Load unpacked" has the path.

### Security

- The extension manifest forbids everything but the loopback: its only host
  permission is `http://127.0.0.1/*`, and there is intentionally no endpoint
  that writes memories from a browser.

### Changed

- ChatGPT-support wording across the README and storefront claims now names the
  mechanism and the scope: works through the manually installed extension, by
  click-to-insert only.

## [0.9.2] - 7 November 2026

### Added

- Automatic capture. Loci can now read the messages you type in OpenCode and
  keep the durable parts — preferences, decisions, identity details — without
  you saying "remember this" first. The extractor is the same offline,
  rule-based one `loci capture` already used; there is no model call, no API
  key, and no network request. Assistant replies and tool output are never
  read; fenced code, logs, quotes, and slash commands are filtered out before
  extraction.
- An OpenCode 2 plugin that hooks each model request, sends only the unseen
  user messages to `loci capture --auto`, and never blocks the request. It keeps
  a per-session cursor so a restart does not replay history and a repeated turn
  does not duplicate. The first time it sees an existing session it captures
  only the newest message.
- A confidence gate for automatic writes. Only durable, high-confidence
  statements (0.8 or above) in the auto-safe kinds — person, preference,
  decision, other — are written. Findings scoring 0.6–0.8, facts, and project
  notes appear in the review queue; weaker or non-durable lines are dropped.
  Every automatic save is marked `inferred` and keeps the exact source sentence
  as its `origin`.
- `loci capture --auto`, with `--messages` for a JSON message list, `--file`,
  `--scope`, `--auto-confidence`, `--dry-run`, and `--json`. This is the same
  path the plugin uses, so a script can capture without the desktop app.
- `loci auto [on|off]` is the single switch for automatic capture. The plugin
  asks the CLI on every turn, so turning it off also stops the plugin without a
  restart. `loci auto` prints the current state.
- `loci plugin opencode [status|on|off]`. `on` installs the single-file plugin
  into the OpenCode config folder, `off` removes only files Loci wrote, and
  `status` prints the path, whether capture is on, and the last capture line.
  The installer offers the same choice once, with a yes/no prompt, but only
  when an OpenCode config folder exists.

### Changed

- Preference extraction accepts an adverb before the verb ("I also prefer tea"
  and "I really love dark mode"), and negation normalisation still produces
  "User does not like tabs" rather than a literal double negative.
- `loci capture` without `--auto` now shows and saves every extracted
  candidate, deduplicated, instead of only the high-confidence subset.
- Input with a UTF-8 BOM is stripped before parsing, so a file written by
  Windows PowerShell does not lose its first statement.
- The build stages `integrations/` and fails if the OpenCode plugin is missing,
  so `loci plugin opencode on` always has a source to install.
- `install.bat --remove` now removes the auto-capture plugin it installed
  before disconnecting, and only when Loci's marker is present. Removing the
  connection stops automatic capture instead of leaving the plugin behind.

### Verification

- `node --test`: 184 tests, 184 pass, 0 fail. New coverage includes 22
  automatic-capture tests (extraction, safety gate, CLI subprocess, dry-run,
  `auto off`) and 10 OpenCode-plugin tests (context hook, real message shape,
  quote stripping, cursor, no replay, idle fallback, missing CLI, auto-off).
- End-to-end on OpenCode 2.0.31: a real `opencode run` loaded the plugin,
  saved one preference, wrote `saved=1` to `~/.loci/auto-capture.log`, did not
  duplicate it on a later turn, honoured `loci auto off`, and captured only
  the new statement after it was turned back on.

### Limits

- The plugin is OpenCode-only. ChatGPT web remains unsupported until the P3.1
  browser extension exists.
- Extraction is rule-based, not a model. It will miss unusual phrasing and can
  occasionally save a durable-sounding sentence you did not mean to store;
  both are visible and reversible in the desktop app. `loci auto off` stops it.
- The plugin waits at most five seconds per turn and skips a capture that
  exceeds it.

## [0.9.1] - 6 November 2026

### Added

- MCP lifecycle tracking. Every stdio server now writes a small run record with
  a five-second heartbeat and answers `loci doctor` honestly after the fact.
  A clean close (stdin ended, Ctrl+C) is recorded as `clean`. A crash is
  recorded as `crashed` with its reason. A process terminated from outside
  never gets to write anything, so the next startup or `doctor` run finds its
  last heartbeat, marks the run `unclean`, and estimates the end time from it.
- The run record includes the parent pid. `doctor` reports whether that parent
  is still alive: a dead parent means the whole tree went down together (the AI
  app quit or was killed), while a live parent means only the MCP process was
  killed. Windows reuses pids, so this is a hint, not proof.
- `loci doctor` prints the most recent run when it did **not** end cleanly:

  ```
  run history   UNCLEAN
    unclean  pid=1234  last heartbeat 2026-11-06T10:20:18Z  parent pid=4360 (dead)
    reason: no clean exit recorded (crash or external kill)
    full log: C:\Users\PC\.loci\logs\loci.log
  ```

  When the latest run is `clean` or `running`, it prints a single quiet line
  instead. Older kills followed by clean starts are history, not a live alarm.
- `resources/templates/list` returns an empty template list instead of a
  method-not-found error. Loci exposes one fixed `loci://context` resource, but
  clients were logging a "failed to list MCP resource templates" warning on
  every connect.

### Changed

- A crash now also writes the full stack to stderr, prefixed `[loci] fatal:`,
  after the existing one-line message. Before this, the stack survived only in
  `~/.loci/server.log`; a host that shows just the last stderr lines displayed
  "exited with code 1" and nothing else. An unhandled rejection keeps its
  non-fatal behaviour and now includes the stack in its warning.
- The lifecycle hooks are deliberately shared, not duplicated. Crash handling
  stays with `crashlog.js` (an unhandled rejection still does not take memory
  offline), the stdio transport stays with readline, and the lifecycle module
  is told to leave both alone. A clean stdin close records the run without
  forcing `process.exit`, so buffered stdout is not truncated on Windows.

## [0.9.0] - 6 November 2026

### Added

- Stale badge refresh in the Files view. While the view is open and the page is
  visible, Loci re-checks imported files every ten seconds and updates only the
  badge when a file changed or vanished on disk. The poll reads metadata through
  `GET /api/documents/status`, so a large library is not re-chunked on a timer,
  and no document text is rewritten on a change.
- Optional OCR for scans and images. `loci docs add --ocr`, `ocr: true` on the
  `doc_add` MCP tool, and an **Read scans with Windows OCR** checkbox in the
  Add-file dialog (shown only when Windows reports an engine) read a scanned PDF
  or photo through `Windows.Media.Ocr`. Normal text extraction runs first and
  always wins; OCR is only a fallback for a file with no text layer.
- OCR is capped on purpose: 20 MB input, 25 PDF pages, 3 minutes, 8 MB of
  captured output, and 2 million characters. Hitting a cap returns a "split the
  file" error instead of silently truncating the text.
- `GET /api/capabilities` reports whether OCR is available and which languages
  Windows has installed, so no surface advertises a feature this machine cannot
  deliver.

### Changed

- The Add-file dialog asks for capabilities only when it opens, and the probe is
  cached for the life of the process. A machine without Windows OCR pays nothing
  and never sees the checkbox.
- A browser upload is written to a private temp folder so Windows OCR has a real
  path, read through the same import path as the CLI, then removed. The temp path
  is never stored, so stale detection stays honest for files imported by path.

### Verification

- `node --test`: 141 tests, 141 pass, 0 fail (109 core, 32 UI). New tests cover
  metadata-only document status, both stale flags, scope on the composite status
  list, normal extraction winning over OCR, OCR's file-type gate, the 20 MB input
  cap, and the capability shape.
- Packaged on 6 November 2026 (32,025,721 bytes). The ZIP was extracted to a
  fresh folder and run against an isolated `LOCI_HOME`: version 0.9.0 and
  **12 tools — 8 memory and 4 document** on the handshake, `doctor` exit 0, and
  `ocr  Windows OCR (en-US, fr-FR) — opt-in with docs add --ocr`. The package
  contains no test folder.

## [0.8.0] - 5 November 2026

### Added

- A larger concept graph for recall. Around fifty equivalence groups cover
  software work and everyday life (home, health, family, finance, travel, food,
  work, education, media). Same-group terms expand at weight `0.4`; groups that
  are directly related expand one more step at `0.18`, so a broad term such as
  *backend* can reach a stored *Postgres* note.
- Expansion is still query-side and one hop only. A generated term is never
  re-expanded, so the graph cannot chain across the whole map, and a literal
  match still outranks any expansion.
- `memory_capture` MCP tool. It scans a transcript, meeting notes, or pasted
  message with the existing rule-based extractor, returns candidates with a
  confidence score, marks near-duplicates against active memories, and **never
  writes**. The assistant shows the list and calls `memory_add` only for the
  lines the user approves.
- `doc_add` MCP tool. It imports a named local file through the same reader as
  the window and CLI (text, Markdown, CSV, JSON, HTML, source, best-effort
  `.docx` and text-based PDF), or accepts pasted text with an optional title and
  tags. Re-importing the same path updates the document in place.

### Changed

- The shipped surface is now **12 tools — 8 memory and 4 document**. The README,
  sales page, plan, and selling notes were updated together.
- README recall wording now describes the two-level graph, the `0.4` and `0.18`
  weights, and the one-hop limit instead of a flat synonym map.

### Verification

- `node --test`: 135 tests, 135 pass, 0 fail. New tests cover a true
  12-tool handshake, candidate-only capture with a duplicate marker, an empty
  capture result, file import by relative path, pasted-text import, search of the
  imported document, and rejection of ambiguous `doc_add` input.
- The packaged ZIP was extracted to a fresh folder and run against an isolated
  `LOCI_HOME`: version 0.8.0 and 12 tools on the handshake, candidate-only
  capture, relative-path `doc_add`, search of the imported text, a 400-character
  brief with a stored memory, and `doctor` exit 0.

## [0.7.0] - 5 November 2026

### Added

- Opt-in at-rest encryption. `loci encrypt` generates a random 256-bit key at
  `~/.loci/encryption.key` and rewrites every memory and document line as an
  AES-256-GCM envelope. `loci encrypt --passphrase` derives the key with scrypt
  from `LOCI_PASSPHRASE` or `LOCI_PASSPHRASE_FILE` and stores only a salt.
  `loci decrypt` rewrites the stores as plain text and, in key-file mode, removes
  the key.
- Stores can be read while they hold a mix of plain and encrypted lines, so a
  migration that is interrupted by a crash is still readable. Writes use the
  configured key from that point on.
- Opt-in credential redaction. `loci redact --enable` replaces recognised
  credentials with `[redacted:<rule>]` before a memory or document is written.
  `loci redact` reports existing matches without printing the secret, and
  `loci redact --apply` rewrites those records. Ten rules cover private keys,
  OpenAI, GitHub, AWS, Google, Slack, Stripe, JWTs, bearer tokens, and
  password/secret assignments.
- Documents redact title, text, and tags on add and update; a redacted document
  gets a fresh hash and character count.
- `loci doctor` reports encryption mode, key availability, and the number of
  encrypted and plain lines. If the key is missing it names the problem and
  stops instead of crashing on the first store read.

### Changed

- A missing key or passphrase now produces a named error instead of an empty or
  unreadable store. The CLI entry point prints `error: <message>` and exits 1
  rather than dumping a stack trace for expected key-management failures.
- The README documents the difference between key-file mode (convenient,
  unattended, does not protect a copied `.loci` folder) and passphrase mode
  (protects a copied folder, requires `LOCI_PASSPHRASE` in every process).

### Verification

- `node --test`: 127 tests, 127 pass, 0 fail (97 core, 30 UI).
- The packaged ZIP was extracted to a fresh folder and run against an isolated
  `LOCI_HOME`: the MCP handshake listed 10 tools, a stored fact survived an
  encrypt/decrypt round-trip, plaintext did not appear on disk while encrypted,
  and `loci redact --apply` swept an existing secret.

## [0.6.0] - 5 November 2026

### Added

- Facts and preferences now get the same correction guard as decisions. Saving a
  fact whose attribute matches an active one ("the office is in Lisbon" after "the
  office is in Porto"), a preference whose head noun matches ("prefer light mode"
  after "prefer dark mode"), or a leading predicate with a different ending is
  refused unless the caller passes `supersedes` or `force`. Different attributes
  and different predicates are still allowed. The check is lexical and only
  compares claims of the same kind.
- The review queue now has a way out. Alongside memories the user flagged, it
  includes decisions and preferences that have not been confirmed for 180 days,
  and each row says whether it is flagged or out of date. Rows can be confirmed,
  edited, retired, or **snoozed for 30 days**; **Snooze all 30 days** handles the
  whole queue. Snoozing hides a row without changing its text, basis, or stored
  flag, and confirming, flagging, or editing wakes it again.
- `memory_list` with `needsReview: true` now includes out-of-date decisions and
  omissions snoozed memories, so the agent sees the same queue the window does.

### Changed

- Blocked writes name the kind that clashed — "this fact", "this preference", or
  "this decision" — and the CLI exits 2 for all three. The window returns 409 and
  shows both retry forms, as it already did for decisions.
- `stats()` reports `needsReview`, `reviewDue`, and `snoozed` separately. The
  window's Review queue count is flagged plus out-of-date, excluding snoozed.
- The README no longer says review is manual or limited to what the user flagged.

### Verification

- `node --test`: 118 tests, 118 pass, 0 fail (88 core, 30 UI).

## [0.5.1] - 5 November 2026

### Fixed

- `memory_context` no longer answers "No memories stored yet." when a small
  `budgetChars` cannot fit the formatted brief but has room for bare lines. It
  falls back to a compact brief with no section headings, and if not even one
  line fits it says how many memories are stored and what budget would include
  one. Before this, `budgetChars: 400` against a populated store returned the
  empty-store message.
- The README no longer implies setup wires VS Code. Setup covers Claude Desktop,
  Cursor, Windsurf, OpenCode, and Zed; the README now says so and documents the
  VS Code `mcp.json` block for anyone who wants it.

### Verification

- `node --test`: 105 tests, 105 pass, 0 fail (78 core, 27 UI).

## [0.5.0] - 5 November 2026

### Added

- A second decision cannot be stored on top of the first. `memory_add` on a
  `decision` compares it with current decisions; when they overlap and no
  `supersedes` or `force` is given, nothing is written and the result lists the
  conflicting ids and both ways forward. Near-identical paraphrases still merge,
  so rephrasing the same decision does not look like a contradiction.
- `supersedes` accepts one id or an array. Every named claim is retired, and the
  replacement is appended before anything is retired, so an interrupted write
  cannot lose both versions.
- A fuzzy lexical fallback for recall. When BM25 returns nothing, Loci compares
  query trigrams with each memory and accepts a hit at 60% coverage. It catches
  typos and missing words and is labelled "fuzzy lexical" — it is not semantic
  search and does not claim to be.
- Imported files remember their size and modification time. Listings and search
  results mark a document `changedOnDisk` or `missing`, and the window shows the
  warning under the file.
- Citations use the document's own heading when it has one: `“Title” — § Refunds`
  instead of a generic part number. Plain text keeps `part n of m`.
- The window reopens the last project folder on the next launch, when the folder
  still exists. An explicit `--root` always wins, and a remembered folder that
  was deleted is ignored.
- Decisions and preferences that have not been confirmed for 180 days are marked
  "may be out of date" in the window. This is a prompt; nothing is rewritten,
  retired, or removed from the brief. Set `review.afterDays` to `0` to disable it.
- **Back up** in the header downloads every memory and imported file as one JSON
  file. The token is sent as a header, so the page builds the download itself.

### Changed

- The CLI `add` command exits 2 when a decision is blocked, prints the
  conflicting ids, and documents `--force`.
- The memory window shows a blocked decision inline, preselects the conflicting
  claims as the replacement, and turns Save into **Replace and save**.
- The capture, setup, and import paths still write in bulk on purpose; they skip
  a blocked candidate and report the count instead of stopping the whole action.

## [0.4.5] - 4 November 2026

### Added

- A crash log at `~/.loci/server.log`. Uncaught exceptions and unhandled
  rejections append a timestamp, message, and stack, rotating at 256 KB.
  `loci doctor` prints the last error and the log path, so "it just stopped"
  has an answer instead of silence.
- `memory_list` accepts `needsReview: true`, returning memories the assistant
  flagged for review and memories that were never confirmed.
- The memory window can confirm one row or the whole review queue. Confirming
  records `lastConfirmedAt`, marks the basis as stated, and clears the flag.

### Changed

- `memory_confirm` now says what its callers must do: call it only after the
  user explicitly approves the memory in the current conversation.
- The review queue's "Looks right" button is now **Confirm** and maps to
  `memory_confirm`, so the window and the tool record the same thing.

### Fixed

- `memory_search` no longer returns pinned memories for questions they have
  nothing to do with. Pinned records entered the ranking pool at zero and were
  kept for every query, so "banana smoothie recipe" returned the installer
  decision. A search now keeps only hits that clear the same relevance floor
  the handover already used.
- The handover brief compiles from up to 32 memories and lets the 2,000
  character budget decide what fits, instead of stopping at 12 records first.
  A 20-record store was shipping 13 items with 700 characters still unused,
  and the oldest decisions were dropped while recent trivia stayed.
- Inferred memories are labelled `(inferred)` in the brief. The header says
  the brief is built from the user's own notes, not guesses, so an
  assistant's inference must not read like a stated fact.
- A decision no longer looks like a conflict merely because both texts begin
  with "Decision:". The warning strips that prefix before it searches.

### Accessibility

- The memory and file lists are no longer live regions. A screen reader used to
  re-announce every row after each keystroke in the search box; the result
  count speaks instead, and the rows stay quiet until asked for.
- The icon buttons on a card name the memory they act on. A screen reader now
  hears "Delete this memory" followed by the memory text, not nineteen
  identical "Delete" buttons.
- The window honours `prefers-reduced-motion`.

### Security

- The memory window rejects any request whose `Host` header is not a loopback
  name. The per-run token is embedded in the page, so a hostile site could read
  it by pointing its own domain at `127.0.0.1` (DNS rebinding) and then calling
  every API as the user. The page and the API are now unreachable under a
  foreign host, and the attacker never learns the token.
- The window refuses to be framed (`frame-ancestors 'none'`,
  `X-Frame-Options: DENY`), so another page cannot sit invisibly on top of it
  and collect clicks on Delete or Confirm.
- The unused `?token=` fallback is gone. The token travels in the
  `x-loci-token` header only, so it cannot leak through a URL.

## [0.4.4] - 4 November 2026

### Fixed

- `install.bat` no longer stops every Node process running from
  `%LOCALAPPDATA%\Loci`. That filter also matched an AI app's live memory
  connection, so upgrading silently disconnected memory until the app was
  restarted — and apps that do not retry a failed local server, such as
  OpenCode, stayed disconnected for the rest of the session.
- The installer now copies the download to `%LOCALAPPDATA%\Loci.staging`
  first, closes only the memory window (a process whose command line runs
  `loci.js ui`), and refuses to continue while a memory connection is still
  running from the install folder, naming the app that holds it. The old tree
  is then replaced with a rename, with rollback if the second rename fails, so
  a failed upgrade can no longer leave a half-old, half-new install.
- The installer always asks for the AI app to be restarted, instead of only on
  the failure path.

### Verification

- `node --test`: 72 tests, 72 pass, 0 fail.
- Upgrade path tested with a live connection: the installer refused and named
  the holding app instead of terminating it.
- Clean upgrade tested with the connection closed: the zip installs, `doctor`
  reports a healthy store, and the MCP handshake reports ten tools.

## [0.4.3] - 4 November 2026

### Added

- `memory_confirm` — marks a memory as confirmed by the user, records
  `lastConfirmedAt`, sets `basis` to `stated`, and clears the review flag.
- `basis` and `lastConfirmedAt` provenance fields on memories. Search and list
  output now shows `created:`, `confirmed:`, and `source:` for each result.
- A conflict warning after adding a decision: the top three similar active
  decisions in the same scope are listed, with instructions to supersede one
  explicitly. Nothing is retired automatically.

### Changed

- `memory_context` calls return the full compiled brief. The `loci://context`
  resource returns the same brief instead of pinned memories only.
- `memory_search` is read-only by default. Usage is updated only when
  `trackUsage` is true.
- The MCP tool list, README, and sales page describe **ten tools** — seven for
  memory and three for documents.
- Document search excerpts are wrapped in a delimited block, and any closing
  marker inside the excerpt is escaped.
- `relevanceFloor`, `baselineLimit`, `documentLimit`, and
  `documentRelevanceFloor` are passed through from config in both MCP and CLI.

### Fixed

- `compileBrief` now returns a budget error when `budgetChars` is too small for
  the header and footer, instead of reporting that no memories are stored.
- Legacy memories without `lastConfirmedAt` or `basis` still load and display;
  their creation date is shown as the confirmed date and their basis is treated
  as stated.

### Verification

- `node --test`: 72 tests, 72 pass, 0 fail.
- The 0.4.3 zip is built, installed into `%LOCALAPPDATA%\Loci`, and checked
  with `doctor` and a ten-tool MCP handshake.
