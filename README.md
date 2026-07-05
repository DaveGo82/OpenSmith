# Smith — live dashboard for your AI agents

Smith shows what your AI agents are doing, live in the browser: agent cards
grouped by discipline (Live / Done / Waiting + which model each runs), a
Live Connectors diagram of who hands work to whom (animated data packets),
built-in terminals for up to 4 agents (watch their output, type interrupts
back), and a Matrix-style "Agent" screen. One self-contained Windows exe —
no install, nothing leaves localhost:1999.

## Files
| File | What it is |
|---|---|
| `Smith.exe` | The app (Windows x64, single file) |
| `smith-skill.zip` / `SKILL.md` | The Claude skill that teaches a session to report into Smith |

## Use — 3 steps
1. **Run `Smith.exe`.** A small "Smith is Running" window appears and the dashboard opens in your browser.
2. **Install the skill** in the project you want monitored: extract the zip so you get `<project>\.claude\skills\smith\SKILL.md` (or put it in `%USERPROFILE%\.claude\skills\smith\` for all projects).
3. **Wake it up:** say "use Smith" once in your Claude session — or make it permanent by adding this line to the project's `CLAUDE.md`: *"Always display agent activity in Smith (skill: smith)."*

That's it. The session registers its agents, statuses, models and hand-offs
automatically, and the dashboard updates live. Right-click any card to open
its terminal. Anything that speaks HTTP can push too (`GET http://localhost:1999/api/state`).
