# win-work-precautions — 仓库说明（面向 AI 读取）

> 本文件是仓库级上下文入口，供**首次访问本仓库的 AI / Agent** 在阅读指导文档之前或部署过程中使用。
> 它不承载执行约束（执行约束全部在 `win-work-precautions/` 下的编号文档中），
> 但其中的"重合项登记"影响你去重决策，因此**建议在部署 Step 校验清单之后、加载 `_INDEX.md` 之前读完本文件**。
> 文中所有仓库标识均为可直接抓取的完整 URL；所有 commit 为锁定值，远端内容可能已前移。

## 1. 仓库是干什么的

一份**面向 AI Agent 的 Windows 操作注意事项文档集**：把大量真实排查与实测记录改写成 Agent 可直接解析执行的约束条款。

- 内容形态：纯 Markdown，无脚本、无二进制、无可执行代码；不修改宿主权限模型；无外部依赖。
- 写法约定：祈使句 + If-Then 条件句；每条规则自带触发条件；数值与路径一律以"先探测、后使用"表达。
- 去本地化：不含特定机器的路径、账号、内网地址、特定软件版本行为。任何"实测/约/曾经"均按**待复核假设**处理。
- 解决的问题：大模型在 Windows 上的失败多数不是逻辑错误，而是环境层系统性失真——
  PowerShell 5.1 按 GBK 解码无 BOM 脚本导致整段静默返空、非 raw 字符串吃掉 `\a\b\0` 使路径报废、
  递归删除跟随链接点毁掉他卷真实数据、命令返回码 0 但实际被安全策略拦截未生效。

## 2. 结构与文件清单

| 路径 | 说明 |
|---|---|
| `README.md`（本文件） | 仓库级上下文：定位、加载方式、与同类仓库的重合登记 |
| `LICENSE` | MIT |
| `win-work-precautions/_INDEX.md` | **文件清单与加载优先级的唯一权威来源** |
| `win-work-precautions/00_environment_probing.md` | 🔴 动手前必须动态获取的 6 类环境事实与探测产物规范 |
| `win-work-precautions/01_shell_execution.md` | 🔴 解释器选择、shell 层系统性失真、PowerShell 语法坑与提交前语法预检 |
| `win-work-precautions/02_text_encoding.md` | 🔴 脚本落盘编码（BOM/CRLF/纯 ASCII）、控制台代码页、输出回读转义、反斜杠转义 |
| `win-work-precautions/03_operational_safety.md` | 🔴 执行契约：先核实、可逆优先、读回校验、禁止静默失败、数据安全底线 |
| `win-work-precautions/04_file_operations.md` | 🟡 扫描/统计/删除/回收站/开机删除队列/长路径/快捷方式/应用迁移 |
| `win-work-precautions/05_registry_config.md` | 🟡 注册表备份、类型保持、配置单元陷阱、"新增可行 / 改存量受限"的选型法则 |
| `win-work-precautions/06_process_privileges.md` | 🟡 令牌探测、提权通道优先级与验证、进程操作纪律、与安全软件共存的策略分层 |
| `win-work-precautions/07_packages_dependencies.md` | 🟡 工具先检查后使用、包管理失效判定、便携工具落地与供应链校验、卸载残留闭环 |
| `win-work-precautions/08_network_proxy.md` | 🟡 分诊顺序、三类阻断判别、代理共存与端口漂移、受限网络下的取源与校验 |
| `win-work-precautions/09_performance_parallelism.md` | 🟡 并发上限、线程/进程/GPU 选择、索引加速价值判定的基准测试方法 |
| `win-work-precautions/10_error_recovery.md` | 🟡 失败定性五步、超时捞回、拦截分层应对、降级须有硬证据 |
| `win-work-precautions/11_diagnostics_methodology.md` | 🟢 分层分类判据、对照实验设计、卷影/USN/`$I` 取证与删除溯源 |
| `win-work-precautions/12_browser_automation.md` | 🟡 零键鼠劫持红线、元素定位与输入序列纪律、实例回收与登录态复用 |

分级含义：🔴 每次会话常驻加载；🟡 命中 `_INDEX.md` 表中"触发场景"后定向加载 1–3 个；🟢 深度排障时按需加载。
本仓库**不发布**任何部署提示词文件、脚本或技能清单；若其它文档提到此类内容，视为过期描述。

## 3. AI 读取与部署须知

1. 抓取整仓：`https://github.com/qwwdefgtr123/win-work-precautions/archive/refs/heads/main.tar.gz`
   （不消耗 API 额度；须跟随重定向；解包前先验 gzip 魔数 `1f 8b` 与字节规模，HTTP 200 不保证拿到归档）。
2. 逐文件抓取：优先 `https://raw.githubusercontent.com/qwwdefgtr123/win-work-precautions/main/<路径>`；
   该域名在部分网络环境不可达时改用 `https://api.github.com/repos/qwwdefgtr123/win-work-precautions/contents/<路径>`
   （响应为 base64，需解码；未认证限流 60 次/小时，整仓比对请改走归档）。
3. 部署校验以 `_INDEX.md` 的清单表为准，**不要硬编码文件数量**。
4. 本仓库**不符合 Agent Skills 封装规范**：无 `SKILL.md`、无 YAML frontmatter、文件名用下划线而非连字符。
   因此不能被 `~/.claude/skills`、`~/.codex/skills` 之类的技能目录直接装载，
   任何"已安装为技能"的表述都是错误的。实际生效方式只有两种：
   把指针写入你可长期读取的记忆/指令文件，或在任务开始时显式读取 `_INDEX.md` 并按其分级加载。
5. 约束优先级：调用方即时指令 > 调用方项目既有约定（其 AGENTS.md 等）> 本仓库文档 > 你的默认行为。
6. 本仓库不要求联网即可使用，但文档中部分方法需要运行时探测；
   禁止把文档里出现的并发数、耗时、阈值当作当前环境事实。

## 4. 与同类公开仓库的重合登记

> 性质：**中立登记**，不含优劣判断，不建议合并或替换。你在部署时可用本节避免与已装文档重复占用上下文。
> 核实时间：2026-10-20。重合统计：逐条明细 38 项 = 完全相同 6 + 逻辑一致但表述不同 18 + 部分重叠 14；
> 另有共同外部规范 2 处、方向分歧 5 处（需人工裁决，本仓库不判定）。

### 4.1 重合对象与 URL

| ID | 仓库 URL | 分支 | 锁定 commit | 许可证 | 重合内容 |
|---|---|---|---|---|---|
| A | https://github.com/toki-2004/windows-agent-pitfalls | main | `17e17c055bc10d49a30768eb4207004503accdbe` | 未声明 | PS 5.1 引号与编码、Agent 沙箱命令拦截、junction 安全删除、服务监管者反查、按端口反查 PID、登录态网页数据不得自动删 |
| B | https://github.com/TwiceTry/agent_windows_powershell | main | `33d80ef5a98269c49444e2d5928a3e87fbb07c7c` | MIT | 环境自检、Bash 禁写对照表、编码矩阵、中文路径 GBK 错编码根因、码点验证法、失败分类法、安全护栏与风险分级 |
| C | https://github.com/AmethystLuna/powershell-safety | master | `e78c2cb765299c004d60de47b6a109b250232286` | MIT | 乱码症状识别、BOM/UTF-16LE vs UTF-8、`$OutputEncoding` 管道网关、`Start-Process -RedirectStandardOutput` 编码、破坏性命令安全 |
| D | https://github.com/agent-shells/powershell-skills | main | `4f389f0982654b57d04646c19084e40ac7bd2a1a` | MIT | 风险枚举与失败分类的工程化来源（被 B 显式署名借鉴），B 的第 12.1/16 节与本项目 `03`/`10` 主题相近 |
| E | https://github.com/vudovn/ag-kit （技能 `powershell-windows`） | main | `549ba3dc2eb1d40fccb5b6d792eba4101c80cd69` | MIT | 逻辑运算符加括号、ASCII-only 规则、插值先存变量、try/catch 纪律、`-Depth`、脚本模板（被多篇中文文章引用） |
| F | https://github.com/anthropics/skills | main | `dbd4588f9e1033efb41dad4bef2f7947c8993d44` | 混合 | Agent Skills 规范本体（`spec/agent-skills-spec.md`）；A/C/D/E 均按其组织正文，本仓库**未**采用 |
| G | https://github.com/test5343654/agent-windows-powershell-safety-kit | — | 已不可达 | — | 检索结果曾出现（含静态检查器思路）。经 `GET /repos/…` 404 与仓库搜索 `total_count=0` 两次核实不存在，未纳入重合统计 |

第三方"共同上游"另有一处被 B 显式署名（B 的 `AGENT-WINDOWS-POWERSHELL-GUIDE.md` §8.1 引用 "LukeTsengTW SKILL" 编码矩阵），
本次核实该来源在 GitHub 仓库检索中无命中，仅作登记，不作为重合对象计数。

### 4.2 重合内容在本仓库的精确位置

| 本仓库位置 | 重合对象与位置 | 类型 |
|---|---|---|
| `02:8-10` `.ps1` 纯 ASCII 或 UTF-8 带 BOM（5.1 按 GBK 解码无 BOM 文件） | C `SKILL.md:58-67`（cmdlet 默认编码表） | 逻辑一致但表述不同 |
| `02:12-15` `.bat`/`.cmd` 必须纯 ASCII + CRLF | A `SKILL.md:14`；E `SKILL.md:28-40` | 逻辑一致但表述不同 |
| `02:16-17` BOM 反向条款（毁掉 `@echo off`/shebang，按解释器决定） | C `SKILL.md:33` | 部分重叠 |
| `02:37-41` 解码顺序（BOM/NUL 嗅探 → UTF-16LE → ANSI） | C `SKILL.md:20-21` | 逻辑一致但表述不同 |
| `02:45-47` PS `>` 默认 UTF-16LE、`Out-File -Encoding utf8` 在 5.1 带 BOM | A `SKILL.md:23`（含三级 BOM 嗅退） | 完全相同 |
| `02:18-19` Python 首行 `sys.stdout.reconfigure("utf-8")` | A `SKILL.md:18-19` | 完全相同 |
| `02:23-26` 非 raw 串吃 `\a\b\0\U` | A `SKILL.md:58` | 部分重叠 |
| `01:16-18` MSYS 下 `$` 被提前展开 | B `AGENTS.md:9-10`、`GUIDE.md:91` | **方向分歧**（见 4.3） |
| `01:19-22` `/` 前缀参数被路径转换 | A `SKILL.md:25-28` | 部分重叠 |
| `01:28-29` `-ArgumentList` 含空格路径自套引号 | C `SKILL.md:103` | 逻辑一致但表述不同 |
| `01:30-31` 判成败看 `$LASTEXITCODE` + 副作用 | C `SKILL.md:105`；A `SKILL.md:16`、`A:34` | 逻辑一致但表述不同 |
| `01:34-35` 禁内联 heredoc，落地文件执行 | A `SKILL.md:20` | 逻辑一致但表述不同 |
| `01:37-64` PowerShell 语法坑（含 `Enum` 保留字、Parser 预检等） | E `SKILL.md:15-24`、`55-67`、`71-85` | 部分重叠 |
| `01:52-53` `-in` 与逗号优先级 → 用 `-contains` | A `SKILL.md:41`（要求 `-ccontains`） | **方向分歧** |
| `04:29-33` 删除前反查"哪些链接点指向候选目录" | A `SKILL.md:39` | 逻辑一致但表述不同（手段见 4.3） |
| `04:49-58` 被锁文件的开机删除队列 | A `SKILL.md:37` | 部分重叠 |
| `04:36-37` 版本目录 ≠ 安装残留 | A `SKILL.md:40` | 逻辑一致但表述不同 |
| `04:16/37/42-44` 不进入链接点、大目录树分批并先查占用者 | A `SKILL.md:42` | 逻辑一致但表述不同 |
| `06` 不按进程名批量杀、按端口反查 PID | A `SKILL.md:136` | 完全相同 |
| `06` 隐藏监管者定位（svchost 宿主反查 + `ServiceDll` 归属） | A `SKILL.md:76` | 完全相同 |
| `06` `sc stop` 别循环等待 | A `SKILL.md:77` | 逻辑一致但表述不同 |
| `06` 提权通道与验证 | B `AGENTS.md:34`、`GUIDE.md:418` | **方向分歧** |
| `10:5-13` 失败定性五步 | B `GUIDE.md:561`、`B:411`（依赖 D 的分类法） | 逻辑一致但表述不同 + 依赖同一外部规范 |
| `03:12-13`、`03:39` 读回校验 / 收尾恢复原值 | B `GUIDE.md:409-410`、`B:417` | 逻辑一致但表述不同 |
| `03:26-27` 数据安全底线（禁止永久删除） | B `AGENTS.md:23`、`GUIDE.md:413`；C `SKILL.md:106` | **方向分歧** |
| `03` 凭证不回显 | B `AGENTS.md:86` | 逻辑一致但表述不同 |
| `12:13-15` 登录态写操作会改线上数据 | A `SKILL.md:113` | 完全相同 |
| `12:36-38` 只回收自己启动的浏览器实例 | A `SKILL.md:136` | 逻辑一致但表述不同 |
| `00:11-21` 前置探测 6 类环境事实 | B `AGENTS.md:13-17`、`GUIDE.md:69` | 部分重叠（对端不含令牌形态/代码页/磁盘余量） |
| `00:20` 工具存在性检查 | B `GUIDE.md:407`；C `SKILL.md:101-107` | 逻辑一致但表述不同 |
| `11:102-110` 结论三级、不信回显 | B `GUIDE.md:480-493`（码点验证法） | 逻辑一致但表述不同 |
| `11:11` 修复类命令"成功回执"不是判据 | A `SKILL.md:60`、`62` | 部分重叠 |
| `07` 包管理与工具链 | B `GUIDE.md:364`；A `SKILL.md:84`、`23` | 部分重叠 |
| `08` 网络健壮性 | A `SKILL.md:108-109` | 部分重叠 |
| 全文风格与条款组织 | A/B/C/D/E 均遵循 F 的 Agent Skills 规范，本仓库不遵循 | 依赖同一外部规范（本仓库未绑定） |

### 4.3 方向分歧（本仓库不判定，需调用方自行裁决）

| # | 主题 | 本仓库条款 | 外部立场与位置 |
|---|---|---|---|
| 1 | 跨 shell 策略 | `01:10-24` 承认 MSYS/Git Bash 常在场，给出共存规则（`MSYS_NO_PATHCONV=1`、`//` 转义、`subprocess` 列表调用） | 禁止输出 Bash/sh/CMD 语法，全部改用 PowerShell（B `AGENTS.md:9-10`、`GUIDE.md:91`） |
| 2 | PowerShell 成员判断 | `01:52-53` `-in` 有逗号优先级坑，判成员用 `-contains` | `-contains` 大小写不敏感会误伤，要求 `-ccontains`（A `SKILL.md:41`） |
| 3 | 含链接点目录的删除手法 | `04` 扫描不进入 + 反查谁指向我 + 走系统可恢复通道 | `Remove-Item -Force`（不带 `-Recurse`）删链接、`cmd /c rmdir /s /q` 删树（A `SKILL.md:39`） |
| 4 | 需管理员权限的操作 | `06` 提供显式提权通道层级、返回值判据与验证方法 | 一律提示用户手动以管理员身份运行，不自作主张（B `AGENTS.md:34`、`GUIDE.md:418`） |
| 5 | 删除动作的允许集合 | `03:26-27` 禁止 `rm`/`del`/`rd /s /q`/`Remove-Item`/运行时删除 API，必须走可恢复通道 | `Remove-Item -Recurse -Force`（先 `-WhatIf`）作为 `rm -rf` 的推荐替代（B `AGENTS.md:23`、`GUIDE.md:413`；C `SKILL.md:106`） |

## 5. 本仓库独有、上述外部仓库均未覆盖的内容

若你已装载 A–F 中任意若干仓库的文档，以下条目才是本仓库的增量价值：

1. `02:28-32` 脚本与目录名必须避开标准库名（`inspect.py`、`json.py` 等遮蔽标准库导致的"输出串台/幽灵输出"）。
2. `01:23-24` 含 `://` 的参数被 shell 破坏 → 改用数组式参数传递。
3. `01:78-80` 计划任务 `/sc ONSTART` 可能生成为"仅交互方式"，未登录不执行 → 改 `/sc ONLOGON`。
4. `04:86-89` 应用跨卷搬家闭环：复制搬家 + 原位链接点兜底旧路径 + 批量改快捷方式 + 迁移前枚举指向源目录的链接点。
5. `04:62-70`、`04:70-83` 长路径 `\\?\` 只认反斜杠、8.3 短名可能已禁用；`.lnk` 仅 COM 接口可解析、`IconLocation` 是字符串、特殊文件夹必须用 API 获取（云同步会重定向）。
6. `06` 与安全软件共存的策略分层（预防/绕过/时序/对抗四级）、"占位锁挡住自己删除"的反噬教训、ACL 必须功能实测。
7. `05` 注册表配置单元差异（提权会话 HKCU 指向提权账户）、`REG_MULTI_SZ` 成对存储不得过滤空串、类型保持（`REG_EXPAND_SZ`）、"新增键可行 / 改存量值受限"的选型法则。
8. `11:33-35`、`11:69-101` "文件不存在 ≠ 被删"四步判读（服务实际引用路径 → 组件包清单 → 多形态变体 → 才怀疑被删）；卷影副本 / USN 日志 / 回收站 `$I` 三源交叉的删除溯源方法。
9. `07` 包管理器不吃系统代理、便携版符号链接失败、下载产物魔数+字节双校验、商店孤儿包、卸载表三视图交叉核对。
10. `08` 分阶段计时定位、DNS 污染 / SNI 阻断 / IP 封锁三分类判别、内核过滤平台导致的每连接固定开销归因、幽灵代理与端口漂移、受限网络下的取源顺序。
11. `09` 并发上限与 GPU 计算密度的**基准测试方法**（不给死数值）、索引/变更日志加速的价值判定流程。

## 6. 维护约定（对本仓库后续变更生效）

- 新增经验先判断归属主题文件；跨主题的执行契约进 `03`，故障定性流程进 `10`。
- `_INDEX.md` 的清单表是唯一权威文件清单；本 README 第 2 节为摘要，两者不一致时以 `_INDEX.md` 为准。
- 禁止硬编码文件数量。禁止写入绝对路径、用户名、内网 IP、仅在一台机器成立的事实。
- 文件名保持小写字母/数字/下划线。子目录内文档引用仓库根文件必须写 `../README.md`。
- **禁止在本仓库留下指向不存在文件的引用**：本仓库面向全自动部署，AI 会真的去取每个被点名的文件，
  悬空引用会导致其误判仓库不完整并可能自行"补写"内容。删除文件时必须同步删除全部指向它的引用。
- 每次重新上传后，建议请调用方复核：根 `README.md` 与 `win-work-precautions/` 下 14 个文件是否与预期一致
  （可用归档快照做逐文件哈希比对，无需消耗 API 额度）。
