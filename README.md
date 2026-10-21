# win-work-precautions — Windows 环境 AI Agent 注意事项文档集

面向 AI Agent 的 Windows 操作约束文档集。全部内容从真实排查与实测记录中提炼，**已完成去本地化**：不含特定机器的路径、账号、内网地址或个性化软件配置；所有环境数值均以"先探测、后使用"的条件句表达。

- 适用对象：任何需要在 Windows 上执行命令、操作文件、改配置、排故障的 Agent（Claude Code / Codex / Cursor / 通用 Agent 均可）
- 文档形态：纯 Markdown，无脚本、无依赖、不修改宿主权限模型
- 文件清单的**唯一权威来源是 `_INDEX.md`**（本 README 的目录表为其摘要，若两者不一致以 `_INDEX.md` 为准）
- 核实日期：2026-10-21

## 为什么需要它

大模型在 Windows 上的失败大多不是逻辑错误，而是**环境层的系统性失真**：PowerShell 5.1 按 GBK 解码无 BOM 脚本导致整段静默返空、非 raw 字符串吃掉 `\a\b\0` 使路径报废、`Remove-Item -Recurse` 跟随 junction 删掉他卷真实数据、命令返回码 0 却被安全软件拦截而未生效。这些问题各自有解，但缺少一份把它们**写成 Agent 可直接解析的执行约束**的文档。本仓库补的就是这一层。

## 目录结构

| 优先级 | 文件 | 内容 |
|---|---|---|
| 🔴 核心必读 | `00_environment_probing.md` | 动手前必须动态获取的 6 类环境事实与探测产物规范 |
| 🔴 核心必读 | `01_shell_execution.md` | 解释器选择、shell 层系统性失真、PowerShell 语法坑与提交前语法预检 |
| 🔴 核心必读 | `02_text_encoding.md` | 脚本落盘编码（BOM/CRLF/纯 ASCII）、控制台代码页、输出回读转码、反斜杠转义 |
| 🔴 核心必读 | `03_operational_safety.md` | 执行契约：先核实、可逆优先、读回校验、禁止静默失败、数据安全底线 |
| 🟡 场景触发 | `04_file_operations.md` | 扫描/统计/删除/回收站/开机删除队列/长路径/快捷方式/应用迁移 |
| 🟡 场景触发 | `05_registry_config.md` | 注册表备份、类型保持、配置单元陷阱、"新增可行 / 改存量受限"的选型法则 |
| 🟡 场景触发 | `06_process_privileges.md` | 令牌探测、提权通道优先级与验证、进程操作纪律、与安全软件共存的策略分层 |
| 🟡 场景触发 | `07_packages_dependencies.md` | 工具先检查后使用、包管理失效判定、便携工具落地与供应链校验、卸载残留闭环 |
| 🟡 场景触发 | `08_network_proxy.md` | 分诊顺序、三类阻断判别、代理共存与端口漂移、受限网络下的取源与校验 |
| 🟡 场景触发 | `09_performance_parallelism.md` | 并发上限、线程/进程/GPU 选择、索引加速价值判定的**基准测试方法** |
| 🟡 场景触发 | `10_error_recovery.md` | 失败定性五步、超时捞回、拦截分层应对、降级须有硬证据 |
| 🟡 场景触发 | `12_browser_automation.md` | 零键鼠劫持红线、元素定位与输入序列纪律、实例回收与登录态复用 |
| 🟢 扩展参考 | `11_diagnostics_methodology.md` | 分层分类判据、对照实验设计、"回执 ≠ 成功"、卷影/USN/`$I` 取证与删除溯源 |

> 上表为指导文档摘要；完整清单（含 ⚪ 非指导文档与各文件的触发场景）见 `_INDEX.md`，新增文件只需更新该表。

加载机制：Agent 读 `_INDEX.md`，常驻只读 🔴 四件，🟡/🟢 按任务命中"触发场景"后定向加载 1–3 个文件，避免一次性占满上下文窗口。

## 使用要点

1. 文档只提供**方法与判据**，不提供环境事实。文中任何"实测 / 约 / 曾经"一律按待复核假设处理。
2. 涉及第三方工具的条款，必须先执行存在性检查（如 `Get-Command`）再使用，禁止预设已安装。
3. 约束优先级：用户即时指令 > 项目既有约定（AGENTS.md 等）> 本文档集 > Agent 默认行为。
4. 本仓库**当前不符合 Agent Skills 封装规范**（无 `SKILL.md` 与 YAML frontmatter），因此不能通过技能目录直接装载；生效方式为"记忆/指令文件写入指针 + 按 `_INDEX.md` 分级加载"。
5. 部署与更新给自有 Agent 的提示词模板见 `_AGENT_PROMPT.md`（含取源降级、完整性校验、持久化登记与报告字段）。

## 维护约定

- 新增经验先判断归属主题文件：跨主题的通用执行契约进 `03`，故障定性流程进 `10`。
- 禁止写入绝对路径、用户名、内网 IP、特定软件版本行为等仅在一台机器成立的事实。
- 文件名保持小写字母 / 数字 / 下划线；新增文档必须同步登记进 `_INDEX.md` 的清单表（该表是文件集合的唯一权威清单）。

---

## 与同类公开文档集的重合项索引

> **性质**：本节为**已识别重合项的中立索引**，供协作者与下游 Agent 自查使用。不提供推荐意见，不做优劣判断，不合并、不删除、不裁定分歧。
> **核实时间**：2026-10-20 22:22（Asia/Shanghai）。对端内容此后可能已变更，引用前请以表中 commit SHA 重新拉取比对。

### 重合类型标签

| 标签 | 含义 | 判断提示 |
|---|---|---|
| `完全相同` | 条款语义与写法高度一致，近似逐字对应 | 多源自同一上游，改一处需同步另一处 |
| `逻辑一致但表述不同` | 同一条约束，措辞/示例/语言不同 | 二选一即可，重复仅增加上下文体积 |
| `部分重叠` | 主题相同但覆盖范围或深度不同 | 关注差集才是各自独有价值 |
| `依赖同一外部规范` | 非互相抄录，而是同引第三方规范 | 处理上游即可，不必两处维护 |
| `方向分歧` | 同一主题给出相反的默认取向 | **需人工裁决**，见末节，不可静默共存 |

### 对端项目登记表

| # | 项目 | 分支 | 锁定 Commit | 提交日期 | 许可证 | 正文文件 |
|---|---|---|---|---|---|---|
| A | `toki-2004/windows-agent-pitfalls` | main | `17e17c055bc10d49a30768eb4207004503accdbe` | 2026-09-12 | 未声明 | `SKILL.md`(138 行)、`README.md`(50 行) |
| B | `TwiceTry/agent_windows_powershell` | main | `33d80ef5a98269c49444e2d5928a3e87fbb07c7c` | 2026-08-28 | MIT | `AGENTS.md`(87 行)、`AGENT-WINDOWS-POWERSHELL-GUIDE.md`(772 行) |
| C | `AmethystLuna/powershell-safety` | master | `e78c2cb765299c004d60de47b6a109b250232286` | 2026-06-26 | MIT | `skills/powershell-safety/SKILL.md`(174 行)、`references/encoding-guide.md`(236 行) |
| D | `agent-shells/powershell-skills` | main | `4f389f0982654b57d04646c19084e40ac7bd2a1a` | 2026-07-02 | MIT | `core/pattern-catalog/powershell-parser.md`、`adapters/{claude-code,codex}/powershell-command-runner/SKILL.md` 等（58 文件） |
| E | `vudovn/ag-kit`（skill `powershell-windows`） | main | `549ba3dc2eb1d40fccb5b6d792eba4101c80cd69` | 2026-08-31 | MIT | `.agents/skills/powershell-windows/SKILL.md`(169 行) |
| F | `anthropics/skills`（Agent Skills 规范） | main | `dbd4588f9e1033efb41dad4bef2f7947c8993d44` | 2026-10-09 | 混合 | `spec/agent-skills-spec.md`；全仓 445 文件中检索 `powershell`/`windows` 命名的 skill：**无命中** |
| G | `test5343654/agent-windows-powershell-safety-kit` | — | **不可达** | — | — | 搜索结果曾列出该项目（含静态检查器 `scripts/Invoke-AgentPwshSafetyCheck.ps1`）。本次经 `GET /repos/... → HTTP 404` 与 `search/repositories?q=... → total_count=0` 两次核实均不存在，仅作线索保留，**未纳入下方重合登记**。 |

### 重合明细

本项目侧行号基于 `00`–`12` 与 `_INDEX.md` 的当前磁盘版本；对端侧行号基于上表锁定 commit，两侧均可精确复现。

**`02_text_encoding.md` ↔ C / B / E / A**

| 本项目位置 | 对端位置 | 重合类型 |
|---|---|---|
| `02:8-10` `.ps1` 纯 ASCII 或 UTF-8 带 BOM（5.1 按 GBK 解码无 BOM 文件） | C `SKILL.md:58-67`（cmdlet 默认编码表：`Set-Content`=ANSI、`>`/`>>`=UTF-16 LE） | `逻辑一致但表述不同` |
| `02:12-15` `.bat`/`.cmd` 必须纯 ASCII + CRLF | A `SKILL.md:14`（keep .ps1/.bat/log pure ASCII，附 `grep -nP '[^\x00-\x7F]'` 校验） | `逻辑一致但表述不同` |
| `02:12-15` 同上 | E `SKILL.md:28-40`（§2 Unicode/Emoji Restriction，ASCII-only 规则） | `逻辑一致但表述不同` |
| `02:16-17` BOM 反向条款（BOM 毁掉 `@echo off`/shebang，按解释器决定别一刀切） | C `SKILL.md:33`（BOM 敏感源码文件须用 `UTF8Encoding($false)`） | `部分重叠`（对端限源码文件，本项目含批处理与 shebang） |
| `02:37-41` 解码顺序：BOM/NUL 字节嗅探 → UTF-16LE → ANSI | C `SKILL.md:20-21`（NUL 间隔 = UTF-16 LE 特征；体积约为预期 2 倍） | `逻辑一致但表述不同` |
| `02:45-47` PS `>` 默认 UTF-16LE、`Out-File -Encoding utf8` 在 5.1 为带 BOM UTF-8 | A `SKILL.md:23`（同结论 + "read back BOM-aware (try utf-8, utf-16, utf-8-sig)"） | `完全相同` |
| `02:18-19` Python 首行 `sys.stdout.reconfigure("utf-8")` | A `SKILL.md:18-19`（`PYTHONIOENCODING=utf-8` / `io.TextIOWrapper`） | `完全相同` |
| `02:23-26` 非 raw 串吃 `\a\b\0\U`，改用 raw/占位符 replace | A `SKILL.md:58`（`\S` `\d` 需 raw 串） | `部分重叠`（对端限正则转义，本项目含 Windows 路径） |
| `02:28-32` 脚本名须避开标准库名（`inspect.py` 等遮蔽） | A/B/C/D/E 均无 | 本项目独有 |

**`01_shell_execution.md` ↔ A / B / C / E**

| 本项目位置 | 对端位置 | 重合类型 |
|---|---|---|
| `01:16-18` MSYS 下 `$` 被提前展开成 `extglob`/空 → 落地 `.ps1` 用 `-File` | B `AGENTS.md:9-10`、`GUIDE.md:91`（明令禁止输出 Bash/sh/CMD 语法） | `方向分歧`（见末节 1） |
| `01:19-22` `/` 前缀参数被路径转换 → `MSYS_NO_PATHCONV=1` / `//` / 不经 shell | A `SKILL.md:25-28`（Git Bash 反引号命令替换、CJK 文件名经 cmd 坏码） | `部分重叠`（同指 Git Bash 层失真，命中机制不同） |
| `01:23-24` 含 `://` 参数被破坏 → 改用 `Start-Process -ArgumentList` | A/B/C/D/E 未见同类条款 | 本项目独有 |
| `01:28-29` `-ArgumentList` 中含空格路径须自套引号 | C `SKILL.md:103`（Quote all paths that contain spaces） | `逻辑一致但表述不同` |
| `01:30-31` 判成败看 `$LASTEXITCODE` + 副作用，返回 0 但产物缺失即失败 | C `SKILL.md:105`；A `SKILL.md:16`（stderr 污染致伪退出码）、`A:34`（`-Wait -PassThru` 取真实退出码） | `逻辑一致但表述不同` |
| `01:34-35` 复杂逻辑禁内联 heredoc，落地文件执行 | A `SKILL.md:20`（PS 无 bash heredoc，`<` 为保留运算符） | `逻辑一致但表述不同` |
| `01:37-64` PowerShell 语法坑章节（try/catch 不可接管道、`-Format` 非法、5.1 无三元、`Enum` 保留字、`Parser::ParseFile` 预检） | E `SKILL.md:15-24`（逻辑运算符须加括号）、`E:55-67`（插值先存变量）、`E:71-85`（try/catch 纪律） | `部分重叠`（插值与 try/catch 纪律一致；括号规则、`-Format`、`Enum`、Parser 预检四条对端未见对应项者已登记） |
| `01:52-53` `-in` 与逗号优先级坑 → 判成员用 `-contains` | A `SKILL.md:41`（`-contains` 大小写不敏感会误伤 → 用 `-ccontains`） | `方向分歧`（见末节 2） |
| `01:78-80` 计划任务 `/sc ONSTART` 可能"仅交互方式"→ 改 `/sc ONLOGON` | A/B/C/D/E 均无 | 本项目独有 |

**`04_file_operations.md` ↔ A / B**

| 本项目位置 | 对端位置 | 重合类型 |
|---|---|---|
| `04:29-33` 删除前四项检查之首：反查"哪些 reparse point 指向候选目录" | A `SKILL.md:39`（`Remove-Item -Recurse` 跟随 junction 删目标数据） | `逻辑一致但表述不同`（风险同源，规避手段不同，见末节 3） |
| `04:86-89` 应用搬家闭环：robocopy `/E /MOVE /XJ` + 原位 junction 兜底 + 批量改快捷方式 | A/B/C/D/E 均无 | 本项目独有 |
| `04:49-58` 被锁文件的开机删除队列：REG_MULTI_SZ **成对**存储、不得过滤空串、文件→子目录→根自底向上 | A `SKILL.md:37`（孤儿进程持句柄 → 先停项目自有进程） | `部分重叠`（同属"文件被锁"，处置层级不同） |
| `04:36-37` 版本目录 ≠ 安装残留；卷标/目录名/图标不可作判定依据 | A `SKILL.md:40`（版本比较须数值化，`v1.9` < `v1.10`） | `逻辑一致但表述不同` |
| `04:16`、`04:37`、`04:42-44` 扫描不进入 reparse point；大目录树分批 ≤100 项并先查占用者 | A `SKILL.md:42`（`glob("*keyword*")` 误删真文件 → 用精确名单并先复核） | `逻辑一致但表述不同` |
| `04:62-70`（长路径 `\\?\` 只认反斜杠、8.3 短名可能禁用）；`04:70-83`（`.lnk` 仅 COM 可解析、`IconLocation` 是字符串、特殊文件夹须 API 获取） | A/B/C/D/E 均无 | 本项目独有 |

**`06_process_privileges.md` ↔ A / B**

| 本项目位置 | 对端位置 | 重合类型 |
|---|---|---|
| 进程操作："不要按进程名批量杀，按端口反查 PID" | A `SKILL.md:136`（never `taskkill /IM python.exe`） | `完全相同` |
| §提权通道：显式 `runas` 层级 + 返回值判据 + 验证方式 | B `AGENTS.md:34`、`GUIDE.md:418`（提示用户手动以管理员运行，"不要自作主张"） | `方向分歧`（见末节 4） |
| §服务停止：`sc stop` 同步等待会挂死 → 异步发令 + 轮询判据 | A `SKILL.md:77`（错误 1052 → 别循环 `sc stop`，验证后直杀进程） | `逻辑一致但表述不同` |
| §隐藏监管者定位：svchost 宿主反查 + `Parameters\ServiceDll` 归属确认 | A `SKILL.md:76` | `完全相同` |
| §"文件凭空消失"诊断顺序：先怀疑安全软件实时隔离 | A `SKILL.md:117`（同一现象归因为测试路径错误） | `部分重叠`（归因方向不同） |
| 与安全软件共存的策略分层、占位锁反噬教训、ACL 功能实测 | A/B/C/D/E 均无 | 本项目独有 |

**`03_operational_safety.md`、`10_error_recovery.md` ↔ B / C / D**

| 本项目位置 | 对端位置 | 重合类型 |
|---|---|---|
| `10:5-13` 失败定性五步（输出层失真 → 编码/路径 → shell 改写 → 权限/策略 → 自身假设） | B `GUIDE.md:561`（§16 结构化命令与失败分类）、`B:411`（报错先分类再调整，禁止盲重试） | `逻辑一致但表述不同`；对端另标 `依赖同一外部规范`（D） |
| `03:12-13`、`03:39` 读回校验（返回码与提示文案不算数）+ 收尾恢复原值 | B `GUIDE.md:409-410`、`B:417`（保守可回滚写法、改配置先读现状） | `逻辑一致但表述不同` |
| `03:26-27` 数据安全底线：禁止 `rm`/`del`/`rd /s /q`/`Remove-Item`/运行时删除 API，须走可恢复通道 | B `AGENTS.md:23`、`GUIDE.md:413`；C `SKILL.md:106`（`Remove-Item -Recurse -Force` 作为 `rm -rf` 的推荐替代） | `方向分歧`（见末节 5） |
| 执行契约 1"先核实再动手"（状态新鲜度） | B `GUIDE.md:400-403`（§12.0 优先用 Agent 原生能力，少 spawn shell） | `部分重叠`（动机相近，落点不同） |
| 凭证不回显、不外传 | B `AGENTS.md:86` | `逻辑一致但表述不同` |

**`12_browser_automation.md` ↔ A**

| 本项目位置 | 对端位置 | 重合类型 |
|---|---|---|
| `12:13-15` 登录态写操作会真实改变线上数据，须先复述后果 | A `SKILL.md:113`（同账号会话列表账户侧共享，禁止自动删除/续写，仅操作自己创建的条目） | `完全相同` |
| `12:36-38` 收尾仅回收自己启动的实例（按专属数据目录匹配命令行） | A `SKILL.md:136` | `逻辑一致但表述不同` |
| 合成事件边界（不可信点击被忽略、脚本填值不触发逐键监听） | A `SKILL.md:134`（首选 `evaluate` 派发点击） | `部分重叠`（首选手段相反） |

**`00_environment_probing.md` ↔ B**

| 本项目位置 | 对端位置 | 重合类型 |
|---|---|---|
| `00:11-21` 前置探测 6 类事实（OS/代码页/令牌/Shell 栈/工具链/硬件） | B `AGENTS.md:13-17`、`GUIDE.md:69`（`$PSVersionTable`、执行策略、`Get-Command`） | `部分重叠`（对端不含令牌形态、ANSI 代码页、磁盘余量） |
| `00:20` 工具存在性检查 | B `GUIDE.md:407`；C `SKILL.md:101-107` | `逻辑一致但表述不同` |

**`11_diagnostics_methodology.md` ↔ A / B**

| 本项目位置 | 对端位置 | 重合类型 |
|---|---|---|
| `11:102-110` 结论三级（已证实 / 高度疑似 / 未知），禁止把"未知"写成"没问题" | B `GUIDE.md:480-493`（§13.1 不信回显、用 Unicode 码点验证磁盘真相） | `逻辑一致但表述不同` |
| `11:11` 核心原则 2：修复类命令的"成功回执"不是判据 | A `SKILL.md:60`、`A:62`（弱断言掩盖失败） | `部分重叠` |
| `11:33-35`、`11:69-101` "文件不存在 ≠ 被删"四步判读；卷影 / USN / `$I` 三源交叉取证 | A/B/C/D/E 均无 | 本项目独有 |

**`05_registry_config.md`、`07`、`08`、`09`（概览）**

| 本项目文件 | 对端最近位置 | 重合类型 | 说明 |
|---|---|---|---|
| `05` | A `SKILL.md:76`（仅读 `ServiceDll` 一处涉注册表） | `部分重叠` | 配置单元（HKCU vs `HKU\<SID>`）、`REG_MULTI_SZ` 配对、类型保持、"新增可写 / 改存量受限"、ACL 继承与功能实测——五个快照均无对应条款 |
| `07` | A `SKILL.md:84`、`A:23`；B `GUIDE.md:364`（§10 包管理与工具链差异） | `部分重叠` | 本项目独有：包管理器不吃系统代理、便携版符号链接失败、魔数+字节双校验、商店孤儿包、卸载表三视图 |
| `08` | A `SKILL.md:108-109`（DNS 无超时须加看门狗、快速失败） | `部分重叠` | 本项目独有：分阶段计时定位、污染/SNI/IP 封锁三分类、WFP 每连接开销归因、幽灵代理与端口漂移、受限网络取源顺序 |
| `09` | — | 无重合 | 五个快照均无并发基准方法学、GPU 计算密度判据、索引价值实测对比 |

### 共同依赖的外部规范（第三方锚点，非本项目与对端之间的重合）

| 规范 / 来源 | 声明方 | 位置 | 本项目是否绑定 |
|---|---|---|---|
| Agent Skills 开放规范（`SKILL.md` + YAML frontmatter + 渐进式披露） | A、C、D、E 均按其组织正文；F 为该规范发布处 | A `SKILL.md:1-4`；E `SKILL.md:1-7`（含 `when_to_use`/`allowed-tools`/`version`） | **否**：本仓库为 `_INDEX.md` + 编号 `.md`，无 frontmatter，不能被 `~/.claude/skills`、`~/.codex/skills` 直接装载 |
| `agent-shells` 风险枚举与失败分类 | B 显式署名借鉴 | B `GUIDE.md:421`（§12.1 标题）、`B:561`（§16 标题） | 间接：本项目 `03`/`10` 与 D 主题相近，但未引用 |
| "LukeTsengTW SKILL" 编码矩阵 | B 显式署名借鉴 | B `GUIDE.md:314`（§8.1 标题） | 间接：与本项目 `02:45-47`、C `SKILL.md:58-67` 为同族内容。本次核实：仓库检索 `LukeTsengTW powershell skill` → `total_count=0`；`LukeTsengTW/powershell` → HTTP 404（仅登记核实结果） |

### 已登记的方向分歧（需人工裁决，本节不判定）

| # | 主题 | 本项目立场 | 对端立场 | 位置对照 |
|---|---|---|---|---|
| 1 | 跨 shell 策略 | 承认 MSYS/Git Bash 常在场，给出共存规则（`MSYS_NO_PATHCONV=1`、`//` 转义、`subprocess` 列表调用） | 明令禁止输出 Bash/sh/CMD 语法，要求全部改用 PowerShell 语法 | `01:10-24` ↔ B `AGENTS.md:9-10`、`GUIDE.md:91` |
| 2 | PowerShell 成员判断 | `-in` 与逗号有优先级坑，判成员用 `-contains` | 认为 `-contains` 大小写不敏感会误伤，要求 `-ccontains` | `01:52-53` ↔ A `SKILL.md:41` |
| 3 | 含链接点目录的删除手法 | 扫描不进入 + 反查谁指向我 + 走系统可恢复通道 | `Remove-Item -Force`（不带 `-Recurse`）删链接、`cmd /c rmdir /s /q` 删树 | `04` §删除安全 ↔ A `SKILL.md:39` |
| 4 | 需管理员权限的操作 | 提供显式提权通道层级、返回值判据与验证方法 | 一律提示用户手动以管理员身份运行，不自作主张 | `06` §令牌与提权通道 ↔ B `AGENTS.md:34`、`GUIDE.md:418` |
| 5 | 删除动作的允许集合 | 明令禁止 `rm`/`del`/`rd /s /q`/`Remove-Item`/运行时删除 API，必须走可恢复通道 | 将 `Remove-Item -Recurse -Force`（先 `-WhatIf`）作为 `rm -rf` 的推荐替代 | `03:26-27` ↔ B `AGENTS.md:23`、`GUIDE.md:413`；C `SKILL.md:106` |

### 可复核性与未纳入范围

- 对端正文取自 `GET /repos/{o}/{r}` → `/commits?per_page=1&sha={branch}` → `/git/trees/{sha}?recursive=1` → `/contents/{path}?ref={sha}`（base64 解码）的一性次快照；本项目侧行号可用 `sed -n 'Np'` 直接复核。
- 同主题但**未逐行核实重合度**、仅登记存在的项目：`ngngsonan/awesome-agent-skills`（253+ skills 聚合清单）、`soahimrahman/ai-agent-skills-collection`（2,611 skills）、`mattpocock/skills`、`obra/superpowers`。
- 本节随仓库演进需人工复核；对端删除或改名的情形已出现一次（见登记表 G），故引用一律锁定 commit SHA。
