# TG Web Proxy（Cloudflare Pages + Worker 中继）

Telegram 最新 **td web-proxy / KWS** 协议部署。主程序跑 Pages（规避 Worker 直代理 1101/522），Worker 只做转发并承接自定义域名。

## 架构

```text
Telegram 客户端
  → https://tg.seurl.eu.org          (账号 A · Worker relay)
  → https://tg-webproxy-f5a.pages.dev (账号 B · Pages 主程序)
  → wss://kws{1..5}[-1].web.telegram.org/apiws
```

内层 MTProto 始终客户端 ↔ Telegram 端到端加密，Worker/Pages 只做传输层 obfuscation 重签，不碰 mtp。

## 接入信息

| 项 | 值 |
|---|---|
| **服务器 / 代理地址** | `tg.seurl.eu.org` |
| **端口** | `443`（HTTPS / WSS） |
| **密钥 SECRET** | `2de42a11ad839359ee77dd7cb2fbde8b` |
| **Pages 直连（备用）** | `tg-webproxy-f5a.pages.dev` |
| **健康检查** | `https://tg.seurl.eu.org/health` → `{"ok":true}` |

## 客户端配置

### Telegram Desktop
1. 设置 → 高级 → 实验性功能：启用 **Web Proxy**（或代理设置里选 MTProto / WebProxy，视版本）
2. 服务器填 `tg.seurl.eu.org`，密钥填上面的 `SECRET`（32 位 hex）
3. 客户端会自行生成 `?bridge=` 签名打开代理页，再建 WebSocket lanes

### Telegram Android
同上，在代理设置中填服务器与 SECRET；应用内 web-proxy 桥走 `#android=<nonce>`。

### 手动验证 bridge 页

```bash
# SECRET 对域名做 HMAC 即 bridge cap（客户端自动算）
# 页面 origin 必须是当前访问域名，relay 会重写
open "https://tg.seurl.eu.org/health"
```

## 重新部署

```bash
# Pages 主程序（账号 B）
export CLOUDFLARE_API_TOKEN=$CLOUDFLARE_API_TOKEN_XIAOW
export CLOUDFLARE_ACCOUNT_ID=d9918cc215f68f240a776c36323702f6
cd dist && npx wrangler pages deploy . --project-name tg-webproxy --branch main

# Worker relay（账号 A）
export CLOUDFLARE_API_TOKEN=<账号 A token>
export CLOUDFLARE_ACCOUNT_ID=00c5e26f7ea053e35c2d9a22b67e9f91
npx wrangler deploy webproxy-relay.js --name tg-webproxy-relay
```

改 `SECRET` / `BOOT` 后需重设密钥并重新部署 Pages。

## 上游

- 主程序协议逻辑：[ToiCF/CF-Workers-TGProxy](https://github.com/ToiCF/CF-Workers-TGProxy)
- 拆分部署思路：[xinyitang3/cftgwebproxy](https://github.com/xinyitang3/cftgwebproxy)
