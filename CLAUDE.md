# CLAUDE.md - Repository Structure

## Repository: xinovate/codex

Fork of OpenAI's Codex CLI with China provider support. This is now an independent project — no longer tracking upstream.

## Branch Structure

- **master** - Main development branch. All work happens here.

## Release Process

Releases are triggered by pushing a tag on the `master` branch.

**完整发版步骤：**

```bash
# 1. 修改版本号（必须！否则 codex --version 显示旧版本）
#    文件: codex-rs/Cargo.toml 第 112 行
#    version = "0.1.3" -> version = "0.1.4"

# 2. 提交版本号变更
git add codex-rs/Cargo.toml
git commit -m "bump version to 0.1.4"

# 3. 打 tag 并推送
git tag v0.1.4
git push origin master
git push origin v0.1.4
```

GitHub Actions 自动构建以下平台：
- Linux x64 / arm64
- Windows x64
- macOS arm64 (Apple Silicon)

**版本号位置：** `codex-rs/Cargo.toml` 第 112 行 `version = "x.y.z"`
- 编译时写入二进制，影响 `codex --version` 输出
- `codex update` 命令用此版本与 GitHub releases 对比

**用户更新方式：** `codex update` — 自动检测平台，从 GitHub releases 下载最新版本并替换当前二进制

## Key Files Modified from Upstream

- `codex-rs/Cargo.toml` - Workspace version (line 112, affects `codex --version`)
- `codex-rs/codex-api/src/endpoint/chat_completions.rs` - Chat Completions API conversion (Responses API -> OpenAI Chat format)
- `codex-rs/model-provider/src/china_provider/mod.rs` - China provider runtime (User-Agent header, models manager)
- `codex-rs/model-provider-info/src/lib.rs` - China provider detection, WireApi::Chat variant
- `codex-rs/core/src/client.rs` - ChatCompletionsClient integration
- `codex-rs/core/src/session/mcp.rs` - MCP tool resolution with namespace fallback for Chat Completions path
- `codex-rs/core/src/session/turn.rs` - MCP image preprocessing (preprocess_images_with_mcp)
- `codex-rs/core/src/config/mod.rs` - image_analysis config propagation
- `codex-rs/config/src/config_toml.rs` - ImageAnalysisConfig struct
- `codex-rs/tui/src/updates.rs` - Update check URL (points to xinovate/codex GitHub releases)
- `codex-rs/tui/src/update_versions.rs` - Tag prefix parsing (uses `v*` instead of `rust-v*`)
- `codex-rs/tui/src/update_action.rs` - Standalone update points to our releases page
- `codex-rs/cli/src/main.rs` - `codex update` command: auto-downloads from GitHub releases
- `codex-rs/tui/tooltips.txt` - Removed OpenAI-specific tips
- `.github/workflows/release.yml` - macOS + multi-platform build targets
- `README.md` - Installation docs for China providers (Chinese)
- `codex-rs/CHINA_PROVIDER.md` - China provider setup guide (Chinese)

## Supported China Providers

| Provider | Base URL | Env Key |
|----------|----------|---------|
| DeepSeek | https://api.deepseek.com | DEEPSEEK_API_KEY |
| Volcengine | https://ark.cn-beijing.volces.com/api/coding/v3 | VOLCENGINE_API_KEY |
| Kimi Code | https://api.kimi.com/coding/v1 | KIMI_CODE_API_KEY |
| Xiaomi Mimo (API) | https://api.xiaomimimo.com/v1 | MIMO_API_KEY |
| Xiaomi Mimo (TokenPlan) | https://token-plan-cn.xiaomimimo.com/v1 | MIMO_TP_API_KEY |
| 智谱 GLM | https://open.bigmodel.cn/api/coding/paas/v4 | ZHIPU_API_KEY |

## CI

- `cargo-deny`, `Codespell`, `ci` (Prettier + ASCII check) - Run on all pushes
- `rust-ci-full`, `sdk`, `Bazel` - Upstream CI, may fail on fork (missing runners/secrets), ignore these
- `release` - Triggered by `v*.*.*` tags, builds release binaries for Linux/Windows/macOS

## Upstream Divergence (Updated 2026-05-29)

This project has diverged permanently from upstream (openai/codex). The `upstream` remote has been removed.

Core value: **Responses API → Chat Completions protocol translation layer** (`chat_completions.rs` + `ChatCompletionsClient`),
the only bridge for China AI models (DeepSeek, GLM, etc.) to work with Codex CLI.

## Build Rules

- **编译命令**：`cargo build --bin codex --manifest-path codex-rs/Cargo.toml`
- **开发迭代**：始终使用 debug 模式（不加 `--release`）
  - 增量编译 ~3 分钟（只重编改动的 crate）
  - Clean build（全量编译）约 40-50 分钟，**绝对不要清缓存**
  - Debug 二进制 ~500MB，Release ~175MB
  - debug 二进制路径：`codex-rs/target/debug/codex`（已在 PATH 中）
- **仅发布时** 使用 `cargo build --release --bin codex --manifest-path codex-rs/Cargo.toml`
- **禁止** `cargo clean`（会删除全部增量编译缓存，恢复需要 40+ 分钟）
- 可安全删除 `codex-rs/target/release/` 释放空间（~12GB），但不要动 `target/debug/`
