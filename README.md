---
name: multiagent
description: 部署多Agent协作系统到OpenCode环境。包含PM主Agent和4个子Agent（analyst、architect、devops、qa），实现从需求分析到质量验证的完整软件开发流程。触发场景：需要配置多Agent协作环境、部署Agent和Skills定义文件、配置Agent权限和模型时使用。
---

# OpenCode 多Agent协作系统部署指南

## 系统架构

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

## 安装部署

### 方式一：项目级安装（推荐）

**步骤**：

1. 复制配置文件到项目根目录：
   ```
   你的项目/opencode.json
   ```

2. 复制 `.opencode/` 目录到项目根目录：
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

3. 修改 `opencode.json` 中的模型配置：
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
   - `glm5/glm5`

4. 启动OpenCode，配置自动加载

### 方式二：全局安装

1. 复制Agent定义到全局目录：
   ```
   Windows: %USERPROFILE%\.config\opencode\agents\
   Linux/macOS: ~/.config/opencode/agents/
   ```

2. 复制Skills定义到全局目录：
   ```
   Windows: %USERPROFILE%\.config\opencode\skills\
   Linux/macOS: ~/.config/opencode/skills/
   ```

3. 合并 `opencode.json` 到全局配置（项目配置优先级更高）

## Agent配置

| Agent | 角色 | 模式 | 职责 | Skill |
|-------|------|------|------|-------|
| pm | 项目经理 | primary | 编排调度子Agent | pm-orchestrator |
| analyst | 需求分析师 | subagent | 编写PRD | analyst-prd |
| architect | 系统架构师 | subagent | 系统设计 | architect-design |
| devops | 开发工程师 | subagent | 编码实现 | devops-coding |
| qa | 质量工程师 | subagent | 质量验证 | qa-verify |

## 工作流程

PM Agent编排顺序：

1. **需求分析** — PM拆解需求，创建任务列表
2. **PRD编写** — 调用 `@analyst`，产出 `docs/prd.md`
3. **系统设计** — 调用 `@architect`，产出 `docs/design.md`
4. **编码实现** — 调用 `@devops`，产出源代码和测试代码
5. **质量验证** — 调用 `@qa`，产出 `docs/qa-report.md`
6. **交付汇总** — PM汇总产出物，向用户报告

## 使用方式

### 启动PM Agent

在OpenCode TUI中按 `Tab` 键切换到 `pm` Agent

### 直接使用子Agent

通过 `@` 提及直接调用：
```
@analyst 帮我写一个用户登录功能的PRD
@architect 分析一下当前项目的代码结构
@devops 实现用户登录接口
@qa 检查最近的代码变更
```

### 触发完整流程

向PM提出需求：
```
我需要开发一个用户注册登录功能，支持手机号和邮箱两种注册方式
```

PM自动编排所有子Agent完成全流程。

## 模型配置建议

| Agent | 推荐模型类型 | 原因 |
|-------|-------------|------|
| pm | 强推理模型 | 需理解需求、编排任务 |
| analyst | 强推理模型 | 需深度理解需求并结构化输出 |
| architect | 强推理模型 | 需分析代码、设计架构 |
| devops | 代码能力强模型 | 需高质量代码输出 |
| qa | 严谨模型 | 需细致审查、不遗漏问题 |

配置示例：
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

## 权限配置

| Agent | 文件编辑 | Bash命令 | Skill | Task调用 |
|-------|---------|----------|-------|----------|
| pm | 允许 | 允许 | pm-orchestrator | analyst/architect/devops/qa |
| analyst | 允许 | 需确认 | analyst-prd | 无 |
| architect | 允许 | 读取类允许 | architect-design | 无 |
| devops | 允许 | 允许 | devops-coding | 无 |
| qa | 允许 | 测试/lint类允许 | qa-verify | 无 |

## 产出物

完整流程执行后产出：

```
docs/
├── prd.md           # 产品需求文档（analyst产出）
├── design.md        # 技术设计文档（architect产出）
└── qa-report.md     # 质量验证报告（qa产出）

src/                 # 源代码（devops产出）
tests/               # 测试代码（devops产出）
```

## 常见问题

### Skills安装路径

**项目级**：`<项目根目录>/.opencode/skills/<skill-name>/SKILL.md`

**全局级**：
- Windows: `%USERPROFILE%\.config\opencode\skills\<skill-name>\SKILL.md`
- Linux/macOS: `~/.config/opencode/skills/<skill-name>\SKILL.md`

### Agent定义文件路径

**项目级**：`<项目根目录>/.opencode/agents/<agent-name>.md`

**全局级**：
- Windows: `%USERPROFILE%\.config\opencode\agents\<agent-name>.md`
- Linux/macOS: `~/.config/opencode/agents/<agent-name>.md`

### 修改Agent模型

两种方式：
1. 修改 `opencode.json` 中对应agent的 `model` 字段
2. 修改 `.opencode/agents/<agent-name>.md` 中frontmatter的 `model` 字段

项目级配置优先于全局配置。

### 禁用Agent

在 `opencode.json` 中添加：
```json
{
  "agent": {
    "qa": { "disable": true }
  }
}
```

### 新增子Agent

1. 在 `.opencode/agents/` 下创建 `<name>.md`，包含frontmatter（description、mode: subagent、model等）
2. 在 `opencode.json` 的 `pm` agent的 `permission.task` 中添加允许调用
3. （可选）在 `.opencode/skills/` 下创建对应的SKILL.md

### Skill文件命名要求

- `SKILL.md` 必须全大写
- 目录名必须与frontmatter中的 `name` 一致
- name只允许小写字母、数字和连字符，正则：`^[a-z0-9]+(-[a-z0-9]+)*$`
