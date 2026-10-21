# 01 Shell 执行（Shell Execution）

> 优先级 🔴 核心必读。本文规定"用哪个解释器、怎么把命令交给它"的纪律。
> 绝大多数"命令明明对却报错"、"脚本静默不执行"的问题都源于 shell 层的系统性失真。

## 前置探测（提交任何命令前）

1. **当前解释器身份**：这条命令实际由谁执行（cmd / Windows PowerShell 5.1 / PowerShell 7 / 第三方 bash），
   版本是多少。`$PSVersionTable`、`$SHELL`、`echo $0` 之类的自述结果要按"你确实在这个 shell 里"验证，
   不能凭会话开头的印象。
2. **路径转换层是否在场**：是否存在 MSYS/Git Bash 这类会把 `/` 开头参数改写成 Windows 路径的中间层。
3. **工具是否真的存在**：要调用的程序先做存在性检查（见 `07`），并确认解析到的是**你想要的那个**
   （同名多版本、别名缺失、商店占位 stub 都会给出"能找到但跑不对"的结果）。
4. **参数的传递方式**：本次调用是"程序 + argv 数组"还是"拼字符串过 shell"；含空格/中文/`://`/引号嵌套的参数
   是否会被二次解析。
5. **成败判据**：本次执行用哪个信号判成败（返回码 + 产物 + 副作用），先想清楚再跑，别事后看输出好看就当成功。

探测结论只对本次会话有效；换了 shell 或换了调用方式，重新探测。

## 解释器选择

- 优先使用结构化、可预期的调用方式：直接 `program + argv[]`（如 Python `subprocess.run([...])`
  列表形式、PowerShell `Start-Process -ArgumentList @(...)` 数组形式），**不要拼字符串再过 shell**。
- If 需要在类 Unix shell（MSYS/Git Bash/Cygwin）里调用 Windows 原生命令，Then 意识到存在
  三层失真（见下），复杂逻辑一律改为落地 `.ps1` / `.py` 文件执行。
- If 命令含重定向、管道、变量、引号嵌套中的任意两项，Then 不要内联，写成文件再跑。

## MSYS / Git Bash 类环境的三类系统性失真（If 当前 shell 是它，Then 遵守）

1. **`$` 变量被提前展开**：`powershell -Command "... $_ ... $var ..."` 中的 `$` 会被外层 shell
   吞成空值或字面量 `extglob`，导致 PowerShell 收到残缺脚本、报缺括号。
   → 规则：含 `$` 的 PowerShell/批处理逻辑一律写 `.ps1` 文件，用 `-File` 执行。
2. **以 `/` 开头的参数被当路径转换**：`powercfg /batteryreport`、`tasklist /FO CSV`、
   `cmd //c ...` 的开关会被改写成宿主机路径（形如 `/FO` → `C:\...\FO`），报"参数无效"。
   → 规则：把该命令放进 PowerShell 内执行；或从 Python `subprocess.run([...])` 直接调用（不过 shell）；
   或设置 `MSYS_NO_PATHCONV=1`。需要向 Windows 传 `/` 前缀参数时用 `//` 转义。
3. **含 `://` 的参数被破坏**：代理 URL（`socks5://127.0.0.1:port`）这类会被转义损坏。
   → 规则：启动带代理参数的浏览器/工具时改用 PowerShell `Start-Process -ArgumentList` 传参。

## 命令写法约束

- If 要在 `Start-Process -ArgumentList` 里传含空格的路径，Then 自己给该参数套引号
  （`'"C:\Program Files\...\x"'`），否则参数在首个空格处断开，被调用方收到半截路径。
- If 需要判断命令成败，Then 读 `$LASTEXITCODE` / 子进程返回码 + 副作用（产物文件是否存在），
  不读控制台美观输出。返回码 0 但产物缺失 = 失败。
- If 命令是长任务（可能 > 60s），Then 必须设置超时并把中间结果落盘；
  超时后先捞取部分产物再决定是否重跑（见 `10`），不要无脑重跑。
- 复杂脚本（多语句、含中文、含正则、含 here-string）禁止塞进命令行 heredoc；
  用文件写入工具落盘后执行，避免转义二次污染。

## PowerShell 语法坑（写 .ps1 前逐条自检）

这些坑的共同特征：**解析期就失败，运行期一行都不执行**，且常表现为 exit 0、无输出、无报错。

- **`try/catch` 是语句，不能直接接管道**：`try{...}catch{...} | Out-File x` 会整脚本解析失败
  （提权进程秒退、完成标记不出现，外在表现就是"超时"）。
  → 写成 `& { try{...}catch{...} } *>&1 | Out-File x`。
- **`-Format` 不是操作符**，只是 cmdlet 参数：`($dt -Format 'yyyy-MM-dd')` 是非法表达式，
  会导致整个脚本一行都不执行。日期时间格式化一律用 `$dt.ToString('yyyy-MM-dd HH:mm')`
  （事件日志、开机时间、安装时间、创建时间全适用）。
- **字符串插值里的链式调用只被部分求值**：`"$([Security...]::GetCurrent()).IsInRole(...)"`
  输出的是"表达式文本 + 前半段结果"，不是最终布尔值。
  → 复杂链式调用先赋给变量，或整体包进 `$( )`；不要在字符串里写链式调用。
- **Windows PowerShell 5.1 不支持 `?:` 三元运算符**（7 才支持）。5.1 下用 `if/else`
  或 `[int]`/`Switch` 替代；需要兼容两版时一律 `if/else`。
- **`-in` 与逗号优先级**：`$x -in 'a','b','c'` 被解析为 `($x -in 'a')` 再拼数组，条件常恒真
  → 判成员用 `$array -contains $x`（并把候选集先存成变量）。
- **保留/半保留标识符不能作函数名或变量名**，典型如 `Enum`（解析错误同样导致 `-File` 静默失败）。
  命名一律加前缀（`BEnum`、`Get-EmuList`）。
- **`Add-Type` 在受管控环境可能被策略禁用**（它会编译并加载运行期 .NET 代码）。
  → 读原始扇区/结构体改用 `[IO.File]::Open*` + 字节偏移解析，不要依赖自定义 C# 内联。
- **`powershell -Command "<脚本>" 值1 值2` 的尾随参数不会进 `$args`**（被当命令文本续接）。
  → 需要传值就把值内联进脚本文本（仅当值是 ASCII 时安全），或改为落地 `.ps1` 用 `-File` + `-ArgumentList`。
- **`Start-Process -Wait` 会阻塞调用方**（长任务/等 UAC 时不要前台等）：
  改后台启动 + 轮询完成标记文件 + 读回结果（与 `06`/`10` 一致）。
- **提交前做语法预检**（比"跑一下看报不报错"快且可靠，能捕获静默失败类问题）：
  `$e=$null; [System.Management.Automation.Language.Parser]::ParseFile($path,[ref]$null,[ref]$e); $e`
  —— 返回非空即语法挂了，先修再跑。产物文件"根本没出现"通常意味着语法挂而非逻辑挂。

## 控制台回显不可信

- If 需要采信命令结果，Then 以**落盘文件**为准：脚本内写文件（UTF-8 无 BOM），执行后读回。
  `Out-File` 直接产出可能与预期编码不符（见 `02`），`Start-Process -RedirectStandardOutput`
  在部分场景会立即返回而吞掉输出。
- 采集型脚本的可靠模式：脚本内用 StringBuilder 累加 → 结束时
  `[IO.File]::WriteAllText(path, text, UTF8Encoding($false))` 一次性写文件 → 外部读文件。
- If 输出里出现零星 NUL 字节（混合 UTF-16 段），Then 不要整份按 UTF-16 解码；
  先判 `b'\x00' in head` 决定解码路径，或去除 NUL 后按 UTF-8 解（见 `02`）。

## 定时与自启

- If 创建计划任务要求开机即执行，Then 注意 `/sc ONSTART` + `/rl HIGHEST` 生成的任务
  可能是"只使用交互方式"，未登录则不跑；按需改 `/sc ONLOGON` 或显式设置登录类型。
- If 只需当前用户登录后自启，Then 优先 HKCU `...\Explorer\Run`/`Startup` 或启动文件夹，
  避免污染系统级服务；写入前后都要读回并确认路径含空格时已加引号。
