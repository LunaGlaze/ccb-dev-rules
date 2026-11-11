# CodeReview

代码审查的任务是确认改动是否实现了目标、是否引入了可证明的缺陷，以及是否符合本仓库的契约和维护方式。审查意见应帮助作者处理问题，不用于展示知识、强加个人风格或把所有可改进处都列成阻塞项。

## 审查前

PR review 应依据 PR 标题、正文和关联需求理解改动意图，并以当前本地存储的master分支代码作为行为对照；不要只看少量差异猜测目的，也不要为重复取代码而改变本地仓库状态。若 PR 信息或基线不完整，说明缺少的证据。

PR review 默认只读取和汇报，不修改本地文件、不替作者修复、不提交意见、不推送远端。用户明确要求修复时，先把审查发现与实现任务分开，避免在未确认问题时扩大改动。

如果对本地代码进行审查，逻辑同理，仅由从PR中读取改动意图变为由会话记录或人类提供的最终文档判断。

### 审查目的

#### 1. The Review Mindset

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

#### 2. Effective Feedback

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

#### 3. Review Scope

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

审查结论可以是请求修改、提出非阻塞意见或未发现明确问题。没有发现问题时，说明审查过的范围和仍未验证的部分，不为凑数量制造意见。格式、措辞和提交说明问题通常不高于 P2。

审查报告描述最终改动和可观察影响，不写审查过程中的闲聊、个人情绪或自动化工具历史。如果环境配置有GitHub-MCP等工具，可尝试同时读取 CI check 中失败项进行相互验证，但不得将 CI check failure 直接视为P0错误或唯一问题；且若项目的 CI 中存在与本 PR 无关的历史失败、待授权生成物或其他已知噪声，只有在它实际影响本次改动时才纳入审查。

## 意图审查

除代码格式、可疑错误等内容外，如果审查是对于非用户的PR发起的，还需要对PR的意图进行判断，且不能默认PR描述是必然权威的：
 - 该PR是否与已有PR重复或冲突？
 - (如果该PR声称修复某些问题)该PR是否是一个有必要性的修复/PR，还是一个纯Agent自我感动式的无意义工作？
 - 该PR是否是针对的是核心内容且可能涉及真实性修改？若是，其是否符合现实直觉？
 - 该PR是否是“公平”的，即例如如果一个机制可能同时对角色和怪物生效，PR是否单独强化或削弱一方，或没有联动的另一方调整或计划，且原因够不合理？
 - 该PR是否有“只是为了给玩家添堵”的嫌疑，或仅仅是为了强化玩家/作弊？
 - 该PR是否是一刀切式的禁止行为？或者是否仅进行删除而不(考虑)增加代替方案？

## 最终审查原则约束

 - PR review 任务除非得到明确授权，否则不应改动/操作/写入任何本地文件(包括临时目录)；
 - 发起 PR review 任务时，本地储存库代码将会已经是最新版本，实际仅需要从GitHub对应PR中获取变更信息；任何对更改可能导致的后果的审计应当对比本地代码，而非尝试越狱进行重复克隆；
 - 在进行 review 时，需要根据线索判断真实意图，不得将仅凭少量代码猜测强加至贡献方；
 - 完成 PR review 任务后，应当对审查出的问题根据严重程度和重要性进行分类整理；
 - review 任务审查需要具体例证，不能仅靠推理猜测，如果无法找到明确证据，就不应认为是必须修复的项目(即无证据禁止列入P0~1)；
   + 明确证据的含义为: 具体的、必定触发的、仓库内已有案例/可被已处于被描述的预期行为触发；
   + 提出 review 回音前，就应对回音内容负责，需要进一步确认的问题应该在审查环节就进行确认；不应给出“可能A或B”一类的车葫芦话，应该验证真实行为后判断是否存在问题；
 - 整理涉及文件代码风格、处理常量导入排序和同文件其他函数代码使用不规范部分等行为，属于通过 CI check 的必须举措，禁止以`无关改动`等理由给出任何修改/拆分PR建议；
 - PR review 任务主要审查代码方面的问题，格式和规范相关内容优先级略低，通常不被视为必须修复项目(一般最高P2)；PR 描述格式问题交由人类审查不属于 agent 工作；
   + 功能实现的方法是否合理？是否属于典型的智能体绕行实现思维，在事实上有更简单规范的方法？
   + 是否存在重复造轮子的行为？是否符合同类功能既有惯例和调用规范？
   + 是否有完全无用的无效注释？是否有没有解释的魔法数字？
 - 并非所有 PR review 任务都要求必须找出可能的问题，当对应 PR 相对完善时可以直接承认自己找不到明显问题；
