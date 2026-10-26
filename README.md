# ACM Problem Creating Workflow

面向 ACM/ICPC 算法竞赛，尤其是 Codeforces Polygon 的 AI 辅助出题 Skills。通过主流程与独立验题两个入口，将数学定义、解法、testlib 程序、数据自测和 TeX 题面组织成有验证依据的工作流。

## 快速开始

先准备 Git 和可用的 Codex。只讨论 idea、数学定义和解法时，可以先完成下面三步；编译器、testlib、TeX 和 Polygon 凭据在需要执行相应阶段时配置。

**1. 克隆仓库。** 以下命令适用于 PowerShell 和 Bash，默认取得 `main` 的最新文档与 Skills：

```bash
git clone --depth 1 https://github.com/ycdada/ACM-Problem-creating-workflow-for-AI-agents.git
cd ACM-Problem-creating-workflow-for-AI-agents
```

**2. 在该目录打开 Codex。** 桌面版打开此项目；CLI 用户在该目录运行：

```bash
codex
```

尚未安装或登录时，按[安装与配置指南](docs/getting-started.md#准备-codex)操作。

**3. 将下面的请求发给 Codex。** `$acm-authoring` 是 Skill 调用名称，整段请求应在 Codex 聊天中输入：

```text
使用 $acm-authoring，在 problems/example/ 中整理一道候选题：
给定整数数组，求一个非空连续子数组的最大元素和。
本次只完成数学定义、解法与证明、约束建议和待决问题，暂不连接 Polygon。
```

预期产出是指定目录下的设计材料与任务报告；文件命名沿用 Skill 和项目约定。需要验题时，在新的干净上下文中调用 `$acm-independent-review` 并提供选手可见材料。

其他入口：[复制到自己的项目](docs/getting-started.md#安装到自己的项目)、[配置 testlib](docs/getting-started.md#配置-testlib)、[配置 Polygon](docs/getting-started.md#配置-polygon)、[常见问题](docs/getting-started.md#常见问题)。

## 你会得到什么

| Skill | 适用场景 | 主要产出 |
| --- | --- | --- |
| [acm-authoring](.agents/skills/acm-authoring/SKILL.md) | 从 idea 出发，或继续完善已有题目 | 数学定义与证明、各类程序、数据计划、自测记录、TeX 题面题解、Polygon 验题报告 |
| [acm-independent-review](.agents/skills/acm-independent-review/SKILL.md) | 根据选手可见题面独立解题验题 | 独立解与自测、首次结论冻结、歧义和反例、按请求进行的交叉核验 |

主流程按本次任务选择阶段，支持只讨论设计、只修复 validator 或只完善题面。两个 Skill 均以中文组织计划和报告。

```mermaid
flowchart LR
    A[Idea 与数学定义] --> B[证明与解题程序]
    B --> C[testlib 与数据设计]
    C --> D[对拍和错误解测试]
    D --> E[TeX 题面与题解]
    E --> F[Polygon 渲染与验题]
    F --> G[独立验题与交付]
```

## 安装

### 在本仓库中使用

按上面的快速开始克隆并打开项目即可。两个 Skill 位于仓库级 `.agents/skills/`，可以直接按名称调用。需要固定首版时，在克隆命令中加入 `--branch v0.1.0`；固定标签的更新方式见[详细指南](docs/getting-started.md#更新)。[Codex Skill 文档](https://developers.openai.com/codex/skills/)

### 在自己的题目项目中使用

将下列两个目录一起复制到目标题目仓库的 `.agents/skills/` 下，保持它们互为同级目录：

```text
.agents/skills/acm-authoring/
.agents/skills/acm-independent-review/
```

可直接使用[Windows PowerShell 或 Bash 安装命令](docs/getting-started.md#安装到自己的项目)。也可以在 Codex 中使用内置 Skill 安装器，安装到该安装器管理的用户级位置：

```text
请使用 skill-installer，从 GitHub 仓库
ycdada/ACM-Problem-creating-workflow-for-AI-agents
安装 .agents/skills/acm-authoring 和 .agents/skills/acm-independent-review，
使用 v0.1.0 版本。
```

两个目录同时安装后，主流程中的独立验题引用才能保持完整。其他支持 Agent Skills 的客户端可按其导入方式使用；当前已验证的入口与界面元数据面向 Codex。

## 使用

### 从 idea 开始

```text
使用 $acm-authoring，在 problems/example/ 中整理这个出题想法：……
本次只完成数学定义、候选解法、正确性证明和需要出题人决定的问题。
```

### 完善已有题目

```text
使用 $acm-authoring，检查 problems/example/ 的已有材料。
使用项目固定的 testlib，补齐 validator、generator、标程、暴力解、
错误解和慢解，完成适用范围内的自测，并保存出题报告。
```

可以进一步指定 TeX 题面、翻译、样例解释、题解或 Polygon 同步等范围。Skill 会读取相应阶段的参考文件，沿用项目已有工具与目录约定。

### 独立验题

在未接触当前题目内部材料的新上下文中，只提供选手可见的题面、样例、时空限制和公开附件：

```text
使用 $acm-independent-review，独立验题 problems/example/statement/ 中的题面。
本次只做盲解与自测，保存独立实现、验证记录和首次结果冻结。
```

需要完整验题时，再说明允许使用的正式数据、checker 或平台结果。独立验题先冻结自己的理解、解法与验证记录，再进行交叉核验。

已经读过该题标程、证明或生成计划的上下文会被标记为“受污染复核”，调用 Skill 不会清除已有信息。

## 程序与执行环境

当前版本交付流程指令、参考资料和 API 示例。编译器、运行器、TeX 渲染环境和平台连接由使用它的题目项目提供。

- **validator、generator、自定义 checker**：使用 C++ testlib；按输出语义选择适合的标准 checker，特殊题型再编写自定义 checker。
- **题面与题解**：维护 TeX 源文件，Polygon 的 HTML 与 PDF 渲染分别验证。
- **本地执行**：沿用现有 runner；也可评估 [rbx](https://rbx.rsalesc.dev/)。采用 rbx 时核对其系统要求，Windows 用户参考其 [WSL 安装说明](https://rbx.rsalesc.dev/intro/installation/)。
- **testlib 依赖**：由题目项目或用户提供，记录实际版本和哈希；本仓库未打包 `testlib.h`。

缺少工具、正式数据或平台连接的检查会如实标记为未执行。局部自测、证明判断与目标平台实测分别记录。

按阶段准备环境：

| 需要做什么 | 需要准备 |
| --- | --- |
| 设计题目、审阅文本 | Codex；通过 Git 克隆时需要 Git |
| 编译和自测 | 题目使用的编译器、项目 runner、固定版本的 `testlib.h` |
| 运行 Python API 示例 | Python 3 和 `requests`，建议使用项目虚拟环境 |
| 本地查看 TeX | 项目选用的 TeX 工具；平台渲染可走 Polygon 接口 |
| Polygon 同步与验题 | API key、secret、题目 ID，以及所需权限；有 PIN 时另行配置 |

具体命令与环境检查见[安装与配置指南](docs/getting-started.md)。本仓库的 API 示例从进程环境变量读取配置；不会自动加载 `.env`。

## Polygon API

[Polygon API 调用说明](.agents/skills/acm-authoring/references/polygon-api.md)包含：

- API key、时间戳与 SHA-512 签名。
- 参数编码、multipart 源文件上传与 Python 请求示例。
- validator、checker、解的 tag、时空限制与生成脚本配置。
- TeX 题面、样例、validator / checker 单测的保存方法。
- HTML / PDF 渲染、构建验题、逐解逐测试结果读取与完整性判断。
- 读回核对、工作副本关联及失败后的恢复处理。

接口以 [Polygon 官方文档](https://codeforces.github.io/polygon-misc/API)为准。示例通过了离线签名与模拟响应检查，尚未使用真实凭据连接 Polygon 实测。平台写操作按用户任务的授权范围执行。

## 设计说明

Skill 承载工作方式，执行工具落实检查条件。每个结论绑定实际材料版本、命令或任务标识，以及验证范围。

主流程要求错误解和慢解有明确测试目的，发现的反例能够复现并沉淀为回归数据。独立验题保留首次结果，区分盲测发现与读取出题方材料后的修正。

建议先使用一道普通非交互题走通完整流程，再根据实际需求扩展到多解构造、浮点、子任务和交互题。

## 仓库结构

```text
.agents/skills/
  acm-authoring/
    SKILL.md
    agents/openai.yaml
    references/
      design.md
      implementation-and-testing.md
      statement-and-editorial.md
      polygon.md
      polygon-api.md
  acm-independent-review/
    SKILL.md
    agents/openai.yaml
    references/review-report.md
docs/
  getting-started.md
  workflow-proposal.md
  research/polygon-testlib-research.md
```

进一步阅读：[工作流方案](docs/workflow-proposal.md)、[Polygon 与 testlib 调研](docs/research/polygon-testlib-research.md)。

## 验证状态

本版本已完成 Skill 格式、界面元数据、本地引用和可移植路径检查，并进行了独立前向试用：

- 设计任务识别出未确定的边界语义，保留问题供出题人决定。
- 独立验题任务完成 C++17 编译、自测与首次结果冻结。
- API 示例完成语法、签名、文件内容编码与模拟响应检查。

具体题目的正确性、数据覆盖和时限仍需在实际使用时验证。当前没有独立的 CLI 或 MCP server 实现。

## 许可

本项目使用 [Apache License 2.0](LICENSE)。使用 testlib、rbx 或其他第三方依赖时，遵循其各自许可。
