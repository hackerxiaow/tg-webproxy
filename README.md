# TG Web Proxy（纯 Cloudflare Pages）

Telegram 最新 **td web-proxy / KWS** 协议，纯 Pages 单层部署（无 Worker，规避 1101/522）。

## 架构

```text
Telegram 客户端
  → https://<自定义域名>   (Pages 自定义域名，CNAME → tgws-a1x.pages.dev)
  → wss://kws{1..5}[-1].web.telegram.org/apiws
```

内层 MTProto 始终客户端 ↔ Telegram 端到端加密，Pages 只做传输层 obfuscation 重签。

## 项目信息

| 项 | 值 |
|---|---|
| **Pages 项目** | `tgws`（账号 B，无 proxy 字样） |
| **默认域名** | `https://tgws-a1x.pages.dev` |
| **自定义域名** | 控制台绑定，CNAME → `tgws-a1x.pages.dev` |
| **SECRET** | `2de42a11ad839359ee77dd7cb2fbde8b` |
| **健康检查** | `https://tgws-a1x.pages.dev/health` → `{"ok":true}` |

## 自定义域名绑定

Cloudflare 控制台 → Pages → `tgws` → Custom domains → 添加域名。
**同账号域名会自动创建 CNAME**（API Token 无 DNS 写权限时需在控制台操作）。

## 客户端配置

- 服务器：绑定的域名（或 `tgws-a1x.pages.dev`）
- 端口：443
- 密钥：SECRET（32 位 hex，见上表）

## 重新部署

```bash
export CLOUDFLARE_API_TOKEN=$CLOUDFLARE_API_TOKEN_XIAOW
export CLOUDFLARE_ACCOUNT_ID=d9918cc215f68f240a776c36323702f6
cd /Users/moe/.zcode/workspace/default/tg-webproxy
npx wrangler pages deploy dist --project-name tgws --branch main
```

## 上游

- 协议/主程序：[ToiCF/CF-Workers-TGProxy](https://github.com/ToiCF/CF-Workers-TGProxy)
