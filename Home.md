# AI 运维 Wiki

这套文档记录 `/home/Hera` 这台机器当前的 AI 运行面、恢复入口和日常运维做法。它面向接手机器的人，不面向开发演示，也不保存任何 secret 真值。

## 当前现网概览

- 主程序：`nanobot`
- 主源码仓：`/opt/ai-stack/nanobot`
- 备份/恢复控制仓：`/home/Hera/nanobot-workspace-skills`
- 公开运维文档仓：`/home/Hera/nanobot-ops-wiki`
- 默认聊天模型：`ollama/kimi-k2.5`
- 主 provider：`custom`，实际走 AxonHub
- Telegram：走 `telegram_planbridge`，不是内置 Telegram channel
- Weixin：已启用，走本地 bridge + `nanobot-weixin-bridge.service`
- 每日天气：`06:30`，Telegram + Weixin 双发
- Mastodon / Bilibili 随机分享：每天 `08:00-20:00` 之间各自随机一次，Telegram 主发 + Weixin 镜像，静默执行
- 长期记忆：已启用 semantic memory（Mem0 / NIM embeddings）
- Node 运行时：默认 `/usr/local/bin/node`，版本线为 Node 22
- Telegram 当前已验证可裸连，不再依赖本地 Xray SOCKS 代理

## 先看哪里

- 新接手先看：[Quick Start](Quick-Start.md)
- 想知道服务和仓库关系，看：[Service Map](Service-Map.md)
- 想知道配置、密钥和 auth state 在哪里，看：[Config and Secrets](Config-and-Secrets.md)
- 想知道各接入渠道和外部能力怎么接进来，看：[Channels and Integrations](Channels-and-Integrations.md)
- 想恢复机器或确认快照边界，看：[Backup and Restore](Backup-and-Restore.md)
- 想升级、切模型、换 Node 或回滚，看：[Upgrade and Rollback](Upgrade-and-Rollback.md)
- 想单独排查微信，看：[Weixin Runbook](Weixin-Runbook.md)

## 当前关键服务

- `nanobot-gateway.service`
- `nanobot-weixin-bridge.service`
- `codex-listener.service`
- `daily-digest.service` / `daily-digest.timer`
- `hera-skills-backup.service` / `hera-skills-backup.timer`

## 当前维护边界

- 文档写运行方式、路径、恢复语义和排障入口，不写任何 key/token 真值。
- Weixin 仍然是本地 bridge 分叉架构，不是上游 direct channel。
- `~/.nanobot/bridge` 是本地重建的运行产物，不作为正式备份对象。
- 正式恢复基线仍然是私有仓 `jacob-sheng/nanobot-workspace-skills`。
