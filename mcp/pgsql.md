# Claude Code + PostgreSQL MCP 配置笔记

> 最后更新：2026-06-09  
> 适用场景：Claude Code CLI（非 Desktop 版）

---

## 一、重要前提（必读）

❌ **不要使用官方 `@modelcontextprotocol/server-postgres`**
- 已于 2025-07 废弃
- 存在已知 SQL 注入漏洞

✅ **推荐替代方案**
1. **Postgres MCP Pro**（`postgres-mcp`）—— 生产级，带性能分析
2. **@sarmadparvez/postgresql-mcp** —— 轻量，支持读写事务

---

## 二、方案 A：Postgres MCP Pro（推荐）

### 1. 全局安装（可选，用于加速首次启动）

bash
npm install -g postgres-mcp


### 2. 项目级配置（仅当前项目生效）

```bash
cd /path/to/your/project

claude mcp add --scope project postgres \
  -e POSTGRES_CONNECTION_STRING="${POSTGRES_CONNECTION_STRING}" \
  -e ACCESS_MODE=restricted \
  -- npx -y postgres-mcp

# 最简单粗暴
claude mcp add --scope project postgres \
  -e POSTGRES_CONNECTION_STRING="postgresql://user:pass@host:5432/mydb" \
  -e ACCESS_MODE=unrestricted \
  -- npx -y postgres-mcp
```
生成的配置文件位于：`./.mcp.json`

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "postgres-mcp"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "${POSTGRES_CONNECTION_STRING}",
        "ACCESS_MODE": "restricted"
      }
    }
  }
}
```

### 3. 权限模式说明

| 模式 | 说明 |
|---|---|
| `unrestricted` | 读写，可执行 DDL |
| `restricted` ✅ | 只读 + Schema 查看 |
| `read-only` | 仅 SELECT |

> ⚠️ 生产库务必使用 `restricted` 或 `read-only`

---

## 三、环境变量管理（防泄露）

### `.env`（不提交 Git）

env
POSTGRES_CONNECTION_STRING=postgresql://user:pass@localhost:5432/mydb


### `.gitignore`

gitignore
.mcp.json
.env


### 加载环境变量

bash
source .env


---

## 四、验证是否生效

bash
查看 MCP Server 列表

claude mcp list

在 Claude Code 中测试

"列出所有表"

"查看 users 表结构"



斜杠命令检查状态：


/mcp


---

## 五、跨机器同步说明

✅ **不需要手动重装 MCP 包**  
✅ **`.mcp.json` 可随代码提交**

### 新机器前置条件

1. **Node.js ≥ 18**
bash
node --version


2. **配置环境变量**
bash
export POSTGRES_CONNECTION_STRING="..."


3. **数据库可访问**
bash
pg_isready -h host -p 5432
psql "$POSTGRES_CONNECTION_STRING"


📌 首次运行时会由 `npx` 自动下载依赖，稍作等待即可。

---

## 六、常见问题排查

### 1. npx 下载超时
bash
npm install -g postgres-mcp
修改 .mcp.json 中 command 为 postgres-mcp



### 2. PostgreSQL 连接失败
- 检查 `pg_hba.conf`
- 云数据库需加 `?sslmode=require`

### 3. 权限不足
sql
GRANT SELECT ON ALL TABLES IN SCHEMA public TO user;
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT ON TABLES TO user;


### 4. JSON 语法错误
bash
python3 -m json.tool .mcp.json


---

## 七、Scope 区别速查

| Scope | 配置文件 | 可见范围 |
|---|---|---|
| `--scope project` ✅ | `.mcp.json` | 当前项目 |
| `--scope local` | `~/.claude.json` | 当前用户 / 当前项目 |
| `--scope user` | `~/.claude.json` | 所有项目 |

优先级：`local > project > user`

---

## 八、团队协作建议

- ✅ `.mcp.json` 入库
- ✅ `.env.example` 入库
- ❌ 凭证、密码不入库
- ✅ 使用 `${VAR}` 引用环境变量

---

## 九、参考命令速查

bash
添加

claude mcp add --scope project postgres ...

删除

claude mcp remove postgres

列表

claude mcp list

MCP 状态

/mcp


---

> 💡 **核心记住一句话**：  
> MCP 配置随项目走，数据库凭证走环境变量，Node.js 是新机器唯一必须预装的依赖。

