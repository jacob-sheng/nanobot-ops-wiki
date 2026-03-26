# Config and Secrets

本页只记录路径、用途、权限和恢复语义，不记录任何 secret 真值。

## 主要配置文件

- `~/.nanobot/config.json`
  - 主聊天配置
  - 包括 provider、默认模型、channels、memory 等
- `~/.nanobot/workspace/cron/jobs.json`
  - workspace 内的 cron store
- `~/.summarize/config.json`
  - summarize CLI 默认模型和网关配置

## 主要 auth / state 目录

- `~/.nanobot/weixin-auth/`
  - 微信登录态、sync 游标、bridge 路由状态
  - `accounts/*.json` 里除 token/account 外还包含持久化的 `contextTokens`
- `~/.config/gws/`
  - Google Workspace CLI 认证材料
- `~/.config/toot/`
  - Mastodon CLI 认证状态

## 主要 secret 文件

- `~/axonhub.key`
  - 当前 AxonHub API key
- `~/NIM.key`
  - NVIDIA NIM embeddings key
- `/etc/default/nanobot`
  - 运行时环境变量入口，实际让 systemd 服务拿到 provider key 等配置

## 运行时与数据路径

- `~/.nanobot/workspace/`
  - prompts、skills、sessions、daily digest 服务脚本
- `~/.nanobot/workspace/sessions/`
  - 各聊天会话历史 JSONL
- `~/.nanobot/media/`
  - 入站媒体缓存
- `~/.nanobot/bridge/`
  - 本地重建的 Weixin bridge 运行目录
- `profiles/hera/mem0/semantic_memories.json`
  - 私有备份仓里的 Mem0 导出

## 权限与恢复语义

- `~/.config/gws/`、`~/.summarize/`、`~/.nanobot/weixin-auth/` 这类目录应保持最小权限。
- `~/.nanobot/bridge/` 不是正式备份对象；恢复时从 `nanobot` 源码本地重建。
- `weixin-auth`、`gws`、`toot` 这类目录保存的是 auth state，恢复后仍可能因为上游 token 失效而要求重新授权。
- 正式恢复基线在私有仓 `jacob-sheng/nanobot-workspace-skills`，不是这个公开文档仓。
