# AmuxAIClean 安全模型

> 这份文档描述 AmuxAIClean 的威胁模型、删除链路设计、红线与漏洞上报方式。
> 任何与本文档不一致的实现都视为 bug，请优先修代码而不是改本文档。

---

<!-- v2026.10.14：移除已退役 Helper/XPC 的当前链路描述与失效链接；
候选、授权和执行三道闸门与产品实际实现、删除合同一致。 -->

现行文件清理只处理 `$HOME` 下允许的用户态候选，以当前用户身份调用
TrashPipeline → `NSWorkspace.recycle`。
完整入口与验证范围见[产品状态](docs/CURRENT_STATE.md)；历史 Helper/XPC 设计已退役。
可选 Sentinel 默认关闭，只在当前用户域做本地发现，不参与删除授权或执行。

---

## 1. 设计原则（Hard Rules）

按重要性递减：

1. **永远不 unlink**：所有"删除"操作只能通过 `NSWorkspace.shared.recycle(_:)` 走废纸篓。代码库内禁止出现 `unlink(2)` / `FileManager.removeItem(at:)` 用于"删除候选"路径，违反者视为 P0 安全 bug。
2. **dryRun 是执行层的一等公民**：删除流水线 TrashPipeline 默认 `dryRun=true`；Release UI 不显示 dry-run/仅预览开关，用户在共享确认页明确选择“移到废纸篓”后才生成真实执行授权。自动化测试默认 dryRun；需要验证真实 recycle 时只能使用 `NSTemporaryDirectory()` 下的隔离 fixture。
3. **三色判定后才能删**：删除前必须经过 SafetyEngine.evaluate 的 `safe / confirm / forbid` 三色。普通模式的 `forbid` 一律拒绝；专业模式只有命中显式可覆盖 reason、取得类型化授权并完成严格确认后才能尝试。`hardForbid` 与结构性边界永不可覆盖。
4. **批级原子拒绝**：普通请求中任意一条命中 `forbid`，整批拒绝并提示用户，不允许"删一半再回滚"。专业覆盖请求必须先把 normal / override-eligible / non-bypassable 分区并分别确认；执行前出现任意新的 non-bypassable 路径时整批拒绝。
5. **确认上下文完整性**：类型化删除授权必须绑定精确路径、来源、扫描 revision、风险 reason 和用户选择；确认后路径身份变化或风险增加必须重新预检。历史 HMAC/XPC 设计不再属于当前链路。
6. **能力最小化**：v1 不部署特权删除 Helper，所有删除操作以普通用户身份运行，依赖 Trash 机制做最后兜底。可选 Sentinel 同样只在登录用户域运行；包内旧 LaunchDaemon plist 仅用于注销历史 root 注册，代码不得重新注册它。

---

## 2. 威胁模型

### 在范围内（v1 必须防）

- 用户误操作：手抖把 `~/Library` 整个删除 -> hardForbid 黑名单 + 三色判定 + 批级原子拒绝
- 错误规则：vendors.yaml 写成 `~/Library` 这种危险前缀 -> RuleLoader 启动期反向校验，命中 hardForbid 的规则直接 reject
- 路径欺骗：候选路径含 `..`、symlink 出 sandbox -> SafetyEngine resolveSymlinks + 真实路径白名单
- vendor 误命中：`com.foo.bar*` 命中 `com.foo.barbarian` -> matchesStrictPrefix 强制点号边界
- 短 appName 模糊：用 "X" 反扫导致大量误删 -> minimumStem 长度限制 + 严格 Levenshtein
- 隐藏文件误删：`.DS_Store` / `.git` 之类 dotfile -> OrphanScanner 显式跳过 dotfile
- TCC 失败被吞：扫描没结果但用户不知道 -> TCCProbe errno=1 累计阈值后弹引导

### 不在范围内（v1 不防）

- 物理攻击者本地 root 提权（系统已沦陷，不在工具能解决的范围）
- 恶意 vendors.yaml 投毒（v1 只用 bundle 内置 yaml，不读用户自定义；如未来开放，再补签名验证）
- 时序侧信道（v1 不处理加密敏感信息，无意义）

---

## 3. 删除链路：三道闸门

### 候选级

SafetyEngine 校验原路径、解析后的身份、
HOME/系统边界、hard-forbid 与厂商规则，产生独立的 verdict/rejection。
扫描完整性、TCC 和实际文件权限分别保留类型化来源，不能把扫描缺口直接改成永久禁止或权限拒绝。
卸载、重复文件、工作流等候选均不能跳过同一安全边界。

### 授权级

TrashActionCoordinator 消费精确
`DeletionIntent` 和确认计划。授权冻结路径集、扫描 session/revision、风险 reason、运行中应用
决定与用户选择；普通审阅或专业严格确认只改变呈现方式，不授予任意 force 能力。
取消、关闭或预检拒绝不能生成系统提交回执，也不能构造成功结果。

### 执行级

TrashPipeline 在耐久 reservation 与系统调用边界
重新检查身份、范围、安全、权限、重复文件内容/保留副本及对应授权。跨进程 owner/attempt
和阶段防止重复提交；历史持久化不可用时不调用系统回收。
真正提交后以系统回执确认已移动项，再同步历史与页面。未知结果保留待核对状态，不伪报失败或成功。

文件清理只调用 `NSWorkspace.recycle`。Simulator Device / Runtime 是封闭类型的系统托管卸载
例外，必须明确标识不可从废纸篓恢复，不能扩展成任意命令或文件删除通道。
具体合同见 DELETION_UX_CONTRACT.md。

### 专业模式覆盖边界

`SafetyVerdict.forbid` 表示普通删除不可提交，但不同 `SafetyRejection` 的安全含义不同；专业模式不能继续用一个裸 `Bool force` 一刀切。

- **永不可覆盖**：relative/empty path、`..`、`/`、HOME/Library/Documents/Desktop/Downloads 等整根、HOME 外路径、系统卷、hardForbid prefix、symlink 解析后越界、不可解释的路径身份变化。
- **可严格覆盖**：当前只允许由生产预检铸造的 `vendorPathRule` 类型化授权；其它产品策略必须逐类加入显式 allowlist 和测试，禁止字符串 contains 判断。
- **扫描不完整**：`incomplete` 是证据质量，不是 OS 删除权限。v2026.10.14 起，行模型独立保留 Safety 判定与 typed 扫描缺口，复制、聚合及选中快照更新不能丢失任何一方。Normal 的选择和可清理容量、Smart Care 的默认清理仍排除缺证据项；专业模式可在目标级 `lstat + resolved scope + parent write/search` 重验、展示“内容未完整检查”并严格确认后尝试移动整个目标，不得因保留原 Safety 判定而自动获得删除权，也不得把缺口统一映射成 Full Disk Access。
- **真实 OS 权限**：TCC、POSIX owner/mode、ACL、immutable flag 必须保留来源。当前无 root helper，不能承诺绕过；`NSWorkspace.recycle` 返回失败时如实展示。
- **能力不可伪造**：覆盖授权必须绑定精确路径、精确 reason、扫描 revision 和确认上下文，执行前重新跑 Safety/路径解析。裸 `[String]` 回调和与 normal 等价的空心 `.force` 不构成授权。
- **统一交互**：完整状态机见 DELETION_UX_CONTRACT.md。预检拒绝不能包装成执行结果，结果页不能承担首次授权。

---

## 4. 数据持久化、隐私与受控网络边界

<!-- v2026.07.18：按生产源码重建隐私合同，移除“完全离线/无路径持久化”的失实描述。 -->

本地安全判定和删除授权不依赖网络，也不得因网络状态改变。当前 App 源码允许主动创建的
HTTP 请求只有：

- 更新检查：只读取固定 GitHub Releases `latest` API，并验证同仓库、同 tag 的 Release 页面
  与 asset 元数据；App 不自动请求 DMG、`*.sha256` 或其它 asset，也不跟随 HTTP redirect。
  首次安装默认开启启动自动检查，每个 App 进程最多一次，用户可关闭或手动触发；
- 可选 AI：默认关闭，只在用户配置端点并主动发起询问/审核时请求 OpenAI-compatible API。
  非环回端点强制 HTTPS，环回开发端点可用 HTTP；base URL 禁止 userinfo/query/fragment，
  客户端不跟随 HTTP redirect；默认会话为禁 Cookie、cache 和共享 credential storage 的
  独立 ephemeral session。AI 技术日志只保留 endpoint host、stage、HTTP status/响应字节数
  或 transport domain/code，不得保存请求正文、API Key、服务端错误正文或模型响应正文；
  请求字段和接收方边界以
  [PRIVACY.md](PRIVACY.md) 为准。

“只有两类主动 HTTP”不等于文件系统永不联网。用户选择 SMB/NFS 网络卷、iCloud/FileProvider
占位文件或 Quick Look 时，文件系统、提供方或 macOS 预览服务可能远端读取/下载；这类系统
代办访问不得被文档伪装成完全离线，也不得在实现中自动转换成 AI 请求。

任何新增或扩大网络请求都必须同时定义目标、触发授权、数据字段、超时/取消、凭据、日志脱敏、
隐私声明和聚焦测试。远程结果永远不能提高删除安全等级、改变默认勾选、铸造删除授权或绕过本地
`SafetyEngine`。项目不收集遥测，也不自动上报崩溃。

本地持久化不是“只有两份且没有路径”。生产状态包括：

- `~/Library/Application Support/AmuxAIClean/heuristic.json`：绝对路径及祖先前缀、
  deleted/restored/kept 计数与时间；
- 同目录的 `clean_stats.json`、`whitelist.json`、`trash-pending-operations.json`、
  `trash-completion-journal.json`、`simulator-managed-operations.json`：分别保存本地清理统计、
  白名单、操作对账状态，以及系统已完成移动但尚未耐久合并进撤销历史的短期交接事实；交接 journal
  包含原路径/废纸篓路径、恢复范围、操作与尝试 UUID、时间和文件身份，只能在 Undo 与 pending
  ledger 都收敛后移除，失败时留待启动重放；
- **v2026.10.14 撤销历史存储变更**：同目录的 `UndoHistory.json.sqlite` 是生产历史权威，保存最近
  10 批恢复身份、逐项目恢复状态及不含路径的 SHA-256 幂等 receipt。确认流程先在 journal
  耐久标记 `receiptPrunePending`，再清 receipt，最后才移除 journal 项。任一阶段崩溃都由
  启动恢复继续，不能在失去最后恢复证据后永久遗留 receipt。旧 JSON 导入时，其精确原始字节、
  文件身份和退役意图也先存入此数据库；原始字节归档用于保留迁移证据，不是第二份可写历史。
  `UndoHistory.json` 随后成为 schema 5 退役 manifest，仅记录存储格式、数据库文件名和迁移 ID；
  兼容范围仍仅为 1.6.2 及以后升级；
- 用户主动开启 Sentinel 后，同目录的 `sentinel-pending.json` 最多保存 100 条热点变化的
  绝对路径、时间和可选大小；当前服务是用户域 `com.amux.ai.Clean.sentinel.agent`。
  历史 root 服务 `com.amux.ai.Clean.sentinel` 使用文件名/Label 对应的 migration-only
  描述符并验证 `notRegistered`；升级机器上的 ServiceManagement 状态才是权威；关闭
  当前 Agent 是降权动作，不得因旧描述符查询失败而被阻断，但
  旧状态未验证时也不得显示“全部已停止”。ServiceManagement 另持有后台项目注册状态，
  删除 defaults 不能代替 `SMAppService.unregister()`；
- `~/Library/Preferences/com.amux.ai.Clean.plist`：UI/扫描/更新/AI 设置和自定义扫描路径；
- macOS Keychain service `com.amux.ai.Clean.ai.apiKey`：AI 凭据，不同步；
- `~/Library/Logs/AmuxAIClean/`：主 App 与用户态 Sentinel 以 7 天为目标滚动保留本地诊断日志，
  可能包含业务阶段、结果、路径和技术错误，禁止记录 API Key、AI 请求正文、服务端错误正文或
  模型响应正文；7 天是主 App 初始化/跨日写入/显式目录准备重试或 Sentinel 每小时检查驱动的
  目标，不是硬实时过期。主 App
  产品源码统一经过 `AmuxLog`，另写由 macOS 管理的 privacy-hashed Unified Logging；文件 sink
  的目录创建、打开和保留枚举必须在后台初始化，UI 启动路径只允许最多 256 行的有界内存缓冲，
  旧初始化 revision 不得覆盖新配置。本地 sink 失败只允许独立公开白名单化的
  `stage/domain/code`，设置页/Help 必须显示同一闭集，stderr 不得回显 raw Error/路径；writer
  health 与 retention/enumerate/export 等 operation issue 必须独立，同阶段成功才清除对应问题。
  枚举、导出、保留和清理只接受逐级 no-follow、精确日期名、普通且单链接的主 App 日志；合并
  导出使用固定 64 KiB 块写入同目录 `0600` 临时文件并 `fsync + renameat`，不得构造无界聚合
  `Data`；清理完成事件会进入新日志，不能宣称留下空文件；
- Trash pending/completion 仍使用 `0600` 随机临时 JSON 做 fsync + rename；进程在 rename/unlink
  前被强制终止时，遗留 `.trash-pending.<UUID>.tmp`、`.trash-completion.<UUID>.tmp` 可能含路径。
  旧 JSON writer 遗留的 `.UndoHistory.<UUID>.tmp` 也可能含路径。对应 store 必须在自身稳定
  跨进程 flock 内按精确形状清理，禁止无锁 glob 或误删相似用户文件；生产 SQLite 的 rollback
  journal 由 SQLite 管理，不得混入上述临时文件清理；
- 用户可把大文件 JSON/TSV 或合并日志导出到任意选择位置；这类副本不受 App 的 7 天保留或
  “清除日志”控制，必须由用户自行删除。完整字段与清除顺序以隐私政策为准。

完整字段、保留和清除方法只在 [PRIVACY.md](PRIVACY.md) 维护，代码或 schema 改动必须同步更新。

**v2026.10.14 历史写入与退役边界**：`UndoHistoryStore` 公共初始化默认使用系统 SQLite；内部
`legacyJSON` 仅供明确的旧格式 fixture/import 验证，不是生产故障后的回退 writer。生产事务在后台
持久化队列与原有稳定 history flock 内执行，使用 `BEGIN IMMEDIATE`、`journal_mode=DELETE`、
`synchronous=EXTRA` 和 `fullfsync=ON`。数据库、sidecar 与父目录身份必须通过 no-follow/身份检查；
路径被替换、未知 schema 或损坏不能触发覆盖重建。SQLite 的提交原子性只覆盖数据库事务，遵循其
[rollback-journal 原子提交](https://sqlite.org/atomiccommit.html)及文件系统同步假设，不覆盖外部
JSON ledger/journal；三者继续使用精确 attempt 的顺序提交、回执和幂等重放协议。

导入先在数据库提交原始字节、身份、记录及完整 manifest，再在同一 history flock 内退役旧路径，
最后提交退役完成标记。已有旧文件只通过验证后的固定 descriptor 写入 manifest；首次创建使用
完整 fsync 的私有临时文件与 `RENAME_EXCL` 发布，不能覆盖并发出现的对象。中断后只接受已提交
退役意图允许的原版本或同一对象上的 manifest 精确前缀，以及完整 manifest；其它替换和歧义内容
必须保留并拒绝。schema 5 使旧 JSON writer 无法把退役文件当作可继续写入的历史。已验证的旧
PREPARED generation 可导入完整 stage，但必须同时证明旧 reserve、marker、目标身份和精确前缀；
不能把任意损坏 JSON 当成可恢复数据。旧 JSON 的 8 MiB / 每条 10,000 pairs 仅是 legacy 解码边界，
不再限制 SQLite 接收已接受的 callback 事实。每次真实删除 reserve 前的 `prepareForDeletion`
必须提交一次真实数据库写事务；空 completion journal 或缓存的 `.ready` 不能证明 writer 可用。

`PrivacyInfo.xcprivacy` 必须覆盖所有平台都适用的 collected-data、tracking 与 tracking-domain
事实。可选 AI 的路径/名称、文件元数据和 Bearer API Key 分别按 `Other User Content`、
`Other Data Types` 和 `User ID` 保守披露；允许自定义兼容端点意味着不能假设所有服务商都只在
实时鉴权期间处理且立即丢弃账户凭据。三类都只能声明 App Functionality、linked、no tracking。
Apple 当前把 Required Reason API 声明范围列为 iOS、iPadOS、tvOS、visionOS 和
watchOS，不包含本项目唯一发行平台 macOS，因此当前清单不写 `NSPrivacyAccessedAPITypes`；
禁止把移动平台理由码当作通用 API 清单，尤其不能用仅允许“展示磁盘空间”的 `85F4.1` 概括
本项目还用于介质/只读卷分类的 `statfs`。目标平台或 Apple 适用范围变化时，必须重新按源码
真实用途审计并同步清单、隐私政策和发行测试。
App 不读取剪贴板或通讯录，也不解析浏览器历史数据库中的访问记录；用户明确点击复制时会把
当前展示的路径/名称/结果/Bundle ID 写入系统通用剪贴板。普通磁盘占用扫描主要处理路径、
类型、时间和占用等文件系统元数据；卸载、孤儿、启动项、开发者构建产物与 Brew 页面还会在
本机解析 Info.plist、launchd plist、receipt/JWS JSON 等结构化记录，Quick Look 由用户主动
触发并交给 macOS。重复文件扫描会额外在本机读取候选文件字节计算 SHA-256，删除前再次完整
校验内容与身份。上述正文和重复哈希不由 App 上传或持久化；完整边界以
[PRIVACY.md](PRIVACY.md) 为准。

---

## 5. v1 已知不防的攻击面（透明）

- 当前正式候选使用 Developer ID 签名，并只提交最终 DMG 公证；Notary、staple、Gatekeeper 与 SHA256 能证明来源和字节完整性，但不能证明业务逻辑没有缺陷
- 官方安装包与默认更新源为 [CarGuo/AmuxAICleanRelease](https://github.com/CarGuo/AmuxAICleanRelease/releases)；本地构建/公证与公开上传是独立操作，本地成功不会自动发布
- v1 不启用特权删除 Helper -> 受 TCC 限制读不到的目录扫不到，这是预期；不会通过任何"曲线救国"绕过 TCC
- vendors.yaml 在 bundle 内只读，但用户如果自己 mod app 资源后果自负

---

## 6. 漏洞上报

如果你发现了**绕过 typed deletion intent / SafetyEngine / TrashPipeline 导致未授权移动**或**导致 SafetyEngine 错判**的漏洞，请：

- 不要公开 issue
- 发邮件到 `guoshuyu.gh@gmail.com`，标题前缀 `[SECURITY]`
- 包含：复现步骤、影响范围、建议修复方向
- 我会在 7 天内回复，确认后 30 天内发补丁

非安全 bug、误命中、性能问题请走正常 GitHub Issue。

---

## 7. 红线汇总（对开发者）

新增/修改代码时必须自检：

1. 没引入 `unlink` / `removeItem(at:)` 删除候选路径
2. 新增删除请求在执行层默认走 dryRun；Release UI 不显示 dry-run/仅预览开关
3. 新加规则跑 RuleLoader 反向校验过 hardForbid
4. 新加候选源必须进入统一删除意图与 SafetyEngine，不能另建删除通道
5. 测试不允许通过减少断言来"修绿"
6. UI 上"危险操作"按钮必须用 `.destructive` role + 二次确认 sheet
7. 不允许 `try?` 静默吞错（错误必须 log 或上抛）
