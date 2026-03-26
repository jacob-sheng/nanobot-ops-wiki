# Upgrade and Rollback

这页收口高风险改动的检查点和回滚锚点。

## nanobot 上游更新

- 主仓：`/opt/ai-stack/nanobot`
- 更新时优先保住本地定制：
  - AxonHub custom provider 路径
  - Weixin 本地 bridge
  - semantic memory / Mem0
  - daily digest 双发
- 合并上游后优先重查：
  - `LOCAL_PATCHES.md`
  - `docs/weixin-bridge-maintenance.md`
  - 当前 workspace prompt / skills 路由是否仍匹配

## Node 22

- 当前默认 `node` 来自 `/usr/local/bin/node`
- Debian 自带 `/usr/bin/node` 仍保留作回滚锚点
- 已有回滚脚本：
  - `~/.nanobot/bin/rollback_node22.sh`
- 任何涉及 bridge 或 summarize 的 Node 变更，都要先确认：
  - bridge 可重新 build
  - `summarize` 可正常运行

## provider 切换与回退

- 当前默认 provider：AxonHub custom
- 当前默认模型：`ollama/kimi-k2.5`
- 本地已有切换脚本：
  - `~/.nanobot/bin/switch_to_axonhub.sh`
  - `~/.nanobot/bin/rollback_custom_provider.sh`
- 不要把 provider 切回 `auto` 再配 `ollama/...` 模型名使用

## Weixin bridge 升级与重建

- 不要直接编辑 `~/.nanobot/bridge` 运行产物
- 正确路径是：
  - 改 `/opt/ai-stack/nanobot/bridge/src/...`
  - 然后本地重建 bridge
- 关键回滚锚点：
  - Git 提交历史
  - `weixin-auth`
  - 私有快照仓

## 升级后最小回归

- `nanobot-gateway.service` 为 `active`
- `nanobot-weixin-bridge.service` 为 `active`
- gateway 日志里重新出现 `Connected to Weixin bridge`
- Telegram 文本正常
- Weixin 文本正常
- Weixin 图片入站不回归
- `daily-digest` 仍在 `06:30`，Telegram + Weixin
