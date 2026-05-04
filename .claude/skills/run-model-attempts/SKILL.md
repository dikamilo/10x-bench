---
name: run-model-attempts
description: Run isolated one-shot model attempts for a programming evaluation case. Use when the user wants to execute multiple AI coding models or agents against the same benchmark prompt while preventing access to scorecards, previous results, or other attempts. Triggers on phrases like "run model attempts", "uruchom proby modeli", "odpal modele na benchmarku", "run eval attempts", "one-shot attempts", "isolated model runs", or requests to compare several models on the same programming task.
---

# run-model-attempts

Runs one-shot implementation attempts for a benchmark case while preserving isolation and comparability. This skill only runs attempts. It does not score, fix, or iterate on results.

## Inputs

Required:

- `benchmark/prompt.md`
- `benchmark/context.md` if present
- `benchmark/baseline-manifest.md` if present
- model or harness list

Optional:

- `benchmark/bootstrap.md` if intentionally model-visible
- number of attempts per model
- starter scaffold path
- output run ID

Never use `benchmark/scorecard.md` as an input for model attempts.

## Isolation Contract

The Docker container is the isolation unit. The entire harness session — every file write, shell command, build, and test the agent performs — runs inside the container. The container sees only `/workspace`. It cannot navigate to the host repo, other attempts, or the scorecard because those paths do not exist inside it.

```
[host: benchmark repo]
       │
       │  copy prompt.md + context.md only
       ▼
[Docker container]
  ├── /workspace/          ← only path the agent can see
  │     ├── prompt.md
  │     └── context.md
  ├── harness binary       ← installed at container start
  └── language runtime     ← from base image
       │
       │  agent runs, writes code, builds, tests — all inside
       ▼
  /workspace/ contains the final implementation
       │
       │  copy out after container exits
       ▼
[eval-attempts/{model-id}-attempt-{n}/]
```

### Standard execution pattern

```bash
WORKSPACE=$(mktemp -d /private/tmp/10x-eval-runs/XXXXXX)

# Copy only allowed benchmark files
cp benchmark/prompt.md "$WORKSPACE/"
cp benchmark/context.md "$WORKSPACE/" 2>/dev/null || true
# copy bootstrap.md only if intentionally model-visible

# Run the entire harness session inside the container
docker run --rm \
  --name "attempt-{model-id}-{n}" \
  -v "$WORKSPACE":/workspace \
  -w /workspace \
  -e OPENROUTER_API_KEY="$OPENROUTER_API_KEY" \
  --network bridge \
  <base-image> \
  bash -c '<install-harness> && <run-harness-with-prompt>' \
  2>&1 | tee "$WORKSPACE/agent-run.log"

# Copy result to repo
cp -r "$WORKSPACE/." "eval-attempts/{model-id}-attempt-{n}/"
```

`docker logs` / tee captures the full session as `agent-run.log` automatically.

### Per-harness container commands

#### OpenCode via OpenRouter

Base image: `golang:1.23-bookworm` (includes Go runtime the agent will need for Go benchmarks; adjust for other stacks)

opencode stores credentials in `~/.local/share/opencode/auth.json` (XDG standard on Linux). Mount it read-only rather than passing the raw key as an environment variable.

```bash
docker run --rm \
  -v "$WORKSPACE":/workspace \
  -v "$HOME/.local/share/opencode/auth.json":/root/.local/share/opencode/auth.json:ro \
  -w /workspace \
  --network bridge \
  golang:1.23-bookworm \
  bash -c '
    curl -fsSL https://opencode.ai/install | bash
    export PATH="$HOME/.opencode/bin:$PATH"
    opencode run \
      -m openrouter/<provider>/<model> \
      --dangerously-skip-permissions \
      "$(cat /workspace/prompt.md)"
  ' 2>&1 | tee "$WORKSPACE/agent-run.log"
```

Notes:
- Pipe the installer to `bash`, not `sh` — the install script uses bash-specific features (`set -o pipefail`) that fail silently under `sh`.
- The `auth.json` mount is read-only (`:ro`) so the container cannot modify host credentials.
- Do not pass `OPENROUTER_API_KEY` as a plain env var; the mounted auth file is the canonical credential source for opencode.

Common OpenRouter provider prefixes:
- Moonshot (Kimi): `openrouter/moonshotai/<model>`
- Z.AI (GLM): `openrouter/z-ai/<model>`
- MiniMax: `openrouter/minimax/<model>`
- Mistral (Devstral): `openrouter/mistral/<model>`
- Alibaba (Qwen): `openrouter/alibaba/<model>`
- xAI (Grok): `openrouter/xai/<model>`

If unsure of the provider prefix, search OpenRouter for the model before running.

#### Claude Code CLI

```bash
docker run --rm \
  -v "$WORKSPACE":/workspace \
  -w /workspace \
  -e ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" \
  --network bridge \
  node:22-bookworm \
  bash -c '
    npm install -g @anthropic-ai/claude-code
    claude --dangerously-skip-permissions \
      -p "$(cat /workspace/prompt.md)"
  ' 2>&1 | tee "$WORKSPACE/agent-run.log"
```

#### Manual harnesses (Cursor, Claude Desktop, Codex Desktop)

Docker is not applicable. The harness runs on the host inside the operator's IDE or desktop app.

Minimum isolation for manual harnesses:
1. Create a temp workspace outside the repo: `mktemp -d /private/tmp/10x-eval-runs/XXXXXX`
2. Copy only allowed benchmark files there
3. Open that directory in the IDE/app — do not open the benchmark repo itself
4. After the run, copy the workspace to `eval-attempts/{model-id}-attempt-{n}/`
5. Record `isolation_deviations: manual harness, container isolation not available` in `run-log.md`

The operator is responsible for ensuring the agent cannot see other attempts or the scorecard during a manual run.

### Base image selection

Choose the base image that matches the benchmark stack so the agent has the right runtime available:

| Stack | Base image |
|-------|------------|
| Go | `golang:1.23-bookworm` |
| Node.js | `node:22-bookworm` |
| Python | `python:3.13-bookworm` |
| Rust | `rust:1.78-bookworm` |
| Multi-language / unknown | `ubuntu:24.04` |

Use `bookworm` (Debian) variants over Alpine when the harness installer requires `bash`, `curl`, or standard libc. Switch to Alpine only when confirmed compatible.

### Preflight check

Before running any attempt, verify Docker is available:

```bash
docker info > /dev/null 2>&1 || { echo "Docker unavailable — cannot run isolated attempts"; exit 1; }
```

If Docker is unavailable, stop and report to the user. Do not silently fall back to running the harness directly on the host. If the user explicitly accepts the risk, document the deviation and proceed with Layer 1 (temp dir) isolation only.

### Forbidden inputs inside the container

The container must not receive:

- `benchmark/scorecard.md`
- `eval-results/`
- other attempt directories
- evaluator notes
- hidden tests unless the benchmark explicitly defines them as model-visible

Do not mount the benchmark repo or any parent directory. Mount only `$WORKSPACE`.

## Workflow

### 1. Preflight

Read the benchmark inputs and confirm:

- `prompt.md` exists
- `scorecard.md` is not included in the model-visible input set
- state mode is known:
  - `greenfield`: workspace starts empty except for allowed benchmark files
  - `brownfield`: workspace starts with a fresh copy of the baseline from `baseline-manifest.md`
- model/harness list is known
- attempt count is known, defaulting to 3 per model if not specified
- Docker is available
- output directories will not overwrite existing attempts without explicit user approval

If `scorecard.md` is referenced inside `prompt.md`, `context.md`, or `bootstrap.md`, stop and ask the user to fix the benchmark package before running attempts.

### 2. Create Isolated Workspaces

For each `{model-id}-attempt-{n}`:

1. Create an empty temp workspace: `mktemp -d /private/tmp/10x-eval-runs/XXXXXX`
2. For greenfield, leave it empty.
3. For brownfield, copy a fresh baseline from `baseline-manifest.md`.
4. Copy only allowlisted benchmark files into the temp workspace.
5. Copy `bootstrap.md` only if it is intentionally model-visible.
6. Record workspace path and start timestamp.

### 3. Run Attempts

For each model and attempt:

- Start a Docker container from the appropriate base image
- Install the harness inside the container at startup (ad hoc download)
- Mount only `$WORKSPACE` as `/workspace`
- Pass the prompt as a single string argument to the harness
- Capture stdout + stderr with `tee $WORKSPACE/agent-run.log`
- Run one-shot with no iterative feedback
- Do not pass scorecard or evaluator notes

Run independent attempts in parallel when resource limits allow.

### 4. Collect Artifacts

After each container exits:

1. Copy the workspace to:

```text
eval-attempts/{model-id}-attempt-{n}/
```

`agent-run.log` is already inside the workspace and will be copied with it.

2. Add `run-log.md` with:
   - model ID and display name
   - harness and container image used
   - attempt number and run ID
   - temp workspace path
   - start and end timestamp
   - container exit status
   - visible benchmark inputs copied
   - state mode and baseline source, if any
   - `isolation_deviations`: any cases where container isolation was not applied and why
   - `log_limitations`: note if `agent-run.log` is incomplete or missing

Do not include scorecard contents in `run-log.md`.

### 5. Finish

Report:

- completed attempts and their output directories
- failed attempts and exit codes
- any isolation deviations
- next recommended step: use `score-eval-attempt` on the generated attempts

## Guardrails

- Do not pass `scorecard.md` to implementation models.
- Do not mount the benchmark repo or any path above `$WORKSPACE` into the container.
- Do not let one attempt's workspace be visible to another container.
- Do not pass `bootstrap.md` when bootstrapping is an evaluated part of a greenfield task.
- Do not reuse a mutated baseline between models; every brownfield attempt gets a fresh workspace copy.
- Do not score or critique attempts during generation.
- Do not overwrite existing attempt directories without explicit approval.
- Do not run the harness directly on the host when Docker is available.
- Do not start attempts if Docker is unavailable without explicit user approval.
- Do not use git worktrees as an isolation substitute — they share the host filesystem and do not prevent the agent from navigating to other paths.
