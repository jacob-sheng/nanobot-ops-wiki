# Quick Start

这页面向“刚接手这台机器的人”。目标是 5 分钟内确认系统是否活着、知道去哪里看日志、知道最小恢复动作是什么。

## 1. 先确认关键服务

```bash
systemctl is-active \
  nanobot-gateway.service \
  nanobot-weixin-bridge.service \
  codex-listener.service \
  daily-digest.timer \
  hera-skills-backup.timer
```

正常预期：

- `nanobot-gateway.service`: `active`
- `nanobot-weixin-bridge.service`: `active`
- `codex-listener.service`: `active`
- `daily-digest.timer`: `active`
- `hera-skills-backup.timer`: `active`

## 2. 再看最关键的日志

```bash
journalctl -u nanobot-gateway.service -n 100 --no-pager
journalctl -u nanobot-weixin-bridge.service -n 100 --no-pager
journalctl -u codex-listener.service -n 100 --no-pager
journalctl -u daily-digest.service -n 50 --no-pager
```

重点关注：

- gateway 日志里有没有 `Connected to Weixin bridge`
- Weixin bridge 有没有持续报 `monitor_error` 或 `send_*_failed`
- daily digest 最近一次是否有 Telegram 和 Weixin 发送结果

## 3. 常用重启动作

```bash
sudo systemctl restart nanobot-weixin-bridge.service
sudo systemctl restart nanobot-gateway.service
sudo systemctl restart codex-listener.service
sudo systemctl restart daily-digest.timer
```

如果只是天气推送要补发一次：

```bash
sudo -u Hera -H /opt/ai-stack/nanobot/.venv/bin/python \
  /home/Hera/.nanobot/workspace/services/daily-digest/daily_digest.py
```

## 4. 关键路径

```text
/opt/ai-stack/nanobot
/opt/ai-stack/codex-listener
/opt/ai-stack/nanobot-telegram-planbridge
/home/Hera/.nanobot/config.json
/home/Hera/.nanobot/workspace
/home/Hera/.nanobot/bilibili-auth
/home/Hera/.nanobot/weixin-auth
/home/Hera/.config/gws
/home/Hera/.summarize
/home/Hera/nanobot-workspace-skills
```

补充：

- Weixin 对话历史在 `~/.nanobot/workspace/sessions/`
- Weixin 登录态和 bridge 路由状态在 `~/.nanobot/weixin-auth/`
- 语义记忆导出在备份仓库 `profiles/hera/mem0/semantic_memories.json`

## 5. 当前接入现状

- 默认聊天模型：`ollama/kimi-k2.5`
- 主 provider：AxonHub custom
- Telegram 当前走 `telegram_planbridge`
- Telegram 当前已验证裸连可用，不再依赖 `xray-client-telegram.service`
- Weixin 当前已启用，本地 bridge 已对齐腾讯官方图片入站协议
- `daily-digest` 当前 `06:30` 发送，Telegram + Weixin 双发
- `gws` 已接入 Gmail / Drive / Calendar
- `summarize` 已接入 AxonHub 的 OpenAI-compatible 路径
- `mastodon` 已接入，支持读时间线、关注、点赞、收藏
- 当前还有两条随机社交分享任务：
  - `mastodon_daily_share`
  - `bilibili_daily_share`
  - 两者都在 `08:00-20:00 Asia/Shanghai` 内随机一次
  - 两者都主发 Telegram，并镜像到 Weixin
  - 两者都 `sendProgress=false`，所以只会看到最终分享，不会看到执行过程
  - Bilibili 依赖登录态首页推荐；登录失效时只发 Telegram 登录提醒
  - Bilibili helper venv 按 `skills/bilibili-daily-share/requirements-bilibili-cli.lock.txt` 重建，并固定要求 `httpx`

## 6. 恢复入口

正式恢复基线在：

- 仓库：`jacob-sheng/nanobot-workspace-skills`
- 本地路径：`/home/Hera/nanobot-workspace-skills`

从干净机器恢复时，优先看主仓 README：

- `/home/Hera/nanobot-workspace-skills/README.md`

## 7. 不要做什么

- 不要把 secrets 真值写进 wiki。
- 不要直接编辑 `~/.nanobot/bridge` 里的运行产物；应从 `nanobot` 源码重建。
- 不要把 provider 切回 `auto` 配合 `ollama/...` 模型名使用。
- 不要假设 Telegram 用的是内置 `telegram` channel；当前实际在跑的是 `telegram_planbridge`。
- 不要把 Weixin 实现误认为上游 direct channel；当前是本地 bridge 分叉方案。
