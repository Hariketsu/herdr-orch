# herdr-orch

[English](README.md) | 简体中文

构建在 `herdr-core`（`hc`）之上的多 Agent 编排技能，用于将协调者 Agent 与不同能力档位的外部编码 Worker（如 Codex 与 Antigravity CLI）可靠地协同起来。

## 为什么需要 herdr-orch

不同模型各有擅长区间、成本曲线与典型失效模式：

- **Codex（`gpt-6.2-sol`）**：擅长跨仓静态事实核对、反向 TDD 红灯测试编写、多文件 rebase 冲突解决和严苛的缺陷审查；但弱于面向人的表达、容易顺手越界重构，且倾向把单次验收探针假设焊死进生产结构。
- **Antigravity（`gemini-4-neon`）**：擅长可长期维护的模块抽象、1M 长上下文跨仓综合与并行工程实现；但偶发遗留测试 fake/桩代码，且开到 `xhigh` 时边际收益偏低、token 消耗显著增加。
- **Antigravity（`gemini-4-hydrogen`）**：具备 1M 平价上下文，成本约为 Neon 的一半、速度约 1.5 倍，适合大仓/长日志初筛摘录和模式明确的批量执行；但缺少可执行校验时容易过早宣称完成，且在超长多轮会话中状态易退化。

`herdr-orch` 在不重复 `herdr-core` 自身 CLI 文档的前提下，沉淀了一套跨模型分派与验收的工程纪律：

- **清晰的委派边界与模型路由**：将边界清晰的模块实现、反向 TDD 出题、大仓初筛、复杂冲突解决和独立 Code Review 按档位派给合适的 Worker；小改动、顶层产品/架构对齐、UI 工艺与面向人的文案始终留在协调者侧。
- **经核实的简报与逃生舱**：写 Brief 前先在代码中核实路径与符号，写明完整校验命令、锁死可编辑文件范围，并显式授权 Worker 在遇到上游规则或设计冲突时中止并回报 `blocked`。
- **工作区与权限隔离**：纯审计/调研使用只读约束；编码、rebase 与合并实验一律在独立 `git worktree` 分支中进行，并保留禁止 `git push` / 外部写操作的硬红线。
- **独立审计与熔断机制**：不凭口头汇报或 `git show --stat` 与绿灯测试直接放行，协调者必须通读核心实现 diff（兼顾防越界与工程可维护性）、独立跑通完整门禁，并将返工/审查收敛轮次上限锁在两轮。
- **模块化模型卡**：所有模型画像与 CLI 细节按单层扁平文件放在 `references/` 下，新增或更新模型无需重写主干编排流程。

## 目录结构

```text
herdr-orch/
├── SKILL.md                           # 核心编排流程与模型路由
├── references/
│   ├── codex-gpt-6.2-sol.md           # Codex (gpt-6.2-sol) 模型卡
│   ├── agy-gemini-4-neon.md           # Antigravity (gemini-4-neon) 模型卡
│   └── agy-gemini-4-hydrogen.md       # Antigravity (gemini-4-hydrogen) 模型卡
├── README.md                          # 英文说明
└── README.zh-CN.md                    # 简体中文说明
```

## 前置条件

- 已安装 `herdr-core`（`hc`）及其对应的 `herdr-core` 技能。
- 已安装一个或多个受支持的 Worker CLI（当前已内置模型卡：OpenAI Codex `gpt-6.2-sol`，以及 Antigravity CLI `agy` 的 `gemini-4-neon` 与 `gemini-4-hydrogen`）。

## 安装

```bash
npx skills add Hariketsu/herdr-orch
```

或从本地检出目录安装：

```bash
npx skills add /path/to/herdr-orch
```

## 扩展新模型

在 `references/<agent>-<model>.md` 新增一张单层扁平模型卡，涵盖以下 5 节：

- Profile（强项与已观测到的失效模式）
- Effort sizing（推理档位选型）
- Briefing guardrails（简报护栏）
- Sandbox / CLI specifics（启动参数与权限隔离）
- Coordinator audit checklist（协调者审计清单）

随后在 `SKILL.md` 的 `## Model profiles` 下新增一行路由说明即可。
