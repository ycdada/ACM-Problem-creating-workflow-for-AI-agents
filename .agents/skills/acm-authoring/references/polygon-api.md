# Polygon API 调用说明

需要直接调用或实现 Polygon 接入时读取。本文件按 2026-10-26 的[官方 API 文档](https://codeforces.github.io/polygon-misc/API)核对；使用时重新确认相关方法与当前客户端能力。示例不会自动执行，远端写操作仍以用户任务的授权范围为准。

## 认证与参数编码

请求地址为 `https://polygon.codeforces.com/api/{methodName}`。在 Polygon 设置中取得 API key 和 secret，通过环境或已有凭据连接读取。题目方法通常还需要 `problemId`，设置了 PIN 的题目需要 `pin`。

每次请求添加 `apiKey`、秒级 UNIX `time` 和 `apiSig`。时间与服务端相差超过五分钟会被拒绝。签名规则为：

```text
pairs = 全部请求参数，但排除 apiSig
按参数名、参数值依次做字典序排序
canonical = rand + "/" + methodName + "?" + join(key + "=" + value, "&") + "#" + secret
apiSig = rand + sha512(canonical 的 UTF-8 字节).hexdigest()
```

`rand` 是每次选择的六个字符。布尔值统一传 `true` / `false`，省略不需要修改的可选参数。使用的参数值与实际传输值保持一致。

文件上传尤其需要注意：`file` 参数是内容，不是本地路径。rbx 客户端把参数转换为 UTF-8 字节，以原始内容构造签名，再通过 multipart 发送；上传文件内容同样进入签名。不能先 URL 编码再计算这个签名，也不能漏掉上传内容。[认证规则](https://codeforces.github.io/polygon-misc/API#authorization)、[rbx 客户端实现](https://github.com/rsalesc/rbx/blob/main/rbx/box/packaging/polygon/polygon_api.py)

## Python 请求示例

下面是基于上述规则的简化示例，依赖已有的 `requests`。优先复用项目客户端；本例适合核对或实现接入，尚未使用真实凭据连接 Polygon 验证。`POLYGON_API_KEY`、`POLYGON_API_SECRET` 仅为环境变量名称约定。

```python
import hashlib
import os
import secrets
import string
import time

import requests


def _parameter_bytes(value):
    if isinstance(value, bytes):
        return value
    if isinstance(value, bool):
        return b"true" if value else b"false"
    return str(value).encode("utf-8")


def api(method, *, response="json", read_timeout=60, **parameters):
    """仅发送一次请求；不自动重试写操作，也不打印认证参数。"""
    if response not in {"json", "text", "bytes"}:
        raise ValueError("未知响应格式")
    pairs = [
        (key.encode("utf-8"), _parameter_bytes(value))
        for key, value in parameters.items()
        if value is not None and key not in {"apiKey", "time", "apiSig"}
    ]
    pairs.extend([
        (b"apiKey", os.environ["POLYGON_API_KEY"].encode("utf-8")),
        (b"time", str(int(time.time())).encode("ascii")),
    ])
    nonce = "".join(
        secrets.choice(string.ascii_lowercase + string.digits)
        for _ in range(6)
    ).encode("ascii")
    canonical = (
        nonce + b"/" + method.encode("utf-8") + b"?"
        + b"&".join(key + b"=" + value for key, value in sorted(pairs))
        + b"#" + os.environ["POLYGON_API_SECRET"].encode("utf-8")
    )
    pairs.append((b"apiSig", nonce + hashlib.sha512(canonical).hexdigest().encode("ascii")))
    result = requests.post(
        "https://polygon.codeforces.com/api/" + method,
        files=[(key.decode("utf-8"), value) for key, value in pairs],
        timeout=(10, read_timeout),
    )
    try:
        payload = result.json()
    except ValueError:
        payload = None
    api_failed = isinstance(payload, dict) and payload.get("status") == "FAILED"
    if not 200 <= result.status_code < 300 or api_failed:
        comment = payload.get("comment", "") if isinstance(payload, dict) else ""
        raise RuntimeError(f"Polygon HTTP {result.status_code}: {comment}")
    if response == "text":
        return result.text
    if response == "bytes":
        return result.content
    if not isinstance(payload, dict) or payload.get("status") != "OK":
        raise RuntimeError("响应不符合预期的 Polygon JSON 格式")
    return payload.get("result")
```

这段示例处理单值参数。若接口需要复杂列表，按该方法的实际定义编码，不能把 Python 列表直接转成字符串。大文件下载使用项目下载器处理流式保存；示例的 `bytes` 模式会把完整响应读入内存。

## 一、先读取当前状态

在实际调用前准备已确认的题目标识和可选 PIN，例如：

```python
problem = {"problemId": int(os.environ["POLYGON_PROBLEM_ID"])}
if os.environ.get("POLYGON_PROBLEM_PIN"):
    problem["pin"] = os.environ["POLYGON_PROBLEM_PIN"]

info = api("problem.info", **problem)
files = api("problem.files", **problem)
solutions = api("problem.solutions", **problem)
statements = api("problem.statements", **problem)
```

使用返回内容核对工作副本、源文件与解的名称、语言和编译类型。后面的 `source_type`、`checker_name`、`lang`、`testset` 均指实际确认的配置，不能照抄成固定标识。

有些方法不返回 JSON。例如读取生成脚本使用：

```python
script = api("problem.script", response="text", testset=testset, **problem)
```

这也是请求函数区分 JSON、文本和二进制响应的原因。[方法及响应格式](https://codeforces.github.io/polygon-misc/API#methods-for-problems)

## 二、配置程序、解和限制

以下是按任务需要选择的调用片段；先形成远端差异清单，再执行授权范围内的变更。

```python
from pathlib import Path

api("problem.saveFile", type="source", name="validator.cpp",
    file=Path("validator/validator.cpp").read_bytes(),
    sourceType=source_type, **problem)
api("problem.setValidator", validator="validator.cpp", **problem)

api("problem.saveFile", type="source", name="gen.cpp",
    file=Path("generators/gen.cpp").read_bytes(),
    sourceType=source_type, **problem)

api("problem.saveSolution", name="main.cpp", tag="MA",
    file=Path("solutions/main.cpp").read_bytes(),
    sourceType=source_type, **problem)
api("problem.saveSolution", name="wrong_greedy.cpp", tag="WA",
    file=Path("solutions/wrong_greedy.cpp").read_bytes(),
    sourceType=source_type, **problem)
api("problem.saveSolution", name="slow.cpp", tag="TL",
    file=Path("solutions/slow.cpp").read_bytes(),
    sourceType=source_type, **problem)

api("problem.setChecker", checker=checker_name, **problem)
api("problem.updateInfo", timeLimit=2000, memoryLimit=256, **problem)
```

- `saveFile` 的 `type` 为 `source`、`resource` 或 `aux`；testlib 头文件及其他资源按实际包的引用和编译规则配置，上传后通过包构建确认可编译。
- 自定义 checker 先作为 source 上传，再绑定；标准 checker 使用平台当前有效的名称。
- 解的 `MA` 是主标程，`OK` 是正确解，`WA` 和 `TL` 分别用于预期错误答案与超时；还有其他 tag，按方法文档及验题目标选择。小规模暴力解的适用范围需要另行配置。
- `sourceType` 是平台编译类型标识，按现有配置或当前平台支持项确定。
- `timeLimit` 单位为毫秒，`memoryLimit` 为 MB；示例为 2 秒、256 MB，实际值由题目的测量结果确定。

依据：[文件上传](https://codeforces.github.io/polygon-misc/API#problemsavefile)、[解与 tag](https://codeforces.github.io/polygon-misc/API#problemsavesolution)、[限制设置](https://codeforces.github.io/polygon-misc/API#problemupdateinfo)。

## 三、保存 TeX 题面、脚本和样例

题面用 `lang` 选择语言，分别传 legend、input、output、notes、tutorial 等字段；它们是 TeX 内容，不是整份源文件的本地路径。未传的字段保留现有值，传空内容属于明确的修改。

```python
api("problem.saveStatement", lang=lang, encoding="UTF-8",
    legend=Path("statement/legend.tex").read_text(encoding="utf-8"),
    input=Path("statement/input.tex").read_text(encoding="utf-8"),
    output=Path("statement/output.tex").read_text(encoding="utf-8"),
    tutorial=Path("editorial/tutorial.tex").read_text(encoding="utf-8"),
    **problem)

api("problem.saveScript", testset=testset,
    source=Path("tests/generation.script").read_text(encoding="utf-8"), **problem)

api("problem.saveTest", testset=testset, testIndex=sample_index,
    testInput=Path("tests/sample.in").read_text(encoding="utf-8"),
    testUseInStatements=True,
    testOutputForStatements=Path("tests/sample.out").read_text(encoding="utf-8"),
    **problem)
```

脚本语法使用 Polygon 当前支持的生成器调用约定，先核对生成器名称、参数和测试编号。`testOutputForStatements` 是样例展示内容，正式标准答案依赖题包构建流程；两者分别验证。[题面](https://codeforces.github.io/polygon-misc/API#problemsavestatement)、[脚本](https://codeforces.github.io/polygon-misc/API#problemsavescript)、[测试](https://codeforces.github.io/polygon-misc/API#problemsavetest)

validator 单测用 `problem.saveValidatorTest`，核心字段为 `testIndex`、`testInput`、`testVerdict=VALID/INVALID`。checker 单测用 `problem.saveCheckerTest`，需要输入、选手输出、标准答案和预期 verdict。保存后旧运行结果会被清除；保存用例与实际运行成功分开判断。[validator 单测](https://codeforces.github.io/polygon-misc/API#problemsavevalidatortest)、[checker 单测](https://codeforces.github.io/polygon-misc/API#problemsavecheckertest)

## 四、渲染、构建和取回验题结果

```python
rendered = api("problem.renderStatements", includeContent=False,
               read_timeout=300, **problem)
before_packages = api("problem.packages", **problem)
api("problem.buildPackage", full=True, verify=True, **problem)
packages = api("problem.packages", **problem)
```

渲染可能耗时较长，应在可汇报进度的任务执行环境中运行。`includeContent=False` 先取状态与哈希；需要文件时改为 `True`，将返回的 Base64 内容解码并保存，避免把大段内容写入报告。

逐项检查 `rendered` 中 statements 与 tutorials 的 HTML / PDF status。单项失败仍可能得到顶层 `OK`。[渲染接口](https://codeforces.github.io/polygon-misc/API#problemrenderstatements)

`buildPackage` 只负责启动构建。先记录构建前已有包，再通过返回内容和 `packages` 读回定位本次 package，核对 revision 与创建时间；并发构建时不能直接选列表第一项。包状态为 `PENDING`、`RUNNING`、`READY` 或 `FAILED`。[构建接口](https://codeforces.github.io/polygon-misc/API#problembuildpackage)

得到确认的 `package_id` 后：

```python
invocations = api("problem.invocations", packageId=package_id, **problem)
for invocation in invocations:
    details = api("problem.invocation", invocationId=invocation["id"], **problem)
    # 将本次任务对应的结果保存到报告，再按完整性规则检查。
```

运行列表只属于当前调用用户的工作副本；它不是所有协作者工作副本的全局结果。完成判断同时要求：

1. 本次包与修订匹配，状态及 comment 没有未处理失败。
2. invocation 已完成，预期的解与测试组合都有结果。
3. 标签违例、跳过的运行及整体 tag 失败都有处理结论。
4. 题目 cautions 与相关测试、渲染问题均已检查。

`FINISHED` 或 runs 中没有 `violateTag` 不能单独证明通过，某些整体错误只记录在 package comment。[运行列表](https://codeforces.github.io/polygon-misc/API#probleminvocations)、[运行详情与完整性语义](https://codeforces.github.io/polygon-misc/API#probleminvocation)

如果本次任务包括提交修订，调用 `problem.commitChanges` 并核对返回的 `committed` 与 `conflictOccurred`；顶层 OK 也可能只是没有需要提交的改动。`minorChanges=true` 表示不发邮件通知，是否采用由任务要求决定。[提交接口](https://codeforces.github.io/polygon-misc/API#problemcommitchanges)

## 失败与重试

认证失败时核对时间、字节编码、参数完整性和凭据来源；权限或 PIN 失败时修正配置。API 限流或长任务未完成时适度退避。

写请求超时、HTTP 500 或响应丢失后，先读回对象、包和任务状态，确认是否已生效。上面的请求函数故意不自动重试；在接入层根据具体方法实现可恢复处理。
