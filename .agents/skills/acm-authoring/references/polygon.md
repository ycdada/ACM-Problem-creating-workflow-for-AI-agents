# Polygon 同步与平台验题

进行 Polygon 读取、同步、渲染、stress、打包或结果分析时读取本文件。具体参数与权限以当前官方文档及连接能力为准。[官方 API](https://codeforces.github.io/polygon-misc/API)

需要直接发送 HTTP 请求、实现接入、配置认证或核对方法参数时，继续读取 [polygon-api.md](polygon-api.md)，其中包含签名、multipart 上传和完整调用顺序。已有连接足以完成任务时，沿用其接口。

## 选择已有接入

优先使用项目现有连接、客户端或经过核实的 rbx 接入。MCP 可用时检查其具体工具能力；没有接入时可以准备可审阅源文件与变更清单，明确平台操作未执行。

rbx 的原生 Windows 支持受限，采用它时核对当前安装说明。官方建议在 WSL 文件系统内维护工作副本。环境选择按用户任务和实际可用工具决定。[rbx 安装说明](https://rbx.rsalesc.dev/intro/installation/)、[Polygon 打包](https://rbx.rsalesc.dev/setters/packaging/polygon/)

平台写操作需要本次会话已有授权；仅请求本地编写或审查时，完成本地材料。凭据由连接或客户端管理，报告使用题目标识与任务标识，省略密钥和签名请求。

## 同步与读回

1. 确认目标题目、用户工作副本、当前修订与访问权限。
2. 比较本地和远端的题面、源文件、解、脚本、测试与相关配置，形成新增、修改和冲突清单。
3. 在授权范围内应用变更，保留并处理远端人工修改；操作后读回核对。提交修订是否属于交付范围由任务决定。
4. 请求失败或响应丢失时，先读取远端实际状态，再决定重试，避免重复创建或覆盖。

常见方法包括 `problem.saveStatement`、`problem.saveFile`、`problem.saveSolution`、`problem.saveScript`、`problem.saveTest`，以及 checker、validator 的绑定配置。调用时核对实际语言标识、文件类型、testset 与 solution tag；方法存在不代表当前客户端已封装它。

## 渲染与验题的完成判定

| 工作 | 可核实接口 | 判读要点 |
| --- | --- | --- |
| 题面与题解 HTML / PDF | `problem.renderStatements` | 检查每项 RenderResult，顶层 OK 仅代表请求层结果 |
| validator / checker 单测 | 相应测试读写接口 | 区分已保存用例、旧结果与本轮实际执行结果 |
| 平台对拍 | `problem.saveStress`、`problem.runStresses`、`problem.stresses` | 记录运行状态、解对、模式、反例与预算 |
| 构建与验题 | `problem.buildPackage`，`verify=true` | 区分已启动、任务结束与成功；核对对应包 |
| 运行结果 | `problem.invocations`、`problem.invocation` | 核对题目工作副本、解、测试、判定、耗时、内存与完整性 |

平台结果需要联合检查 package 状态与 comment、逐解逐测试结果、标签匹配和相关 cautions。任务 FINISHED、顶层 OK、结果中没有某个错误字段，均不足以单独证明完整验题通过。

列出预期运行的解与测试范围，再与实际结果核对；缺失、过期、工作副本不匹配或无法解析的结果标为未知。平台耗时采用目标语言与编译器，和本地测量分别保留。

长任务记录任务标识与最后状态，按客户端能力等待或查询；没有新变化时适度退避。状态不明的写操作先读回再处理；持续失败时保存可恢复信息并报告具体缺口。

完成条件：授权范围内的变更已读回，渲染与验题各项结果完整且对应当前版本，警告有处理结论；剩余工作明确区分“本地完成”“平台未执行”和“平台失败”。
