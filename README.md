<p align="center">
  <img src="assets/salvation.png" alt="Salvation" width="640"/>
</p>

<h1 align="center">Salvation</h1>

A protocol for picking a project up in any fresh chat, agent or model, and
putting it down again without losing *why* things were decided.

Every session starts by reading one file (`RESUME.md`: the project card, the
newest session entries, an index of the code) and ends by writing one entry
back (`done / decided / rejected / state / blockers / next / files`). Plain
markdown, no daemon, no database, no harness hooks — so it works in claude.ai,
Cowork, Claude Code, Codex, Cursor or a terminal.

| File | What it is |
|---|---|
| `SPEC.md` | the interface: where files live, their formats, what the resume contains. Binding. |
| `SKILL.md` | the same protocol as session instructions, for a model to follow. Drop it into any harness that reads skills, or paste it into a chat. |

## Reference implementation

- **[docket](https://github.com/andrewrgarcia/docket)** (`dk`) — project cards, books, and `dk resume`
- **[yggdrasil](https://github.com/andrewrgarcia/yggdrasil-cli)** (`ygg`) — the code index from a project's `WHITE.md`
- **[fur](https://github.com/andrewrgarcia/fur-cli)** — the plain-markdown archive format the session ledger is kept in

Each is replaceable. Anything that reads and writes the files `SPEC.md`
describes conforms; a session with no tools installed can follow it by hand.

Named, with a nod, after another Andrew Ryan's city — but this one is meant to
hold.

## License

MIT
