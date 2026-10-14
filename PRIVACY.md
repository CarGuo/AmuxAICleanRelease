# 隐私政策 / Privacy Policy

最后更新：2026-10-14

当前文档基线：AmuxAIClean v1.6.12（升级兼容下限见安装指南）

## 一句话说明

AmuxAIClean 不包含遥测、广告、行为埋点或崩溃自动上报。扫描、规则判断、删除预检和历史记录默认都在本机完成。App 源码主动发起的 HTTP 请求只有检查更新，以及用户主动启用并调用 AI 助手；扫描网络卷、云端占位文件或调用系统 Quick Look 仍可能由 macOS、文件提供方或远端文件系统产生网络访问。AI 请求可能把下文列明的文件路径上下文和元数据发送给用户配置的模型服务商，因此本项目不再作“任何路径永不离开设备”或“本地扫描绝不会触发网络 I/O”的错误承诺。

## 1. 不收集的内容

- 不收集设备 UUID、MAC 地址、序列号或广告标识符；
- 不做点击、停留时长、扫描频率、删除数量等遥测或分析；
- 不自动上传扫描结果、日志或崩溃报告；
- 不嵌入 Google Analytics、Firebase、Sentry、Crashlytics、Bugsnag 等第三方统计 SDK；
- 不读取剪贴板、通讯录、日历、相机、麦克风、位置、照片图库或屏幕内容，也不解析浏览器历史数据库中的访问记录。用户点击“复制”时，App 会把明确展示的路径、名称、结果或 Bundle ID 写入系统通用剪贴板，见第 2.6 节；
- 普通磁盘占用扫描主要读取名称、类型、时间与占用等文件系统元数据，但并非整个产品都“只读元数据”：卸载/孤儿/启动项/开发者构建产物/Brew 等专门页面会在本机解析相关 Info.plist、launchd plist、receipt/JWS JSON 等结构化文件；重复文件扫描还会读取候选文件字节计算 SHA-256。具体边界见第 2.4–2.5 节。

本地清理统计（例如累计移入废纸篓的字节数）只保存在本机，不会作为遥测上传。

可选的 Sentinel 后台监控默认关闭。用户在“设置 → 高级”主动开启后，它只在本机监听
废纸篓以及 Downloads、Caches、DerivedData 等清理热点的文件系统变化，用于提示卸载
残留或建议重新扫描；当前实现以登录用户的非特权 LaunchAgent 运行，不建立网络连接。
具体本地记录与旧版迁移边界见第 3 节。

## 2. App HTTP、本地解析与系统代办行为

下列 2.1 与 2.2 是当前 App 源码主动创建的两类 HTTP 请求；这一定义不把网络卷、云文件
提供方、默认浏览器或 Quick Look 的系统代办访问误算成 App 的 HTTP API。

### 2.1 检查更新

| 项目 | 真实行为 |
|---|---|
| 触发 | “启动时自动检查更新”首次安装默认开启，每个 App 进程最多自动检查一次；用户可在设置中关闭，也可手动触发 |
| 目标 | 只请求公开发行仓库的 `https://api.github.com/repos/CarGuo/AmuxAICleanRelease/releases/latest`。App 会校验并解析同仓库、同 tag 的 GitHub Release 页面与 asset 元数据，但不会自动下载 DMG、`*.sha256` 或其它 Release asset |
| 方法与数据 | `GET`，无请求体；包含 HTTP 必要头以及标识 App 版本的 `User-Agent` |
| 本地数据 | 不发送扫描路径、文件内容或清理记录 |
| 接收方 | GitHub；GitHub 可能按其政策处理 IP 地址、请求时间和 User-Agent |
| 关闭方式 | 设置中关闭“启动时自动检查更新”；手动检查仍只在用户点击后发生 |

### 2.2 可选 AI 助手

AI 助手默认关闭。只有用户在设置中主动开启、配置模型端点，并对某个项目发起询问或启动 Smart Care AI 审核时才会发送请求。

| 项目 | 真实行为 |
|---|---|
| 默认端点 | `https://api.openai.com`；用户可改为其他 OpenAI-compatible HTTPS 服务 |
| 本地端点 | 仅环回地址允许使用明文 HTTP，例如 `http://localhost`；非环回端点必须使用 HTTPS |
| 端点约束 | base URL 必须有明确 host，且不得携带 userinfo、query 或 fragment；请求不跟随 HTTP 重定向，服务商若迁移地址，用户需配置最终端点，避免凭据和正文被转送到另一个 origin |
| 请求认证 | 如端点需要认证，API Key 通过 `Authorization: Bearer …` 发送给该端点。该凭据可识别用户在服务商处的账户或项目，因此隐私清单按 Apple 的 `User ID` 类型保守披露；本项目不读取服务商账户资料，也不把 Key 写入日志 |
| 发送字段 | 所选路径、是否目录、占用字节数、文件计数、修改时间、本地 Safety verdict、匹配到的 vendor 名称/说明、最多 5 个直接子项名称、用户配置的 model/temperature、`response_format=json_object`，以及固定的 system/user 提示模板 |
| 不发送 | 文件内容、完整目录树、运行日志、撤销历史、白名单或其他未被询问的扫描项 |
| 路径脱敏 | 默认开启，会替换用户名、邮箱样式片段、UUID 和非系统默认的自定义卷名；用户可在明确警告后关闭。脱敏降低暴露程度，但不保证数据匿名化 |
| 会话存储 | 使用独立 ephemeral `URLSession`，禁用 Cookie 接收/发送、URL cache 和共享 credential storage；模型响应不会形成 App 管理之外的持久网络缓存 |
| 错误诊断 | 本地日志保留经验证的端点 host、阶段、HTTP status/响应字节数，或 transport domain/code；不记录请求正文、API Key、服务端错误正文或模型响应正文 |
| 接收方与保留 | 由用户配置的模型服务商决定；其日志、保留、训练和账户政策不受本项目控制。使用前请查看该服务商政策 |

AI 返回值只用于解释和建议，不能提高本地安全等级、绕过 `SafetyEngine` 或直接授权删除。

### 2.3 用户主动打开的网页

App 可能把经仓库归属验证的 Releases 页面、隐私政策、条款或 Issues 链接交给默认浏览器
打开。这是用户点击后的浏览器访问，不是后台遥测；浏览器后续下载及其重定向由浏览器管理，
不属于 App 的自动下载链路。

### 2.4 重复文件的本地内容比对

重复文件页面默认检查 `~/Documents`、`~/Downloads` 与 `~/Desktop`，用户也可以主动加入
其它目录。它先按大小筛选，再读取候选文件前 64 KiB 计算 SHA-256；仍然冲突且不超过
256 MiB 的候选会读取完整文件计算 SHA-256，超过 256 MiB 的候选只形成“前缀匹配”证据。
用户确认把重复项移入废纸篓前，执行闸门会再次读取目标与保留参照的完整内容并校验哈希及
文件身份，内容或身份变化会拒绝提交。

这些读取全部在本机进行。App 不解析文件业务内容，不把文件字节或哈希上传给更新服务或
AI 服务，也不把重复文件哈希写入上述 JSON、偏好、日志或导出文件。读取失败、权限拒绝、
取消或扫描不完整会保留为类型化 incomplete，而不是被解释成“没有重复项”。

### 2.5 本地结构化解析、Quick Look 与网络支持存储

- 卸载与孤儿发现会读取 App 的 `Info.plist` 及相关安装元数据；启动项页面会解码用户选择
  范围内的 launchd plist；DerivedData/开发者构建产物与 Homebrew 页面会读取其
  `Info.plist`、receipt 或 JWS/JSON 等结构化记录。这些解析用于本地分类、归属和完整性
  判断，不会把文件正文上传。Brew/DerivedData 的诊断可能包含相关绝对路径、闭集
  source/stage/reason/kind、error domain/code 或 POSIX errno、rejected-entry count 与
  hashed-fallback；不记录 receipt/JWS/plist 原始正文或 `localizedDescription`。缺失保持正常
  布局/cache 状态；已有记录不可读或损坏时进入本地技术日志，并在没有独立分类证据时形成
  类型化 incomplete，不能伪装成“没有更新”或“不是工程”。
- 用户主动点击 Quick Look 时，App 把选中文件 URL 交给 macOS `QLPreviewPanel`。预览的
  解码、缓存和保留由 macOS/Quick Look 管理，不受 App 的“清除日志”控制；若文件是
  iCloud/FileProvider 占位文件或位于网络卷，预览可能触发下载或远端读取。
- 当用户扫描 SMB/NFS 等网络卷、FileProvider/iCloud 目录或其它按需下载位置时，即便 App
  只调用文件系统 API，读取元数据、内容哈希或预览也可能联系相应服务器/提供方并下载数据。
  App 不另行把这些数据转发给更新服务；是否发送到 AI 仍只取决于用户是否主动发起第 2.2
  节的请求。

### 2.6 用户主动复制

App 不读取系统剪贴板。用户点击页面中的“复制路径 / 复制名称 / 复制结果 / 复制 Bundle ID”
等操作时，当前界面显示的相应文本会写入 `NSPasteboard.general`。通用剪贴板由 macOS 管理，
在被覆盖前可能被其它获准访问剪贴板的 App、通用剪贴板或系统历史能力读取；App 不跟踪、
上传或自动清除该副本。

### 2.7 普通模式的系统状态

<!-- v2026.09.06：新增只读状态卡，明确本机计数与网络请求的区别。 -->
普通模式在 App 活跃时读取本机 CPU 时间、虚拟内存统计、磁盘容量、内置电池状态、运行时间、
系统热状态，以及活动 Wi-Fi/以太网接口的收发字节计数。采样和最近最多 60 秒的曲线仅保留在
进程内存，不写历史文件，不读取网络数据包内容，不上传，也不为采样发送网络请求。

Normal mode reads local CPU time, memory statistics, disk capacity, battery status, uptime, thermal
state, and byte counters for active Wi-Fi/Ethernet interfaces while the app is active. Samples and
up to 60 seconds of chart history remain in process memory. They are not persisted or uploaded;
sampling neither reads packet contents nor sends network requests.

## 3. 本地数据存储

<!-- v2026.10.14：生产撤销历史改用 SQLite；披露旧 JSON 原始字节归档及退役标记，避免把最近批次保留误写成全部迁移证据的保留上限。 -->

以下列出 App 管理的本地状态与用户主动导出的副本。除表中明确写为“用户选择位置”的导出外，
默认路径都位于当前 macOS 用户账户内：

| 位置 | 内容 | 保留策略 |
|---|---|---|
| `~/Library/Application Support/AmuxAIClean/heuristic.json` | 标准化绝对路径及其祖先前缀、删除/恢复/保留计数和更新时间 | 直到用户清除 |
| `…/clean_stats.json` | 清理事件时间、字节数和类别；事件最多保留 365 天，另保留累计总量 | 直到重置或清除 |
| `…/whitelist.json` | 用户加入白名单的绝对路径、可选原因和时间 | 直到移除或清除 |
| `…/UndoHistory.json.sqlite` | 生产撤销历史数据库：最近 10 个移入废纸篓批次的原路径、废纸篓路径、文件身份和恢复状态；另含不带路径的 SHA-256 幂等 receipt，用于跨崩溃去重 | 可见批次超限时移除最旧记录；确认流程先在 completion journal 耐久保留 `receiptPrunePending` 证据，再清 receipt，最后才移除 journal 项，任一崩溃阶段由启动恢复继续 |
| 同一数据库内的旧历史导入证据 | 升级时验证过的旧 `UndoHistory.json` 精确原始字节、文件身份、已导入记录/receipt 标识和退役意图；原字节可能包含旧历史中的绝对路径、恢复状态及经证据验证的中断写入片段 | 为避免迁移时丢弃恢复证据而保留，不随最近 10 批可见历史淘汰；当前无独立自动过期，直到用户按第 4 节清除 App 数据 |
| `…/UndoHistory.json` | 从 1.6.2 及以后升级时的旧 JSON 历史源；源数据先耐久导入 SQLite，再原位退役为 schema 5 manifest，包含存储格式、数据库文件名和迁移 ID | 退役后保留标记，使旧 writer 拒绝继续写旧格式；SQLite 提交与文件退役分步执行，中断后按已提交证据继续，不丢弃未知损坏或覆盖外部替换对象 |
| `…/UndoHistory.json.sqlite-journal` | 系统 SQLite 在 DELETE journal 模式下使用的事务回滚文件，可能包含同一数据库的历史页数据 | 由 SQLite 在提交或恢复时管理；异常退出后可能留待恢复，运行中不得单独删除，不能当作普通临时垃圾清理 |
| 同目录旧 `.UndoHistory.schema4-cas.stage-0`、`.UndoHistory.schema4-cas.stage-1`、`.UndoHistory.schema4-cas.marker-0`、`.UndoHistory.schema4-cas.marker-1` 及 `.UndoHistory.json.sqlite-cutover.<UUID>.tmp` | 旧 JSON/WAL 的历史图像、摘要/身份和事务状态，以及首次发布退役 manifest 的临时文件；旧 stage 可能包含历史路径，manifest 临时文件只含退役信息 | 生产新写入不再使用旧 WAL；升级导入可读取旧证据，不将它们按普通缓存删除。manifest 临时文件正常发布或失败时清理，异常退出可能残留；均可在退出 App 后随 Application Support 目录清除 |
| `…/trash-pending-operations.json` | 未完成 Trash 操作的原路径、冻结文件身份、恢复范围、operation/attempt、进程 owner 与阶段，用于崩溃后对账 | 对账到可信终态后清理；已提交但未知的记录 fail closed |
| `…/simulator-managed-operations.json` | CoreSimulator 托管操作的 simulator identifier 与 kind、attempt、进程 owner、scan/review identity、阶段、时间和可选退出码；不保存普通文件路径或 inode | 权威 absent、可重试失败或对应终态收敛后按状态机清理 |
| `…/trash-completion-journal.json` | 已由 macOS 确认移入废纸篓、但尚未完成撤销事务收尾的短期交接记录；包含操作/尝试 UUID、原路径、废纸篓路径、恢复范围、时间、文件身份，以及已确认 ledger、等待清理 receipt 的阶段标记 | 撤销历史耐久写入、对应 pending ledger 精确确认、receipt 清理全部完成后才删除相应记录（JSON envelope 可以继续作为空文件存在）；写入或对账失败时保留，供下次启动在跨进程恢复锁内幂等重放，不能按普通缓存提前丢弃 |
| 上述 Trash/Simulator JSON 及 `UndoHistory.json` 旁的 `.lock` 文件 | 跨进程 `flock` 所需的稳定空载荷 inode；SQLite 历史继续使用旧 history lock 协调导入与写入，不保存路径、身份或用户内容 | 可随所属 Application Support 目录一起清除；运行中不得单独删除或替换 |
| 同目录的 `.trash-pending.<UUID>.tmp`、`.trash-completion.<UUID>.tmp`、旧 `.UndoHistory.<UUID>.tmp` | 两个短期 Trash JSON 原子替换前及旧 Undo JSON writer 遗留的 `0600` 临时文件，可能包含同一批绝对路径、文件身份和操作状态；生产 SQLite 不用这些文件提交历史 | 正常 JSON 写入会 rename 或失败时 unlink；若进程在两者之间被强制终止，下一次访问对应 store 时会在其稳定跨进程锁内只按精确前缀 + UUID + `.tmp` 形状清理，避免误删相似文件或与活跃 writer 竞态 |
| `…/sentinel-pending.json` | 用户启用 Sentinel 后，本地记录热点变化的绝对路径、检测时间和可选大小；相同路径去重，最多 100 条 | Sentinel 运行时滚动覆盖；关闭 Sentinel 不自动删除，直到用户清除 App 数据 |
| `$TMPDIR/AmuxAIClean-CommonCache-<UUID>/request.json` | 隔离 Common App Cache worker 的短期请求，包含 HOME 绝对路径、已展开的 vendor/path rules 和扫描选项；文件权限 `0600` | 正常路径在启动 worker 前先打开只读 fd 并立即 unlink，worker 只继承 fd；若进程恰在“创建后、unlink 前”异常退出，命名临时目录可能留待 macOS 临时目录管理或用户确认后清理 |
| `~/Library/Preferences/com.amux.ai.Clean.plist` | 语言/界面/扫描偏好、自定义扫描绝对路径、更新设置、AI 开关/端点/模型/温度/脱敏选项等 | 直到重置或清除 |
| macOS Keychain：service `com.amux.ai.Clean.ai.apiKey`、account `default` | 用户保存的 AI API Key；明确标记为不同步 | 直到在设置中删除或手动清除 |
| `~/Library/Logs/AmuxAIClean/AmuxAIClean-YYYY-MM-DD.log`、`…/AmuxAIClean-Sentinel-YYYY-MM-DD.log` | 主 App 与用户态 Sentinel 的用户可读本地诊断日志；包含扫描/删除/恢复等业务阶段、结果、技术错误，并可能包含绝对路径，但不记录 API Key、AI 请求正文、服务端错误正文或模型响应正文。主 App 后台初始化 file sink 期间最多 256 行同类内容暂存在进程内存；正常写入还会把上一次可见路径身份核验后的日志保留在最多 4 MiB 的进程内 replay window，每 256 行或 1 秒重新核验。若日志 pathname 被 unlink 或普通文件替换，会安全重绑并按原顺序重放；symlink/其它异物保持 fail closed。窗口超限不静默：恢复安全路径后写入 `file_log_visibility_replay_overflow` 及丢失行/字节计数。禁用或替换 sink 时，旧 sink 会按 FIFO 完成已接受写入、最终可见性核验和关闭 | 按天写入，目标保留 7 天：主 App 在日志系统初始化、跨日后的下一次写入，以及设置页/Help 验证日志目录并重试待处理保留操作时清理；长期运行的 Sentinel 在启动、每小时和休眠后的下一次写入时清理。不是文件一满 7 天就由后台硬删除 |
| macOS Unified Logging（subsystem `com.amux.ai.Clean`） | 主 App 同时写入系统日志；普通 `AmuxLog` 事件的整条预格式化消息使用 private hash。本地文件 sink 初始化、打开、写入、访问准备、枚举、导出、清理或保留失败时，独立 `LogFileSink` category 只公开白名单化的 `stage / error domain / numeric code`，设置页或 Help 入口也显示同一闭集诊断。写入健康与保留/枚举/导出等维护问题分开显示，不会用“仍能写入”掩盖维护失败；stderr 只写同一闭集，不写 raw Error 或路径。Sentinel 本地日志失败只写固定失败标记 | 保留、收集和清理由 macOS 管理；App 内“清除日志”不清除此系统副本 |
| 用户选择的 Large Files `.json` / `.tsv` 导出位置 | 当前可见大文件行的绝对路径、字节数、修改时间和类型 | 由用户选择位置并长期保留；App 不跟踪也不自动删除 |
| 用户选择的 `AmuxAIClean-Logs.txt` 导出位置 | 导出当时通过完整性校验的主 App 文件日志合并副本，可能包含绝对路径和技术错误；以固定 64 KiB 块写入同目录 `0600` 随机临时文件，`fsync` 后原子替换所选目标，不把全部日志同时载入内存 | 由用户选择位置并长期保留；后续“清除日志”不删除已导出的副本 |
| `/tmp/com.amux.ai.Clean.sentinel.{out,err}.log` | 历史版本 root LaunchDaemon 启用时可能留下的 stdout/stderr；可能含 `/var/root` 下路径和技术错误，当前用户 Agent 不写这些位置 | 旧版不会自动清除；确认历史 root 服务已注销后可用管理员权限单独删除 |
| `/var/root/Library/Application Support/AmuxAIClean/` | 历史版本 root Sentinel 把 `homeDirectoryForCurrentUser` 误解析到 `/var/root` 后可能创建的目录或 `sentinel-pending.json` | 当前 App 以普通用户运行，不能声称自动检查或删除；确认旧服务注销后由管理员按第 4 节检查并清除 |

设置页的“清理日志”只管理经逐级 `O_NOFOLLOW` 目录校验、精确日期文件名、普通文件且
`link count == 1` 共同确认的主 App 日志；同名目录、符号链接、硬链接、Sentinel 日志、Unified
Logging 和已经导出的副本都不在该操作范围内。清理成功后会重新开始当日日志，并把“清理已完成”
本身写入新日志，因此界面不再宣称会留下一个空文件。失败时不会显示“已清理”，而会显示上述
阶段、错误域和错误码。

当前版本启用 Sentinel 时，会通过 macOS ServiceManagement 注册用户域 Login Agent
`com.amux.ai.Clean.sentinel.agent`。历史版本曾使用 root Label `com.amux.ai.Clean.sentinel`。
App 包内为该历史 Label 保留“文件名与 Label 对应”的 migration-only LaunchDaemon 描述符，
仅用于尝试读取状态并注销已经存在的注册；代码没有重新注册旧服务的路径。真实升级以目标
机器上 ServiceManagement 返回的权威状态为准。

用户期望开启 Sentinel 时，只有历史 root 服务明确返回 `notRegistered`，当前用户 Agent
才可继续注册，避免新旧服务并存；`notFound` 表示系统框架无法定位描述符/服务，不能当作
“已经注销”。用户期望关闭时，当前 Agent 的注销属于降权动作，不会被旧描述符查询失败
阻断；但旧 root 状态未验证时，界面仍必须显示“关闭但未完全验证”，不能把 toggle 为 OFF
写成所有后台项已停。系统注册状态不是普通偏好文件；删除 defaults、JSON 或日志都不能代替
`SMAppService.unregister()`。

首次安装、Sentinel 保持默认 OFF，且没有成功注册时间、已注册系统状态或持久化关闭核验
标记等 authority 证据时，`.notFound` / `.unknown` 查询失败本身不能证明后台服务存在。App 会保留
“状态未验证”的界面事实并写结构化本地日志，但不弹出“关闭失败”，也不要求这类从未开启的用户
进入系统设置。用户显式执行 OFF，或 App 已观察到 current/legacy authority 活跃、成功注册或持久化核验证据时，
查询失败仍会 fail closed，并用可操作提示说明无法确认后台项已停止。

旧版 root 服务的根因是它把 `homeDirectoryForCurrentUser` 解析为 `/var/root`，并不能正确
监听已登录用户的废纸篓和热点目录。当前版本不会把该旧域当成可用 Sentinel；迁移注销失败时
会停止注册新 Agent，并在 UI 中明确呈现失败，而不是让两个后台服务同时运行。

废纸篓中的实际文件由 Finder/macOS 管理，不属于上述 App 数据。删除 App 数据不会自动清空废纸篓，也不会恢复其中项目。

## 4. 清除本地数据

如果曾启用 Sentinel，先保持 App 打开，在“设置 → 高级”关闭 Sentinel，并确认状态显示
“未注册 / Not registered”。这是调用 `SMAppService.unregister()` 的必要步骤；不要先删除偏好，
否则 UI 开关会消失，但系统注册的后台服务可能仍继续运行。若注销失败，请先按 App 提示处理，
不要把文件已删除误认为后台服务已停止。升级自旧版时，这一步也会检查并尝试注销历史
root LaunchDaemon；设置页必须把当前 Agent 与历史 root 状态分开验证。

确认 Sentinel 已注销后退出 AmuxAIClean。确认不再需要撤销历史、白名单、清理统计、设置和
本地日志，再在终端分别执行。清除 Application Support 目录会同时移除 SQLite 历史、数据库内
保留的旧 JSON 原始字节、退役 manifest、旧 WAL 遗留和锁文件；只清除可见历史或日志不等于
清除了这些迁移证据，运行中不要拆开删除数据库与事务文件：

```bash
rm -rf "$HOME/Library/Application Support/AmuxAIClean"
rm -rf "$HOME/Library/Logs/AmuxAIClean"
defaults delete com.amux.ai.Clean
security delete-generic-password -s com.amux.ai.Clean.ai.apiKey -a default
```

某项本来不存在时，相应命令可能报告“未找到”，不代表其他项目未清除。上述命令不会删除
用户另存的大文件/日志导出、系统通用剪贴板内容或 Quick Look/macOS 管理的预览缓存，也不会
清除 macOS Unified Logging；导出文件需由用户在其选择的位置自行删除，剪贴板可通过复制其它
内容覆盖，系统预览/日志由 macOS 的保留与管理机制控制。完全磁盘访问、App 数据访问、
App 管理、桌面/文稿/下载、可移动卷和网络卷等系统授权需在“系统设置 → 隐私与安全”中
由用户单独撤销。

若设备上曾运行并启用过历史版本 root Sentinel，确认后台服务均已注销后，可由管理员
检查并清理 root 域遗留；这些命令不应在旧服务仍运行时执行：

```bash
sudo rm -f /tmp/com.amux.ai.Clean.sentinel.out.log
sudo rm -f /tmp/com.amux.ai.Clean.sentinel.err.log
sudo rm -rf "/var/root/Library/Application Support/AmuxAIClean"
```

当前用户 Agent 不会新建这些 root 域路径。命令使用 `sudo` 是因为历史文件可能属于 root，
不是授权当前 App 获得 root 删除能力。

## 5. 权限与访问范围

| 权限/系统提示 | 用途 | 拒绝后的行为 |
|---|---|---|
| 完全磁盘访问 | 读取 macOS 受 TCC 保护且本产品被允许分析的位置，以形成更完整的磁盘占用结果 | 保持可用，但明确显示部分结果和不可访问位置，不把权限失败伪装成 0 项 |
| 其他 App 数据访问（`NSAppDataUsageDescription`） | 读取其他 App 数据容器的元数据；若用户把相应目录加入重复文件扫描，还会在本机读取候选文件字节计算 SHA-256 | 相关容器不可读，结果标记不完整 |
| App 管理访问（`NSAppBundlesUsageDescription`） | 读取已安装 `.app` 的 Info.plist 等元数据；若用户把相应目录加入重复文件扫描，还会在本机读取候选文件字节计算 SHA-256 | 相关 App 元数据可能缺失，结果标记不完整 |
| 桌面、文稿、下载（`NSDesktopFolderUsageDescription` / `NSDocumentsFolderUsageDescription` / `NSDownloadsFolderUsageDescription`） | 磁盘占用扫描读取文件系统元数据；重复文件扫描还会在本机读取候选文件字节计算 SHA-256 | 对应目录不可读，结果保留 scoped incomplete；不会把拒绝伪装成空目录 |
| 可移动卷、网络卷（`NSRemovableVolumesUsageDescription` / `NSNetworkVolumesUsageDescription`） | 当磁盘或自定义扫描包含这些卷时读取文件系统元数据；若用户把卷内目录加入重复文件扫描，还会读取候选文件字节计算 SHA-256。网络卷读取由文件系统联系远端服务器，不等同于 App 主动 HTTP 上传 | 对应卷不进入可信完整结果；已生成的其它卷结果不受影响 |
| 用户选择的文件夹 | 添加自定义扫描根 | 无法添加该目录 |
| 后台项目注册（Sentinel，可选） | 用户主动开启后，以当前登录用户的非特权 Login Agent 在本机监听废纸篓与清理热点并提示重新扫描 | 默认关闭；先清理旧版 root 注册，再注册/注销当前 Agent，并以 ServiceManagement 权威状态验证；失败会明确提示 |
| App 主动 HTTP | 上述更新检查及用户主动启用的 AI 请求 | 对应联网功能失败；本地安全判定不受影响 |
| 网络支持存储/系统代办 | 用户选择网络卷、云端占位文件或 Quick Look 时，文件系统、FileProvider、iCloud 或 Quick Look 可能远端读取/下载 | 对应文件可能不可读或预览失败，并形成 incomplete；不会自动转成 AI 请求 |

所有清理候选只在用户确认后通过 `NSWorkspace.recycle` 移入废纸篓；不使用直接抹除，也不因获得磁盘权限而自动删除。

Apple 当前的 [Privacy manifest files](https://developer.apple.com/documentation/bundleresources/privacy-manifest-files)
文档要求所有平台说明收集的数据，但把 Required Reason API 声明范围明确列为 iOS、iPadOS、
tvOS、visionOS 和 watchOS，不包含本项目唯一发行平台 macOS。因此当前 Developer ID macOS
清单保留可选 AI 的 collected-data、tracking 和 tracking-domain 事实：路径/名称上下文按
`Other User Content`、文件元数据按 `Other Data Types`、可识别服务商账户或项目的 API Key 按
`User ID` 保守披露，三者都只用于 App Functionality、视为 linked、且不用于 tracking。即使
某个服务商只在实时鉴权期间处理 token，项目允许用户配置其它兼容端点，无法替所有接收方保证
不保留，因此不能用实时服务例外低报。清单不写
`NSPrivacyAccessedAPITypes`。这不是声称源码没有读取时间、容量、uptime 或 UserDefaults；而是
避免把移动平台理由码当作通用 API 清单。尤其 `statfs` 还用于本地介质/只读卷分类，并不只符合
`85F4.1` 的“向用户显示磁盘空间”用途。如果将来增加其它 Apple 平台，或 Apple 扩大 macOS
适用范围，必须按当时源码的每一种真实用途重新选择批准理由，并同步修改清单、本文和发行测试。

## 6. 第三方依赖与安全

主程序仅使用 Apple 系统框架和仓库内第一方 Swift Package，不嵌入第三方分析 SDK。App 主动
HTTP 使用系统 `URLSession`；网络卷、云文件和 Quick Look 的传输由相应 macOS 框架或文件
提供方管理。非环回 AI 端点必须为 HTTPS且不跟随重定向，但服务商如何处理请求仍由该服务商
条款决定。

本项目采用 MIT License，关键实现可从以下位置审计：

- 更新：UpdateChecker.swift
- AI 请求：OpenAIChatClient.swift
- AI 字段：PathInquiry.swift
- Keychain：KeychainAPIKeyStore.swift
- 删除：TrashPipeline.swift

## 7. 儿童隐私、变更与联系

AmuxAIClean 不以 13 岁以下儿童为目标用户，也不建立用户账户或年龄档案。隐私行为发生实质变化时，应同步更新本文件、`PrivacyInfo.xcprivacy` 和 CHANGELOG 的 Security / Privacy 说明；不得只改文案来掩盖实现偏差。

问题与普通缺陷请通过 GitHub Issues 提交；安全漏洞请按 [SECURITY.md](SECURITY.md) 的私密上报方式处理。

---

## English summary

AmuxAIClean includes no telemetry, ads, behavioral analytics, or automatic crash reporting. Scanning, safety decisions, cleanup previews, and history are local by default.

The App itself creates HTTP requests for two workflows: update checks against GitHub Releases, and
the optional AI assistant. This does not mean that scanning a user-selected SMB/NFS share,
iCloud/FileProvider placeholder, or using Quick Look can never cause network I/O by macOS or the
storage provider. AI is off by default and runs only after the user enables it and initiates a
review. An AI request can include a selected path (redacted by default), file/directory metadata,
the local safety verdict, vendor context, and up to five direct child names; it never includes
file contents or a full directory tree. A configured AI base URL may not contain userinfo, query,
or fragment components, and the client does not follow HTTP redirects. The configured provider
receives the request under its own privacy and retention terms. AI output can never override local
safety policy or authorize deletion.

Local AI diagnostics retain the validated endpoint host, request stage, HTTP status and response
byte count, or transport domain and numeric code. They do not retain the request body, API key,
provider error body, or model response body.

When authentication is configured, the Bearer API key can identify the user's provider account or
project. The privacy manifest therefore conservatively declares it as linked `User ID`, alongside
`Other User Content` for path/name context and `Other Data Types` for file metadata. All three are
used only for App Functionality and never for tracking. Because users may configure another
compatible provider, the project cannot assume every recipient discards the credential immediately.

“Check for updates at launch” is enabled on first install and performs at most one automatic check
per App process; it can be disabled in Settings. Manual update checks remain user initiated. The
request targets `https://api.github.com/repos/CarGuo/AmuxAICleanRelease/releases/latest`, the public
distribution repository. The App validates Release-page and asset metadata from that repository, but it does
not automatically download a DMG, checksum file, or any other Release asset. A verified Release
page is opened in the default browser only after the user chooses to proceed.

Local JSON stores and rolling file logs can contain absolute paths. Large-file and combined-log
exports are written to a location selected by the user and are not removed when App data is
cleared. While the main-App file sink is initialized off the UI thread, up to 256 rendered log
lines are held in process memory; overflow drops the oldest lines and records a count, and a failed
initialization releases the buffer. During normal writing, lines accepted since the last visible
path-identity proof are also retained in a process-memory replay window capped at 4 MiB. The path is
revalidated every 256 lines or one second. If the pathname was unlinked or replaced by an ordinary
file, the sink safely rebinds and replays that window in order; a symlink or other foreign entry
remains fail closed. Capacity loss is made explicit by a
`file_log_visibility_replay_overflow` line with dropped-line and dropped-byte counts after a safe
path returns. Disabling or replacing a sink retains it until every accepted append, final visibility
fence, and close has completed in FIFO order. The main App also emits privacy-hashed messages to macOS Unified
Logging, whose retention is system-managed. If local initialization, writing, validation, retention,
enumeration, export, or clearing fails, a separate Unified Logging category and the Settings/Help UI
expose only a normalized stage, error domain, and numeric code—never the raw error or path. Writer
health and maintenance issues are reported independently. Combined-log export streams fixed 64 KiB
blocks into a sibling `0600` temporary file and atomically replaces the user-selected destination.
Settings clears only exact, no-follow, single-link regular main-App log files; it does not clear
Sentinel logs, Unified Logging, or prior exports. The clear event starts the new daily log, so the
replacement file is not described as empty. Seven days is an activity-driven retention target, not
an instantaneous hard expiry. Crash-interrupted atomic writes may briefly leave precisely named `0600` Trash/Undo temp
JSON files; the corresponding store removes them under its cross-process lock on next access.
The AI credential is stored separately in macOS Keychain. See Section 4 for local data and credential removal commands.

Duplicate scanning is local but is not metadata-only. It hashes the first 64 KiB of candidate
files, fully hashes remaining candidates up to 256 MiB, and performs a full content and identity
verification before a confirmed Trash submission. File bytes and duplicate hashes are neither
uploaded nor persisted by the App.

Some focused local workflows parse structured metadata such as application and launchd plists,
developer-build metadata, and Homebrew receipt/JWS JSON. User-invoked Quick Look delegates the
selected URL to macOS and may hydrate cloud or network-backed content. The App never reads the
clipboard, but explicit Copy actions write displayed paths, names, results, or bundle IDs to the
system general pasteboard, where system and other-app retention rules apply.

Brew and DerivedData diagnostics distinguish an expected missing file from an unreadable or
malformed record. Local diagnostics may contain the related absolute path and stable source,
stage, reason, error-domain/code, POSIX errno, rejected-entry count, or hashed-fallback fact, but
never the original receipt/JWS/plist body or a localized raw error description. Apple's current
privacy-manifest documentation requires collected-data declarations on all platforms, but lists
Required Reason API declarations for iOS, iPadOS, tvOS, visionOS, and watchOS—not this macOS-only
Developer ID target. The manifest therefore omits `NSPrivacyAccessedAPITypes` instead of treating
mobile approved-reason codes as a generic API inventory. A future platform or policy-scope change
requires a fresh source-use audit before any such reasons are declared.

The optional Sentinel background monitor is off by default. When enabled, the current service
(`com.amux.ai.Clean.sentinel.agent`) runs as a non-privileged login agent for the signed-in user,
locally watches that user's Trash and selected cleanup hotspots, and keeps up to 100
path/time/size events in `sentinel-pending.json`; it does not use the network. Historical builds
previously registered a root-domain label (`com.amux.ai.Clean.sentinel`), which resolved the wrong
home directory. The current app keeps a migration-only descriptor whose filename matches that
historical label, never registers it again, and requires it to be authoritatively unregistered
before enabling the new agent. Turning the current agent off is never blocked by an unavailable
legacy descriptor, but Settings remains unverified until legacy status is known. On a
fresh/default-OFF install with no durable, current, or legacy evidence of an active Sentinel
authority, a `.notFound` or unknown lookup result is recorded in structured local logs and may
remain visibly unverified, but it does not present a shutdown-failure alert or instruct the user to
open System Settings. An explicit OFF action or prior authority evidence remains fail-closed when
shutdown cannot be verified. See Sections 3 and 4 for data and credential removal.
