# 安全

## 报告 Node.js 中的 Bug

通过 [HackerOne](https://hackerone.com/nodejs) 报告 Node.js 中的安全漏洞。

通常，您的报告会在 5 天内得到确认，您会在 10 天内收到更详细的回复，说明处理您提交内容的后续步骤。当我们的分类志愿者休假时，尤其是在年底，这些时间线可能会延长。

在初次回复您的报告后，安全团队将努力让您了解修复和全面公告的进展，并可能就报告的问题询问更多信息或指导。

如果您在 6 个工作日内未收到报告的确认，或者找不到项目的私密安全联系人，您可以向 OpenJS 基金会 CNA 升级，邮箱为 `security@lists.openjsf.org`。

如果项目确认了您的报告但在 14 天内未提供任何进一步回复或互动，也适合升级。

### Node.js 漏洞奖励计划

Node.js 项目不再设有漏洞奖励计划。

## 报告第三方模块中的 Bug

第三方模块中的安全漏洞应报告给其各自的维护者。

## 披露政策

以下是 Node.js 的安全披露政策：

* 收到安全报告并指定主要处理人。此人将协调修复和发布流程。针对所有受支持的 Node.js 版本验证问题。一旦确认，确定所有受影响版本的列表。审计代码以查找任何潜在的类似问题。为所有受支持的版本准备修复。
  这些修复不会提交到公共仓库，而是暂存在本地，等待公告发布。

* 为此漏洞选择建议的禁运日期，并为该漏洞申请 CVE（通用漏洞披露 (CVE®)）。

* 在禁运日期，将公告副本发送到 Node.js 安全邮件列表。更改被推送到公共仓库，新构建部署到 nodejs.org。在邮件列表收到通知后的 6 小时内，将在 Node.js 博客上发布公告的副本。

* 通常，禁运日期将设定为 CVE 发布后的 72 小时。但是，这可能会根据漏洞的严重性或应用修复的难度而有所不同。

* 此过程可能需要一些时间，尤其是当我们需要与其他项目的维护者协调时。我们将尽可能快地处理漏洞；但是，我们必须遵循上述发布流程，以确保我们一致地处理披露。

## 行为准则和漏洞报告指南

报告安全漏洞时，报告者必须遵守以下指南：

1. **遵守行为准则**：所有安全报告必须符合我们的[行为准则](CODE_OF_CONDUCT.md)。违反我们行为准则的报告将不被考虑，并可能导致被禁止参与未来的活动。

2. **无有害行为**：安全研究和漏洞报告不得：
   * 对正在运行的系统或生产环境造成损害。
   * 干扰 Node.js 开发或基础设施。
   * 影响其他用户的应用程序或系统。
   * 包含可能伤害用户的实际利用代码。
   * 涉及社会工程学或网络钓鱼尝试。

3. **负责任的测试**：测试潜在漏洞时：
   * 使用隔离的、可控的环境。
   * 未经事先授权，请勿在生产系统上进行测试。请联系 Node.js 技术指导委员会（<tsc@iojs.org>）获取许可，或打开 HackerOne 报告。
   * 请勿试图访问或修改其他用户的数据。
   * 如果意外获得未经授权的访问，立即停止测试。

4. **报告质量**
   * 提供清晰、详细的步骤来重现漏洞。
   * 包含用 JavaScript 编写的可重现代码。
   * 仅包含证明问题所需的最少概念验证。
   * 移除任何可能造成伤害的恶意负载或组件。

未能遵循这些指南可能导致：

* 漏洞报告被拒绝。
* 放弃任何潜在的漏洞奖励。
* 暂时或永久禁止参与漏洞奖励计划。
* 在恶意意图的情况下采取法律行动。

## Node.js 威胁模型

在 Node.js 威胁模型中，存在受信任的元素，例如底层操作系统。需要破坏这些受信任元素的漏洞不在 Node.js 威胁模型的范围内。

要使漏洞有资格获得漏洞奖励，它必须是 Node.js 威胁模型上下文中的漏洞。换句话说，它不能假设受信任的元素（如操作系统）已被破坏。

### 实验性平台

Node.js 为操作系统和硬件组合维护基于层级的支持系统（一级、二级和实验性）。对于在[支持的平台](BUILDING.md#supported-platforms)文档中归类为“实验性”的平台：

* 仅影响实验性平台的安全漏洞将**不**被接受为有效的安全问题。
* 实验性平台上的任何问题都将被视为普通 Bug。
* 不会为仅影响实验性平台的问题发布 CVE。
* 实验性平台特定问题不提供漏洞奖励。

此政策承认实验性平台可能无法编译，可能无法通过测试套件，并且没有与一级和二级平台相同级别的测试和支持基础设施。

### 位于编译时标志和 V8 标志之后的实验性特性

Node.js 包含某些仅在编译时使用特定标志时才可用的实验性特性。这些特性旨在用于开发、调试或测试目的，不会在正式发布中启用。

Node.js 也可能暴露由 V8 命令行标志（例如 `--js-staging`、`--max_old_space_size`）控制的 V8 特性。这些标志启用或修改 V8 级别的 JavaScript 引擎行为，这些行为不是 Node.js 实现的 ECMAScript 规范的一部分，也不是 Node.js 文档化 API 表面的一部分。

* 仅影响位于编译时标志或 V8 标志之后的特性的安全漏洞将**不**被接受为有效的安全问题。
* 这些特性的任何问题都将被视为普通 Bug。
* 不会为仅影响编译时标志或 V8 标志特性的问题发布 CVE。
* 编译时标志或 V8 标志特性问题不提供漏洞奖励。

此政策承认位于编译时标志之后的实验性特性尚未准备好供公众使用，可能有不完整的实现、缺少安全加固或其他限制，使其不适合生产使用。同样，V8 标志暴露了内部的 V8 引擎选项，这些选项不是 Node.js 文档化 API 表面的一部分，在生产构建中默认不启用，可能有不完整的实现或缺少安全加固。

### 什么构成漏洞

能够通过控制 Node.js 不信任的元素导致以下情况被视为漏洞：

* 通过正确使用 Node.js API 保护的数据的披露或完整性/机密性丧失。
* 运行时的不可用性，包括其性能的无限制下降。

如果 Node.js 在默认情况下（未经用户特定请求）加载配置文件或运行代码，且未记录在案，则视为漏洞。
与此情况相关的漏洞可能通过文档更新来修复。

#### 拒绝服务（DoS）漏洞

要被视为 DoS 漏洞，PoC 必须满足以下标准：

* API 被正确使用。
* API 没有针对在生产环境中使用的警告。
* API 是公开且文档化的。如果 API 来自 JavaScript，其行为必须在 [ECMAScript 规范](https://tc39.es/ecma262/) 中有明确定义。
* API 具有稳定（2.0）状态。
* 行为足够严重，足以快速或在 Node.js 应用程序开发者无法控制的上下文中（例如 HTTP 解析）导致拒绝服务。
* 行为可直接被不受信任的来源利用，无需应用程序犯错。
* 无法通过标准运营实践（如进程回收）合理缓解。
* 行为在正常使用模式下具有确定性，而非边缘情况。
* 行为发生的速率会在典型工作负载下，在实际时间范围内导致实际资源耗尽。
* 攻击展示了[不对称资源消耗](https://cwe.mitre.org/data/definitions/405.html)，即攻击者消耗的资源远少于服务器处理攻击所需的资源。攻击者端需要相当资源（可通过常见实践如限流缓解）的攻击可能不符合条件。

**Node.js 不信任**：

* 从入站网络连接远程端接收的数据，这些连接通过使用 Node.js API 接受，并在传递给应用程序之前由 Node.js 转换/验证。这包括：
  * HTTP API（所有形式）服务器 API。
* 从出站网络连接远程端接收的数据，这些连接通过使用 Node.js API 创建，并在传递给应用程序之前由 Node.js 转换/验证，**但有效载荷长度除外**。Node.js 信任应用程序会建立/请求连接，以避免导致拒绝服务的有效载荷大小。
  * HTTP API（所有形式）客户端 API。
  * DNS API。
* 通过使用 Node.js API 保护的数据的消费者（例如，有权访问通过 Node.js 加密 API 加密的数据的人）。
* 通过使用 Node.js API 打开读取或写入的文件内容或其他 I/O（例如：stdin、stdout、stderr）。

换句话说，如果通过 Node.js 在应用程序之间传递的数据能触发 API 文档中所述之外的操行，则可能存在安全漏洞。不良操作的示例包括污染全局对象、导致不可恢复的崩溃，或任何其他可能导致机密性、完整性或可用性丧失的意外副作用。

例如，如果受信任的输入（如安全的应用程序代码）是正确的，那么不受信任的输入不得导致任意 JavaScript 代码执行。

**Node.js 信任其他一切**。示例包括：

* 运行它的开发者和基础设施。
* Node.js 运行所在的操作系统及其配置，以及操作系统控制下的任何内容。
* 它被要求运行的代码，包括 JavaScript、WASM 和原生代码，即使该代码是动态加载的，例如，从 npm 注册表安装的所有依赖项。
  运行的代码继承执行用户的所有权限。
* 由它被要求运行的代码提供给它的输入，因为执行所需的输入验证是应用程序的责任，例如 `JSON.parse()` 的输入。
* 用于检查器（调试器协议）的任何连接，无论是由命令行选项还是 Node.js API 打开的，也无论远程端是在本地机器还是远程。
* 模块加载时的文件系统。
  请参阅 <https://nodejs.org/api/modules.html#all-together>。
* `node:wasi` 模块目前不提供某些 WASI 运行时提供的全面文件系统安全属性。
* 执行路径是受信任的。此外，Node.js 路径操作函数（如 `path.join()` 和 `path.normalize()`）信任其输入。关于这些函数依赖未净化输入的漏洞报告不被视为需要 CVE 的漏洞，因为用户有责任根据其安全要求净化路径输入。

来自 Node.js 内部函数的数据操作的任何意外行为，如果可通过不受信任的资源利用，则可能被视为漏洞。

除了基于上述内容处理漏洞外，项目还致力于避免 API 和内部实现使应用程序代码“容易”错误使用 API，从而导致应用程序代码本身出现漏洞。虽然我们不认为这些是 Node.js 本身的漏洞，也不一定会发布 CVE，但我们希望它们首先被私下报告给 Node.js。我们通常选择根据这些报告改进我们的 API，并根据对社区构成的风险程度，在常规或安全发布中发布修复。

### 漏洞示例

#### 证书验证不当 (CWE-295)

* Node.js 提供 API 来验证用于连接到 TLS/SSL 端点的证书中的主题备用名称（SAN）处理。如果可构造导致 Node.js API 验证错误的证书，则视为漏洞。

#### HTTP 请求解释不一致 (CWE-444)

* Node.js 提供 API 来接受 HTTP 连接。这些 API 解析连接接收的标头并将其传递给应用程序。解析这些标头中的错误可能导致请求走私，视为漏洞。

#### 缺少加密步骤 (CWE-325)

* Node.js 提供 API 来加密数据。允许攻击者在无需解密密钥的情况下获取原始数据的错误被视为漏洞。

#### 系统或配置设置的外部控制 (CWE-15)

* 如果 Node.js 自动加载一个未记录在案的配置文件，并且修改该配置可能影响使用 Node.js API 保护的数据的机密性，则视为漏洞。

### 非漏洞示例

#### 深度防御问题

* 其修复仅能在另一个安全边界已被攻破后提高弹性，或降低 Node.js 威胁模型之外问题的影响的漏洞，被视为深度防御问题。
* 深度防御问题从不被视为 Node.js 安全漏洞，不会获得 CVE，并作为常规 Bug 或加固改进处理。

#### 恶意协议对等端

* Node.js 将来自远程网络对等端的数据视为不受信任的，解析器或协议实现中的漏洞可能是安全漏洞。
* Node.js 将来自 HTTP/1.1 持久连接的数据视为受信任的，意味着在同一个 HTTP/1.1 连接重用生命周期内，Node.js 客户端消费未经请求或顺序错乱的响应通常不被视为 Node.js 漏洞。

#### 恶意第三方模块 (CWE-1357)

* 代码被 Node.js 信任。因此，任何需要恶意第三方模块的场景都不能导致 Node.js 中的漏洞。

#### 原型污染攻击 (CWE-1321)

* Node.js 信任应用程序代码提供给它的输入。由应用程序适当地进行净化。因此，任何需要控制用户输入的场景都不被视为漏洞。

#### 不受控制的搜索路径元素 (CWE-427)

* Node.js 信任其可访问的环境中的文件系统。因此，如果它从其可访问的任何路径访问/加载文件，则不是漏洞。

#### 系统或配置设置的外部控制 (CWE-15)

* 如果 Node.js 自动加载一个记录在案的配置文件，则任何需要修改该配置文件的场景都不被视为漏洞。

#### 出站连接上的不受控制的资源消耗 (CWE-400)

* 如果 Node.js 被要求连接到远程站点并返回一个工件，且该工件的大小足够大以影响性能或导致运行时资源耗尽，则不被视为漏洞。

#### 影响 Corepack 下载的软件的漏洞

* Corepack 默认下载用户请求的最新版本软件，或用户请求的特定版本。因此，Node.js 发布版本不会受此类漏洞影响。用户有责任保持他们通过 Corepack 使用的软件为最新。

#### 向不受信任的用户暴露应用程序级 API (CWE-653)

* Node.js 信任使用其 API 的应用程序代码。当应用程序代码以不安全的方式将 Node.js 功能暴露给不受信任的用户时，任何由此产生的崩溃、数据损坏或其他问题不被视为 Node.js 本身的漏洞。应用程序有责任：
  * 在将不受信任的输入传递给 Node.js API 之前，对其进行验证和净化。
  * 设计适当的访问控制和安全边界。
  * 避免直接将低级或危险的 API 暴露给不受信任的用户。

* 以下场景示例**不**是 Node.js 漏洞：
  * 允许不受信任的用户通过 `node:sqlite`（`DatabaseSync`）注册 SQLite 用户定义函数，这些函数可以执行任意操作（例如，在查询执行期间关闭数据库连接，导致崩溃或释放后使用条件）。
  * 在 `DatabaseSync` 中使用 `allowExtension` 选项加载 SQLite 扩展 — 此选项必须由应用程序显式设置为 `true`，启用它是应用程序操作员的责任。
  * 使用 `node:sqlite` 内置 SQL 函数或编译指示（例如 `ATTACH DATABASE`）读取或写入文件 — `DatabaseSync` 具有与进程本身相同的文件系统访问权限，限制执行什么 SQL 是应用程序的责任。
  * 在未进行适当输入验证的情况下，将 `child_process.exec()` 或类似 API 暴露给不受信任的用户，导致命令注入。
  * 允许不受信任的用户控制传递给文件系统 API 的文件路径而不进行验证，导致路径遍历问题。
  * 允许不受信任的用户定义以应用程序权限执行的自定义代码（例如自定义转换、插件或回调）。

* 这些场景代表应用程序级安全问题，而非 Node.js 漏洞。根本原因是应用程序未能在受信任的应用程序逻辑和不受信任的用户输入之间建立适当的安全边界。

#### 需要控制构建环境的构建系统攻击 (CWE-78, CWE-114, CWE-276)

* Node.js 构建系统（例如 `configure`、`configure.py`、`Makefile`、`vcbuild.bat`）旨在在受信任的构建环境中运行。构建环境，包括环境变量、文件系统和本地安装的工具，是 Node.js 威胁模型中的受信任元素。
* 关于构建脚本中通过环境变量（如 `CC`、`CXX`、`PKG_CONFIG`、`RUSTC`）进行命令注入、构建输出目录中的路径劫持或构建工件的文件权限的报告**不**被视为漏洞。这些场景要求攻击者已经控制构建环境，这意味着系统已被攻破。
* 构建脚本不是安全边界。它们被期望执行由环境指定的工具和脚本，并信任它们操作的文件系统。

#### EventEmitter 上未处理的 'error' 事件 (CWE-248)

* 可以触发 `'error'` 事件的 EventEmitter 要求应用程序附加 `'error'` 事件处理程序。这包括 HTTP 流和其他 Node.js 核心流。如果应用程序未能附加 `'error'` 处理程序，EventEmitter 将抛出未捕获的异常，可能导致进程崩溃。
* 因缺少 `'error'` 处理程序导致的崩溃不被视为 Node.js 中的拒绝服务漏洞。应用程序有责任通过为可能触发错误的 EventEmitter 附加适当的 `'error'` 事件监听器来正确处理错误。

#### 应用程序回调抛出的异常 (CWE-248)

* Node.js 信任其被要求运行的应用程序代码，包括由 Node.js API 调用的回调。如果应用程序回调抛出未捕获的异常，任何由此产生的崩溃不被视为 Node.js 中的漏洞。
* 例如，[CVE-2026-21637](https://www.cve.org/CVERecord?id=CVE-2026-21637) 被分类为 Node.js 漏洞，但要求 TLS 回调（如 `ALPNCallback`、`SNICallback` 或 `pskCallback`）抛出的场景不在 Node.js 威胁模型内。未来类似问题的报告，即崩溃取决于应用程序回调抛出未捕获异常，将不被视为 Node.js 漏洞。应用程序有责任处理意外的回调输入，并在不抛出未捕获异常的情况下报告错误。

#### 权限模型边界（`--permission`）

Node.js [权限模型](https://nodejs.org/api/permissions.html)（`--permission`）是一种选择加入机制，限制 Node.js 进程可以访问哪些资源。它旨在减少受信任应用程序代码中错误的爆炸半径，**不**作为针对故意滥用或受损进程的安全边界。

以下**不**是 Node.js 中的漏洞：

* **操作员控制的标志**：由操作员显式传递的标志（例如 `--localstorage-file`）解锁的行为是操作员的责任。权限模型不限制当操作员有意配置时 Node.js 的行为方式。

* **`node:sqlite` 和权限模型**：`DatabaseSync` 具有与进程相同的文件系统权限。使用 SQL 编译指示或内置 SQLite 机制（例如 `ATTACH DATABASE`）访问文件不会绕过权限模型 — 权限模型不会拦截 SQL 级别的文件操作。

* **路径解析和符号链接**：`fs.realpathSync()`、`fs.realpath()` 和类似函数在应用权限检查之前将路径解析为其规范形式。通过解析到允许路径的符号链接访问文件是预期行为，不是绕过。在允许列表内解析的符号链接上的 TOCTOU 竞争同样不被视为权限模型绕过。

* **带有修改 `execArgv` 的 `worker_threads`**：工作线程继承其父进程的权限限制。向工作线程传递空或修改的 `execArgv` 不会授予其额外权限。

#### V8 沙箱

V8 沙箱是 V8 内部的进程内隔离机制，不是 Node.js 安全边界。Node.js 不保证或记录 V8 沙箱作为安全特性，并且未以在生产 Node.js 构建中提供安全保证的方式启用。关于逃逸 V8 沙箱的报告不被视为 Node.js 漏洞；应直接报告给 [V8 项目](https://v8.dev/docs/security-bugs)。

#### `writeEarlyHints()` 中的 CRLF 注入

`ServerResponse.writeEarlyHints()` 接受一个由应用程序设置的 `link` 标头值。将任意字符串（包括 CRLF 序列）作为 `link` 值传递是应用程序级 API 滥用，不是 Node.js 漏洞。Node.js 根据 HTTP 规范验证 Early Hints 的结构，但不净化传递给的应用程序自由格式数据；这是应用程序的责任。

## 评估实验性特性报告

实验性特性与任何其他稳定特性一样，有资格获得安全报告。它们也可能获得与稳定特性相同的严重性评分。

## 接收安全更新

安全通知将通过以下方式分发。

* <https://groups.google.com/group/nodejs-sec>
* <https://nodejs.org/en/blog/vulnerability>

### CVE 发布时间线

当安全发布被发布时，相应的 CVE 公开披露之前有一个内置延迟。此延迟发生是因为：

1. 安全发布后，我们请求漏洞报告者在 HackerOne 上披露详细信息。
2. 如果报告者未在一天内披露，我们会进行强制披露以发布 CVE。
3. 然后披露会经过 HackerOne 的批准流程，然后 CVE 才会公开可用。

因此，CVE 可能在安全发布发布时并非立即可用，但通常会在发布后几天内披露。

## 对此政策的评论

如果您对此流程如何改进有建议，请访问 [nodejs/security-wg](https://github.com/nodejs/security-wg) 仓库。

## 事件响应计划

如果发生安全事件，请参阅[安全事件响应计划](https://github.com/nodejs/security-wg/blob/main/INCIDENT_RESPONSE_PLAN.md)。

## Node.js 安全团队

Node.js 安全团队成员被期望将他们在团队中获得特权的所有信息完全保密。这包括同意在问题尚未公开披露之前，不将任何问题通知团队外的任何人，包括问题的存在、即将发布的版本的预期，以及除作为安全团队成员工作过程之外的对任何问题的修补。

### Node.js 安全团队成员资格政策

Node.js 安全团队有权访问不适合公开的安全敏感问题和补丁。

纳入政策如下：

1. @nodejs/TSC 的所有成员都有权访问私密安全报告和私密补丁。
2. @nodejs/releasers 团队的成员有权访问私密安全补丁以生成发布版本。
3. 在个案基础上，技术指导委员会之外的个人可由 TSC 邀请访问私密安全报告或私密补丁，以便将他们的专业知识应用于问题或补丁。此访问可能是临时的或永久的，由 TSC 决定。

可以通过 TSC 仓库中的议题请求安全团队成员资格。

## 负责分类安全报告的小组

分类的责任是确定 Node.js 是否必须采取任何行动来缓解问题，如果是，则确保采取行动。

缓解可能采取多种形式，例如，包含修复的 Node.js 安全发布、文档、信息性 CVE 或博客文章。

* [@mcollina](https://github.com/mcollina) - Matteo Collina
* [@RafaelGSS](https://github.com/RafaelGSS) - Rafael Gonzaga
* [@vdeturckheim](https://github.com/vdeturckheim) - Vladimir de Turckheim
* [@BethGriggs](https://github.com/BethGriggs) - Beth Griggs

## 有权访问 Node.js 私密安全报告的小组

[TSC 投票成员](https://github.com/nodejs/node#tsc-voting-members) 有访问权限。

此外，以下个人有访问权限：

* [BethGriggs](https://github.com/BethGriggs) - **Beth Griggs**
* [MylesBorins](https://github.com/MylesBorins) -  **Myles Borins**
* [bengl](https://github.com/bengl)- **Bryan English**
* [bnoordhuis](https://github.com/bnoordhuis) **Ben Noordhuis**
* [cjihrig](https://github.com/cjihrig) **Colin Ihrig**
* [joesepi](https://github.com/joesepi) - **Joe Sepi**
* [juanarbol](https://github.com/juanarbol) **Juan Jose Arboleda**
* [sxa](https://github.com/sxa) - **Stewart X Addison**
* [ulisesgascon](https://github.com/ulisesgascon) **Ulises Gascón**
* [vdeturckheim](https://github.com/vdeturckheim) - **Vladimir de Turckheim**

该列表来自 HackerOne 上 Node.js 项目的[成员页面](https://hackerone.com/organizations/nodejs/settings/users)。

## 有权访问 Node.js 私密安全补丁的小组

<!-- ncu-team-sync.team(nodejs-private/security) -->

* [@aduh95](https://github.com/aduh95) - Antoine du Hamel
* [@anonrig](https://github.com/anonrig) - Yagiz Nizipli
* [@bengl](https://github.com/bengl) - Bryan English
* [@benjamingr](https://github.com/benjamingr) - Benjamin Gruenbaum
* [@BethGriggs](https://github.com/BethGriggs) - Beth Griggs
* [@bmeck](https://github.com/bmeck) - Bradley Farias
* [@bnoordhuis](https://github.com/bnoordhuis) - Ben Noordhuis
* [@BridgeAR](https://github.com/BridgeAR) - Ruben Bridgewater
* [@gireeshpunathil](https://github.com/gireeshpunathil) - Gireesh Punathil
* [@guybedford](https://github.com/guybedford) - Guy Bedford
* [@indutny](https://github.com/indutny) - Fedor Indutny
* [@jasnell](https://github.com/jasnell) - James M Snell
* [@joaocgreis](https://github.com/joaocgreis) - João Reis
* [@joesepi](https://github.com/joesepi) - Joe Sepi
* [@joyeecheung](https://github.com/joyeecheung) - Joyee Cheung
* [@juanarbol](https://github.com/juanarbol) - Juan José
* [@legendecas](https://github.com/legendecas) - Chengzhong Wu
* [@marco-ippolito](https://github.com/marco-ippolito) - Marco Ippolito
* [@mcollina](https://github.com/mcollina) - Matteo Collina
* [@MoLow](https://github.com/MoLow) - Moshe Atlow
* [@panva](https://github.com/panva) - Filip Skokan
* [@RafaelGSS](https://github.com/RafaelGSS) - Rafael Gonzaga
* [@richardlau](https://github.com/richardlau) - Richard Lau
* [@ronag](https://github.com/ronag) - Robert Nagy
* [@ruyadorno](https://github.com/ruyadorno) - Ruy Adorno
* [@santigimeno](https://github.com/santigimeno) - Santiago Gimeno
* [@ShogunPanda](https://github.com/ShogunPanda) - Paolo Insogna
* [@sxa](https://github.com/sxa) - Stewart X Addison
* [@targos](https://github.com/targos) - Michaël Zasso
* [@tniessen](https://github.com/tniessen) - Tobias Nießen
* [@UlisesGascon](https://github.com/UlisesGascon) - Ulises Gascón
* [@vdeturckheim](https://github.com/vdeturckheim) - Vladimir de Turckheim

<!-- ncu-team-sync end -->