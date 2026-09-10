# 10 — Windows Local Operations

## 1. Target environment

The MVP is designed to run natively on the owner’s Windows workstation.

Preferred repository workspace:

```text
D:\Test_Poly
```

Do not use the entire `D:\` root as the repository itself. Keeping a dedicated repo directory prevents runtime artifacts and unrelated files from entering Git scope.

Suggested runtime layout after implementation:

```text
D:\Test_Poly\
  backend\
  scripts\
  config\
  docs\
  data\                 # gitignored
  logs\                 # gitignored
  browser_profiles\     # gitignored
  .venv\                # gitignored
  .env                   # gitignored
```

## 2. GitHub access model for local Codex

Preferred model: a repository-specific SSH deploy key with write permission.

Benefits:

- private key stays on the workstation;
- credential scope is limited to `MaksimUnimax/Test_Poly`;
- Codex does not need a broad personal access token;
- architect can independently review remote commits through GitHub.

The private key must never be committed or copied into `D:\Test_Poly`.

## 3. Deploy-key creation

Run under the owner’s Windows account in PowerShell:

```powershell
$KeyPath = "$env:USERPROFILE\.ssh\test_poly_codex"
ssh-keygen -t ed25519 -f $KeyPath -C "test-poly-codex"
Get-Content "$KeyPath.pub"
```

Only the `.pub` contents are added to GitHub:

```text
Repository -> Settings -> Deploy keys -> Add deploy key
```

Enable write access for the key because Codex is expected to push task branches.

Never paste the private key into ChatGPT, GitHub issues, task documents or source files.

## 4. Dedicated SSH alias

Recommended `%USERPROFILE%\.ssh\config` entry:

```text
Host github-test-poly
    HostName github.com
    User git
    IdentityFile ~/.ssh/test_poly_codex
    IdentitiesOnly yes
```

Then repository remote can be:

```text
git@github-test-poly:MaksimUnimax/Test_Poly.git
```

This makes the repository-specific identity explicit.

## 5. Initial clone

After key installation:

```powershell
Set-Location D:\
git clone git@github-test-poly:MaksimUnimax/Test_Poly.git
Set-Location D:\Test_Poly
git remote -v
git status
```

If the repository was cloned by HTTPS first, update the remote:

```powershell
git remote set-url origin git@github-test-poly:MaksimUnimax/Test_Poly.git
git fetch origin
```

Verify repository access with:

```powershell
git ls-remote origin
```

## 6. Git identity

The owner may configure a normal Git author identity for commits. Do not put authentication tokens in Git config URLs.

Check:

```powershell
git config user.name
git config user.email
```

If project-specific identity is needed, set it locally inside this repository, not globally unless desired by the owner.

## 7. Branch workflow

Codex normally uses:

```text
codex/p1-tooling-bootstrap
codex/p1-polymarket-probe
codex/p1-fonbet-probe
...
```

Start task:

```powershell
git fetch origin
git switch main
git pull --ff-only origin main
git status --short
git switch -c codex/<task-name>
```

If branch already exists remotely:

```powershell
git switch --track origin/codex/<task-name>
```

Never use `git push --force` unless the owner/architect explicitly changes the repository policy; current policy forbids it.

## 8. Python environment

Exact interpreter/dependency versions are pinned during Phase 1 tooling bootstrap. Architectural target is a modern supported CPython release compatible with required packages.

Use an in-repository virtual environment:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python --version
python -m pip --version
```

The bootstrap task will replace ad-hoc installation commands with reproducible project scripts/lock configuration.

Do not install project packages globally merely to make a task pass.

## 9. Playwright environment

If Phase 1 Fonbet probe uses Playwright, install browser binaries through the pinned project environment/script rather than manually relying on whichever browser happens to exist.

A dedicated headful Chromium context is preferred for diagnostics.

Browser profile/session data lives under a gitignored runtime path. Do not point probes at the owner’s normal browser profile unless an explicit task requires it.

## 10. Environment/secrets

Expected pattern after bootstrap:

```text
.env                 # real values, never committed
.env.example         # variable names/placeholders, committed
```

Potential variables:

```text
TELEGRAM_BOT_TOKEN=
TELEGRAM_CHAT_ID=
SCANNER_CONFIG_PATH=
```

No Polymarket wallet/private key belongs in MVP configuration.

Third-party provider keys may be added only after the provider is architect-approved.

## 11. Running locally

Final script names are pinned during implementation, but the target operator experience is:

```powershell
.\scripts\setup_windows.ps1
.\scripts\run_tests.ps1
.\scripts\run_scanner.ps1
```

And explicit bounded probes such as:

```powershell
.\scripts\probe_polymarket.ps1
.\scripts\probe_fonbet.ps1
```

Probe scripts must not silently transition into indefinite daemons.

## 12. Local binding

Any local FastAPI diagnostics server binds to loopback by default, e.g. `127.0.0.1`. Do not expose it to all interfaces (`0.0.0.0`) in default scripts.

## 13. Windows sleep/power behavior

Long observation runs can be invalidated by system sleep.

Before a controlled research run:

- record Windows sleep/power state in the run checklist;
- ensure the machine will not sleep unexpectedly for the intended run;
- do not silently change global power settings in setup scripts;
- if the owner changes power behavior, record it as an operator action.

## 14. Clock/time synchronization

Accurate relative source timing is central to the research.

Before research runs:

- Windows time synchronization should be healthy;
- application startup logs local UTC wall time;
- source timestamps are never assumed trustworthy without comparison;
- process duration uses monotonic timing where applicable.

If significant clock skew is observed, mark the affected run degraded rather than hiding it.

## 15. Runtime data policy

Never commit:

```text
.venv/
.env
*.db
*.sqlite*
data/
logs/
browser_profiles/
playwright-report/
test-results/
__pycache__/
.pytest_cache/
.mypy_cache/
.ruff_cache/
coverage artifacts
network dumps/HAR unless deliberately sanitized as fixtures
```

A root `.gitignore` is part of repository bootstrap.

## 16. Log/data disk monitoring

The scanner must expose disk-pressure warnings. For long runs, verify free space before start and ensure bounded raw capture is enabled.

## 17. Graceful operator stop

Ctrl+C / shutdown should:

1. stop discovery;
2. stop accepting new source messages;
3. flush essential queued persistence;
4. close WebSockets/browser contexts;
5. finish run metadata;
6. exit without leaving a supposedly active research run.

Killing the process may be necessary during crashes, but recovery must mark the previous run incomplete.

## 18. Startup safety

On startup:

- configuration validates;
- DB migration state validates;
- no old quote is treated as fresh;
- source connections initialize independently;
- Telegram connectivity is not required to begin data acquisition unless policy explicitly says otherwise.

## 19. Optional Task Scheduler/service operation

Do not configure persistent Windows Task Scheduler/service startup during early probes. After the scanner passes soak tests, the architect may define a separate task for unattended operation.

## 20. Recovery checklist

If local/remote Git diverges or a task crashes:

- preserve unexpected local changes;
- run `git status` and `git log --oneline --decorate -n 20`;
- fetch remote;
- do not reset/force-push automatically;
- report exact divergence to the architect;
- preserve runtime evidence if it may explain a source defect.
