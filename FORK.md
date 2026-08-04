# CC Switch Fork：Codex 模型热切换（5.2 → DeepSeek V4 Flash）

本 fork 基于 CC Switch v3.19.1，核心改动是让 Codex 桌面端在同一会话内按模型自动选择供应商，无需重启、无需手动切换。

## 功能

- Codex 模型菜单选择 **5.2**（实际请求模型 `gpt-5.2` / `gpt-5.2-*`）→ 自动走 **DeepSeek**，上游模型强制为 `deepseek-v4-flash`。
- 选择 **5.5 / 5.6 Sol / 5.6 Terra / 5.6 Luna** 等 → 继续走 **OpenAI Official**（ChatGPT Plus / Codex 订阅额度）。
- 同一会话可在 DeepSeek 与 GPT 之间往返切换；CC Switch 主界面仍显示 OpenAI Official，不修改全局当前供应商。
- 界面菜单中仍显示 "5.2" 是预期行为，它只是 DeepSeek 的触发入口。

## 修改的文件

- `src-tauri/src/proxy/handler_context.rs`：Codex 请求按 `model` 临时选择供应商（`gpt-5.2*` → DeepSeek；其余 → `codex-official`，回退按名称 "OpenAI Official" 查找）。
- `src-tauri/src/proxy/forwarder.rs`：目标为 DeepSeek 时强制改写上游模型为 `deepseek-v4-flash`；从 DeepSeek 切回 OpenAI Official 时清理第三方明文 `reasoning` 项并移除 `previous_response_id`。

## 前置配置（CC Switch 内）

- OpenAI Official：ChatGPT Plus OAuth，地址 `https://chatgpt.com/codex`，供应商 ID `codex-official`。
- DeepSeek：DeepSeek API Key，地址 `https://api.deepseek.com`。
- 路由总开关开启、Codex 路由接管开启、统一 Codex 会话历史开启；本地代理 `127.0.0.1:15721`。

## 使用方式

- 菜单选 5.2 → DeepSeek V4 Flash；选 5.5/5.6 → OpenAI Official。
- 真实路由以 CC Switch 日志为准（请求目标指向 `api.deepseek.com`、`model=deepseek-v4-flash`）。

## 自动打包（fork 专属 CI）

- 新增 `.github/workflows/fork-build.yml`：**不需要任何签名密钥或 Apple 证书**。
- 推送 `v*` 标签（或在 Actions 页面手动触发）→ macOS 通用版构建 `CC Switch.app` + `CC Switch.dmg`，上传为 Artifact；打标签时自动创建 Release。
- 上游 `.github/workflows/release.yml` 已加 `if: github.repository == 'farion1231/cc-switch'`，只在上游仓库运行（它依赖上游私有 secrets，fork 内无法执行）。

## 本地开发

- `pnpm install`
- `pnpm dev`（开发模式；会占用 `127.0.0.1:15721`，需先退出已安装的正式版）
- `pnpm build`（本地发布版构建）

## 注意事项

- 当前为个人使用快速实现，触发模型（`gpt-5.2`）、供应商名称（`DeepSeek`）、上游模型（`deepseek-v4-flash`）为硬编码，如需通用化可后续改为可配置。
- 打包产物为 ad-hoc 签名（未使用 Apple Developer 证书），仅适合个人使用。
