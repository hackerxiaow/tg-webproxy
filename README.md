# TG Web Proxy（单账号 · Cloudflare Pages + Worker 中继）

Telegram 最新 **td web-proxy / KWS** 协议。整套部署在**账号 B**（xiaow@mail.com）：
Pages 跑主程序（规避 Worker 直代理 1101/522），同账号 Worker 只做转发并承接自定义域名。

## 架构

```text
Telegram 客户端
  → https://tg.moew.eu.org              (账号 B · Worker relay)
  → https://tg-webproxy-f5a.pages.dev   (账号 B · Pages 主程序)
  → wss://kws{1..5}[-1].web.telegram.org/apiws
```

内层 MTProto 始终客户端 ↔ Telegram 端到端加密，边缘只做传输层 obfuscation 重签。

## 接入信息

| 项 | 值 |
|---|---|
| **服务器 / 代理地址** | `tg.moew.eu.org` |
| **端口** | `443`（HTTPS / WSS） |
| **密钥 SECRET** | `2de42a11ad839359ee77dd7cb2fbde8b` |
| **Pages 直连（备用）** | `tg-webproxy-f5a.pages.dev` |
| **健康检查** | `https://tg.moew.eu.org/health` → `{"ok":true}` |

## 客户端配置

### Telegram Desktop
1. 设置 → 高级 → 实验性功能：启用 **Web Proxy**（或代理设置里的 WebProxy/MTProto）
2. 服务器 `tg.moew.eu.org`，密钥填 `SECRET`（32 位 hex）
3. 客户端自动算 `?bridge=` 签名打开代理页，再建 WebSocket lanes

### Telegram Android
代理设置填同一服务器 + SECRET；应用内 web-proxy 桥走 `#android=<nonce>`。

## 部署资源（全部在账号 B）

| 层 | 名称 | 域名 |
|---|---|---|
| Pages 主程序 | `tg-webproxy` | `tg-webproxy-f5a.pages.dev` |
| Worker 转发 | `tg-webproxy-relay` | `tg.moew.eu.org` |

账号：`d9918cc215f68f240a776c36323702f6`（xiaow@mail.com）
Token：`CLOUDFLARE_API_TOKEN_XIAOW`

## 重新部署

```bash
export CLOUDFLARE_API_TOKEN=$CLOUDFLARE_API_TOKEN_XIAOW
export CLOUDFLARE_ACCOUNT_ID=d9918cc215f68f240a776c36323702f6
cd /Users/moe/.zcode/workspace/default/tg-webproxy

# Pages 主程序
npx wrangler pages deploy dist --project-name tg-webproxy --branch main

# Worker relay
npx wrangler deploy webproxy-relay.js --name tg-webproxy-relay
```

改 `SECRET` / `BOOT` 后需重设密钥并重新部署 Pages。

## 上游

- 主程序协议：[ToiCF/CF-Workers-TGProxy](https://github.com/ToiCF/CF-Workers-TGProxy)
- 拆分思路：[xinyitang3/cftgwebproxy](https://github.com/xinyitang3/cftgwebproxy)
