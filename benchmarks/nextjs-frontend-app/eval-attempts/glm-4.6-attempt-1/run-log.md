# Run Log

## Benchmark
- name: nextjs-frontend-app
- display_name: Next.js Frontend App
- state_mode: greenfield
- baseline_source: n/a (greenfield)

## Model
- model_id: glm-4.6
- model_display: GLM-4.6
- resolved_model: openrouter/z-ai/glm-4.6
- attempt: 1
- run_id: 20260506-184243

## Harness
- harness: opencode
- harness_version: 1.14.28
- mode: lite
- image: (declared: none — runner.yaml has no image; lite-only benchmark)
- image_digest: not-pulled (lite mode)

## Host runtime versions
- node: v22.17.0
- npm: 10.9.2
- platform: darwin (Darwin 25.3.0)

## Workspace
- temp_workspace: /tmp/10x-eval-lite/20260506-184243/glm-4.6-attempt-1
- visible_inputs:
  - prompt.md
  - context.md

## Execution
- start: 2026-05-06T16:43:02Z
- end: 2026-05-06T16:46:25Z
- duration: ~3min 23s
- exit_status: 0
- timeout_flag: false

## Isolation
- isolation_deviations: lite mode, no container isolation; harness ran on host

## Logs
- log_limitations: agent-run.log captured opencode stdout/stderr in full; ANSI color escapes preserved.
