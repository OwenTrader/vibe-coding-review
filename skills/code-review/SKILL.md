---
name: code-review
description: 多维度深度代码审查技能。当用户要求"代码审查 / code review / 审查代码 / review 这个 PR / 检查代码质量 / 审查这个模块 / 审计代码 / 上线前审查"时使用。覆盖 UI/UX 样式一致性与公共样式抽离、代码质量、安全边界、关键参数硬编码审计、CRUD 完整性、产品缺口、功能链路、日志规范（关键路径有日志且日志不无限堆积）、可观测性、架构合理性、业务逻辑正确性、数据一致性、缓存策略、消息队列、容灾回滚、部署发布、数据库设计、第三方依赖、技术债务、工具链与静态代码检测（Lint 接入）、测试策略与质量门禁、时间与时钟正确性、环境隔离与生产数据流转、AI/LLM 集成安全与成本、移动端特有安全、客户端错误上报与用户反馈等 55 个维度，输出分级（P0-P3）结构化审查报告。
license: Apache-2.0
agent_created: true
---

# Code Review — 多维度深度代码审查

## 目的

对目标代码（PR diff、指定目录、指定文件或整个模块）执行系统化多维度审查，产出按严重程度分级、可直接执行修复的结构化报告。审查基于代码证据，禁止纯推测——每个问题必须给出文件路径与行号。

## 审查工作流

1. **确定审查范围与输入**：与用户确认审查对象（diff / 目录 / 文件列表）与重点维度。未指定审查对象时默认全维度审查当前变更（`git diff`）及其直接关联代码。同时确认以下输入（缺失时向用户询问；无法获取时基于代码推断，并在报告"审查假设"中显式声明）：
   - **技术栈**：语言与运行时版本、前端框架、样式方案（CSS/Less/Sass/Tailwind/CSS-in-JS/组件库）、后端框架与 ORM、数据库与版本。
   - **设计规范来源**：设计 token / 主题变量 / 组件库文档的出处——这是维度 1「样式一致性」的比对基准，缺失时不得套用无关设计教条。
   - **接口契约**：接口文档 / OpenAPI / proto / 类型定义（维度 14、39 的比对基准）。
   - **已知约束**：性能目标、兼容范围（浏览器 / 机型 / 最低客户端版本）、合规要求（个保法 / 等保 / 财务留存 / 未成年人保护）。
   - **当前分支与改动状态**：`git status` / `git log -1` / 是否存在未提交改动（决定维度 21「变更集」的审查对象，避免漏审本地未提交改动）。
2. **建立上下文**：先读项目结构、构建配置、既有公共组件/样式/工具函数，再读被审代码。发现可复用资产是"公共样式/公共逻辑抽离"维度的前提。
3. **逐维度审查**：按下表加载对应 references 检查清单逐项核查。每个检查项标注**三态**（`已检查` / `发现问题` / `不适用`），每个发现记录：维度、严重级别、文件:行号、问题描述、证据代码片段、修复建议。
4. **交叉验证**：对 P0/P1 发现，必须二次确认（读调用方、读配置、读测试），排除误报后再写入报告。
5. **输出报告**：按 `references/report-template.md` 的格式输出，必须包含**审查覆盖度声明**（实跑 / 未跑维度及原因，三态汇总）。P0/P1 必须给出具体修复代码或补丁级建议。
6. **修复（可选）**：用户确认后按报告逐项修复，修复后复核不引入新问题。
7. **误报反馈回路**：报告交付后，若用户驳回某条发现，记录其裁定归类（规则过严 / 证据不足 / 项目约定例外）到报告"误报反馈"区；**同一项目后续审查对该类已裁定例外不再重复报出**，避免同类误报反复出现、降低报告可信度。

## 审查维度与检查清单

| # | 维度 | 检查清单 | 默认启用 |
|---|------|----------|---------|
| 1 | UI/UX 样式一致性 + 公共样式抽离 | `references/ui-ux-consistency.md` | ✅ |
| 2 | 代码质量 | `references/code-quality.md` | ✅ |
| 3 | 安全边界 | `references/security.md` | ✅ |
| 4 | 关键参数硬编码审计 | `references/hardcoded-parameters.md` | ✅ |
| 5 | CRUD 完整性 | `references/crud.md` | ✅ |
| 6 | 产品缺口 | `references/product-gap.md` | ✅ |
| 7 | 功能链路完整性 | `references/functional-flow.md` | ✅ |
| 8 | 日志规范（关键路径覆盖 + 防无限堆积） | `references/logging.md` | ✅ |
| 9 | 可观测性（指标/追踪/告警/健康检查） | `references/observability.md` | ✅ |
| 10 | 错误处理与边界条件 | `references/extra-dimensions.md` | ✅ |
| 11 | 性能与资源管理 | `references/extra-dimensions.md` | ✅ |
| 12 | 并发、幂等与事务 | `references/extra-dimensions.md` | ✅ |
| 13 | 配置与魔法数 | `references/extra-dimensions.md` | ✅ |
| 14 | API 契约与接口设计 | `references/extra-dimensions.md` | ⬜ 按需 |
| 15 | 隐私与数据合规 | `references/extra-dimensions.md` | ⬜ 按需（涉个人数据时默认启用） |
| 16 | 国际化与无障碍 | `references/extra-dimensions.md` | ⬜ 按需 |
| 17 | 可测试性 | `references/extra-dimensions.md` | ⬜ 按需 |
| 18 | 兼容性与迁移 | `references/extra-dimensions.md` | ⬜ 按需 |
| 19 | 依赖与供应链 | `references/extra-dimensions.md` | ⬜ 按需 |
| 20 | 文档与注释 | `references/extra-dimensions.md` | ⬜ 按需 |
| 21 | 变更集与提交质量 | `references/extra-dimensions.md` | ✅（审查 diff/PR 时） |
| 22 | 架构合理性 | `references/advanced-dimensions.md` | ⬜ 大改动/新模块时 |
| 23 | 业务逻辑正确性 | `references/advanced-dimensions.md` | ✅ |
| 24 | 状态管理 | `references/advanced-dimensions.md` | ⬜ 含复杂状态时 |
| 25 | 权限与身份认证（深化） | `references/advanced-dimensions.md` | ⬜ 多角色系统（与维度 3 联动） |
| 26 | 数据一致性（跨系统） | `references/advanced-dimensions.md` | ⬜ 多存储系统时 |
| 27 | 缓存策略 | `references/advanced-dimensions.md` | ⬜ 有缓存时 |
| 28 | 异步任务与消息队列 | `references/advanced-dimensions.md` | ⬜ 有 MQ/异步任务时 |
| 29 | 容灾与故障恢复 | `references/advanced-dimensions.md` | ⬜ 上线前审查 |
| 30 | 限流与资源保护 | `references/advanced-dimensions.md` | ⬜ 公网服务 |
| 31 | 灾备与数据恢复 | `references/advanced-dimensions.md` | ⬜ 上线前审查 |
| 32 | 安全输入输出（深化） | `references/advanced-dimensions.md` | ⬜ 与维度 3 联动 |
| 33 | Secrets 管理（深化） | `references/advanced-dimensions.md` | ⬜ 与维度 3 联动 |
| 34 | 数据生命周期 | `references/advanced-dimensions.md` | ⬜ 按需 |
| 35 | Feature Flag / 灰度发布 | `references/advanced-dimensions.md` | ⬜ 有 flag 体系时 |
| 36 | 部署与发布 | `references/advanced-dimensions.md` | ⬜ 发布审查 |
| 37 | 可回滚性 | `references/advanced-dimensions.md` | ⬜ 发布审查 |
| 38 | 数据库设计 | `references/advanced-dimensions.md` | ⬜ schema 变更时 |
| 39 | API 版本与兼容策略 | `references/advanced-dimensions.md` | ⬜ 对外 API（与维度 14 联动） |
| 40 | 第三方服务依赖 | `references/advanced-dimensions.md` | ⬜ 有外部依赖时建议启用 |
| 41 | 任务调度 | `references/advanced-dimensions.md` | ⬜ 有定时任务时 |
| 42 | 资源生命周期 | `references/advanced-dimensions.md` | ✅（与维度 11 联动） |
| 43 | 前端交互状态 | `references/advanced-dimensions.md` | ✅ 前端项目（与维度 1 联动） |
| 44 | SEO / Web 性能 | `references/advanced-dimensions.md` | ⬜ 面向搜索引擎的站点 |
| 45 | 数据迁移 | `references/advanced-dimensions.md` | ⬜ 有迁移时 |
| 46 | 技术债务 | `references/advanced-dimensions.md` | ⬜ 定期体检 |
| 47 | 可维护性（工程级） | `references/advanced-dimensions.md` | ⬜ 定期体检 |
| 48 | 可扩展性 | `references/advanced-dimensions.md` | ⬜ 架构评审 |
| 49 | 工具链与静态代码检测（Lint 接入） | `references/tooling-static-analysis.md` | ✅ |
| 50 | 测试策略与质量门禁 | `references/extended-dimensions.md` | ✅ |
| 51 | 时间与时钟正确性 | `references/extended-dimensions.md` | ✅（交易/分布式/定时任务必启） |
| 52 | 环境隔离与生产数据流转 | `references/extended-dimensions.md` | ✅ |
| 53 | AI / LLM 集成安全与成本 | `references/extended-dimensions.md` | ⬜ 含 AI 功能时 |
| 54 | 移动端特有（存储/传输/完整性/生命周期） | `references/extended-dimensions.md` | ⬜ 移动端项目 |
| 55 | 客户端错误上报与用户反馈 | `references/extended-dimensions.md` | ⬜ 有前端/客户端时 |

前端项目（含 Astro/React/Vue/Flutter/Electron 界面层）必须启用维度 1、43；纯后端/脚本项目可跳过。长期运行的服务（守护进程、心跳任务、relay 服务）必须启用维度 9，建议叠加维度 29、30、41。移动端项目必须启用维度 54；含 AI/LLM 功能必须启用维度 53；有前端/客户端的产品启用维度 55；交易/资金/行情类项目必须启用维度 51。"上线前审查 / 发布前检查"场景默认追加维度 29、31、36、37。标注「联动」的维度与既有低编号维度共享边界，审查时合并执行、去重报告，避免同一问题计两次。

维度 49 与维度 2/7 的分工：维度 2/7 负责"人审查出控制流/缩进/死代码类错误"；维度 49 负责"这类错误为什么没被 ruff/ESLint 等工具自动拦住"——人审查出的低级错误必须同时在维度 49 追记工具链根因。

## 严重级别定义

| 级别 | 含义 | 处理要求 |
|------|------|----------|
| P0 | 阻断性：安全漏洞、数据丢失风险、功能不可用、生产事故隐患 | 必须修复后才能合并/发布 |
| P1 | 严重：逻辑错误、链路断裂、日志缺失导致无法排查、明显性能问题 | 应当修复，例外需记录理由 |
| P2 | 一般：一致性问题、可维护性缺陷、重复代码 | 建议修复，可排期 |
| P3 | 建议：风格优化、可选改进 | 酌情采纳 |

## 硬性原则

- **证据驱动**：每个问题附文件路径与行号，引用实际代码片段；禁止"可能存在"式的无证据推测（推测性风险单独标注为"待验证"）。
- **先读后评**：审查前先读相关既有实现（公共组件、样式变量、工具函数、日志封装），基于项目现状判断，不套用与项目技术栈无关的通用教条。
- **不改坏东西**：审查只读；修复阶段先获用户确认，且遵守项目既有规范（如：破坏性 git 操作前确认、magic numbers 提取到配置文件）。
- **报告闭环**：报告末尾给出修复优先级排序与"必须修复 / 可延后"的明确清单，不留模糊结论。
