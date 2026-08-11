# 🚀 Katabump 自动续期脚本 (GitHub Actions)

本项目是根据eooce/katabump-renew进行优化

这是一个基于 GitHub Actions 的自动化脚本，用于定时登录并自动续期 [Katabump](https://dashboard.katabump.com/) 应用。

## ✨ 项目特点

*   **智能周期计算**：采用“每日唤醒 + 严格计算天数”的逻辑。通过设定起始日期，精准匹配官方 **4天一续** 的规则，彻底告别跨月、大小月导致的时间错位问题。
*   **安全省额度**：未到续期时间时，脚本会自动判定并安全跳过，不强行启动耗时的浏览器自动化任务，最大限度节省 GitHub Actions 的运行额度。
*   **智能防风控**：配合代理配置支持，有效绕过 Cloudflare (CF) 盾和人机验证。
*   **Telegram 通知**：支持绑定 Telegram 机器人，自动推送续期成功、失败或跳过的状态报告。

---

## ⚙️ 需要配置的参数

要让项目正常运行，你需要分别配置 **GitHub Secrets** 和 **代码中的起始日期**。

### 1. 配置 GitHub Secrets (仓库 Settings -> Secrets and variables -> Actions)

| Secret 名称 | 是否必填 | 说明 |
|---------------------|----------|---------------------------------------------------|
| `KATABUMP_EMAIL` | ✅ **必填** | 你的 Katabump 登录邮箱 |
| `KATABUMP_PASSWORD` | ✅ **必填** | 你的 Katabump 登录密码 |
| `NODE_LINK` | ❌ 可选 | 代理节点链接，用于绕过 CF 盾。建议使用稍微干净点的节点（如住宅代理）。不配置则使用直连。 |
| `TG_BOT_TOKEN` | ❌ 可选 | Telegram Bot Token（用于发送通知） |
| `TG_CHAT_ID` | ❌ 可选 | Telegram Chat ID（接收通知的用户或群组 ID） |

### 2. 修改起始续期日期 (修改 `.github/workflows/renew.yml` 文件)

为了让脚本能够精准匹配你的续期周期，请打开 `.github/workflows/renew.yml` 文件，找到 `START_DATE` 变量，将其修改为你明确知道的**下一次可续期日期**：

```yaml
      - name: 🚀 运行续期脚本
        env:
          # ... 其他环境变量
          # 👇 在这里填写你的起始续期日期 (格式：YYYY-MM-DD)
          START_DATE: '2026-08-11'  # <--- 请务必修改为你自己的续期日

💡 运行逻辑说明：
脚本每天 UTC 0:20 (北京时间 8:20) 自动唤醒。唤醒后会计算当前日期与 START_DATE 的相差天数。
只有相差天数是 4 的倍数（如 0天, 4天, 8天...）时，才会真正执行续期任务；否则直接跳过


━━━━━━━━━━━━━━━━━━━━━━
### 代理格式（确认在v2rayN里使用正常的节点）

`NODE_LINK` 支持以下任意一种代理协议的完整分享链接（不配置则直连）：

- **VLESS**：`vless://uuid@server:port?security=reality&sni=...&type=ws&...`
- **VMess**：`vmess://base64encoded...`
- **Trojan**：`trojan://password@server:port?sni=...&type=ws&...`
- **tuic**：`tuic://uuid:password@server:port...`
- **anytls**：`anytls://uuid@server:port...`
- **hysteria2**：`hysteria2://base64@server:port...`
- **SOCKS5**：`socks5://user:pass@server:port` 或 `socks://user:pass@server:port`

### 注意事项
⚠️ 注意事项
节点纯净度：如果有 CF 盾拦截提示，极大概率是因为机房 IP 被风控，建议更换为干净的代理节点（如 B2proxy 住宅代理）。

手动触发：如果你中途想强制续期，可以在仓库的 Actions 页面手动点击 Run workflow，此操作会无视 START_DATE 的规则强制执行一次续期。

