# Polygon 与 testlib 官方资料核对

核对日期：2026-10-26。范围是公开官方文档与维护者源码；未登录 Polygon，未调用带凭据的 API，未在平台创建、修改或发布题目。以下“支持”指公开文档记载的能力，接入时仍须验证目标账号权限、实际服务版本与返回结构。

## Polygon API 能覆盖的工作

官方入口是 `https://polygon.codeforces.com/api/{methodName}`，使用 API key、时间戳与签名鉴权；这与 Codeforces 比赛/题库查询 API 是不同接口。密钥应由本地适配器持有。[官方 API：介绍与鉴权](https://codeforces.github.io/polygon-misc/API#authorization)

| 工作 | 公开文档中的接口与边界 |
| --- | --- |
| 创建题目、配置限制 | `problem.create`、`problem.info`、`problem.updateInfo`；时限、内存等有平台允许范围。 |
| 上传题面与题解 | `problem.saveStatement` 按语言保存 `legend`、`input`、`output`、`notes`、`tutorial` 等字段；资源另有 statement resource 接口。 |
| 检查题面渲染 | `problem.renderStatements` 返回当前工作副本各语言题面及非空题解的 HTML/PDF 结果；必须逐项检查渲染结果，顶层 `OK` 可能包含某项失败。 |
| 上传源文件、设置角色 | `problem.saveFile` 上传 source/resource/aux；`problem.setValidator`、`problem.setChecker` 设置选中的源码。生成器源码上传后由测试脚本引用。 |
| 配置参考解与错误解 | `problem.saveSolution` 支持 `MA`、`OK`、`WA`、`TL` 等标签；平台还支持 `TO`、`TM` 等混合预期。 |
| 配置测试 | `problem.saveScript` 保存指定 testset 的生成脚本；`problem.saveTest` 保存手工测试，并可标记题面样例。 |
| 测 validator/checker | `problem.saveValidatorTest` 配置 `VALID/INVALID` 输入；`problem.saveCheckerTest` 配置输入、选手输出、参考输出与预期判定。读取接口可返回已存的运行结果。 |
| 运行对拍 | `problem.saveStress` 保存生成模式及对比解，`problem.runStresses` 异步启动，`problem.stresses` 轮询，`problem.addStressCountertest` 可把发现的反例加入测试。 |
| 提交工作副本、构建题包 | `problem.commitChanges` 提交；`problem.buildPackage` 发起构建，`verify=true` 要求按解的标签验题并进行 checker stress；`problem.packages` 查状态，`problem.package` 下载题包。 |
| 读取平台运行报告 | `problem.invocations` 列出当前工作副本的运行，包括包验证；`problem.invocation` 返回解与测试的运行判定、耗时和内存等已存结果。无法读取其他用户工作副本的 invocation。 |
| 检查平台警告 | `problem.cautions` 返回平台检查结果；其 package readiness 检查为快速检查，不能代替编译和实际题包验证。 |

各行依据：[题面保存](https://codeforces.github.io/polygon-misc/API#problemsavestatement)、[双格式渲染](https://codeforces.github.io/polygon-misc/API#problemrenderstatements)、[源文件](https://codeforces.github.io/polygon-misc/API#problemsavefile)、[validator 绑定](https://codeforces.github.io/polygon-misc/API#problemsetvalidator)、[checker 绑定](https://codeforces.github.io/polygon-misc/API#problemsetchecker)、[解与标签](https://codeforces.github.io/polygon-misc/API#problemsavesolution)、[脚本](https://codeforces.github.io/polygon-misc/API#problemsavescript)、[手工测试](https://codeforces.github.io/polygon-misc/API#problemsavetest)、[validator 单测](https://codeforces.github.io/polygon-misc/API#problemsavevalidatortest)、[checker 单测](https://codeforces.github.io/polygon-misc/API#problemsavecheckertest)、[stress 配置](https://codeforces.github.io/polygon-misc/API#problemsavestress)、[stress 启动](https://codeforces.github.io/polygon-misc/API#problemrunstresses)、[构建](https://codeforces.github.io/polygon-misc/API#problembuildpackage)、[运行列表](https://codeforces.github.io/polygon-misc/API#probleminvocations)、[运行详情](https://codeforces.github.io/polygon-misc/API#probleminvocation)、[警告](https://codeforces.github.io/polygon-misc/API#problemcautions)。

### 接入时必须保留的语义

- 平台操作围绕工作副本、revision、commit 与 package 进行。应记录上传内容、平台 revision、packageId 和 invocationId 的对应关系，避免把旧运行报告作为新文件的验证证据。[提交](https://codeforces.github.io/polygon-misc/API#problemcommitchanges)、[Package 对象](https://codeforces.github.io/polygon-misc/API#package)
- `problem.invocation` 的结果是调用时已经存储的运行记录。只收到 HTTP 成功或 invocation 结束标记都不足以证明整题通过：还要核对所需运行完整性、标签冲突和 package 的状态/评论；文档明确某些整体错误只写在 package comment。[运行详情](https://codeforces.github.io/polygon-misc/API#probleminvocation)
- 遇到某解违反预期标签时，平台包验证可能跳过该解剩余测试，并跳过后续 testset。`RJ` 不能解释为 AC，也不一定包含时间/内存测量。[运行详情](https://codeforces.github.io/polygon-misc/API#probleminvocation)、[InvocationRun](https://codeforces.github.io/polygon-misc/API#invocationrun)
- `InvocationRun.timeMillis` 中的 TL 耗时受时限截断；它不是慢解完整执行耗时。平台实测可以检查 AC 与 TL 的分离程度，本机耗时不能直接换算为正式平台时限，这是由运行环境差异得出的工程建议。[InvocationRun](https://codeforces.github.io/polygon-misc/API#invocationrun)
- `saveValidatorTest` 与 `saveCheckerTest` 会规范化换行但不补文件末尾换行，保存会清空旧运行结果。因此必须保留“有无末尾换行”的测试意图；保存成功不能等同于已重新运行通过。[validator 单测](https://codeforces.github.io/polygon-misc/API#problemsavevalidatortest)、[checker 单测](https://codeforces.github.io/polygon-misc/API#problemsavecheckertest)
- 当前公开方法清单中未找到可单独启动 validator/checker 单测的对应 `run...Tests` 接口。不要据此虚构接口；本地执行并以平台题包验证和现有运行结果复核，必要时使用网页完成独立运行。[官方方法清单](https://codeforces.github.io/polygon-misc/API#methods)

## testlib 代码约定

维护者 README 给出三类程序的入口：validator 用 `registerValidation(argc, argv)`，generator 用 `registerGen(argc, argv, 1)`，checker 用 `registerTestlibCmd(argc, argv)`。validator 示例通过严格读取、行尾和 EOF 检查拒绝非法输入；generator 示例通过命令行参数影响随机序列。[testlib 官方 README](https://github.com/MikeMirzayanov/testlib/blob/master/README.md)

源码进一步显示，`registerGen` 将随机算法版本设为参数值，并用命令行参数初始化 `rnd`；命令行 seed 计算跳过可执行文件名。可复现记录应包含生成器源码/构建标识、testlib 固定版本、完整参数与种子约定；“只存一个 seed 数字”不足以复现已经修改过的程序。生成器应使用 testlib 的 `rnd`，避免额外使用时间或其他随机源。[testlib 源码](https://github.com/MikeMirzayanov/testlib/blob/master/testlib.h)

已有 checker 应按输出语义选择，而不是所有题都采用字符串比较：

| 输出语义 | 维护者仓库中的示例 |
| --- | --- |
| token 序列完全相同、忽略 token 间空白 | [`wcmp.cpp`](https://github.com/MikeMirzayanov/testlib/blob/master/checkers/wcmp.cpp) |
| 有序整数序列按数值相同 | [`ncmp.cpp`](https://github.com/MikeMirzayanov/testlib/blob/master/checkers/ncmp.cpp) |
| 浮点序列允许规定误差 | [`rcmp6.cpp`](https://github.com/MikeMirzayanov/testlib/blob/master/checkers/rcmp6.cpp) 是绝对/相对误差 `1e-6` 的示例；应匹配题面误差要求。 |
| 构造、多解、证书、最优性 | 依题意编写 testlib checker，并专门测试错误输出。该项是工作流设计建议，不能用文本比较替代题意判定。 |

这里核实的是维护者仓库源码；未登录目标 Polygon 账号逐一核对其标准 checker 菜单。适配器应先读取实际配置并验证平台接受的引用方式，不能凭仓库文件名臆测 `problem.setChecker` 的标准库名称格式。

## Polygon 的 TeX 题面

官方手册明确区分 PDF 与 HTML：HTML 排版支持的 TeX 命令受限，数学公式另由 MathJax 渲染。平台亦提供 `defs.toml` 定义自有命令的方式。建议本地维护与平台字段对应的 TeX 片段，导出为 API 的 legend/input/output/notes/tutorial 等字段，并检查平台 HTML 与 PDF 两种产物；本地完整 LaTeX 文档能编译不代表 Polygon HTML 可用。[Statements TeX manual](https://polygon.codeforces.com/docs/statements-tex-manual)、[题面保存字段](https://codeforces.github.io/polygon-misc/API#problemsavestatement)、[平台渲染接口](https://codeforces.github.io/polygon-misc/API#problemrenderstatements)

## 对工作流设计的直接建议

下列属于基于上述能力的工程建议：

1. 让本地 CLI 固定负责构建、生成、validator 检查、差分对拍、反例保存、checker 单测和覆盖报告；Skill 负责各阶段的推理步骤与审查规范。
2. 先实现本地闭环，再做 Polygon API 适配；MCP 作为 CLI 与平台适配器的结构化调用入口，不承担题意正确性的证明。
3. 将平台同步拆成“读取远端状态、生成差异、上传、回读核对、提交、构建验证、收集结果”，逐步记录执行结果，支持失败后继续。公开 API 没有承诺一个能把整题全部变更原子提交的统一事务。
4. 数据验收同时要求合法性和错误解覆盖；每个预设错误解应有可解释的击败测试。随机生成通过 validator，只能证明数据满足已实现约束，不能证明数据强度或数学模型正确。
5. 保存最小反例、生成命令、源码 hash 和运行环境；冻结版本后再收集最终平台证据。对自定义 checker 增加“合法多解接受、非法证书拒绝、错误目标值拒绝、截断/多余输出处理”等针对语义的用例。
6. 平台包验证后仍进行独立验题与题面阅读。AI 生成的解法、validator、checker 若来自相同错误理解，可能彼此一致地通过测试。
