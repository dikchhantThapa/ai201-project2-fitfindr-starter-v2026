# Running FitFindr

Everything about how the starter works and how to use it.

---

## Before your first class

Setup happens **before class**. The [environment setup page](../pages/ide_setup)
has the per-operating-system commands and the exact versions.

**Same API key you set up in unit 1.** Nothing new to sign up for. There is one
new package — the MCP SDK, for unit 4 — and it installs with everything else.

From inside this repo after you've forked and cloned it:

**macOS / Linux**

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env          # then paste your key into .env
python test.py
```

**Windows (PowerShell)**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
copy .env.example .env         # then paste your key into .env
python test.py
```

**You're ready when `python test.py` passes.**

If it doesn't and you've made one honest attempt at the error, post the
**whole** output in the help channel.

> Activate the virtual environment every time you open a new terminal. You
> should see `(.venv)` at the start of your prompt.

---

## Your first five minutes

Start with the data. Your tools will only be as good as your understanding of
what they're reading, and ten minutes here saves an hour of confused debugging.

```bash
python app.py fields                    # what a listing and a wardrobe item hold
python app.py listings --full -n 6      # read six listings all the way through
python app.py examples                  # queries worth trying
python app.py ask 'vintage graphic tee under $30'
```

> **Quote your queries with single quotes**, on every operating system. In
> PowerShell, `"vintage graphic tee under $30"` in double quotes silently
> becomes `vintage graphic tee under ` — PowerShell reads `$30` as a variable
> and substitutes nothing. You get results with no price ceiling and no error
> saying so. Single quotes are literal in PowerShell, bash and zsh alike.

That last command runs the completed FitFindr agent loop.
---

## Every command

| Command | What it does |
|---|---|
| `python test.py` | Checks your environment. Run it whenever something feels off |
| `python app.py fields` | The fields a listing and a wardrobe item have — **Milestone 1** |
| `python app.py listings` | Browse the data. `--full` for whole records, `-n` for how many |
| `python app.py examples` | Queries worth trying, including one that matches nothing |
| `python app.py ask '...'` | Run the agent on one query — **single quotes** |
| `python app.py ask` | Keep asking until you press Enter on an empty line |
| `python agent.py` | Runs both example paths — one that matches, one that can't |
| `python mcp_server.py` | Starts your MCP server — **unit 4** |
| `python mcp_client.py` | Asks the server what it offers — **unit 4** |
| `python run_eval.py --label before` | Runs every scenario five times and writes the run log — **unit 4** |
| `python serve.py` | Serves the agent over HTTP instead of exiting — **unit 9** |

Useful flags on `ask`:

| Flag | What it does |
|---|---|
| `--trace` | Print the loop step by step, once you've added the trace calls |
| `--empty-wardrobe` | Run as a user with nothing saved — one of unit 4's failure modes |

---

## Which command goes with which milestone

### Unit 3 — the build

| Milestone | What you're doing | Where |
|---|---|---|
| 1 | Read the data, run the starter | `app.py fields`, `app.py listings --full`, `app.py ask` |
| 2 | Spec your three tools | README, **Tool Inventory** |
| 3 | Write your acceptance criteria | `criteria.md` |
| 4 | Build the three tools | `tools.py`, tested one at a time from a terminal |
| 5 | Wire the loop and the state | `agent.py::run_agent` |
| 6 | Write it up | README |

### Unit 4 — the test

| Milestone | What you're doing | Where |
|---|---|---|
| 1 | Move one tool onto MCP | `mcp_server.py`, then `mcp_client.call_tool` in `run_agent` |
| 2 | Break it on purpose, then trace it | `trace.py`, `app.py ask --trace` |
| 3 | Run your test | `scenarios.py`, then `run_eval.py --label before` |
| 4 | Call each criterion, diagnose the misses | README |
| 5 | Fix one thing and re-run | `run_eval.py --label after` |
| 6 | Say what's still broken | README |

---

## Running it somewhere else — **unit 9**

FitFindr is deployed on Render:

**https://dikchhant-fitfindr.onrender.com**

The hosted version is the easiest way to test the project. No local setup or API key is required.

### Check that the service is running

**Bash / Git Bash**

```bash
curl -sS https://dikchhant-fitfindr.onrender.com/health
```

**Windows PowerShell**

```powershell
curl.exe -sS https://dikchhant-fitfindr.onrender.com/health
```

Expected response:

```json
{"status":"ok"}
```

The free Render service may take longer on the first request after it has been idle.

### Send a FitFindr request

**Bash / Git Bash**

```bash
curl -sS -X POST https://dikchhant-fitfindr.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"query":"vintage graphic tee under $30"}'
```

**Windows PowerShell**

```powershell
$body = @{ query = 'vintage graphic tee under $30' } | ConvertTo-Json -Compress

Invoke-RestMethod `
  -Uri "https://dikchhant-fitfindr.onrender.com/ask" `
  -Method Post `
  -ContentType "application/json" `
  -Body $body
```

The response contains the full FitFindr session, so it is long. For a successful run, check that:

- `"error"` is `null`
- `"selected_item"` contains an item
- `"outfit_suggestion"` contains an outfit
- `"fit_card"` contains a generated caption

You can replace the example query with another clothing request.


#### Optional quick check

Instead of reading the full JSON response, this command gives a simple yes/no check for the main FitFindr outputs.

**Bash / Git Bash**

```bash
curl -sS -X POST https://dikchhant-fitfindr.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{"query":"vintage graphic tee under $30"}' \
  | python -c "import sys,json; d=json.load(sys.stdin); print('error:', 'yes' if d.get('error') else 'no'); print('selected_item:', 'yes' if d.get('selected_item') else 'no'); print('outfit_suggestion:', 'yes' if d.get('outfit_suggestion') else 'no'); print('fit_card:', 'yes' if d.get('fit_card') else 'no')"
```

**Windows PowerShell**

```powershell
$body = @{ query = 'vintage graphic tee under $30' } | ConvertTo-Json -Compress

$d = Invoke-RestMethod `
  -Uri "https://dikchhant-fitfindr.onrender.com/ask" `
  -Method Post `
  -ContentType "application/json" `
  -Body $body

if ($d.error) { "error: yes" } else { "error: no" }
if ($d.selected_item) { "selected_item: yes" } else { "selected_item: no" }
if ($d.outfit_suggestion) { "outfit_suggestion: yes" } else { "outfit_suggestion: no" }
if ($d.fit_card) { "fit_card: yes" } else { "fit_card: no" }
```

Expected successful result:

```text
error: no
selected_item: yes
outfit_suggestion: yes
fit_card: yes
```


### Try unexpected input

**Bash / Git Bash**

```bash
curl -sS -X POST https://dikchhant-fitfindr.onrender.com/ask \
  -H "Content-Type: application/json" \
  -d '{}'
```

**Windows PowerShell**

```powershell
curl.exe -sS -X POST https://dikchhant-fitfindr.onrender.com/ask `
  -H "Content-Type: application/json" `
  -d '{}'
```

Expected response:

```json
{"error":"Send JSON with a 'query' field, e.g. {\"query\": \"vintage graphic tee under $30\"}"}
```

The service should return this clear error instead of crashing.

### Run locally

```bash
python serve.py
```

The local server runs at:

```text
http://localhost:5000
```

The hosted start command is:

```bash
gunicorn serve:app
```

`POST /ask` runs the existing FitFindr agent. `GET /health` checks whether the service is awake.

FitFindr handles one agent request at a time. Structured logs record the timestamp, input, output or error, and execution time. Local runs are written to `logs/runs.jsonl`, and the deployed service also emits the structured entries to the Render runtime logs.

---

## Where everything lives

| File | What it does |
|---|---|
| `config.py` | Every setting worth changing — **TEMPERATURE and the cache are at the top** |
| `tools.py` | The three FitFindr tools: listing search, outfit suggestion, and fit-card generation |
| `agent.py` | The completed planning loop and session state |
| `generate.py` | The only thing that calls out to a service. Handles pacing and quota |
| `trace.py` | The trace helper, and the loop's stop condition |
| `app.py` | The command line |
| `serve.py` | The same agent over HTTP, for deploying — **unit 9** |
| `mcp_server.py` | Your MCP server — **unit 4, you register the tool** |
| `mcp_client.py` | Calling an MCP tool from your agent. Given to you |
| `scenarios.py` | What your test runs. **You fill this in** |
| `run_eval.py` | Runs the scenarios repeatedly and writes the run log |
| `criteria.md` | Your five acceptance criteria. **You fill this in** |
| `data/` | 40 listings and the wardrobe schema |
| `utils/data_loader.py` | Loading the data. Use this rather than reading the files yourself |
| `results/` | Run logs. **Commit these** — they're your evidence |

---

## The two settings people go looking for

Both are at the top of `config.py`, and the brief points at them by name.

If your fit cards come out **word-for-word identical** every time, it is one of:

- **`CACHE_ENABLED`** — while you build, identical prompts reuse the answer
  already received. That's deliberate; it makes debugging free. `run_eval.py`
  turns it off automatically.
- **`TEMPERATURE`** — at `0.0` the model gives you the same words every time.
  It ships at `0.9`.

---

## About rate limits

**This is the heaviest pair in the course for call volume.** Two of your three
tools call the model, so one agent run is several requests, and a few runs back
to back while debugging can cross the per-minute limit.

`generate.py` handles it:

- **It paces itself** and says when it's waiting. **A pause is the starter
  doing its job, not a bug.**
- **It reuses answers** to prompts already sent, while you build.
- **It stops** if a session makes an unreasonable number of calls, rather than
  draining your day's allowance. For an agent, that usually means a loop isn't
  ending — look for that before raising the number.
- **It counts your calls** and prints the total when you exit.

`config.MAX_ITERATIONS` is the loop's own stop condition. Your loop this unit
is short enough that you may never hit it. Keep it — a missing stop condition
is the most common architectural failure in production agents.

---

## When something goes wrong

| What you see | What it means |
|---|---|
| `The planning loop isn't built yet` | Correct, at the start. That's the TODO in `agent.py` |
| `No GEMINI_API_KEY found` | No `.env`, or the key wasn't pasted in. On Windows check it didn't save as `.env.txt` |
| A message saying the model couldn't be reached | Usually a bad key. This is also **exactly what unit 4 Milestone 2 asks you to trigger on purpose** |
| `[rate limit] ... Waiting 34s` | Working as intended. Leave it |
| `[serve] another request is still running` | Also working as intended. `serve.py` answers one request at a time; yours is queued behind one that's mid-run |
| `QuotaGuard: This session has made 300 requests` | A loop isn't ending. Find it before raising the budget |
| `The loop ran 11 times, past MAX_ITERATIONS` | Same thing, caught earlier |
| `The server doesn't offer a tool called '...'` | Nothing registered in `mcp_server.py` yet, or the name doesn't match |
| MCP returns a string where you expected a list | You're not going through `mcp_client.call_tool`, which unwraps it |
| Fit cards identical every run | `CACHE_ENABLED` or `TEMPERATURE`. See above |
| `--trace` prints nothing | You haven't added `trace.step()` calls yet — unit 4, Milestone 2 |
| `IncompleteFieldDefinitionWarning: Field 'lifespan'` | Harmless noise from the MCP library's dependencies. Not your code, not an error. Ignore it |
| A price ceiling you typed is being ignored | You used double quotes in PowerShell. `$30` vanished. Use single quotes |
| `UnicodeEncodeError: 'charmap' codec can't encode` | Shouldn't happen — `config.py` sets the console to UTF-8. If you see it, you're running a file that doesn't import `config` |

Still stuck after one honest attempt? Post in the help channel with the full
error, what you ran, and what you expected.

---

## A note on committing

At least four new commits in unit 3, at least four more in unit 4. Your commit
history is what shows your criteria existed before your results did.

**Do not delete and recreate this repository.** You submit the same URL both
units.

`results/` is deliberately **not** in `.gitignore`.
