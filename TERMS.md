# 使用条款 / Terms of Use

最后更新：2026-10-14
当前文档基线：AmuxAIClean v1.6.12（升级兼容下限见安装指南）

## 0. 接受条款

下载、安装或运行 AmuxAIClean 即表示你已阅读并同意本条款。如不同意，请勿安装或立即卸载（卸载方式见 [README.md](README.md)）。

<!-- v2026.10.14：内置规则的事实描述与独立发行/源码仓库一致；不改变确认、恢复及责任边界。 -->

## 1. 许可协议

AmuxAIClean 基于 **MIT License** 发布，完整许可文本见仓库根目录 [LICENSE](LICENSE)。

> 简要说明：你可以免费使用、复制、修改、合并、出版、分发、再授权及/或销售本软件副本，唯一的要求是在所有副本中保留原版权声明与本许可声明。

## 2. 软件性质

- AmuxAIClean 是一款本地 macOS 清理 / 优化工具
- 核心分析、安全判定和清理在用户本机运行；App 主动 HTTP 仅包括访问 GitHub 的更新检查，以及用户主动启用后访问其配置模型服务的 AI 助手。用户选择网络卷、云端占位文件或 Quick Look 时，macOS 或存储提供方仍可能远端读取/下载
- 不收取费用，不内购，无广告
- 详细隐私政策见 [PRIVACY.md](PRIVACY.md)

## 3. 免责声明（重要 —— 请务必阅读）

### 3.1 移入废纸篓与恢复边界

AmuxAIClean 的文件清理统一调用 macOS 系统回收接口，将项目**移入废纸篓**；
软件不提供“直接抹除”或绕过废纸篓的删除模式。

移入废纸篓后，在你清倒废纸篓之前，通常可通过软件的「最近删除」面板或 Finder
尝试还原；如果废纸篓已清倒、项目被系统或其他工具移除，软件无法保证恢复。请务必：

1. 重要数据先做 Time Machine / 云盘备份
2. 删除前认真核对预览列表
3. 确认文件无误前不要清倒废纸篓

### 3.2 启发式扫描的局限

AmuxAIClean 的「可清理项」基于内置 Vendor Rules 和启发式规则推断，**无法保证 100% 准确**：

- 极少数情况下可能把仍在使用的缓存识别为可清理
- 第三方 App 的私有数据格式如发生变化，旧规则可能误判
- 用户自定义路径仍会经过安全边界检查，但软件无法判断文件对用户业务是否仍有价值

**因此，所有删除前的最终判断责任在用户，软件仅做建议性提示。**

### 3.3 系统兼容性

- 支持 macOS 13.5 Ventura 或更高版本
- macOS 13.4 及更早版本不在支持范围内，安装或运行结果不受保证
- 不支持 iOS / iPadOS / visionOS / watchOS / tvOS

### 3.4 "AS IS" 条款（与 MIT 一致）

本软件按"现状"提供，**不附带任何明示或暗示的担保**，包括但不限于：

- 适销性（merchantability）
- 适用于特定用途（fitness for a particular purpose）
- 不侵权（non-infringement）

**在任何情况下，作者或版权持有人均不对任何索赔、损害或其他责任负责**，无论该等责任源于合同、侵权或其他原因，亦无论是否源于、起因于或与本软件或其使用或交易有关。这包括但不限于：

- 因误删文件造成的数据丢失
- 因清理系统缓存导致第三方 App 短暂异常
- 因停止维护、不再更新导致的兼容性问题
- 因 macOS 更新导致功能不可用
- 任何形式的间接损失、利润损失、业务中断

## 4. 用户责任

使用本软件即表示你确认：

- ✅ 你拥有被扫描磁盘上数据的合法访问权
- ✅ 你已理解恢复窗口有条件：只有匹配的废纸篓项目仍存在且身份可验证时才能尝试恢复；清倒废纸篓后不保证可恢复
- ✅ 你不会在生产服务器、关键业务系统上未经备份地运行删除操作
- ✅ 你已知晓本软件不附带商业级 SLA、不保证 7×24 可用
- ✅ 你不会将本软件用于任何违反所在地法律法规的目的

## 5. 第三方内容

- 主程序只链接 Apple 系统框架和仓库内第一方 Swift Packages，不嵌入第三方 SDK、静态库或动态库
- 检查更新功能访问 GitHub Releases API，使用即表示同意 [GitHub 服务条款](https://docs.github.com/en/site-policy/github-terms/github-terms-of-service)
- AI 助手默认关闭；用户启用并配置第三方或自托管模型端点后，相应请求受该服务商条款与隐私政策约束。具体发送字段见 [PRIVACY.md](PRIVACY.md)

## 6. 商标

- “AmuxAIClean”是当前产品与项目仓库名，不主张已注册商标
- "macOS"、"Apple"、"Finder"、"Sequoia"、"Apple Silicon" 等是 Apple Inc. 的注册商标，本项目仅做指称性使用，与 Apple Inc. 无任何关联或背书

## 7. 终止

你可以随时通过卸载本软件终止本条款；作者保留随时停止维护、归档项目的权利，停止维护后已分发版本仍按本条款继续可用。

## 8. 适用法律

本条款的解释和执行不指定特定司法管辖区。如使用本软件产生争议，应在用户所在地依据当地法律解决，作者不承担跨境诉讼成本。

## 9. 联系方式

技术问题与改进建议：请通过 GitHub Issues 提交。

---

**English Summary**

AmuxAIClean is released under the MIT License and provided "AS IS" without warranty of any kind. Cleanup moves items to the macOS Trash; there is no Direct Erase mode. Restoration is not guaranteed after the Trash is emptied or an item is otherwise removed, so back up important data and verify every preview. Update checks use GitHub, and the optional user-configured AI feature is governed by the selected provider's terms. Accessing a network share, cloud placeholder, or Quick Look preview may cause macOS or the storage provider to read remotely. The author is not liable for data loss or consequential damages. By installing the software you accept these terms.
