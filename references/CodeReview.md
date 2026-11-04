# CodeReview

代码审查的任务是确认改动是否实现了目标、是否引入了可证明的缺陷，以及是否符合本仓库的契约和维护方式。审查意见应帮助作者处理问题，不用于展示知识、强加个人风格或把所有可改进处都列成阻塞项。

## 审查前

PR review 应依据 PR 标题、正文和关联需求理解改动意图，并以当前本地存储的master分支代码作为行为对照；不要只看少量差异猜测目的，也不要为重复取代码而改变本地仓库状态。若 PR 信息或基线不完整，说明缺少的证据。

PR review 默认只读取和汇报，不修改本地文件、不替作者修复、不提交意见、不推送远端。用户明确要求修复时，先把审查发现与实现任务分开，避免在未确认问题时扩大改动。

如果对本地代码进行审查，逻辑同理，仅由从PR中读取改动意图变为由会话记录或人类提供的最终文档判断。

## 审查目的

### 1. The Review Mindset

**Goals of Code Review:**
- Catch bugs and edge cases
- Ensure code maintainability
- Share knowledge across team
- Enforce coding standards
- Improve design and architecture

**Not the Goals:**
- Show off knowledge
- Nitpick formatting (use linters)
- Block progress unnecessarily
- Rewrite to your preference

### 2. Effective Feedback

**Good Feedback is:**
- Specific and actionable
- Educational, not judgmental
- Focused on the code, not the person
- Prioritized (critical vs nice-to-have)

```markdown
❌ Bad: "This is wrong."
✅ Good: "This could cause a race condition when multiple users
         access simultaneously. Consider using a mutex here."

❌ Bad: "Why didn't you use X pattern?"
✅ Good: "Have you considered the Repository pattern? It would
         make this easier to test. Here's an example: [link]"

❌ Bad: "Rename this variable."
✅ Good: "[nit] Consider `userCount` instead of `uc` for
         clarity. Not blocking if you prefer to keep it."
```

### 3. Review Scope

**What to Review:**
- Logic correctness and edge cases
- Security vulnerabilities
- Performance implications
- Test coverage and quality
- Error handling
- Documentation and comments
- API design and naming
- Architectural fit

**What Not to Review Manually:**
- Code formatting (use Prettier, Black, etc.)(Formatting is a task for the CI check.)
- Import organization
- Linting violations

## 审查顺序

1. **行为与正确性**：主路径、空值、边界、错误路径、状态转换、重复调用、并发或生命周期是否符合需求和现有契约。
2. **接口与兼容性**：公共名称、参数、返回值、序列化、事件注册、Lua 绑定、JSON Schema 和调用方是否同步；旧数据或旧调用是否仍有定义明确的行为。
3. **资源与安全**：所有权、指针和引用有效期、文件和网络输入、权限边界、注入风险、敏感数据及错误信息泄露。只在有具体触发条件时提出安全结论。
4. **性能与规模**：算法复杂度、重复加载、无界增长、无意义复制、热点路径上的分配和锁；考虑真实调用频率与数据规模，不因理论上的优化空间阻塞正常改动。
5. **测试**：测试是否覆盖本次改变的行为、失败路径和关键边界。指出具体缺口与风险，不用“缺少测试”作为没有说明的通用意见，也不得强求影响范围较小的改动必须建设测试工具。
6. **维护性**：是否复用仓库已有实现，是否新增了不必要的层次、重复逻辑、过度防御、无效注释或与相邻代码冲突的惯例。风格意见通常低于行为缺陷。

## 证据与严重度

每条发现都应能指向文件和位置，并说明触发条件、实际后果和证据。证据可以来自测试、调用链、契约、可复现输入或明确的源码行为。单纯的可能性和个人偏好不能包装成确定缺陷。

建议采用以下等级：

- **P0**：明确会导致无法构建、严重数据损坏、严重安全问题或游戏核心功能不可用、玩家无法载入存档，且有直接证据。
- **P1**：影响较大或容易触发的功能缺陷，应尽快处理；需要有具体案例或清晰的调用路径。
- **P2**：存在较大可能的行为问题、兼容问题、维护风险或明显的非必要复杂度，建议修复。
- **P3**：影响有限、证据不足或属于可选改进；不应阻塞合并；或仅能作为建议性。

发现格式应包含：严重度、文件与行号、问题、触发条件或证据、建议的处理方向。若问题不能确认，写成待确认的问题，并说明需要检查什么。

## 审查结论

审查结论可以是请求修改、提出非阻塞意见或未发现明确问题。没有发现问题时，说明审查过的范围和仍未验证的部分，不为凑数量制造意见。格式、措辞和提交说明问题通常不高于 P2；PR 描述是否符合发布习惯交由项目维护者决定。

审查报告描述最终改动和可观察影响，不写审查过程中的闲聊、个人情绪或自动化工具历史。如果环境配置有GitHub-MCP等工具，可尝试同时读取 CI check 中失败项进行相互验证，但不得将 CI check failure 直接视为P0错误或唯一问题；且若项目的 CI 中存在与本 PR 无关的历史失败、待授权生成物或其他已知噪声，只有在它实际影响本次改动时才纳入审查。
