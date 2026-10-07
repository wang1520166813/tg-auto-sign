# Telegram Auto Sign-in

一个基于 [Telethon](https://github.com/LonamiWebs/Telethon) 的 Telegram 自动签到工具，通过 **GitHub Actions** 定时运行，向指定的 Bot / 频道发送签到命令。

无需服务器、无需常驻进程：**Fork 本仓库 → 配置 Secrets → 每天自动签到。**

---

## ✨ 功能特性

- **多目标签到** — 支持配置任意数量的 Bot / 频道，签到列表通过环境变量注入，**无需修改代码**
- **防卡死设计** — 全局超时 + 连接超时 + 自动断开，避免任务长时间挂起
- **智能重试** — 网络波动自动重试 3 次；触发 Flood 限流时自动等待后重试
- **北京时间日志** — 所有日志时间戳自动转换为北京时间（UTC+8）
- **日志脱敏** — 自动过滤 Session、Token、IP 等敏感内容
- **零服务器部署** — 完全依赖 GitHub Actions，无需自建服务器或常驻进程
- **环境兼容** — Python 3.11+ / Telethon 1.36+

---

## 📦 项目结构

```
.
├── main.py                                        # 主程序
├── requirements.txt                               # 依赖清单
└── .github/
    └── workflows/
        └── telegram-signin.yml                    # GitHub Actions 工作流
```

---

## ⚙️ 配置教程

### 步骤 1 · 获取 Telegram API 凭证

1. 访问 [my.telegram.org](https://my.telegram.org) 并登录你的 Telegram 账号
2. 进入 **API development tools**
3. 创建一个应用（名称随意），获得：
   - `API_ID`：纯数字，例如 `12345678`
   - `API_HASH`：32 位字符串，例如 `a1b2c3d4e5f6...`

### 步骤 2 · 生成 Session 字符串

在本地运行以下脚本（需先 `pip install telethon`），按提示输入手机号与验证码：

```python
from telethon.sync import TelegramClient
from telethon.sessions import StringSession

API_ID = 12345678              # 换成你的 API_ID
API_HASH = "your_api_hash"     # 换成你的 API_HASH

with TelegramClient(StringSession(), API_ID, API_HASH) as client:
    print(client.session.save())
```

终端输出的那一长串字符就是 `SESSION`。

> 💡 提示：登录时请使用**手机号**（含国际区号，如 `+8613800138000`）。
> 若账号开启了**两步验证**，脚本会提示你输入密码。
> 生成的 Session 等同于账号登录凭证，**务必妥善保管，不要提交到代码仓库**。

### 步骤 3 · 配置签到列表

签到列表使用 JSON 数组表示，每项为 `["@Bot用户名", "签到命令"]`：

```json
[["@ExampleBot", "/sign"], ["@DemoBot", "/qd"]]
```

- `@Bot用户名`：目标 Bot 或频道用户名，以 `@` 开头
- `签到命令`：该 Bot 需要的命令，例如 `/sign`、`/qd`、`签到`、`/checkin`
- 想签到几个就写几项，**数量不限**

### 步骤 4 · 设置仓库 Secrets

进入你的仓库 → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

| 名称 | 必填 | 说明 |
|------|------|------|
| `API_ID` | ✅ | Telegram API ID（纯数字） |
| `API_HASH` | ✅ | Telegram API Hash |
| `SESSION` | ✅ | 步骤 2 生成的 Session 字符串 |
| `SIGN_LIST_JSON` | ✅ | 签到列表 JSON，如 `[["@ExampleBot","/sign"]]` |

> ⚠️ **这四个变量请统一放在 Secrets 标签页。**
> 特别注意：`SIGN_LIST_JSON` **也要放 Secrets**——工作流是从 `secrets.SIGN_LIST_JSON` 读取的，
> 如果误放到 Variables，脚本读不到签到列表，任务会直接失败。
>
> 填写 `SIGN_LIST_JSON` 时请保持**一行、半角引号**，例如：
> ```
> [["@ExampleBot","/sign"],["@DemoBot","/qd"]]
> ```
> 不要换行，不要用中文引号 “ ” ，多个频道之间用英文逗号分隔。

### 步骤 5 · 调整运行时间（可选）

编辑 `.github/workflows/telegram-signin.yml`：

```yaml
on:
  schedule:
    - cron: '30 16 * * *'   # 每天 UTC 16:30 触发 = 北京时间 00:30
```

cron 表达式使用 **UTC 时间**，北京时间 = UTC + 8：

| 想要的北京时间 | 应填写的 cron |
|---------------|--------------|
| 00:30 | `30 16 * * *` |
| 08:00 | `0 0 * * *` |
| 20:00 | `0 12 * * *` |

---

## 🚀 使用方式

### 自动运行

工作流按 `cron` 表达式定时触发，无需人工干预。

### 手动运行

1. 进入仓库的 **Actions** 标签页
2. 左侧选择 **Telegram Auto Sign-in**
3. 点击右侧 **Run workflow** → **Run workflow**

> 首次配置后建议**手动触发一次**，确认所有 Secrets 填写正确。

---

## 📊 查看运行结果

1. 进入 **Actions** 标签页
2. 点击最新的运行记录
3. 展开 **Run Telegram Sign-in** 步骤查看日志
4. 日志中会逐条显示 `✅ [成功]` 与 `❌ [失败]`，末尾输出统计：

```
📊 签到统计结果 (2026-01-01 00:30:00)
✅ 成功：5 个
❌ 失败：0 个
📈 成功率：100.0%
```

---

## 🛡️ 安全说明

- 所有配置（`API_ID` / `API_HASH` / `SESSION` / `SIGN_LIST_JSON`）均通过 **GitHub Secrets** 管理，不写入代码
- 代码中**不硬编码**任何密钥
- 日志输出自动**脱敏**：超过 40 字符的长串与 IP 地址会被替换为 `[REDACTED]` / `[IP]`
- 建议每 **3–6 个月**轮换一次 Session

> ⚠️ **公开仓库的 Actions 运行日志对所有人可见。**
> 请确保敏感信息只存放于 Secrets，不要在代码或日志中输出账号信息。
> 如果希望日志私有，可将仓库设为 **Private**。

---

## ❓ 常见问题

**Q1：报错 `AuthKeyUnregistered`**
Session 已失效或被撤销。请重新执行步骤 2 生成新 Session，并更新仓库 Secret。

**Q2：报错 `API_ID 必须是正整数`**
`API_ID` 填写有误。确认是纯数字，且没有多余空格或引号。

**Q3：报错 `SessionPasswordNeededError`**
账号开启了两步验证（2FA）。生成 Session 时需输入你的 2FA 密码。

**Q4：日志显示 `没有有效的签到条目`**
`SIGN_LIST_JSON` 格式不正确。确认是合法的 JSON 数组，且每项都是 `["@bot", "命令"]` 两个元素。

**Q5：某些 Bot 一直失败**
Bot 可能已停用、改名，或你的账号未被允许使用。可从 `SIGN_LIST_JSON` 中移除该条目。

**Q6：Actions 没有按时间运行**
GitHub 的定时任务存在排队延迟（通常几分钟到几十分钟），高负载时段可能更久，属正常现象。

**Q7：会不会被 Telegram 封号？**
脚本内置了任务间隔（每个任务间隔 3 秒、发送后等待 2 秒）与 Flood 限流处理，行为接近人工操作。但请勿把间隔调得过短或频繁手动触发。

---

## 📄 许可证

本项目采用 [MIT License](LICENSE)。

---

## 🤝 贡献

欢迎提交 Issue 反馈问题，或提交 Pull Request 改进功能。

---

## ⭐ 支持

如果这个项目对你有帮助，欢迎点个 Star。
