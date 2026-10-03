---
name: salvation
description: Session protocol that joins three tools — docket (`dk`, project cards + session ledger), yggdrasil (`ygg`, the code index) and fur (the plain-markdown archive the ledger lives in) — so a project can be picked up by any fresh chat, agent or model and put down again without losing why things were decided. Use in EVERY session on a project that has a docket card, or when the user says "salvation", pastes a `RESUME.md` (`<!-- dk:resume v1 -->`), a ygg codex, or asks to resume, continue, hand off or close work on a project. Use it even when the user does not name any of the tools.
license: MIT
metadata:
  author: Andrew Ryan Garcia
  argument-hint: "[card name, or path to a RESUME.md]"
---

# Salvation

A chat ends and everything it worked out is gone. The code survives because git
keeps it; the *reasoning* — what was tried, what was rejected and why, what the
next step is and which model it deserves — survives only if someone writes it
down in a place the next session will read. Salvation is that loop, with three
small tools each doing one job:

| Tool | Question it answers | Where its output lives |
|---|---|---|
| **docket** (`dk`) | What is this project, what is its state, what is next? | a *card*: one markdown file in a *book* (a folder of cards) |
| **fur** | What was decided and rejected in earlier sessions, and why? | a *session ledger*: a fur conversation inside the book, `sessions/chats/<slug>-<id>/` |
| **yggdrasil** (`ygg`) | Which files matter, and how big are they? | a *codex*: an index (and, on request, bodies) of the files a `WHITE.md` names |

`dk resume <card>` joins the three into one file, `RESUME.md`, ordered by what
cannot be rebuilt: card, then ledger, then code index. Every session **starts
by reading it and ends by writing one entry back**. Nothing else is required.
No daemon, no database, no vector store, no harness hooks: plain files, so the
loop works in claude.ai, Cowork, Claude Code, Codex, Cursor or a terminal.

Why this shape, in four lines. Context is finite and accuracy falls as it
fills (Chroma measured it across 18 models; Anthropic's guidance is "the
smallest set of high-signal tokens"), so the resume carries an *index* of the
code, not the code. Decisions are what a fresh session cannot rederive from a
repo (the "vanishing why"; PROJECTMEM estimates 5–20k tokens per session spent
re-deriving context without a memory layer), so the ledger records `decided`
and `rejected` with reasons, not a transcript. Agents working across context
windows fail by over-reaching or by declaring the job done early (Anthropic's
long-running-agent harness), so each entry names the task in flight and its
remaining criteria. The frozen contract is `SPEC.md` in the salvation repo
(D1–D5; formerly docket's `docs/resume-contract.md`); if it is reachable and
disagrees with this skill, it wins. The tools named here are the reference
implementation: anything that reads and writes the same files conforms.

This skill supersedes session-pilot, session-resume and session-close, if
they exist. If any of them is loaded too, follow this one where they differ.
A project's *own* rules are different: its `CLAUDE.md` / `AGENTS.md`, or a project-specific skill
(a `<project>-session-pilot` with its own routing table and review
boundaries), are binding on what they cover and outrank §2 here. This skill
still governs how the session resumes and how it saves.

The user does not need to explain any of this. "Use salvation", plus a card
name or just a folder (a path, a shell prompt, a pasted `git status`), is a
complete instruction.

## 0. Find out where you are

Decide this once, at the start, and say which path you took.

- **A — a shell with `dk`.** You can run commands on the user's machine and
  `dk --version` answers. Use the fast paths below.
- **B — files, no `dk`.** You can read the book (in Cowork: the connected
  folder; `dk where` would print it) but cannot run `dk`. Assemble by hand
  (§1 hand path) and save by hand (§3).
- **C — plain chat.** You can read only what the user pastes. Ask for
  `RESUME.md` (and `ygg --white WHITE.md --contents` beside it if the task
  needs file bodies). At the end, print what must be saved and where.

**Which card.** If the user named it, use that. If they only pointed at a
folder: on path A run `dk here` from it; on path B read the header of each
top-level card in the book and take the one whose `path:` or a `place:`
contains that folder (the deepest match wins; two cards tied: ask which). On
path C ask for `dk resume "$(dk here)"` run from that folder. **The book must
be reachable** for A and B: if it is not connected, ask once for that folder
(`dk where` prints it), and do nothing else until it is.

A project with **no card yet** gets one before anything else: `dk add <path>`
(no path = an idea), then a `WHITE.md` in the repo listing the files that
matter. That is the whole onboarding. Several folders for one project: keep
one card and add `place: <label> <path>` lines under `path:`.

## 1. Orient — before any work

**A:** `dk resume <card> --out <path outside the book and outside any repo>`
(`dk resume book/card` or `-b <book>` when there are several books; `--place
<label>` to limit the code index to one folder; `dk here` from inside a project
folder prints its card). Read the file it prints. Token cost per part is on
stderr.

**B, hand path**, same order and same output:

1. **Card** — `<book>/<name>.md`, by name or a prefix of its `id:`. Keep all
   of it above `## readme`. Note `id:` (8 hex), `path:`, `place:` and `white:`.
2. **Ledger** — the one `<book>/sessions/chats/*/convo.md` whose front-matter
   `tags:` holds `dk-<id>`. None: `[no sessions yet]`. Two or more: stop and
   name the folders; do not pick. Unreadable as text: say the archive may be
   locked (`fur unlock`) and stop.
3. **Linked files** — each `<!-- fur:msg … link=<file> -->` line in
   `convo.md`, oldest first, names a file in the same folder. Classify by first
   line only: `<!-- dk:session` is an entry, `<!-- dk:doc` is a document,
   anything else is chatter to ignore. Never follow a link that is absolute or
   has `..`; say so. A missing link gets one line.
4. **Sessions, newest first** — the newest three entries whole, separated by
   `---`. Older ones as one line each under `### earlier`:
   `- SES-<stamp> · <first line of its ## next> · <date>`.
5. **Documents** — never inlined, never counted among the three. One line each
   under `### documents`: `- DOC-<name> · <title> · <status> · <~tokens> · <date>`.
   Open one in full only when the task needs it, and say that you did.
6. **Code** — if the card has `white:` or the project has `WHITE.md`, run
   `ygg --white <manifest> --out <temp>` from the project folder and read the
   index. No shell: read the manifest and list the files with sizes. One
   block per place when the card has several. No manifest: say so; never load
   the whole repo. No `path:`: `[no project path — this card is an idea]`.

**C:** the user's pasted `RESUME.md` is the result of the above. Treat it as
current unless a later paste says otherwise.

**Then report, before working, in this order and briefly:**

1. **State** — the newest entry's `state` and the card's `state:` line, and
   whether they agree.
2. **Next** — the newest entry's `next`, with the model and effort it names.
3. **Open items** — the newest entry's `blockers`, labelled *claimed,
   unverified* unless the card's ticked boxes or other evidence show them done.
   Only the newest entry counts; older blockers are stale.
4. **What you could not load** — no `dk`, no code index, an unreadable file —
   and which path (A/B/C) you took.

Then wait. Do not start work the user has not asked for.

## 2. Work — stay in lane, read what you touch

- **The code part is a map, not the territory.** It lists files; it does not
  contain them. Before changing a file or describing what it does, open it
  (shell) or ask for it. Never describe code from its name and size. Never
  invent a path: a `mod x;` or an import proves a module exists, not whether it
  is `x.rs` or `x/mod.rs`. If you cannot see it, ask for `ygg tree -L 3 <dir>`.
- **Asking for files (path C):** answer everything the current codex supports
  first, then one fenced block of bare paths, last thing in the message, spelt
  as the INDEX spells them, with one sentence naming what you cannot determine
  without them. Do not ask twice; proceed on stated assumptions.

  ````
  ```
  src/scanner/mod.rs
  src/scanner/archive.rs
  ```
  ````

  To regenerate: `ygg --only <paths> --printed`, `ygg tree`, `ygg pick`.
  `--printed` is a format, not a selector. Do not suggest `--sniff`.
- **Decided and rejected are settled.** Reopen one only if the user does or
  you have new evidence, and then name the entry it came from.
- **Scope is literal.** Discoveries become new `[ ]` items on the card, never
  extra diff. If a discovery blocks the task, say so and stop.
- **Acceptance criteria first.** Restate them as a test plan before code; if
  the task has none, write them and get agreement.
- **Silently-wrong-output files** (numerics, geometry, offsets, money, dates,
  migrations, auth): minimal diff, a test pinning exact values, and
  `⚠ needs human review` in the entry. State the assumption; don't bury it.
- **Leave the repo runnable.** Finish the seam or revert it.
- **Task ids are a session's own labels.** Spell them out in words wherever
  they appear; never fill one in from the card's checklist.
- **No secrets** in anything you write back.

### Routing: tier by failure mode, not by size

Ask "if this comes out wrong, how will I find out?"

| Failure mode | Tier |
|---|---|
| A test, type-checker or linter catches it at once | cheapest model that can write it |
| Compiles and passes but can be subtly wrong — numerics, spans, concurrency, money, dates, migrations, security | premium, high effort, human reviews |
| The cost is a bad *decision* — architecture, schema, protocol, public API, naming | premium, design review only; no implementation that session |
| Mechanical — renames, conversions, boilerplate, scaffolding from a pattern | cheapest tier, batched |
| Reading and explaining existing code | mid tier; size the context, not the cleverness |
| One-line answer, triage | cheapest tier |

Prefer raising effort on a cheaper model before jumping a tier. Escalate one
tier only after failing the same criterion twice. De-escalate the moment the
remaining work is mechanical. Never recommend the top tier for implementation.
Recommend a model and effort, nothing else — not a surface or product.

## 3. Save — every session, even discussion-only or aborted

Two things may be saved. **An entry, every time.** **A document**, only when
the session produced reasoning worth rereading whole (an option analysis, a
phased plan, a draft spec): if a `decided` line cannot hold its reason in one
line, the reason is a document. A spec the *code* obeys goes in the repo (e.g.
`docs/`), not the ledger; ask before writing into a repo.

### The entry — `SES-YYYYMMDD-HHMMSS.md`, UTC, all eight headings, this order

```markdown
<!-- dk:session v1 -->
# <card> · <YYYY-MM-DD HH:MM> · <harness> · <model> <effort>

## done
- <what changed, one line each>

## decided
- <choice> — <why>          (or: — see DOC-YYYYMMDD-<slug>)

## rejected
- <option> — <why not>

## state
<task in words · criteria met/remaining, or "no task in flight">

## blockers
<the human action needed, or none>

## next
<task in words · model effort · failure mode that justifies the tier>

## files
- <path> — <new | changed | removed>
```

An empty section says `none`. Every `decided`/`rejected` line carries its
reason or points at a document — `rejected` is why this ledger exists. Record
only what was said or done; never record your own advice as the user's
decision unless they took it. Under about 40 lines: the newest three are
loaded whole by every resume.

### The document — `DOC-YYYYMMDD-<slug>.md`

Starts `<!-- dk:doc v1 -->`, a `# <title>`, then `status: draft | adopted |
superseded` (required). A distillation, not a transcript: the question, the
options and their costs, the choice and why, what would change it. Revise in
place; replace by writing a new one and setting the old `status:` to
`superseded`.

### Where (paths A and B)

```
<book>/sessions/chats/<slug>-<first 8 of conversation_id>/
    convo.md
    SES-…md
    DOC-…md
```

1. Find the conversation tagged `dk-<id>` (as in §1.2). None: create the
   folder — `<slug(title)>-<first 8 of a new uuid v4>`, title `<card>
   sessions` — with `convo.md` starting:

   ```
   ---
   fur_schema: 1
   conversation_id: <uuid v4>
   title: <card> sessions
   created_at: <RFC3339 UTC>
   tags:
     - dk-<id>
     - session
   ---
   ```
2. Write the document first if there is one, then the entry, beside `convo.md`.
3. **Append** one marker per new file to `convo.md` — a blank line, then
   `<!-- fur:msg id=<uuid v4> avatar=claude ts=<RFC3339 UTC> link=<file> -->`
   (`sha256=<hex>` optional). Never rewrite or reorder the file.
4. **Card write-back, exactly three changes:** replace the first `state:` line
   in `## now` with the entry's state (insert one if missing); tick `[ ]` →
   `[x]` for items finished, matched by existing text, never reworded; append
   new `[ ]` items to the end of `## next`. Nothing else in the card moves.
5. **Verify from disk, then tell the truth.** Re-read each file you wrote
   (not the tool's "written" reply); check the markers are the last lines of
   `convo.md`; use a fresh output name per write. With `dk`, `dk resume
   <card>` should now show the new entry first. Say what was saved and where.
   A failed write is reported, never claimed.

### Path C — nothing can be written

Print the entry (and document) in fenced blocks and, under them, the exact
marker lines to append and the folder to put them in. Then the footer.

### The footer — last thing in the final message, always

```
────────────────────────────────
SESSION CLOSE — <project>
Done this session : <one line, or "discussion only">
State             : <task in words · criteria done/remaining, or "no task in flight">
Blockers          : <human action, or "none">
NEXT TASK         : <task in words>
RECOMMENDED AI    : <model · effort>
Why this tier     : <one line naming the failure mode>
Saved             : <paths of the entry and any document, or "not saved: <reason>">
────────────────────────────────
```

NEXT TASK is the first open, unblocked item on the card in order — unless the
current task is unfinished, in which case it is *finishing it*. Blockers names
the human action ("review the coordinate math in `frame.rs`"), not a vague
dependency. `state`, `blockers` and `next` are the same text in the entry and
the footer. If you cannot name the failure mode in one line, you have not
picked the tier deliberately.

## Reference — the commands

```bash
dk                         # the list (several books: the index)
dk add <path>              # new card; no path = an idea
dk <card> / dk show <card> # read a card
dk edit <card>             # edit it in the terminal
dk resume <card> --out F   # RESUME.md: card → ledger → code index
dk resume <card> --place L # one of several places
dk here                    # which card owns this folder
dk book                    # the books; -b <book> or book/card for one command
dk where [-b book]         # a book's folder

ygg --white WHITE.md --out F         # index of the manifest's files (what dk runs)
ygg --white WHITE.md --contents      # with bodies, for a chat that cannot read the repo
ygg tree [-L 2] [-s]                 # structure, optionally with symbols
ygg --only a.rs b.rs --printed       # just these files

fur convo                  # list conversations in a fur project (the book is one)
fur printed                # flat transcript of the active conversation
fur unlock / fur lock      # the ledger is encrypted at rest if the user chose to
fur rebuild                # .fur/ is only an index; the markdown is the archive
```

docket never writes the ledger and never calls fur; whoever ends the session
writes the entry — you, through this skill. The three tools stay separate on
purpose: each is replaceable, and the files outlive all of them.
