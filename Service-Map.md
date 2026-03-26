# Service Map

这页说明当前 AI 运行面的服务、timer、仓库和它们之间的关系。

## 主链路

### `nanobot-gateway.service`

- 主聊天网关
- 工作目录：`/opt/ai-stack/nanobot`
- 负责：
  - agent loop
  - provider 调用
  - semantic memory
  - Telegram PlanBridge 与 Weixin channel 接入

### `nanobot-weixin-bridge.service`

- 本地 Weixin bridge
- 工作目录：`~/.nanobot/bridge`
- 负责：
  - 微信扫码登录态复用
  - bridge websocket
  - polling / 发送
  - `contextTokens` 持久化
  - 微信图片入站下载与解密

### `codex-listener.service`

- Codex Listener API
- 工作目录：`/opt/ai-stack/codex-listener`
- 负责：
  - 复杂任务下发
  - `plan_bridge`
  - 与 Telegram 计划流联动

## 定时任务

### `daily-digest.timer`

- 每天 `06:30`
- 触发 `daily-digest.service`
- 当前行为：
  - Telegram 发送天气摘要
  - Weixin 向 `channels.weixin.allowFrom` 广播同一份天气

### `hera-skills-backup.timer`

- 每 3 天跑一次
- 触发私有快照脚本
- 负责把 workspace、overlay、weixin-auth、gws、summarize、Mem0 导出等内容推到私有备份仓

### `nanobot-update-backups-prune.timer`

- 每天运行
- 清理本机历史更新备份，只保留最近几份

## 相关支撑服务

### `xray-client-telegram.service`

- Telegram 相关网络支撑服务
- 不直接承担 bot 逻辑，但当前链路里属于实际依赖

## 当前关键仓库

- `/opt/ai-stack/nanobot`
  - 主程序源码
- `/opt/ai-stack/codex-listener`
  - 任务桥 / 计划桥源码
- `/opt/ai-stack/nanobot-telegram-planbridge`
  - Telegram PlanBridge 相关实现
- `/home/Hera/nanobot-workspace-skills`
  - 私有备份与恢复控制仓
- `/home/Hera/nanobot-ops-wiki`
  - 公开运维文档仓

## 关系速记

- Telegram 聊天主入口实际走 `telegram_planbridge`
- Weixin 走本地 bridge，不是上游 direct channel
- `nanobot-gateway` 是中心，bridge / listener / digest 都围绕它工作

相关页：

- [Quick Start](Quick-Start.md)
- [Channels and Integrations](Channels-and-Integrations.md)
