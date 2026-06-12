直接给你结论：Claude Code 装 filesystem 不需要单独安装什么，本质是用 claude mcp add 注册一个由 npx 动态拉起的 MCP Server 进程。一行命令就能跑起来。

一行命令跑通（推荐）

claude mcp add filesystem --transport stdio -- \
  npx -y @modelcontextprotocol/server-filesystem \
  /Users/yourname/projects


把最后一行的路径换成你想让 Claude 访问的真实目录即可。

💡 --transport stdio 是 Claude Code 连接本地 MCP Server 的标准方式，-- 后面的全部内容会原样传给 npx 启动进程。

几个新手必踩的坑

1. 路径必须是绝对路径，不能是相对路径或 ~
✅ /Users/alice/projects/myapp
❌ ./myapp
❌ ~/projects
❌ ../projects


相对路径会被当成字符串字面量处理，Server 启动时报 ENOENT。

2. 想开多个目录，直接追加在命令后面
claude mcp add filesystem --transport stdio -- \
  npx -y @modelcontextprotocol/server-filesystem \
  /Users/alice/projects \
  /Users/alice/Downloads


Server 的白名单是启动参数决定的，运行时不能动态扩充——要加目录必须改配置重启。

3. 选对项目级还是全局级

Claude Code 的 MCP 配置有两档，按场景选：

scope 命令 配置落点 适合场景

项目级（默认） claude mcp add filesystem ... .claude/settings.json（随项目提交 git） 团队协作，团队 clone 后直接能用

全局级 claude mcp add --scope user filesystem ... ~/.claude/settings.json 个人常用目录，跨项目复用

如果不确定，先用默认的项目级，提交到 git 对队友最友好。

验证是否装成功

claude mcp list


看到 filesystem 状态为 Connected 即 OK。

进 Claude Code 后随便试一句：

列出 /Users/yourname/projects 下所有 .md 文件

如果 Claude 直接调用工具返回结果而不是说"我无法访问文件系统"，那就稳了。

报错了按这张表对照

⚠️ 以下三类报错几乎覆盖了 90% 的安装问题，对着排查比瞎搜快得多。

🔴 Path not allowed / is not in my allowed list
• 原因：访问的路径不在启动时声明的白名单里

• 排查：claude mcp get filesystem 看当前配置，确认路径确实是绝对路径且拼写无误

🔴 EACCES / Permission denied
• 原因：Node 进程本身没权限读那个目录（比如 macOS 的 Downloads、系统目录）

• 排查：ls -la /path/to/dir 看权限；macOS 还要去"系统设置 → 隐私与安全 → 完全磁盘访问"里给终端/Claude 授权

🔴 MCPConnectionError: Server 'filesystem' closed unexpectedly
• 原因：99% 是 npx 参数格式不对，或 Node 版本太老

• 排查：node -v 确认 ≥ 18；直接在终端跑一遍 npx -y @modelcontextprotocol/server-filesystem /tmp，看是不是 Node/npm 本身的问题

顺手提一句安全边界

这个 Server 的权限模型是白名单制——只认启动时声明过的目录，根目录 /、系统目录 /etc、~ 一律碰不到。所以别图省事把 / 或 ~ 加进去，按"最小权限"原则只开项目目录就够了。另外它会跟随软链接，如果白名单里有含 symlink 的目录，记得 realpath 看一眼实际指向哪里。

如果你是想装别的 filesystem 实现（比如 @agent-infra/mcp-server-filesystem 或带 SSH 远程的 mcp-remote-fs），告诉我具体包名，命令略有差异。
