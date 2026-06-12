以下是为 Claude Code 安装 `filesystem` MCP 并授权**多个目录**的笔记总结，可直接用于记录。

---

## Claude Code 安装 filesystem MCP（多目录授权）

### 核心目的
让 Claude Code 能够读写本地指定文件夹（**无法删除文件**，写入时避免意外覆盖）。

### 安装命令（多目录）
```bash
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem /路径/目录1 /路径/目录2 /路径/目录3
```
- `filesystem`：服务器名称（可自定义）
- 多个目录用**空格分隔**
- 路径必须使用**绝对路径**（如 `/Users/name/Projects` 或 `~/Projects`）

### 轻量版（节省 token，长对话推荐）
```bash
claude mcp add filesystem -- npx -y filesystem-slim@latest /目录1 /目录2
```

### 配置管理
- **查看所有 MCP**：`claude mcp list`
- **移除服务器**：`claude mcp remove filesystem`
- **修改已授权的目录**：编辑配置文件 `~/.claude/mcp.json` 或项目下的 `.claude/mcp.json`，在 `args` 数组中增减路径

### 安全建议
- ✅ 只授予需要被 Claude 访问的文件夹（如 `~/Projects`、`~/Documents`）
- ❌ 避免授予根目录 `/` 或整个用户家目录 `~`

### 验证方法
在对话中提问：
> “请列出 `~/Projects` 下的文件结构”

若能正常返回，说明配置成功。
