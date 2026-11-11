# 共享记忆工作规则

适用于操作本仓库的所有 AI。共享内容以已同步的 GitHub `main` 为准。当前任务的实际授权由用户指令决定。

## 工作开始前

1. 具备本地 Git 读取条件时，在干净的 `main` 上先用 `git pull --ff-only` 同步。遇到未提交改动或分支分歧，保留现场并检查状态。
2. 阅读 [MEMORY.md](MEMORY.md) 和本页。
3. 阅读 [PRINCIPLES](01_PRINCIPLES/README.md) 中与任务相关的内容。
4. 阅读 [CURRENT_STATE](02_CURRENT_STATE/current-project.md)、[项目目录](03_PROJECTS/README.md) 中对应项目的 README 和进度文件。
5. 按需检索 [DECISIONS](04_DECISIONS/README.md) 与 [WIKI](05_WIKI/README.md)。若关键文件读不到，说明缺失路径和对本次判断的影响。

## 工作过程中

- 标明已核实事实、推断和待验证信息。事实附可复核证据，推断写明依据，待验证项写明缺口。
- 长期原则的修改必须经用户明确确认。未经验证的信息留在 INBOX，等待核实。
- 重要发现、经验和猜想先进入 [INBOX](06_INBOX/README.md)，逐条记录来源、日期、所属项目、类型和验证状态。
- 尊重其他 AI 已提交的改动。出现不同结论时先核对日期、口径和证据。分歧未解决时并列记录，保留双方有效内容。
- 任何文件与提交都不得包含 API Key、密码、Token、Cookie、私密个人信息或其他敏感凭据。证据只摘录必要内容，确认来源可以共享。

## 工作结束后

1. 总结本轮完成事项和未验证项。项目实际有进展时更新当前状态与项目进度，并给出下一步。
2. 记录重要决策，将新发现写入 INBOX。经过核实并适合复用的经验可按 [Dreaming](05_WIKI/dreaming-protocol.md) 沉淀到 Wiki。
3. 用 `git status --short`、`git diff` 和 `git diff --cached` 复核本轮内容，确认来源、链接及敏感信息检查通过。只暂存本轮有权修改的文件。
4. 有可提交改动且具备写权限时，执行 Git Commit 并推送到 GitHub。推送冲突按 [README](README.md) 处理，禁止强推。只读会话交付拟更新的文件和内容，由有权限的工具回写。
