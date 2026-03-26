# Weixin Runbook

这页是当前本地 Weixin bridge 的简版运行手册。

## 当前架构

- Python channel：
  - `nanobot/channels/weixin.py`
- Node bridge：
  - `bridge/src/weixin-api.ts`
  - `bridge/src/weixin-auth.ts`
  - `bridge/src/weixin-index.ts`
  - `bridge/src/weixin.ts`
- 状态目录：
  - `~/.nanobot/weixin-auth/accounts/*.json`
  - `~/.nanobot/weixin-auth/sync/*.json`
- 常驻服务：
  - `nanobot-weixin-bridge.service`

## 职责边界

- bridge 负责：
  - QR 登录
  - polling / send
  - `contextTokens`
  - 微信图片入站下载与解密
  - 出站媒体上传与发送
- Python channel 负责：
  - websocket transport
  - `InboundMessage` 封装
  - prompt-safe metadata 过滤
  - transport 心跳与重连

## 状态文件语义

- `accounts/*.json`
  - account token
  - user/account IDs
  - base URL
  - 持久化 `contextTokens`
- `sync/*.json`
  - `getUpdatesBuf`
- 对话正文历史不在 `weixin-auth`，而在：
  - `~/.nanobot/workspace/sessions/`

## 图片入站协议

当前图片入站不是“抓图片 URL”，而是按腾讯官方 `@tencent-weixin/openclaw-weixin` 协议对齐：

- 解析 `item_list`
- 匹配 `type=IMAGE`
- 读取 `image_item.media.encrypt_query_param`
- 使用 `image_item.aeskey` 或 `image_item.media.aes_key`
- 走 CDN 下载 + 解密
- 落盘后作为 inbound media 转给 agent

## 常见日志关键字

- bridge 层：
  - `weixin.inbound_item_summary`
  - `weixin.image_download_start`
  - `weixin.image_saved`
  - `weixin.image_download_failed`
  - `weixin.send_invalid_context_token`
  - `weixin.send_session_expired`
  - `weixin.monitor_session_expired`
  - `weixin.monitor_api_error`
  - `weixin.monitor_network_error`
- gateway 层：
  - `Connected to Weixin bridge`
  - `Weixin transport disconnected`

## 常见排查

- 图片没进模型：
  - 看 `journalctl -u nanobot-weixin-bridge.service`
  - 看 `~/.nanobot/media/weixin/` 有没有新文件
  - 如果目录为空，说明 bridge 还没完成 CDN 下载/解密
- 回复失败：
  - 看 `accounts/*.json` 里的 `contextTokens`
  - 看 bridge 日志里的 invalid context / session expired
- bridge 看起来活着但 gateway 收不到：
  - 看 `nanobot-gateway.service` 日志里的 websocket 断线和重连

## 升级 checklist

- 保持本地 bridge 架构，不要直接替换成上游 direct channel
- 上游 Weixin 改动优先重查：
  - `routeTag`
  - `contextTokens` 持久化
  - session 失效处理
  - QR 刷新
  - 图片入站协议
  - 出站媒体发送
- 重建 bridge 后，至少回归：
  - 文本收发
  - 图片入站
  - 出站媒体
  - daily digest 微信广播
