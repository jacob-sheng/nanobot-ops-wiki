# Backup and Restore

正式恢复基线在私有仓：

- `jacob-sheng/nanobot-workspace-skills`

这页只讲恢复语义，不替代私有仓 README。

## snapshot 覆盖什么

- `skills/` 快照
- `profiles/hera/home_overlay/`
- `profiles/hera/root_overlay/`
- `profiles/hera/mem0/semantic_memories.json`
- `profiles/hera/secrets/`
- `manifests/repos.lock.yaml`
- `~/.nanobot/bilibili-auth/`
- `~/.nanobot/weixin-auth/`
- `~/.config/gws/`
- `~/.config/toot/`
- `~/.summarize/`
- 本地运维脚本与 systemd unit
- workspace cron store：`~/.nanobot/workspace/cron/jobs.json`
- workspace 服务状态，例如：
  - `~/.nanobot/workspace/services/mastodon-daily-share/state.json`
  - `~/.nanobot/workspace/services/bilibili-daily-share/state.json`

## 不覆盖什么

- `~/.nanobot/bridge/` 运行产物
- append 式本地日志
- 临时缓存和可重建 node_modules / pycache
- `~/.nanobot/bilibili-auth/qr.png`
- `~/.nanobot/workspace/services/bilibili-daily-share/last_prepare.json`
- 旧的 `~/.nanobot/cron/` legacy 路径

## 恢复语义

- restore 是 commit-based，不是 branch-based
- `/opt/ai-stack` 下的源码仓会按锁定 commit 恢复
- Weixin bridge 会从恢复后的 `nanobot` 源码本地重建
- `weixin-auth`、`bilibili-auth`、`gws`、`toot` 这些 auth state 会恢复，但如果上游 token 失效，仍可能要求重新授权
- Telegram 当前按直连恢复，不默认启用 `xray-client-telegram.service`
- `xray-client-telegram.service` 和 `/usr/local/etc/xray/client-telegram.json` 仅保留作回滚材料；若未来明确恢复代理路径，再重新安装 `/usr/local/bin/xray` 并启用该服务
- `MEMORY.md` 不是主长期记忆来源；长期记忆以导出的 Mem0 为主
- 当前两条随机社交分享任务会随着 workspace overlay 一起恢复：
  - `mastodon_daily_share`
  - `bilibili_daily_share`
  - cron store 在 `workspace/cron/jobs.json`
  - Mastodon 去重状态在 `services/mastodon-daily-share/state.json`
  - Bilibili 去重状态在 `services/bilibili-daily-share/state.json`
  - helper skills 来自 `skills/mastodon-daily-share/` 和 `skills/bilibili-daily-share/`
  - Bilibili 专用 `~/.nanobot/venvs/bilibili-cli` 不备份，restore 时按 `skills/bilibili-daily-share/requirements-bilibili-cli.lock.txt` 重建，并固定使用 `httpx` backend

## 当前恢复入口

干净机器上的主入口是私有仓 README：

- `nanobot-workspace-skills/README.md`

以及：

- `scripts/bootstrap_restore.sh`

## 运维注意

- 当前 GitHub snapshot 是唯一正式灾备链
- 本地 rsync 备份链已退役
- `daily-digest` 恢复后默认是 `06:30`，Telegram + Weixin 双发
- `mastodon_daily_share` / `bilibili_daily_share` 恢复后仍是 Telegram 主送达 + Weixin 镜像，且 `sendProgress=false`
