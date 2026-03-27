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

### workspace cron: `mastodon_daily_share`

- live store: `~/.nanobot/workspace/cron/jobs.json`
- 调度：`daily_random`
- 时间窗：`08:00-20:00 Asia/Shanghai`
- 行为：
  - 读取 Mastodon 首页时间线
  - 仅在发现真正值得分享的内容时，主发 Telegram 私聊，并镜像同文案到 Weixin
  - `sendProgress=false`，所以不会把执行步骤外发到聊天渠道
- 去重状态：
  - `~/.nanobot/workspace/services/mastodon-daily-share/state.json`

### workspace cron: `bilibili_daily_share`

- live store: `~/.nanobot/workspace/cron/jobs.json`
- 调度：`daily_random`
- 时间窗：`08:00-20:00 Asia/Shanghai`
- 行为：
  - 读取登录态 Bilibili 首页推荐
  - 仅在发现真正值得分享的内容时，主发 Telegram 私聊，并镜像同文案到 Weixin
  - `sendProgress=false`，所以不会把执行步骤外发到聊天渠道
  - 登录失效时不分享内容，只发一条 Telegram 登录提醒
- 去重与状态：
  - `~/.nanobot/workspace/services/bilibili-daily-share/state.json`
  - `~/.nanobot/workspace/services/bilibili-daily-share/last_prepare.json` 仅作为短期 callback 缓存使用，不属于长期备份真相面

### `daily-digest.timer`

- 每天 `06:30`
- 触发 `daily-digest.service`
- 当前行为：
  - Telegram 发送天气摘要
  - Weixin 向 `channels.weixin.allowFrom` 广播同一份天气

### `hera-skills-backup.timer`

- 每 3 天跑一次
- 触发私有快照脚本
- 负责把 workspace、overlay、weixin-auth、bilibili-auth、gws、summarize、Mem0 导出等内容推到私有备份仓

## 相关支撑服务

### `xray-client-telegram.service`

- 旧的 Telegram SOCKS 支撑服务
- 当前已不再是主链路依赖，只保留作回滚备用件
- 仍保留的回滚材料有：
  - `xray-client-telegram.service`
  - `/usr/local/etc/xray/client-telegram.json`
- `xray` 二进制本体不是快照内容；只有在未来明确回滚到代理方案时，才需要重新安装并重新启用该服务

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
