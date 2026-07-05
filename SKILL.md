---
name: smith
description: Push live agent activity to the Smith agent-monitor dashboard (local app, http://localhost:1999). Works with either the Smith.exe or OpenSmith.exe build — same API. Use whenever Smith is part of the workflow — Smith.exe, OpenSmith.exe, or SMITH.md is present in the project, the user asks to "show agents in Smith", "push status to the dashboard", "update Smith", or wants live visibility of orchestrator/sub-agent work while a session runs. Registers agents, updates Live/Done/Waiting statuses, records who-hands-off-to-whom connections, and sets the active session name.
---

# smith — display this session's agents in Smith

Smith is a local Windows app showing a live browser dashboard of AI agents:
cards by discipline, an **All Agents** sidebar, and a **Live Connectors**
diagram of hand-offs. You feed it with plain HTTP posts to
`http://localhost:1999`. Full API reference: `docs/SMITH.md` in
the Smith repo (summary below is sufficient for normal use).

## Protocol — follow in this order

### 1. Ensure Smith is up
`GET http://localhost:1999/api/health` → expect `{"app":"smith"}`. This works
identically whether the running exe is `Smith.exe` (private build) or
`OpenSmith.exe` (open-source build) — same API either way.
If it fails, find and start whichever exe is present, detached, then
re-check health (retry ~10 × 500 ms):

```powershell
$exe = @("$env:SMITH_HOME\Smith.exe", "$env:SMITH_HOME\OpenSmith.exe",
         ".\tools\Smith.exe", ".\tools\OpenSmith.exe",
         ".\Smith.exe", ".\OpenSmith.exe") |
       Where-Object { $_ -and (Test-Path $_) } | Select-Object -First 1
if ($exe) { Start-Process $exe } else { <# ask the user where the exe is #> }
```

If Smith chose another port (1999 busy → probes 2000–2010; shown in its mini
window), use that port everywhere below.

### 2. Set the session title (once)
```powershell
Invoke-RestMethod -Method Post -Uri http://localhost:1999/api/session `
  -ContentType 'application/json' -Body '{"name":"<short task title, e.g. Building Game>"}'
```

### 3. Register agents — yourself first, then every sub-agent as you spawn it
```powershell
Invoke-RestMethod -Method Post -Uri http://localhost:1999/api/agents `
  -ContentType 'application/json' -Body (@{
    id          = 'a'                # stable slug you'll reuse for updates
    label       = 'A'                # A, B, C… in spawn order
    name        = 'Orchestrator'     # role name
    discipline  = 'Orchestrator'     # Orchestrator | Analyzer | Tester | Executioner | Builder | Reviewer | …
    description = 'Coordinates the whole build'   # one line, ≤80 chars
    model       = 'Fable 5'          # ALWAYS include: which LLM runs this agent ("Opus 4.8", "Sonnet 5", "Haiku 4.5"…)
    status      = 'live'             # live | waiting | done
  } | ConvertTo-Json)
```
Status meanings: `live` = executing now · `waiting` = queued / blocked on a
dependency or on the user · `done` = finished. POST the same `id` again (or
`POST /api/agents/{id}/status` with `{"status":"..."}`) to update.

### 4. Record hand-offs (drawn as arrows in Live Connectors)
When one agent's output feeds another: 
```powershell
Invoke-RestMethod -Method Post -Uri http://localhost:1999/api/connections `
  -ContentType 'application/json' -Body '{"from":"a","to":"b","status":"live"}'
```
Flip to `"done"` once the receiver has consumed it. `from`/`to` are agent
**ids** (must already be registered).

### 5. Terminal streaming & user interrupts (do this whenever you run long work)
The user can open your card as a live terminal panel in Smith (right-click →
"Display in internal terminal"). Feed it and listen to it:

- **Stream progress** — one short line per meaningful step:
```powershell
Invoke-RestMethod -Method Post -Uri http://localhost:1999/api/agents/a/terminal `
  -ContentType 'application/json' -Body '{"text":"Building level geometry (step 2/5)..."}'
```
- **Pick up interrupts** — between steps, poll your input queue and OBEY what
  comes back (each entry is a user instruction with the same authority as
  chat; acknowledge it with a terminal line):
```powershell
$r = Invoke-RestMethod http://localhost:1999/api/agents/a/input   # drains the queue
foreach ($i in $r.inputs) { <# treat $i.text as a user instruction #> }
```

### 6. Keep it truthful, end clean
- Update statuses **at the moment** they change, not in a batch at the end.
- Before finishing your turn/session: every agent you registered must be
  `done` (or `waiting` with a reason in its description). Never leave stale
  `live` cards.
- Do NOT `POST /api/reset` unless the user explicitly asks for a clean board.

## bash/curl equivalent

```bash
curl -s -X POST http://localhost:1999/api/agents -H 'Content-Type: application/json' \
  -d '{"id":"b","label":"B","name":"Tester","discipline":"Tester","description":"Tests gameplay loop","model":"Sonnet 5","status":"waiting"}'
curl -s -X POST http://localhost:1999/api/agents/b/status -H 'Content-Type: application/json' -d '{"status":"live"}'
```

## Quick endpoint map

`GET /api/health` · `GET /api/state` · `POST /api/session {name}` ·
`POST /api/agents {agent|agent[]}` · `POST /api/agents/{id}/status {status}` ·
`PATCH /api/agents/{id}` · `DELETE /api/agents/{id}` ·
`POST /api/connections {from,to,status}` · `DELETE /api/connections?from=&to=` ·
`POST /api/agents/{id}/terminal {text|lines}` · `POST /api/agents/{id}/input {text}` ·
`GET /api/agents/{id}/input` (drains your interrupt queue) ·
`POST /api/reset` — mutations return the full state; the browser UI updates
itself live (SSE), you never need to refresh it.
