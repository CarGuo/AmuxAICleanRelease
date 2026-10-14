# 官方发布与下载

<!-- v2026.10.14：明确用户下载位置与两个仓库职责，不把公开仓库自动源码归档当作 App 源码。 -->

官方安装包统一发布在 [CarGuo/AmuxAICleanRelease Releases](https://github.com/CarGuo/AmuxAICleanRelease/releases)。
[最新版入口](https://github.com/CarGuo/AmuxAICleanRelease/releases/latest)与应用默认更新检查使用同一发行仓库。

每个正式版本提供 DMG、对应 SHA256 校验文件和 provenance 构建来源记录。下载同一版本的
DMG 与校验文件，在下载目录运行（以下为当前版本示例）：

```bash
shasum -a 256 -c AmuxAIClean-1.6.12.dmg.sha256
```

输出 `AmuxAIClean-1.6.12.dmg: OK` 表示下载字节与发行时一致。签名、公证、staple 和
Gatekeeper 的验证范围另见该版本发行说明；SHA256 不代替这些身份检查。

当前发行基线为 [1.6.12（236）](https://github.com/CarGuo/AmuxAICleanRelease/releases/tag/v1.6.12)：
macOS 13.5+、Apple Silicon/Intel 通用 Developer ID 公证 DMG，支持从 1.6.2 及以后升级。

本仓库保存产品介绍、用户文档、截图和发行附件。源码、开发验证与本地公证流程在独立源码
仓库维护；安装包身份以 provenance 中的源码 commit/tag 为准。公开仓库的版本 tag 指向
发行文档，“Source code”自动归档仅含该公开仓库内容，不能作为 App 源码或安装包使用。

文档与截图更新不更换已经发布的 DMG，也不移动已发布的源码 tag。
