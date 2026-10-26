# 安装、配置与首次运行

本指南面向 Codex。先按需要的阶段准备工具：设计工作可以先开始，编译、自测、TeX 渲染和平台操作各自需要相应环境。Skill 是流程指令，实际操作使用当前项目可用的工具。

## 准备 Codex

1. 准备 Git 和 Codex 桌面版或 CLI。Codex 的安装方式以[官方 CLI 安装指南](https://learn.chatgpt.com/docs/cli)为准；已经能在桌面版打开本地项目时，可直接使用桌面版。
2. CLI 用户在终端检查并登录：

```bash
git --version
codex --version
codex login
codex login status
```

已有登录会话时可以直接检查状态。`codex login` 使用官方登录流程，具体认证方式见[官方认证说明](https://learn.chatgpt.com/docs/auth)。

3. 按 [README 快速开始](../README.md#快速开始)克隆本仓库，在项目目录中打开 Codex。CLI 运行 `codex`；桌面版选择该目录作为项目。

Codex 从项目的 `.agents/skills/` 发现仓库级 Skill。可以在聊天中输入 `$acm-authoring`、`$acm-reconstruct` 或 `$acm-independent-review`；CLI / IDE 也可使用 `/skills` 查看。安装后没有出现时，确认当前项目目录，再重启 Codex。[官方 Skill 发现与调用说明](https://learn.chatgpt.com/docs/build-skills)

## 安装到自己的项目

下面的复制命令用于首次安装。先在克隆的本仓库根目录打开终端，将目标目录改为自己的题目项目；已有同名 Skill 时先比较本地修改，再更新。

### Windows PowerShell

```powershell
$skillSource = (Resolve-Path '.agents/skills').Path
$contestRoot = 'D:\contests\my-contest'
New-Item -ItemType Directory -Force -Path "$contestRoot\.agents\skills" | Out-Null
Copy-Item -LiteralPath "$skillSource\acm-authoring", "$skillSource\acm-reconstruct", "$skillSource\acm-independent-review" -Destination "$contestRoot\.agents\skills" -Recurse
Set-Location -LiteralPath $contestRoot
```

### Linux / macOS：Bash

```bash
skill_source="$PWD/.agents/skills"
contest_root="$HOME/contests/my-contest"
mkdir -p "$contest_root/.agents/skills"
cp -R "$skill_source/acm-authoring" "$skill_source/acm-reconstruct" "$skill_source/acm-independent-review" "$contest_root/.agents/skills/"
cd "$contest_root"
```

在目标目录打开 Codex，或在该终端运行 `codex`。三个 Skill 应为同级目录，且每个目录直接包含 `SKILL.md`，避免多复制一层外部文件夹。

如果希望跨项目使用，可采用 README 中的 Skill 安装器请求。用户级安装和项目级安装选择一种即可；重复安装同名 Skill 可能产生多个候选项。

## 首次运行与验收

在 Codex 聊天中发送 README 的首次设计请求。第一次只做设计时，检查：

- 请求中的题目目录已生成设计材料。
- 数学定义、算法、证明、约束和未确定事项已区分。
- 报告列出本次范围，未运行的编译和平台检查如实说明。

继续开发同一题时，明确题目目录与下一阶段，例如：

```text
使用 $acm-authoring，继续 problems/example/。
读取已有设计和报告，使用当前项目的编译工具与固定 testlib，
补齐 validator、标程、暴力和数据计划，先完成本地小规模自测。
```

已有原题时可以直接请求还原与补数据：

```text
使用 $acm-reconstruct，根据我提供的原题题面，在 problems/original-example/ 中建立题包。
保留原题明确的语义和约束，用 testlib 编写 validator 与 generator，
制作新的测试数据并核对答案，报告材料来源和未确定的内容。
```

独立验题使用新的干净上下文，只给题面、样例、限制和公开附件。示例：

```text
使用 $acm-independent-review，仅根据 problems/example/statement/ 的选手可见材料，
编写独立解并自测，保存首次结果冻结。本次不读取出题方的证明、标程或生成计划。
```

报告中的“已验证”应附实际范围与依据。平台未连接或缺少正式数据时，对应项目仍为未执行。

## 配置 testlib

项目已有固定版本时直接沿用。新项目可将用户选定的 `testlib.h` 放在以下位置，并保留许可：

```text
third_party/testlib/testlib.h
problems/example/
```

例如在项目根目录使用 PowerShell 完成首次复制：

```powershell
$testlibSource = Read-Host 'testlib.h 的完整路径'
New-Item -ItemType Directory -Force -Path 'third_party/testlib' | Out-Null
Copy-Item -LiteralPath $testlibSource -Destination 'third_party/testlib/testlib.h'
Get-FileHash -LiteralPath 'third_party/testlib/testlib.h' -Algorithm SHA256
```

把项目相对路径、实际版本和哈希告知主流程 Skill。编译时由项目配置加入正确的头文件搜索路径，例如 GCC / Clang 的 `-I`；路径以编译命令的工作目录为准。

validator、generator 和自定义 checker 使用同一固定版本。`testlib.h` 来自用户或[维护者仓库](https://github.com/MikeMirzayanov/testlib)，本项目不自动下载或升级它。

进行编译阶段前，确认所选工具在当前环境可用，例如使用 GCC 时运行 `g++ --version`。MSVC、Clang 或已有 runner 按其实际配置使用。

若选择 rbx，参考其[安装文档](https://rbx.rsalesc.dev/intro/installation/)。Windows 使用 rbx 时按官方说明准备 WSL，并在 WSL 文件系统内克隆项目；原生 Windows 工具链与 WSL 是不同执行环境，分别检查路径和依赖。

## 配置 Polygon

只有需要同步、渲染或平台验题时才配置这些项目：

| 变量 | 用途 |
| --- | --- |
| `POLYGON_API_KEY` | Polygon 设置中生成的 API key |
| `POLYGON_API_SECRET` | 对应 secret，用于请求签名 |
| `POLYGON_PROBLEM_ID` | 已确认的目标题目 ID |
| `POLYGON_PROBLEM_PIN` | 仅有 PIN 的题目需要 |

这几个名称与仓库内的 [Python API 示例](../.agents/skills/acm-authoring/references/polygon-api.md)一致。已有客户端可能使用自己的配置方式，以其实际接口为准。

### PowerShell：设置当前会话

```powershell
$env:POLYGON_API_KEY = Read-Host 'Polygon API key'
$polygonSecret = Read-Host 'Polygon API secret' -AsSecureString
$env:POLYGON_API_SECRET = [System.Net.NetworkCredential]::new('', $polygonSecret).Password
$env:POLYGON_PROBLEM_ID = Read-Host 'Polygon problem ID'
```

需要 PIN 时用同样方式设置 `$env:POLYGON_PROBLEM_PIN`。

### Bash：设置当前会话

```bash
read -r -p 'Polygon API key: ' POLYGON_API_KEY
read -r -s -p 'Polygon API secret: ' POLYGON_API_SECRET
printf '\n'
read -r -p 'Polygon problem ID: ' POLYGON_PROBLEM_ID
export POLYGON_API_KEY POLYGON_API_SECRET POLYGON_PROBLEM_ID
```

需要 PIN 时设置并导出 `POLYGON_PROBLEM_PIN`。配置适用于当前 shell 及其子进程；CLI 应从这个 shell 启动。已经运行的桌面进程不会因此更新环境，桌面版应使用其现有连接或按启动方式提供配置。

当前示例不自动读取 `.env`。若项目 runner 会加载配置文件，沿用其机制；本仓库的 `.gitignore` 忽略常见本地环境文件。首次连接先读取题目信息，核对工作副本与权限，再按任务授权同步。

### 可选：准备 Python API 示例依赖

仅当需要直接运行参考文档中的 Python 请求代码时，在项目根目录创建虚拟环境。

PowerShell：

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install requests
.\.venv\Scripts\python.exe -c "import requests; print('requests ready')"
```

Bash：

```bash
python3 -m venv .venv
.venv/bin/python -m pip install requests
.venv/bin/python -c "import requests; print('requests ready')"
```

后续使用同一个虚拟环境中的解释器运行代码。参考文档提供请求函数和调用片段，仓库当前没有 `polygon sync` 或一键出题 CLI。

## 更新

- **直接克隆 `main` 使用**：保存本地改动后，在源仓库运行 `git pull --ff-only`，再更新目标项目中使用的三个 Skill 目录。
- **复制安装**：更新源仓库后比较同名目录的差异，保留自己修改的内容；三个 Skill 一起更新。
- **使用 Skill 安装器**：指定希望安装的版本；已有同名目录时，先确认更新方式和本地修改。
- **固定标签安装**：标签工作区通常处于 detached HEAD。通过 `git fetch --tags` 获取后续版本，再选择目标标签；需要在其中开发题目时先创建工作分支。

首版标签 `v0.1.0` 包含主流程和独立验题两个 Skill；新增的原题还原 Skill 需使用 `main` 或包含它的后续版本。发布版本见 [GitHub Releases](https://github.com/ycdada/ACM-Problem-creating-workflow-for-AI-agents/releases)。

## 常见问题

| 现象 | 检查方式 |
| --- | --- |
| 找不到 Skill | 确认打开了目标项目；检查 `.agents/skills/<名称>/SKILL.md`，三个目录同级；仍未出现时重启 Codex |
| `$acm-authoring` 被 shell 当成变量 | 将整段调用请求发给 Codex 聊天，而不是在 PowerShell / Bash 直接执行 |
| Skill 列表出现重复项 | 检查是否同时进行了用户级和项目级安装 |
| 编译找不到 `testlib.h` | 核对固定文件位置、`-I` 搜索目录和命令的实际工作目录 |
| 缺少编译器、TeX 或 runner | 可以先继续设计和文本工作；相应执行检查保持未执行，按所选工具配置环境 |
| API 示例缺少环境变量 | 在执行代码的同一环境中配置；检查桌面进程、CLI、WSL 与宿主 shell 是否不同 |
| 设置了 `.env` 仍无效 | 当前示例不会自动加载该文件，使用进程环境或项目已有的加载机制 |
| 签名、权限或 PIN 失败 | 核对系统时间、完整参数、凭据来源、题目 ID、访问权限和 PIN，见 API 参考 |
| 题目还未在 Polygon 验题 | 本地自测与平台验题是不同阶段，按报告检查实际执行范围 |
