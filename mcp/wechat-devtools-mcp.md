# wechat-devtools-mcp（uvx Python版）安装踩坑完整笔记
## 一、整体安装流程（最终成功顺序）
### 1. 前置软件准备
1. 安装微信开发者工具，记录两个关键路径：
   - CLI程序路径：`C:\Program Files (x86)\Tencent\微信web开发者工具\cli.bat`
   - 小程序项目根目录（含app.json）：`D:\xxx\你的小程序文件夹`
2. 打开微信开发者工具 → 设置 → 安全
   勾选：服务端口、HTTP调试、自动化测试，保持工具窗口全程开启
3. 安装uv环境（提供uvx命令）
   以**管理员身份**打开PowerShell，执行官方一键安装脚本：
   ```powershell
   irm https://astral.sh/uv/install.ps1 | iex
   ```
4. 关闭所有终端、VS Code，重新打开CMD验证环境
   ```cmd
   uv --version
   uvx --version
   ```
   输出版本号即uv环境就绪。

### 2. 全局安装MCP服务（避免uvx每次在线拉包闪退）
管理员CMD执行：
```cmd
uv tool install wechat-devtools-mcp --force
```

### 3. 配置Claude Code全局MCP文件
文件路径（必须修改用户根目录全局文件，项目内配置会重启丢失）：
`C:\Users\kingschan\.claude.json`
完整可用配置（替换两处自定义路径）：
```json
{
  "mcpServers": {
    "wechat-devtools": {
      "type": "stdio",
      "command": "wechat-devtools-mcp",
      "args": [],
      "env": {
        "WECHAT_DEVTOOLS_CLI": "C:\\Program Files (x86)\\Tencent\\微信web开发者工具\\cli.bat",
        "WECHAT_PROJECT_PATH": "D:\\你的小程序项目完整路径"
      },
      "autoApprove": ["*"]
    }
  }
}
```
- Windows路径必须使用双反斜杠`\\`，防止JSON语法报错
- `autoApprove:["*"]` 自动放行全部工具调用，避免权限弹窗导致进程关闭

### 4. 加载验证
1. 完全关闭所有VS Code窗口后重新打开
2. Claude Code聊天框输入 `/mcp` 打开MCP服务面板
3. 列表出现`wechat-devtools`，点击`Reconnect`显示正常即安装完成

---

## 二、全程遇到的问题+解决方案汇总
### 问题1：Node版npx拉包报404 Not Found
- 现象：`@sensen0326/wechat-devtools-mcp` 淘宝镜像源找不到安装包
- 原因：该npm包发布异常、镜像源资源缺失
- 解决：放弃Node版本，切换uvx Python版wechat-devtools-mcp

### 问题2：执行`npm config set registry`报EPERM权限不足
- 现象：无法写入`.npmrc`配置文件，操作被系统拒绝
- 原因：普通CMD无系统文件修改权限
- 解决：使用**管理员终端**执行npm配置命令；或直接切换uv方案绕开npm

### 问题3：输入`uvx`提示不是内部/外部命令
- 现象：`where uv` 查询无结果，系统无法识别uvx指令
- 原因：uv未安装，或安装后PATH环境变量未刷新
- 解决：
  1. 管理员PowerShell执行官方一键安装脚本自动配置PATH
  2. 安装完成必须**关闭全部终端再重新打开**，旧终端不会加载新环境变量

### 问题4：uvx wechat-devtools-mcp 启动失败：缺少WECHAT_PROJECT_PATH
- 现象：仅配置`WECHAT_DEVTOOLS_CLI`，启动直接报错退出
- 原因：Python版MCP强制要求两个环境变量，小程序项目路径为必填项
- 解决：在`.claude.json`的env中补充`WECHAT_PROJECT_PATH`，填写小程序根目录绝对路径

### 问题5：VS Code重启后MCP服务直接消失
- 现象：配置完重启VS Code，/mcp列表看不到wechat-devtools
- 核心原因&对应方案：
  1. 修改了项目文件夹内局部`.claude.json`
     → 改为修改用户根目录全局配置：`C:\Users\kingschan\.claude.json`
  2. JSON语法错误（多余逗号、单斜杠路径），插件跳过加载配置
     → 使用提供的完整标准配置，路径统一双反斜杠
  3. MCP启动多次失败，Claude Code自动隐藏故障服务
     → 提前手动打开微信开发者工具并开启安全端口；全局uv tool安装MCP避免uvx在线拉包超时闪退
  4. VS Code插件缓存损坏
     → 关闭VS Code，删除`\.vscode\extensions\anthropic.claude-code-*\mcp-cache`缓存文件夹后重启

### 问题6：MCP面板提示 MCP error -32000: Connection closed
- 现象：服务加载后标记Failed，连接直接关闭
- 诱因汇总：
  1. env环境变量缺失/路径填写错误
  2. 微信开发者工具未开启自动化服务端口
  3. 未手动启动微信开发者工具，无进程可连接
  4. uvx在线下载包超时，进程闪退
- 根治方案：uv tool全局安装MCP服务、补全双环境变量、提前启动并配置开发者工具安全设置

---

## 三、关键避坑备忘录
1. 路径规范：Windows JSON配置内文件路径**必须双反斜杠`\\`**
2. 终端权限：安装uv、npm修改配置等操作尽量用管理员终端
3. 环境刷新：安装uv/修改环境变量后，**必须全关终端重开**才能生效
4. 配置文件区分：全局`.claude.json`在用户根目录，项目内配置重启失效
5. 微信工具前置：必须手动打开工具并开启安全端口，否则MCP无法建立连接
6. uvx优化：优先`uv tool install`全局安装，避免uvx每次启动在线下载引发闪退
