# 共享 AI 记忆仓库

以 GitHub 私有仓库的 `main` 分支为共享版本。ChatGPT、Codex、Claude Code、Z Code 等工具从同一份 Markdown 资料读取背景，具备写权限的工具按同一套规则更新。Git 提交保留变更历史。

## 开始使用

先读 [MEMORY.md](MEMORY.md) 和 [AGENTS.md](AGENTS.md)，再按任务打开原则、当前状态和对应项目。首次在其他机器使用时，从目标工作区克隆。

```bash
gh repo clone yik900655-bot/shared-agent-memory
cd shared-agent-memory
git status --short --branch
```

看到 `main...origin/main` 即已进入跟踪远程主分支的本地副本。已有本地副本时，直接打开它。

| 工具 | 如何读取 | 如何回写 |
| --- | --- | --- |
| Codex | 以本仓库根目录打开本地项目或启动会话。Codex 会加载 `AGENTS.md`，再按其指引读取 `MEMORY.md` 和任务相关文件。 | 使用有写权限的本地副本提交和推送。 |
| Claude Code | 在根目录启动。[`CLAUDE.md`](CLAUDE.md) 导入 `AGENTS.md`，后续按统一入口读取。 | 使用有写权限的本地副本提交和推送。 |
| Z Code | 将本仓库根目录设为 Workspace，使用 ZCode Agent 新建任务。它读取 Workspace 的 `AGENTS.md`，再按文内路径打开其余文件。若当前版本或 Agent 没有加载它，在任务首条消息中明确要求读取 `AGENTS.md` 与 `MEMORY.md`。 | 使用有写权限的本地副本提交和推送。 |
| Gemini CLI | 在根目录启动。[`GEMINI.md`](GEMINI.md) 导入 `AGENTS.md`。 | 使用有写权限的本地副本提交和推送。 |
| ChatGPT | 在支持 GitHub 读取的会话中，连接 GitHub 并允许访问此私有仓库。明确指定 `yik900655-bot/shared-agent-memory`，要求先查找 `MEMORY.md` 与 `AGENTS.md`，再按任务打开其余文件。若会话没有仓库读取能力，从最新版副本提供本轮所需 Markdown。 | 使用只读 GitHub 连接时，请 ChatGPT 输出带来源的 Markdown 草稿，交由具有写权限的工具归档和推送。 |

在别的项目中使用共享记忆时，在任务中提供此仓库的位置并明确要求读取两个入口。接入成功的判断是工具能说出实际打开的文件路径，以及对应项目的当前阶段和一项待验证问题。

工具文档见 [Codex](https://learn.chatgpt.com/docs/agent-configuration/agents-md)、[Claude Code](https://code.claude.com/docs/en/memory)、[ZCode Agent](https://zcode.z.ai/cn/docs/agents) 和 [Gemini CLI](https://geminicli.com/docs/cli/gemini-md/)。

## 更新和推送

1. 在干净的 `main` 上运行 `git pull --ff-only`，确认同步成功后再编辑。
2. 更新涉及的记录，新发现先放入 [INBOX](06_INBOX/README.md)。用 `git status --short` 确认改动范围，用 `git add -i` 选择本轮文件。
3. 检查暂存差异，确认只包含本轮改动并检查链接、证据与敏感信息，再提交和推送。

```bash
git diff --cached --check
git diff --cached
git commit -m "Update shared memory"
git push
git status --short --branch
```

推送成功后，工作区应干净，分支状态无 `ahead` 或 `behind`。如果推送被远程新提交拒绝，先 `git fetch origin`，检查 `git log --oneline --left-right HEAD...origin/main`，再运行 `git merge origin/main`。发生冲突时核对双方来源，保留有效改动，再用 `git status` 检查并完成合并。复查后重新推送。禁止强制推送。多个工具同时写入时使用各自的本地副本，避免共享暂存区。

## 阶段整理

阶段结束或积累了一批记录后，按 [Dreaming 整理流程](05_WIKI/dreaming-protocol.md) 手动处理。设计参考 [Cognition Agent Memory Repo](https://cognition.com/agent-memory-repo)。
