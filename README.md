# Telegram 自动签到脚本 (完美版 v7)

## 🚀 功能特性
- ✅ **20 个 Bot/频道自动签到**：内置你的专属签到列表，无需配置
- ✅ **防卡死设计**：全局超时控制、连接超时、自动断开
- ✅ **智能重试**：网络波动自动重试 3 次，Flood 限制自动等待
- ✅ **北京时间日志**：所有日志时间戳自动转换为北京时间 (UTC+8)
- ✅ **敏感信息脱敏**：日志自动过滤 Session、Token、IP 等敏感信息
- ✅ **自动运行**：每天北京时间 00:30 自动执行
- ✅ **环境兼容**：兼容 Python 3.11+ 和 Telethon 1.36+

## ⚙️ 配置要求
在 GitHub Secrets (Settings → Secrets and variables → Actions) 中设置以下三个变量：

| 变量名 | 说明 | 示例 |
|--------|------|------|
| `API_ID` | Telegram API ID (纯数字) | `12345678` |
| `API_HASH` | Telegram API Hash (字符串) | `a1b2c3d4e5f6...` |
| `SESSION` | Telegram Session 字符串 (长字符串) | `1B5...` |

## 🔄 运行方式
- **自动运行**：每天北京时间 **00:30** 自动触发
- **手动运行**：进入 Actions 标签页 → "Telegram Auto Sign-in" → "Run workflow"

## 📊 查看结果
1. 进入 **Actions** 标签页
2. 点击最新的运行记录
3. 查看日志中的 `✅ 成功` 或 `❌ 失败` 状态
4. 日志时间均为 **北京时间**

## ⚠️ 注意事项
1. **账号安全**：确保 Telegram 账号已登录，建议开启两步验证
2. **频率限制**：脚本已内置防 Flood 机制，请勿频繁手动触发
3. **Session 过期**：如果提示 `AuthKeyUnregistered`，请重新生成 Session 并更新 Secrets
4. **私密仓库**：本仓库为私密项目，日志已做脱敏处理，请妥善保管 Secrets

## 🛡️ 安全说明
- 所有敏感信息（API_ID, API_HASH, SESSION）均通过 **GitHub Secrets** 管理
- 代码中**绝不硬编码**任何密钥
- 日志输出自动**脱敏**，防止泄露敏感信息
- 建议定期更新 Session 字符串（每 3-6 个月）

---
**最后更新**：2026-08-21  
**版本**：v7.0 (移除失效 Bot @XJSGK_bot/@yang_SGKbot，cryptography 升级至 49.0.0)  
**维护者**：wang1520166813