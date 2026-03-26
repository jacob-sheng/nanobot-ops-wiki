# Channels and Integrations

这页说明当前机器上实际接入了哪些聊天入口、外部能力和辅助 CLI。

## Telegram

- 当前实际聊天入口不是内置 `telegram` channel
- 现网使用的是 `telegram_planbridge`
- 相关仓库：
  - `/opt/ai-stack/nanobot-telegram-planbridge`
  - `/opt/ai-stack/codex-listener`
- 当前已验证 Telegram 裸连可用，主链路不再依赖本地 SOCKS 代理
- `xray-client-telegram.service` 仅作为回滚备用件保留

## Weixin

- 当前使用本地 bridge 架构
- 关键服务：
  - `nanobot-weixin-bridge.service`
  - `nanobot-gateway.service`
- 登录态与桥状态：
  - `~/.nanobot/weixin-auth/`
- 当前图片入站已按腾讯官方 `@tencent-weixin/openclaw-weixin` 协议对齐：
  - `item_list`
  - `image_item.media.encrypt_query_param`
  - `image_item.aeskey` / `image_item.media.aes_key`
  - CDN 下载 + 解密
- 当前 daily digest 会把天气同步广播到 `channels.weixin.allowFrom`

## Google Workspace

- 当前通过 `gws` 接入
- 已接范围：
  - Gmail
  - Drive
  - Calendar
- 认证状态目录：
  - `~/.config/gws/`

## Mastodon

- 当前通过 `toot` + 本地 wrapper 接入
- 已支持：
  - 时间线读取
  - 通知
  - 搜索
  - follow
  - favourite
  - bookmark
- 当前不支持：
  - reply
  - 私信
  - 删除
  - 批量运营
- 当前还启用了一条 `mastodon_daily_share` 后台任务：
  - 每天 `08:00-20:00 Asia/Shanghai` 随机一次
  - 通过 Mastodon 首页时间线挑值得看的内容
  - 直接发到 Telegram 私聊
  - `sendProgress=false`，只发最终分享，不外发过程播报

## summarize

- 当前通过 `@steipete/summarize`
- 运行时依赖 Node 22
- 配置在：
  - `~/.summarize/config.json`
- 当前走 AxonHub 的 OpenAI-compatible 路径，不走 Google-native Gemini API

## Provider / Embedding

- 主聊天 provider：
  - AxonHub custom
- 默认聊天模型：
  - `ollama/kimi-k2.5`
- semantic memory embeddings：
  - NVIDIA NIM

## Long-term Memory 行为说明

- `memory_add` 和 `memory_search` 运行时仍然存在，没有被删掉。
- 内置 `memory` skill 也还在，而且是 `always: true`。
- 当前“模型不够主动记忆”的主要原因不是工具失效，而是之前精简 `TOOLS.md` 时，把中文语境下的记忆触发路由写得太弱了。
- 当前推荐心智：
  - 明确要求“记住 / 存长期记忆 / 保存偏好”时，优先 `memory_add`
  - 明确在问“你记得吗 / 我会什么 / 我有什么证书 / 以前提过什么”时，优先 `memory_search`
  - 天气、新闻摘要、系统状态、一次性运行流水不进长期记忆
