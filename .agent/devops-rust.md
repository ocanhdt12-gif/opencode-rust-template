# DevOps — Rust stack (Axum + Leptos)

Bổ sung cho `.agent/devops.md` khi project con là **Rust** (`package_manager: cargo`). Thay các lệnh
`npm ci/build` bằng cargo, giữ nguyên thứ tự: `git init` → remote → **generate CI files** → commit → push.

## Phase 1.5: Generate CI/CD (Rust)

Sinh `.github/workflows/ci.yml` cho project con:

```yaml
name: CI
on:
  push: { branches: [main, develop] }
  pull_request: { branches: [main] }
jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
        with: { components: clippy, rustfmt }
      - uses: Swatinem/rust-cache@v2
      - run: cargo fmt --all --check
      - run: cargo clippy --workspace --all-targets -- -D warnings
      - run: cargo build --workspace --release
      - run: cargo test --workspace
      # 📊 Monitoring gate — bắt buộc có health endpoint khi project có web
      - name: Verify health endpoint
        run: grep -rqE '/health' src crates --include='*.rs' || { echo "⚠️ thiếu GET /health — xem skills/monitoring/rust-monitoring.md"; exit 1; }
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: dtolnay/rust-toolchain@stable
      # 🔒 Security gates (chặn merge nếu fail)
      - uses: rustsec/audit-check@v2
        with: { token: "${{ secrets.GITHUB_TOKEN }}" }
      - uses: EmbarkStudios/cargo-deny-action@v2
        with: { command: check }
```

Copy `deny.toml` từ template vào repo con (license/supply-chain gate).

## Deploy (Rust)

### Docker (DEPLOY_PLATFORM=docker / vps-docker)

Multi-stage Dockerfile (build nhỏ, chạy distroless):

```dockerfile
FROM rust:1-slim AS build
WORKDIR /app
COPY . .
RUN cargo build --release --bin <server-bin>
# Leptos: thêm `cargo leptos build --release` nếu dùng cargo-leptos

FROM gcr.io/distroless/cc-debian12
COPY --from=build /app/target/release/<server-bin> /app/server
# COPY --from=build /app/target/site /app/site     # Leptos static assets
EXPOSE 8080
ENV RUST_LOG=info
CMD ["/app/server"]
```

- Health check deploy trỏ `GET /health` (xem rust-monitoring.md).
- Auto-rollback khi health check fail (giữ nguyên nguyên tắc từ devops.md).

### Binary / systemd (VPS không Docker)

- `cargo build --release` → copy `target/release/<bin>` lên VPS → systemd unit + env từ `.env.local` (không commit secret).

## Khác biệt so với Node

| Node | Rust |
|------|------|
| `npm ci` | `cargo fetch` |
| `npm run build` | `cargo build --release` (+ `cargo leptos build` nếu có FE) |
| `npm test` | `cargo test --workspace` |
| `npm audit` | `cargo audit` |
| (eslint/semgrep) | `cargo clippy -- -D warnings` + `cargo deny` |

Security chi tiết: `skills/security/rust-security.md` · Monitoring: `skills/monitoring/rust-monitoring.md`.
