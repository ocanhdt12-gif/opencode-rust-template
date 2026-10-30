# project-config.md — Cấu hình dự án Rust (canonical mẫu)

> Mẫu cho project con sinh từ opencode-rust-template (stack Rust).
> `/brainstorm` ghi file thật `.context/project-config.md` — KHÔNG điền tay.
> `opencode-tools detect` đã nhận Cargo.toml và gợi ý sẵn các giá trị dưới.

## Config

```yaml
# ── Git ──
default_branch: main
integration_prefix: layer   # layer-N-integration (initial build)

# ── Package / source ──
package_manager: cargo
source_roots: [src]         # hoặc [crates/core, crates/server, crates/web] nếu workspace nhiều crate

# ── Rust ──
rust_edition: "2021"
rust_toolchain: stable      # stable | nightly (chỉ nightly khi thật cần)
workspace: true             # Cargo workspace (khuyến nghị kể cả 1 crate)
web_backend: axum           # backend web mặc định
web_frontend: leptos        # frontend WASM mặc định (SSR + hydrate)
async_runtime: tokio

# ── Verify commands (null/placeholder = skip) ──
install_command: cargo fetch
build_command: cargo build --workspace --release
test_command: cargo test --workspace
lint_command: cargo clippy --workspace --all-targets -- -D warnings
typecheck_command: cargo check --workspace
# fmt là gate riêng, luôn chạy:
fmt_command: cargo fmt --all --check
web_typecheck_command: null
web_lint_command: null
api_typecheck_command: null
api_lint_command: null
migration_command: null

# ── Database ──
db_tool: none               # none | sqlx | diesel | sea-orm
migration_required: false

# ── Worktree hooks (optional — parallel loop) ──
# Có env: $WORKTREE_PATH, $TASK_ID, $WORKTREE_SLUG. cwd = worktree. Rỗng = bỏ qua.
worktree_setup_cmd: null
worktree_teardown_cmd: null
```

## Check commands (chạy trước khi báo xong)

Thứ tự chuẩn cho Rust:
1. `cargo fmt --all --check`
2. `cargo clippy --workspace --all-targets -- -D warnings`
3. `cargo build --workspace --release`
4. `cargo test --workspace`

> Nếu command là `null`/placeholder hoặc chưa có code (`source_roots: []`) → ghi `skip, no app configured`.

## UI rules (nếu project có UI — Leptos)

- SSR + hydrate cho trang cần SEO/tải nhanh.
- Component thuần + signal cho state; server function `#[server]` thay vì tự fetch.
- DTO chung giữa Axum/Leptos đặt ở crate `core`, dùng `serde`.

## Notes

- Verify theo skill `skills/rust-guidelines/SKILL.md`.
- `opencode-tools perms` sẽ auto-allow mấy lệnh cargo an toàn (fmt/clippy/test/build/check) vào reviewer.
