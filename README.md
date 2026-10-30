# opencode-rust-template

A Rust edition of the opencode project template. It keeps the **entire AI
orchestration flow** of `opencode-project-template` unchanged and swaps the
machine tooling + default project stack from Node/React to **Rust**.

> Status: **Pha 1 (scaffold) complete.** Pha 2–4 pending — see [Roadmap](#roadmap).

## Two layers (important)

The template is made of two separate layers. "Switching to Rust" only touches the
second one.

| Layer | What it is | Rust change? |
|-------|-----------|--------------|
| **A — AI orchestration** | `AGENTS.md`, `.opencode/agent/*.md`, `.opencode/command/*.md`, `.agent/*.md`, `skills/`, `spec/`, `opencode.jsonc`. Pure Markdown/config that routes intents and drives agents. | **No.** Copied verbatim in Pha 4. The flow `/start → brainstorm → design → graph → loop`, human checkpoints and worktree-per-task are unchanged. |
| **B — Machine tooling** | The scripts that actually run on the machine. Previously Node `.mjs`. | **Yes.** Ported to a single Rust binary `opencode-tools`. |

## The `opencode-tools` binary

One multi-command binary replaces the legacy scripts. Each subcommand keeps the
**same CLI contract** (flags, stdout, generated files) as the script it replaces,
so Layer A can call it without noticing the change.

| `opencode-tools <cmd>` | Legacy script | Role |
|------------------------|---------------|------|
| `dispatch` | `scripts/parallel-dispatch.mjs` | Parallel layer tasks via git worktree-per-task + auto-merge (`plan`/`setup`/`merge`/`cleanup`) |
| `sync` | `scripts/template-sync.mjs` | Pull managed paths from the canonical template (`--project`, `--dry-run`, `--prune`, `--template`, `--no-config`) |
| `detect` | `scripts/detect-profile.mjs` | Detect project stack/profile → JSON (or `--md`) for `/brainstorm` |
| `inventory` | `scripts/generate-inventory.mjs` | Write `docs/generated/inventory.md` |
| `perms` | `scripts/apply-verify-permissions.mjs` | Sync reviewer allow-rules with verify commands (dry-run by default; `--write`) |
| `live` | `.claude/hooks/live-studio.mjs` | vibe-studio enter/leave hook |

## Build

```sh
# install Rust if needed: https://rustup.rs
cargo build --release
./target/release/opencode-tools --help
```

## Key decisions

1. **vibe-studio stays TypeScript** (browser UI; not worth forcing to Rust in Pha 1).
2. **Default web stack for generated projects = full-stack Rust**: **Axum** (backend) + **Leptos** (WASM frontend).
3. **Tooling packaging = one multi-command binary** `opencode-tools <subcommand>`.
4. **Git = shell-out** to the `git` binary (no libgit2), matching legacy behaviour.
5. Template name: **opencode-rust-template**. Git repo to be created later.

## Roadmap

- [x] **Pha 1** — Cargo workspace, `core` lib skeleton, `opencode-tools` with 6 subcommands (CLI contract only), CI, docs.
- [ ] **Pha 2** — Port the 6 tools for real, each with golden tests comparing output against the legacy `.mjs`.
- [x] **Pha 3** — Rust-ify the generated-project stack: `detect` recognises `Cargo.toml`; default verify = `cargo build/test/clippy/fmt`; `skills/rust-guidelines/`; `.context/project-config.rust.md` Rust group.
- [ ] **Pha 4** — Copy Layer A (AGENTS/agents/commands/skills/spec), rewire tool invocations to the Rust binary, smoke-test `/start` end-to-end.
- [ ] **Pha 5 (optional)** — Port vibe-studio sprite scripts to Rust (`image` crate).

## Layout

```
opencode-rust-template/
├── Cargo.toml                 # workspace
├── crates/
│   ├── core/                  # shared lib: config, git (shell-out), fsx, log
│   └── tools/                 # bin: opencode-tools (6 subcommands)
├── .github/workflows/ci.yml   # fmt + clippy -D warnings + build + test
├── docs/ARCHITECTURE.md
└── (Pha 4) AGENTS.md, .opencode/, .agent/, skills/, spec/, tools/vibe-studio/
```
