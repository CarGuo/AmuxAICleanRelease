# AmuxAIClean

面向 macOS 的原生磁盘空间分析与清理工具。看清空间被哪些目录占用，找到缓存、大文件与开发者构建产物，在审阅后完成清理。

本仓库用于 AmuxAIClean 的产品介绍、界面展示和安装包发布。

[下载最新版](https://github.com/CarGuo/AmuxAICleanRelease/releases/latest) · [版本记录](https://github.com/CarGuo/AmuxAICleanRelease/releases) · [问题反馈](https://github.com/CarGuo/AmuxAICleanRelease/issues)

## 功能

| 功能 | 说明 |
| --- | --- |
| 可视化空间分析 | 用矩形面积展示已测得的磁盘占用，支持选择扫描范围、逐层浏览目录、名称搜索和列表分页。较小项目合并展示，并提供完整明细入口。 |
| 一键扫描清理 | 扫描系统与日志、应用缓存及可选的开发缓存，查看候选与风险提示后确认清理。 |
| 大文件与重复文件 | 定位占用较大的文件，检查重复内容，辅助整理长期积累的数据。 |
| 开发者空间管理 | 检查 Xcode、Android、Flutter、Node.js 等开发环境的构建产物、依赖缓存及旧版本资源。 |
| 应用与残留管理 | 在专业模式中检查已安装应用及关联残留，按安全规则审阅处理。 |
| 系统维护 | 提供用户空间维护任务、传统启动代理查看及 Homebrew 等专业工具入口。 |
| 实时系统状态 | 查看 CPU、内存、磁盘、电池、网络速率与近期变化，以及运行时间和散热状态。 |
| 可选 AI 辅助 | 辅助理解扫描结果和清理建议，默认关闭，需要自行配置服务并主动使用。 |

普通模式提供 **分析、清理、优化、状态** 四个栏目，保留一键扫描清理入口。专业模式提供更细的扫描分类、文件列表和管理工具，应用卸载位于专业模式。

## 界面截图

以下截图由 1.6.3 对应界面生成。目录名称和占用使用演示数据，展示中文浅色与英文深色界面。

### 空间分析

从磁盘概览进入每个目录，结合占用图和列表定位需要整理的内容。

![普通模式空间分析](https://github.com/CarGuo/AmuxAICleanRelease/releases/download/v1.6.3/screenshot-analyze.png)

### 扫描清理

按需要选择扫描范围，扫描完成后审阅候选，再决定清理哪些内容。

![普通模式扫描清理](https://github.com/CarGuo/AmuxAICleanRelease/releases/download/v1.6.3/screenshot-clean.png)

### 系统状态

集中查看系统实时读数，支持深色外观与英文界面。

![英文深色系统状态](https://github.com/CarGuo/AmuxAICleanRelease/releases/download/v1.6.3/screenshot-status.png)

## 下载与安装

| 项目 | 要求或支持范围 |
| --- | --- |
| 系统 | macOS 13.5 及以上 |
| 芯片 | Apple Silicon 与 Intel，同一份通用安装包 |
| 语言 | 简体中文、English |
| 外观 | 浅色、深色 |

1. 从 [Releases](https://github.com/CarGuo/AmuxAICleanRelease/releases/latest) 下载 DMG。
2. 打开 DMG，将 AmuxAIClean 拖入 Applications。
3. 从 Applications 启动应用，选择需要扫描的范围。
4. 如需访问受保护的目录，可在“系统设置 → 隐私与安全 → 完全磁盘访问”中为 AmuxAIClean 授权。无法访问的内容会保留不完整提示。

当前版本为 **1.6.3（207）**，使用 **Developer ID Application: SentenceEnd Inc** 签名，已通过 Apple 公证、附加公证票据并完成 Gatekeeper 校验。

[下载 1.6.3 DMG](https://github.com/CarGuo/AmuxAICleanRelease/releases/download/v1.6.3/AmuxAIClean-1.6.3.dmg) · [下载 SHA256 校验文件](https://github.com/CarGuo/AmuxAICleanRelease/releases/download/v1.6.3/AmuxAIClean-1.6.3.dmg.sha256)

同时下载 DMG 和校验文件后，在下载目录运行：

```bash
shasum -a 256 -c AmuxAIClean-1.6.3.dmg.sha256
```

输出 `AmuxAIClean-1.6.3.dmg: OK` 表示文件与发布时的校验值一致。

## 清理与隐私

- 扫描结果表示已测得的占用，不等于全部可清理容量。权限受限或未完成测量的内容会明确标识。
- 文件清理先展示候选与确认信息，再通过废纸篓处理；关键路径与敏感数据受安全规则保护。最近清理记录可用于尝试恢复，前提是废纸篓中的项目仍存在且身份未变化。
- 本地扫描与安全判定不依赖 AI。启用 AI 后，主动发送的上下文由你配置的服务处理。
- 应用会检查 GitHub 版本更新；访问网络卷、云端占位文件或使用系统预览时，macOS 与存储提供方可能产生网络读取。

## 版本更新与反馈

请以本仓库的 [Releases 页面](https://github.com/CarGuo/AmuxAICleanRelease/releases) 为安装包发布入口。1.6.3 的应用内更新检查尚未迁移到本仓库，获取后续版本请查看此页面。

遇到问题可提交 [Issue](https://github.com/CarGuo/AmuxAICleanRelease/issues)，附上应用版本、macOS 版本、芯片类型和复现步骤。分享截图或日志前，请移除私人路径与敏感信息。

<!-- v1.6.3: Initial distribution-only README. Screenshots are versioned Release assets so the Git repository contains only this document. -->
