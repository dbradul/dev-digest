# `e2e/` — specs = executable flows

In this module a spec **is** a flow: `NN-name.flow.json`, a JSON list of agent-browser
commands that `../run.ts` executes in order (it picks up only `*.flow.json`, so this README
is ignored). Flow format, locator rules and env knobs: `../README.md`. Markdown design
notes about e2e coverage go to `../docs/`.

- New flow → next free `NN` prefix, then add it to the coverage table in `../README.md`.
- Deterministic locators only (`--url`, `--text`, `find`); never the AI `chat` command.
