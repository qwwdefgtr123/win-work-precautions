# 00 环境探测（Environment Probing）

> 优先级 🔴 核心必读。本文规定 Agent 在 Windows 上动手前**必须动态获取**的环境事实。
> 禁止把探测结果当成常量写进脚本、配置或长期记忆；环境会变，结论必须现场复核。

## 前置探测指令（每次任务开始时执行）

按顺序探测以下 6 类事实，全部用只读命令完成，任何一项探测失败都要如实记录为"未知"，
不得用假设填补：

1. **系统与区域设置**：OS 版本与位数、是否为服务器/桌面 SKU、系统 ANSI 代码页
   （决定 GBK/CP1252 类解码路径）、当前时区与本地时间（相对时间与计划任务必须换算）。
   探测法：`Get-CimInstance Win32_OperatingSystem`；代码页用 `chcp` 或 Python `locale.getpreferredencoding()`。
2. **进程令牌形态**：当前进程是否真正具备管理权限。
   探测法：`([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)`
   与 `whoami /groups`（看 Mandatory Level）。**账户在 Administrators 组 ≠ 进程是完整令牌**，
   UAC 会降权；据此决定后续是否需要提权通道（见 `06`）。
3. **可用 Shell 栈**：cmd / Windows PowerShell 5.1 / PowerShell 7（`pwsh`）/ 第三方 bash（MSYS/Cygwin）
   各自是否存在、当前实际使用的是哪一个。
   探测法：`Get-Command pwsh, powershell, bash, cmd -ErrorAction SilentlyContinue`。
   结论直接决定 `01`/`02` 中哪些坑会命中（例如 MSYS 在场才会有路径转换与变量吞并问题）。
4. **工具链与第三方工具**：任何要调用的外部工具（编译器、解释器、CLI、归档器、驱动工具）
   必须先 `Get-Command <tool>` 或 `where.exe <tool>` 确认存在，再确认版本；
   **禁止假设任何第三方工具已安装**。找不到时按 `07` 的探测—取源—校验流程处理。
5. **硬件与资源余量**：CPU 逻辑/物理处理器数、是否混合架构、可用内存、目标磁盘剩余空间、
   是否存在独显。探测法：`Get-CimInstance Win32_ComputerSystem/Win32_Processor/Win32_PhysicalMemory`、
   `Get-PSDrive`、`Get-CimInstance Win32_VideoController`。结果用于 `09` 的并发与 GPU 决策。
6. **既有档案与现成工具**：若工作目录或用户指定目录中已有工具库、配置文件、
   实测档案（硬件画像、并发推荐、既往经验文档），**优先读取复用，不要重新探测硬件或造轮子**；
   但若档案时间戳陈旧或与当前现象矛盾，必须重测并更新档案。

## 状态新鲜度规则

- If 距上次扫描已超过一个对话回合，或用户可能在使用电脑（启动/关闭应用、手动增删文件），
  Then 重新枚举进程与目标路径的实际状态，再动手。早先的扫描结果只用于形成假设。
- If 需要判断"某文件/进程/服务现在是否存在"，Then 必须现场查询，不接受上下文中的历史结论。
- If 用户可能已并行修改过你要操作的对象，Then 先做差异比对（列目录、比哈希）再写入。

## 探测产物规范

- 把探测结果写成**一个结构化文件**（JSON/YAML）落在工作目录，字段至少包含：
  `os`、`ansi_codepage`、`token_elevated`、`shells_available`、`tools`（名称→路径→版本→可用性）、
  `hardware`（cpu/mem/disk/gpu）、`probed_at`。
- 后续任务读取该文件前，先检查 `probed_at` 的时效；超过当日或环境已变更（新装软件、重启）则重探。
- 探测脚本本身必须纯 ASCII（见 `02`），输出用 UTF-8 无 BOM 写文件后再读取，不依赖控制台回显。

## 输出纪律

- If 需要展示环境事实给用户或后续步骤，Then 只输出结论行与必要字段，不倾倒原始命令输出。
- If 某项探测返回空，Then 先判断是"权限不足导致静默空"还是"确实不存在"（两者处置完全不同，见 `06`）。
  典型：SMART/温度/电池类 WMI 计数器、`fltmc` 在非提权下常静默返空，不得据此判定硬件或驱动状态。
