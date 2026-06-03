# OpenCode 多Agent协作 Skills 部署说明

## 概述

本套 Skills 实现了一个 PM 编排驱动的多 Agent 协作系统，包含 1 个 PM 主 Agent 和 4 个子 Agent，覆盖从需求分析到质量验证的完整软件开发流程。

## 架构说明

```
用户需求
   │
   ▼
┌──────────┐
│   PM     │  主Agent（primary）- 编排调度
│ 编排者    │
└────┬─────┘
     │ Task 工具调度
     ├──▶ @analyst    → 编写PRD
     ├──▶ @architect  → 系统设计 + 存量代码解析
     ├──▶ @devops     → 编码实现
     └──▶ @qa         → 质量验证
```

## 文件结构

```
项目根目录/
├── opencode.json                              # 项目级配置（agent定义、模型、权限）
└── .opencode/
    ├── agents/                                # Agent 定义文件（Markdown格式）
    │   ├── pm.md                              # PM 主Agent
    │   ├── analyst.md                         # 需求分析师子Agent
    │   ├── architect.md                       # 系统架构师子Agent
    │   ├── devops.md                          # 开发工程师子Agent
    │   └── qa.md                              # 质量工程师子Agent
    └── skills/                                # Skills 定义文件
        ├── pm-orchestrator/SKILL.md           # PM编排技能
        ├── analyst-prd/SKILL.md               # PRD编写技能
        ├── architect-design/SKILL.md          # 架构设计技能
        ├── devops-coding/SKILL.md             # 编码实现技能
        └── qa-verify/SKILL.md                 # 质量验证技能
```

## 安装部署步骤

### 方式一：项目级安装（推荐）

将文件放置在项目根目录下，仅对当前项目生效。

1. **复制配置文件**

   将 `opencode.json` 复制到项目根目录：

   ```
   你的项目/opencode.json
   ```

2. **复制 .opencode 目录**

   将 `.opencode/` 目录复制到项目根目录：

   ```
   你的项目/.opencode/
   ├── agents/
   │   ├── pm.md
   │   ├── analyst.md
   │   ├── architect.md
   │   ├── devops.md
   │   └── qa.md
   └── skills/
       ├── pm-orchestrator/SKILL.md
       ├── analyst-prd/SKILL.md
       ├── architect-design/SKILL.md
       ├── devops-coding/SKILL.md
       └── qa-verify/SKILL.md
   ```

3. **修改模型配置**

   编辑 `opencode.json`，将各 agent 的 `model` 字段改为你可用的模型。默认使用 `glm5/glm5`：

   ```json
   {
     "agent": {
       "pm": { "model": "glm5/glm5" },
       "analyst": { "model": "glm5/glm5" },
       "architect": { "model": "glm5/glm5" },
       "devops": { "model": "glm5/glm5" },
       "qa": { "model": "glm5/glm5" }
     }
   }
   ```

   模型格式为 `provider/model-id`，例如：
   - `anthropic/claude-sonnet-4-20250514`
   - `openai/gpt-4o`
   - `glm5/glm5`（当前配置的GLM5兼容接口）

4. **启动 OpenCode**

   在项目根目录运行 `opencode`，配置将自动加载。

### 方式二：全局安装

将文件放置在用户全局配置目录下，对所有项目生效。

1. **复制 Agent 定义**

   将 `agents/` 下的 `.md` 文件复制到全局目录：

   ```
   Windows: %USERPROFILE%\.config\opencode\agents\
   Linux/macOS: ~/.config/opencode/agents/
   ```

2. **复制 Skills 定义**

   将 `skills/` 下的目录复制到全局目录：

   ```
   Windows: %USERPROFILE%\.config\opencode\skills\
   Linux/macOS: ~/.config/opencode/skills/
   ```

3. **合并 opencode.json 配置**

   将 agent 配置合并到全局 `opencode.json`：

   ```
   Windows: %USERPROFILE%\.config\opencode\opencode.json
   Linux/macOS: ~/.config/opencode/opencode.json
   ```

   注意：全局配置与项目配置会合并，项目配置优先级更高。

## Agent 说明

| Agent | 角色 | 模式 | 职责 | 默认模型 |
|-------|------|------|------|----------|
| pm | 项目经理 | primary | 编排调度子Agent，驱动全流程 | glm5/glm5 |
| analyst | 需求分析师 | subagent | 编写PRD产品需求文档 | glm5/glm5 |
| architect | 系统架构师 | subagent | 系统设计、存量代码解析 | glm5/glm5 |
| devops | 开发工程师 | subagent | 编码、功能实现 | glm5/glm5 |
| qa | 质量工程师 | subagent | 代码审查、测试验证 | glm5/glm5 |

## Skill 说明

| Skill | 对应Agent | 功能描述 |
|-------|-----------|----------|
| pm-orchestrator | pm | PM编排流程，定义工作流和产出物标准 |
| analyst-prd | analyst | PRD文档结构、编写规范和产出要求 |
| architect-design | architect | 架构设计流程、存量代码解析步骤、设计文档结构 |
| devops-coding | devops | 编码工作流、编码规范和产出要求 |
| qa-verify | qa | 验证流程、质量报告结构和质量标准定义 |

## 工作流程

PM Agent 接收到用户需求后，按以下顺序编排：

1. **需求分析** — PM 拆解需求，创建任务列表
2. **PRD编写** — 调用 `@analyst`，产出 `docs/prd.md`
3. **系统设计** — 调用 `@architect`，产出 `docs/design.md`（含存量代码分析）
4. **编码实现** — 调用 `@devops`，产出源代码和测试代码
5. **质量验证** — 调用 `@qa`，产出 `docs/qa-report.md`
6. **交付汇总** — PM 汇总所有产出物，向用户报告

## 使用方式

### 启动后切换到 PM Agent

在 OpenCode TUI 中按 `Tab` 键切换到 `pm` Agent。

### 直接使用子Agent

也可以通过 `@` 提及直接使用子Agent：

```
@analyst 帮我写一个用户登录功能的PRD
@architect 分析一下当前项目的代码结构
@devops 实现用户登录接口
@qa 检查最近的代码变更
```

### 典型用法

直接向 PM 提出需求即可触发完整流程：

```
我需要开发一个用户注册登录功能，支持手机号和邮箱两种注册方式
```

PM 会自动编排所有子Agent完成从PRD到质量验证的全流程。

## 模型配置说明

每个 Agent 可以独立配置使用的模型，在 `opencode.json` 或 `.md` Agent文件中修改 `model` 字段：

```json
"model": "provider/model-id"
```

### 推荐模型搭配

| Agent | 推荐模型类型 | 原因 |
|-------|-------------|------|
| pm | 强推理模型 | 需要理解需求、编排任务 |
| analyst | 强推理模型 | 需要深度理解需求并结构化输出 |
| architect | 强推理模型 | 需要分析代码、设计架构 |
| devops | 代码能力强模型 | 需要高质量代码输出 |
| qa | 严谨模型 | 需要细致审查、不遗漏问题 |

### 配置不同模型示例

```json
{
  "agent": {
    "pm":       { "model": "anthropic/claude-sonnet-4-20250514" },
    "analyst":  { "model": "anthropic/claude-sonnet-4-20250514" },
    "architect":{ "model": "anthropic/claude-sonnet-4-20250514" },
    "devops":   { "model": "anthropic/claude-sonnet-4-20250514" },
    "qa":       { "model": "anthropic/claude-haiku-4-20250514" }
  }
}
```

## 权限说明

| Agent | 文件编辑 | Bash命令 | Skill | Task调用 |
|-------|---------|----------|-------|----------|
| pm | 允许 | 允许 | pm-orchestrator | analyst/architect/devops/qa |
| analyst | 允许 | 需确认 | analyst-prd | 无 |
| architect | 允许 | 读取类允许，其他需确认 | architect-design | 无 |
| devops | 允许 | 允许 | devops-coding | 无 |
| qa | 允许 | 测试/lint类允许，其他需确认 | qa-verify | 无 |

## 产出物

完整流程执行后，项目将产出以下文件：

```
docs/
├── prd.md           # 产品需求文档（analyst产出）
├── design.md        # 技术设计文档（architect产出）
└── qa-report.md     # 质量验证报告（qa产出）

src/                 # 源代码（devops产出，路径视项目结构而定）
tests/               # 测试代码（devops产出，路径视项目结构而定）
```

## 常见问题

### Q: Skills 安装在哪个目录？

**项目级**：`<项目根目录>/.opencode/skills/<skill-name>/SKILL.md`

**全局级**：
- Windows: `%USERPROFILE%\.config\opencode\skills\<skill-name>\SKILL.md`
- Linux/macOS: `~/.config/opencode/skills/<skill-name>/SKILL.md`

### Q: Agent 定义文件放在哪个目录？

**项目级**：`<项目根目录>/.opencode/agents/<agent-name>.md`

**全局级**：
- Windows: `%USERPROFILE%\.config\opencode\agents\<agent-name>.md`
- Linux/macOS: `~/.config/opencode/agents/<agent-name>.md`

### Q: 如何修改 Agent 使用的模型？

有两种方式：
1. 修改 `opencode.json` 中对应 agent 的 `model` 字段
2. 修改 `.opencode/agents/<agent-name>.md` 中 frontmatter 的 `model` 字段

项目级配置优先于全局配置。

### Q: 如何禁用某个 Agent？

在 `opencode.json` 中添加 `"disable": true`：

```json
{
  "agent": {
    "qa": { "disable": true }
  }
}
```

### Q: 如何新增一个子Agent？

1. 在 `.opencode/agents/` 下创建 `<name>.md`，包含 frontmatter（description、mode: subagent、model 等）
2. 在 `opencode.json` 的 `pm` agent 的 `permission.task` 中添加允许调用
3. （可选）在 `.opencode/skills/` 下创建对应的 SKILL.md

### Q: Skill 文件名有什么要求？

- `SKILL.md` 必须全大写
- 目录名必须与 frontmatter 中的 `name` 一致
- name 只允许小写字母、数字和连字符，正则：`^[a-z0-9]+(-[a-z0-9]+)*$`
