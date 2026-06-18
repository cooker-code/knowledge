> 已吸收至：[[02_Agent与AI工程/0205_AI编程工具/020502_Claude Code/020502_核心知识点/ClaudeCode工程使用与治理边界|ClaudeCode工程使用与治理边界]]
---
title: CLAUDE.md决定了你Vibe Coding的开发风格和效率
author: AI编程实验室
date:
url: https://mp.weixin.qq.com/s?__biz=MzE5ODY5MDU4Mw==&mid=2247483674&idx=1&sn=08da0f3cad7622ddd22ea0f3c58de4c0&chksm=97afd129734b0940d63b09718b135abc9be89caeccab2e0bbf546327c24d6315e3a303c10861&mpshare=1&scene=24&srcid=1107nMZMnUdu7Ow6KMXPyYln&sharer_shareinfo=78cad3134374794ff9859f7f7c349f72&sharer_shareinfo_first=78cad3134374794ff9859f7f7c349f72#rd
---

大家好，我是鲁工。

经常用Claude Code的朋友都知道，在正式给模型提需求前，一定要写好CLAUDE.md文档，它决定了你用Claude Code进行Vibe Coding的风格和效率。

CLAUDE.md是什么呢？

CLAUDE.md文档是为Claude Code提供项目特定上下文和指令的配置文件。这些文件充当项目的持久记忆（Memory），Claude Code在运行时会自动读取并添加到上下文中作为系统提示词（System Prompt）。

具体地，CLAUDE.md分为四个层级：

作为个人用户，我们只需要关注用户级别和项目级别的CLAUDE.md文档配置即可。其中用户级别的配置可以视作你的Claude Code编码全局配置，配置地址为~/.claude/CLAUDE.md，用于跨所有项目的全局指令。项目配置的CLAUDE.md可以直接放置在项目根目录下或者一些子目录下：./CLAUDE.md或.claude/CLAUDE.md。

CLAUDE.md文档一般可以包含编码指南和标准、项目特定上下文、常用命令或工作流程、API约定以及测试要求等信息。

可能有朋友就要问了，有没有现成的CLAUDE.md文档可以抄？那必须有。下面我把我珍藏了好几个月的全局配置CLAUDE.md分享给大家，这套全局配置浓缩了一个几十年经验开发者的开发风格，具体如下：

```
# Development Guidelines
## Philosophy
### Core Beliefs
- **Incremental progress over big bangs** - Small changes that compile and pass tests- **Learning from existing code** - Study and plan before implementing- **Pragmatic over dogmatic** - Adapt to project reality- **Clear intent over clever code** - Be boring and obvious
### Simplicity Means
- Single responsibility per function/class- Avoid premature abstractions- No clever tricks - choose the boring solution- If you need to explain it, it's too complex
### Permission ManagementExcept for file deletion and database operations, all other operations do not require my confirmation.
## Process
### 1. Planning & Staging
Break complex work into 3-5 stages. Document in `IMPLEMENTATION_PLAN.md`:
```markdown## Stage N: [Name]**Goal**: [Specific deliverable]**Success Criteria**: [Testable outcomes]**Tests**: [Specific test cases]**Status**: [Not Started|In Progress|Complete]```- Update status as you progress- Remove file when all stages are done
### 2. Implementation Flow
1. **Understand** - Study existing patterns in codebase2. **Test** - Write test first (red)3. **Implement** - Minimal code to pass (green)4. **Refactor** - Clean up with tests passing5. **Commit** - With clear message linking to plan
### 3. When Stuck (After 3 Attempts)
**CRITICAL**: Maximum 3 attempts per issue, then STOP.
1. **Document what failed**:   - What you tried   - Specific error messages   - Why you think it failed
2. **Research alternatives**:   - Find 2-3 similar implementations   - Note different approaches used
3. **Question fundamentals**:   - Is this the right abstraction level?   - Can this be split into smaller problems?   - Is there a simpler approach entirely?
4. **Try different angle**:   - Different library/framework feature?   - Different architectural pattern?   - Remove abstraction instead of adding?
## Technical Standards
### Architecture Principles
- **Composition over inheritance** - Use dependency injection- **Interfaces over singletons** - Enable testing and flexibility- **Explicit over implicit** - Clear data flow and dependencies- **Test-driven when possible** - Never disable tests, fix them
### Code Quality
- **Every commit must**:  - Compile successfully  - Pass all existing tests  - Include tests for new functionality  - Follow project formatting/linting
- **Before committing**:  - Run formatters/linters  - Self-review changes  - Ensure commit message explains "why"
### Error Handling
- Fail fast with descriptive messages- Include context for debugging- Handle errors at appropriate level- Never silently swallow exceptions
## Decision Framework
When multiple valid approaches exist, choose based on:
1. **Testability** - Can I easily test this?2. **Readability** - Will someone understand this in 6 months?3. **Consistency** - Does this match project patterns?4. **Simplicity** - Is this the simplest solution that works?5. **Reversibility** - How hard to change later?
## Project Integration
### Learning the Codebase
- Find 3 similar features/components- Identify common patterns and conventions- Use same libraries/utilities when possible- Follow existing test patterns
### Tooling
- Use project's existing build system- Use project's test framework- Use project's formatter/linter settings- Don't introduce new tools without strong justification
## Quality Gates
### Definition of Done
- [ ] Tests written and passing- [ ] Code follows project conventions- [ ] No linter/formatter warnings- [ ] Commit messages are clear- [ ] Implementation matches plan- [ ] No TODOs without issue numbers
### Test Guidelines
- Test behavior, not implementation- One assertion per test when possible- Clear test names describing scenario- Use existing test utilities/helpers- Tests should be deterministic
## Important Reminders
**NEVER**:- Use `--no-verify` to bypass commit hooks- Disable tests instead of fixing them- Commit code that doesn't compile- Make assumptions - verify with existing code
**ALWAYS**:- Commit working code incrementally- Update plan documentation as you go- Learn from existing implementations- Stop after 3 failed attempts and reassess- Always use context7 when I need code generation, setup or configuration steps, orlibrary/API documentation. This means you should automatically use the Context7 MCPtools to resolve library id and get library docs without me having to explicitly ask.
```

这个全局CLAUDE.md配置参考的是国外顶级开发者Chris Dzombak写的Claude Code深度使用经验总结，其中核心的开发哲学是：

渐进式改进而非一次性大改动：进行小规模修改并确保通过测试

从现有代码中学习：在实施前进行研究和规划

务实而非教条：适应项目实际情况

我们可以在Claude Code中查看Claude对其全局配置的总结：

需要注意的是，跟MCP配置数量一样，CLAUDE.md文档也并不是需要面面俱到，越长越好。太长的系统提示词会占据较多的上下文空间，一定程度上也会稀释真正的项目信息。

但我这套系统配置大概总共占到1.6%的上下文空间，长度上还是相对合理的。

除此之外，在项目开发过程中，我们也可以随时使用 # 符号添加记忆，也可以使用/memory命令随时对CLAUDE.md进行编辑：

相信你用了这套配置，使用Claude Code来Vibe Coding的效率会有所提升。

感谢您阅读我的文章。我是鲁工，八年AI算法老兵，AI全栈开发者，深耕AI编程赛道。欢迎关注我，一起Vibe Coding。感兴趣的朋友也可以加我微信（louwill\_）交个朋友。

>/ 作者：鲁工