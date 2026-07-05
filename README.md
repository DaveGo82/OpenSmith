# OpenSmith — live dashboard for your AI agents

OpenSmith shows what your AI agents are doing, live in the browser: agent
cards grouped by discipline (Live / Done / Waiting + which model each runs),
a **Live Connectors** diagram of who hands work to whom (animated data
packets), built-in terminals for up to 4 agents (watch their output, type
interrupts back), and a Matrix-style **Agent** screen. One self-contained
Windows exe — no install, nothing leaves `localhost:1999`.

This is the open-source build: same features and API as the private "Smith"
project, with original, MIT-friendly branding — no third-party characters,
photos, or trademarks.

## Files in this repo
| File | What it is |
|---|---|
| `OpenSmith.exe` | The app (Windows x64, self-contained single file) |
| `smith-skill.zip` / `SKILL.md` | The Claude skill that teaches a session to report into OpenSmith |

## Use — 3 steps
1. **Run `OpenSmith.exe`.** A small dark "OpenSmith is Running" window appears and the dashboard opens in your browser.
2. **Install the skill** in the project you want monitored: extract the zip so you get `<project>\.claude\skills\smith\SKILL.md` (or `%USERPROFILE%\.claude\skills\smith\` to cover every project).
3. **Wake it up:** say "use Smith" once in your Claude session — or make it permanent by adding this line to that project's `CLAUDE.md`:
   > Always display agent activity in Smith (skill: smith).

That's it. The session registers its agents, statuses, models, and hand-offs
automatically, and the dashboard updates live over the wire — no refresh
needed. Right-click any card to open a live terminal for that agent. Anything
that speaks HTTP can push data too (`GET http://localhost:1999/api/state` and
friends) — see `SKILL.md` for the endpoint summary.

## Branding
Every icon and the "Agent" tab visual are original, procedurally-generated
artwork (drawn from geometric primitives — see `scripts/make-open-assets.ps1`
in the main Smith repo). No third-party characters, photos, or trademarks are
used anywhere in this build.

## License
MIT — see [LICENSE](LICENSE).
