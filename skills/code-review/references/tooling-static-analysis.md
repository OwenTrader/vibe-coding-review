# 维度 49：工具链与静态代码检测（Lint 接入）

**定位**：不审"代码写得对不对"（那是维度 2/7），审"**项目有没有自动拦截低级错误的机制**"。
低级错误（不可达代码、未使用变量/导入、缩进作用域漂移、条件里写赋值、吞异常）不应依赖人眼——必须交给语言对应的静态检测工具，并在 CI/提交环节强制阻断。

**边界**：维度 2（代码质量）与维度 7（功能链路）负责"人审查出控制流/缩进类错误"；本维度负责"工具链为什么没拦住"。人审查出的低级错误 → 除修复本身外，**必须在本维度追记一条根因**：工具链缺失或规则未启用。

---

## 49.1 按语言选型（对照表）

| 语言 | 首选 | 备选/补充 | 配置文件 |
|---|---|---|---|
| **Python** | **ruff**（检查+格式化一体，默认规则集已覆盖低级错误） | mypy（类型）、pylint、vulture（死代码）、bandit（安全） | `ruff.toml` / `pyproject.toml [tool.ruff]` |
| **TypeScript/JS** | **ESLint** + `@typescript-eslint` + `tsc --noEmit` | prettier、biome（一体化替代） | `eslint.config.js` / `.eslintrc.*` |
| **Go** | `go vet` + `golangci-lint`（含 staticcheck） | revive | `.golangci.yml` |
| **Rust** | `cargo clippy` | rustfmt | `clippy.toml` |
| **Dart/Flutter** | `dart analyze` + `flutter_lints` | — | `analysis_options.yaml` |
| **Java/Kotlin** | Checkstyle + SpotBugs / detekt | Error Prone | `checkstyle.xml` |
| **C/C++** | `clang-tidy` | cppcheck | `.clang-tidy` |
| **CSS** | stylelint | — | `.stylelintrc` |
| **Markdown** | markdownlint | vale | `.markdownlint.json` |

**规则**：项目主语言**必须**有"检查 + 格式化"至少其一；首选 ruff/ESLint 这类"开箱即用、默认规则集就能抓低级错误"的工具，不接受"装了但默认规则全关"。

## 49.2 接入完整性（装了 ≠ 接好了）

逐项核对（每项在报告中记 已检查/发现问题/不适用）：

- [ ] **配置文件存在且随代码入库**：团队共享同一规则集；不是"个人机器装了插件"。
- [ ] **抓低级错误的核心规则未关闭**（默认规则集即含，关闭 = P2）：
  - Python/ruff：`F401` 未使用导入、`F841` 未使用变量、`F811` 重复定义、`E722` 裸 except、**`PLW0101` 不可达代码**（return / `while True:` 之后的死代码）、`B006` 可变默认参数、`RUF100` 无效 noqa。
  - TS/ESLint：`no-unused-vars`（或 `@typescript-eslint/no-unused-vars`）、`no-unreachable`、`no-constant-condition`（`while(true)` 包裹范围）、`no-redeclare`、`no-shadow`、`no-cond-assign`、`no-empty`、`no-floating-promises`；`tsc --noEmit` 类型错误。
- [ ] **CI 或 pre-commit 强制阻断**：lint 非零退出码 → 合入失败；不是"提醒但可跳过"。接受形式：pre-commit 框架 / husky / lefthook / CI step，任一即可。
- [ ] **一键脚本入口**：`package.json scripts` / `Makefile` / `pyproject` 有 `lint`、`lint:fix`、`format`。
- [ ] **告警基线**：存量告警清零，或显式列入 baseline 并逐迭代偿还；**新增代码必须零告警**。
- [ ] **豁免有理由**：`# noqa` / `// eslint-disable` 必须带原因注释；大面积豁免 = P2。
- [ ] **缩进源头治理**（加分项）：`.editorconfig` 统一缩进与换行；`.vscode/extensions.json` 推荐插件。

## 49.3 低级错误清单（人审发现 = 工具链失守，双记）

审查中发现以下任一类时，除修复错误本身外，必须追记"工具链未拦截"根因（见 49.4 定级）：

| 低级错误 | 拦截规则（Python/ruff · TS/ESLint） |
|---|---|
| 不可达代码：return / `while True:` 之后仍有语句；**新增循环只包裹内层调用、外层循环体成死代码** | `PLW0101` · `no-unreachable`、`no-constant-condition` |
| 缩进/大括号作用域漂移：for 块未随 while 重新缩进，逻辑跑出循环 | 静态结构分析可部分发现；`.editorconfig` + 格式化器降低发生率（仍须维度 2/7 人审） |
| 未使用变量 / 导入 | `F841` · `F401` / `no-unused-vars` |
| 重复定义 / 遮蔽 | `F811` / `no-redeclare`、`no-shadow` |
| 条件里写赋值（`if x = 1`） | — / `no-cond-assign` |
| 裸 except / 空 catch 吞异常 | `E722` / `no-empty` |
| 未 await 的 Promise / 未闭合资源 | — / `no-floating-promises` |
| 魔法数（联动维度 4） | `PLR2004` / `no-magic-numbers` |

**典型事故形态**（作为触发联想，不指向具体项目）：给脚本加 `while True:` 心跳循环时只包裹了内层调用，外层 `for` 块未同步缩进 → 外层逻辑永不执行、进程零 IO 空转。这类错误 `PLW0101`/`no-unreachable` 可在 CI 直接拦下——有工具链就不会上线。

## 49.4 判定级别

| 级别 | 情形 |
|---|---|
| **P0** | 主线代码存在本可被默认规则集拦截的低级错误，且已造成运行时后果（死循环空转、资源泄漏、错误静默） |
| **P1** | 项目无 lint 配置；或配置存在但未入库/未在 CI 强制；或人审发现的低级错误本可被 linter 拦截（根因记"工具链缺失"） |
| **P2** | 装了但大面积关闭默认规则；无 CI 阻断；无一键脚本；存量告警无基线管理 |
| **P3** | 缺 `.editorconfig` / IDE 推荐插件；格式化未统一 |

## 49.5 报告输出：工具链缺口清单（本维度专有）

除常规发现外，单独输出：

1. **现状**：主语言、装了什么、配置是否入库、CI 是否阻断、告警数量。
2. **最小接入建议**：给出可直接粘贴的命令与配置文件内容（见 49.6）。
3. **"本可被 linter 拦截"清单**：本次人审查出的低级错误 → 文件:行号 → 对应规则编号 → 应开启的工具。

## 49.6 最小接入样例（可直接粘贴）

**Python（ruff）**：
```bash
pip install ruff
ruff check .            # 检查
ruff format --check .   # 格式
```
```toml
# ruff.toml —— 默认即覆盖低级错误，无需大改
target-version = "py312"
select = ["E","F","W","I","UP","B","SIM","PLW","PLR","RUF"]
line-length = 120
```

**TypeScript（ESLint + tsc）**：
```bash
pnpm add -D eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
npx eslint .
npx tsc --noEmit
```
```js
// eslint.config.js
import tseslint from "typescript-eslint";
export default [
  ...tseslint.configs.recommended,
  { rules: { "no-unreachable": "error", "no-constant-condition": "error" } },
];
```

**pre-commit（任选其一，跨语言通用）**：
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.6.0
    hooks:
      - id: ruff
      - id: ruff-format
```

**CI 阻断（GitHub Actions 示例）**：
```yaml
- run: ruff check .          # Python
- run: npx eslint . && npx tsc --noEmit   # TS
```
