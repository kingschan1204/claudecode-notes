`planning-with-files` 是一个为 Claude Code 设计的 Agent Skill，它通过将任务规划、进度和知识持久化到本地 Markdown 文件中，来解决 AI 在处理复杂任务时的“健忘”和“目标漂移”等问题。

它的核心思想是为 Claude Code 打造一个可靠的“外部硬盘”记忆系统，使其拥有持久化的工作记忆。

### 🧠 它主要解决什么问题？

`planning-with-files` 通过“**文件化规划**”的方式，解决了 Claude Code（以及大多数 AI Agent）在处理长期、多步骤任务时常见的几个核心痛点：

*   **易失记忆**：当会话上下文被重置或 Agent 停止后，之前的计划和结论容易丢失，导致工作无法延续，需要重新开始。
*   **目标漂移**：在大量工具调用后，AI 可能偏离最初设定的目标，导致交付成果不符合预期。
*   **错误不复盘**：过程中遇到的错误不会被系统性地记录，可能导致 AI 在同一个问题上反复犯错。
*   **上下文“过载”**：试图把所有信息都塞进有限的上下文窗口中，而非结构化地存储，这会使信息变得难以追溯和复用。

### 📝 它的工作原理是什么？

它的工作方式很简单：为每个复杂任务在项目目录下创建并维护三个 Markdown 文件，形成一套稳定的信息管理闭环。

*   **任务计划文件 ( `task_plan.md` )**：这是核心。它将一个大目标分解为多个阶段（Phase），并追踪每个阶段的详细进度和状态，确保 Claude Code 时刻清楚自己在哪里、该做什么。
*   **研究发现文件 ( `findings.md` )**：用于记录所有研究发现、重要决策和关键信息，作为知识库供随时查阅，防止“看过就忘”。
*   **会话日志文件 ( `progress.md` )**：记录会话日志、操作步骤和测试结果，详细记录整个过程，以便事后追溯。

当需要开始一个复杂任务时，你可以这样开启这个工作流：
```
启动任务时：
Claude，请使用 "planning-with-files" 工作流。我们的目标是 [在此描述你的任务目标]。
```
Claude 会立即在项目目录中创建那三个核心文件，并开始分步执行与追踪。

### 🔧 如何在 Claude Code 中安装

根据你的使用偏好，可以通过以下三种方式安装：

1.  **最推荐：通过 Plugin Marketplace 安装**
    这是官方推荐且最便捷的方式，直接在 Claude Code 对话中输入以下两条命令即可。
    ```shell
    /plugin marketplace add OthmanAdi/planning-with-files
    /plugin install planning-with-files@planning-with-files
    ```
    安装后，你需要重启 Claude Code 才能使 Skill 生效。

2.  **备用：通过 `npx skills add` 命令安装**
    你还可以使用 `skills` 命令行工具进行全局安装。
    ```shell
    npx skills add planning-with-files -y -g
    ```

3.  **手动：从 GitHub 克隆安装**
    如果上述方法不适用，可以手动将 Skill 克隆到全局 Skills 目录。
    ```shell
    git clone https://github.com/OthmanAdi/planning-with-files.git ~/.claude/skills/planning-with-files
    ```

### 💎 总结

简单来说，`planning-with-files` 是一个优秀的技能，能为 Claude Code 在应对复杂项目时提供稳定可靠的“工程化记忆”。这就好比一个长期项目需要一份详尽的待办清单和会议记录，让工作变得井井有条，也让你和其他人（或AI）能在任何时候无缝衔接。

强烈建议在处理大型项目、研发任务或需要跨会话完成的工作时使用它。如果你在安装或使用过程中遇到任何问题，随时可以告诉我，我乐意提供进一步的帮助。
