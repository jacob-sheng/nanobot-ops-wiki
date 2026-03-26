# AI 运维 Wiki

这套 wiki 用来记录 `/home/Hera` 这台机器当前的 AI 运行面、恢复入口和日常运维做法。

## 当前现网概览

- 主程序：`nanobot`
- 主仓库：`/opt/ai-stack/nanobot`
- 备份仓库：`/home/Hera/nanobot-workspace-skills`
- 默认聊天模型：`ollama/kimi-k2.5`
- 主 provider：`custom`，实际走 AxonHub
- Telegram：走 `telegram_planbridge`，不是内置 Telegram channel
- Weixin：已启用，走本地 bridge + `nanobot-weixin-bridge.service`
- 每日天气：`06:30`，Telegram + Weixin 双发
- 语义记忆：已启用，Mem0 导出保存在备份仓库
- Node 运行时：默认 `/usr/local/bin/node`，版本线为 Node 22

## 关键服务

- `nanobot-gateway.service`
- `nanobot-weixin-bridge.service`
- `codex-listener.service`
- `daily-digest.timer`
- `hera-skills-backup.timer`
- `nanobot-update-backups-prune.timer`
- `xray-client-telegram.service`

## 关键仓库

- `/opt/ai-stack/nanobot`
- `/opt/ai-stack/codex-listener`
- `/opt/ai-stack/nanobot-telegram-planbridge`
- `/home/Hera/nanobot-workspace-skills`

## 文档目录

- [[Quick-Start]]
- [[Service-Map]]
- [[Config-and-Secrets]]
- [[Channels-and-Integrations]]
- [[Backup-and-Restore]]
- [[Upgrade-and-Rollback]]
- [[Weixin-Runbook]]

## 当前维护边界

- 这里写路径、职责、运行方式和恢复语义，不写任何 secrets 真值。
- Weixin 仍然是本地 bridge 架构，不是上游 direct channel。
- `~/.nanobot/bridge` 是运行产物，不作为正式备份对象。
- GitHub 私有备份仓库是当前正式恢复基线。
