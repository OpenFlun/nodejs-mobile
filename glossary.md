# 术语表

本文件记录了 Node.js 社区中使用的各种术语和定义。

* **ABI**：[应用程序二进制接口][] — 定义两个二进制程序模块之间的接口。
* **AFAICT**：据我所知（As Far As I Can Tell）。
* **AFAIK**：据我所知（As Far As I Know）。
* **API**：[应用程序编程接口][] — 一组规则和协议，允许不同的软件应用程序相互通信。API 用于实现不同系统之间的集成。
* **ASAP**：尽快（As Soon As Possible）。
* **ASLR**：地址空间布局随机化。一种安全技术，通过随机化内存地址来防止某些攻击。
* **Backport**：将修复或功能从较新分支应用到较旧的受支持分支的过程（例如，将安全修复应用到 LTS 版本）。
* **BE**：大端 [字节序][] — 最高有效位在前的一种字节序。[LE](#le) 的对立面。
* **Bootstrap**：Node.js 进程启动的早期阶段 — 设置执行环境并加载内部模块。
* **CI**：[持续集成][] — 一种开发实践，其中代码变更频繁地合并到共享仓库中。
* **CITGM**：金丝雀测试（Canary In The Gold Mine）— 一种冒烟测试，使用流行的 npm 包来测试代码变更。
* **CJS**：[CommonJS][] — JavaScript 模块的标准，在大多数情况下指 [CommonJS 模块][]。
* **CLDR**：[通用区域数据仓库][] — 软件工程中使用的区域数据仓库。
* **CLI**：[命令行界面][] — 使用文本命令与计算机程序交互的方式。
* **Code cache**（代码缓存）：存储已编译 JS 代码及其元数据的字节块。
* **CVE**：[通用漏洞披露][] — 维护已报告安全漏洞的数据库。
* **Deps**：依赖项 — 本项目所依赖的上游项目。
* **DOM**：[文档对象模型][] — Web 文档的编程接口。它将文档的结构表示为对象树，允许程序员动态操作网页的内容和结构。
* **ECMA**：[Ecma International][] — 一个非营利标准组织，负责制定和发布国际标准，包括 **ECMA-262**。
* **ECMA-262**：**Ecma** 关于 [**ECMAScript** 的规范文档][ECMAScript 规范文档]，由 **TC39** 维护和更新。
* **ECMAScript**：一种脚本语言标准，包括 **JavaScript**。
* **EOF**：[文件结束符][] — 表示文件或流的结束。
* **EOL**：[生命周期结束][]（在项目文档中使用时），[行结束符][]（在程序中使用时），通常此术语指的是生命周期结束。
* **ESM**：[ECMAScript 模块][] — **ECMA-262** 模块系统的实现。
* **ETW**：[Windows 事件跟踪][] — 提供一种在 Windows 系统中跟踪事件的方式。
* **FFDC**：首次故障数据捕获 — 程序出错时默认生成的日志、跟踪和转储。
* **FIPS**：[联邦信息处理标准][] — 一套供非军事政府机构和政府承包商在计算机系统中使用的标准。
* **FS**：文件系统。
* **Godbolt**：[编译器浏览器][] — 用于在 Web 浏览器中交互式运行编译器的工具。
* **HTTP**：[超文本传输协议][] — 一种用于分布式、协作式、超媒体信息系统的应用层协议。它是万维网上数据传输的基础。
* **ICU**：[Unicode 国际组件][] — 提供对 Unicode 支持的库。
* **IDE**：[集成开发环境][] — 一种为计算机程序员提供全面的软件开发设施的软件应用程序。
* **IETF**：[互联网工程任务组][] — 负责制定和推广互联网标准的国际组织。
* **IIRC**：如果我没记错的话（If I Recall Correctly）。
* **IIUC**：如果我没理解错的话（If I Understand Correctly）。
* **IMHO**：依我拙见/恕我直言（In My Humble/Honest Opinion）。
* **IMO**：依我看（In My Opinion）。
* **IPC**：[进程间通信][] — 允许进程之间相互通信的机制。
* **JIT**：[即时编译][] — 在运行时执行计算机代码的方法。
* **JS**：[JavaScript][] — 一种符合 **ECMAScript** 规范的高级解释型编程语言。
* **JS/C++ boundary**（JS/C++ 边界）：V8 运行时与 JS 代码执行之间的边界，通常在使用 C++ 链接调用 JS 函数时跨越。
* **JSON**：[JavaScript 对象表示法][] — 一种轻量级的数据交换格式，易于人们阅读和编写，也易于机器解析和生成。它通常用于在服务器和 Web 应用程序之间传输数据。
* **LE**：小端 [字节序][] — 最低有效位在前的一种字节序。[BE](#be) 的对立面。
* **LGTM/SGTM**：看起来/听起来不错（Looks/Sounds good to me）。
* **LTS**：[长期支持][] — 对软件版本提供的长期支持。
* **MDN**：[Mozilla 开发者网络][] — Web 开发者的资源。
* **MVC**：[模型-视图-控制器][] — 一种常用于开发用户界面的软件设计模式。它将应用程序分为三个相互关联的组件：模型（数据）、视图（展示）和控制器（逻辑）。
* **Native modules/addons**（原生模块/插件）：从非 JavaScript 语言（如 C 或 C++）编译为本机代码的模块，它们暴露可从 JavaScript 调用的接口。
* **npm**：[npm][] — 一种包管理器和注册表，广泛用于管理 Node.js 项目中的依赖项以及与其他人共享代码。
* **OOB**：越界（Out Of Bounds）— 用于数组访问的上下文中。
* **OOM**：内存不足（Out Of Memory）— 计算机程序超出其内存分配的情况。
* **OOP**：[面向对象编程][] — 一种基于“对象”概念的编程范式，对象可以包含数据以及操作这些数据的代码。OOP 语言包含封装、继承和多态等特性。
* **PPC**：[PowerPC][] — 一种微处理器架构。
* **Primordials**（原始对象）：JavaScript 中不受原型污染影响的纯净内置对象。
* **Prototype Pollution**（原型污染）：用户修改对象原型从而影响其他代码的过程。
* **PTAL**：请看一下（Please Take A Look）。
* **RAII**：[资源获取即初始化][] — 用于在 C++ 中管理资源的编程惯用法。
* **REPL**：[读取-求值-输出循环][] — 交互式编程环境。
* **RFC**：[意见征求文档][] — 用于标准化过程中的文档。
* **RSLGTM**：橡皮图章式“看起来不错”（Rubber-Stamp Looks Good To Me）— 审阅者未经完整代码审查即批准。
* **RSS**：[常驻内存大小][] — 进程在 RAM 中占用的内存量。
* **SMP**：[对称多处理器][] — 多个处理器共享同一内存的架构。
* **Snapshot**（快照）：包含从 V8 堆序列化数据的字节块。
* **TBH**：说实话（To Be Honest）。
* **TC39**：[Ecma 技术委员会 39][]，负责 **ECMAScript** 的管理机构。
* **TSC**：技术指导委员会 — 项目内的管理机构。
* **UI**：[用户界面][] — 用户与计算机程序之间的交互点。它包括按钮、菜单和其他允许用户与软件交互的图形元素。
* **URL**：[统一资源定位符][] — 对 Web 资源的引用，指定其在计算机网络上的位置以及获取机制，通常使用 HTTP 或 HTTPS 协议。
* **UTF-8**：[Unicode 转换格式 - 8位][] — 一种变宽字符编码，广泛用于在面向字节的系统中高效表示 Unicode 字符。
* **V8**：[JavaScript 引擎][]，驱动 Node.js 和 Chrome 浏览器。
* **Vendoring**（供应商化）：通过复制源代码将外部软件集成到项目中。
* **VM**：[Node.js VM 模块][] — 提供在 V8 虚拟机上下文中执行代码的方式。
* **W3C**：[万维网联盟][] — 一个国际社区，为 Web 生态系统的各个方面制定标准和指南。
* **WASI**：[WebAssembly 系统接口][] — WebAssembly 的接口。
* **WASM**：WebAssembly — 一种基于栈的虚拟机的二进制指令格式。
* **WDYT**：你怎么看？（What Do You Think?）
* **WG**：工作组 — 项目中的自治团队，具有特定的关注领域。
* **WHATWG**：[Web 超文本应用技术工作组][] — 开发 Web 标准的社区。
* **WIP**：进行中的工作（Work In Progress）— 未完成的工作，可能值得提前一看。
* **WPT**：[web-platform-tests][] — 用于 Web 平台 API 的测试套件。

[应用程序二进制接口]: https://en.wikipedia.org/wiki/Application_binary_interface
[应用程序编程接口]: https://en.wikipedia.org/wiki/Application_programming_interface
[命令行界面]: https://en.wikipedia.org/wiki/Command-line_interface
[通用区域数据仓库]: https://en.wikipedia.org/wiki/Common_Locale_Data_Repository
[通用漏洞披露]: https://cve.org
[CommonJS]: https://en.wikipedia.org/wiki/CommonJS
[CommonJS 模块]: https://nodejs.org/api/modules.html#modules-commonjs-modules
[编译器浏览器]: https://godbolt.org/
[持续集成]: https://en.wikipedia.org/wiki/Continuous_integration
[文档对象模型]: https://en.wikipedia.org/wiki/Document_Object_Model
[ECMAScript 模块]: https://nodejs.org/api/esm.html#modules-ecmascript-modules
[Ecma International]: https://ecma.org
[Ecma 技术委员会 39]: https://tc39.es/
[文件结束符]: https://en.wikipedia.org/wiki/End-of-file
[生命周期结束]: https://en.wikipedia.org/wiki/End-of-life_product
[行结束符]: https://en.wikipedia.org/wiki/Newline
[字节序]: https://en.wikipedia.org/wiki/Endianness
[Windows 事件跟踪]: https://en.wikipedia.org/wiki/Event_Viewer
[联邦信息处理标准]: https://en.wikipedia.org/wiki/Federal_Information_Processing_Standards
[超文本传输协议]: https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol
[集成开发环境]: https://en.wikipedia.org/wiki/Integrated_development_environment
[进程间通信]: https://en.wikipedia.org/wiki/Inter-process_communication
[Unicode 国际组件]: https://icu.unicode.org/
[互联网工程任务组]: https://www.ietf.org/
[JavaScript]: https://developer.mozilla.org/en-US/docs/Web/JavaScript
[JavaScript 对象表示法]: https://www.json.org/
[即时编译]: https://en.wikipedia.org/wiki/Just-in-time_compilation
[长期支持]: https://en.wikipedia.org/wiki/Long-term_support
[模型-视图-控制器]: https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
[Mozilla 开发者网络]: https://developer.mozilla.org/en-US
[npm]: https://www.npmjs.com/
[面向对象编程]: https://en.wikipedia.org/wiki/Object-oriented_programming
[PowerPC]: https://en.wikipedia.org/wiki/PowerPC
[读取-求值-输出循环]: https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop
[意见征求文档]: https://en.wikipedia.org/wiki/Request_for_Comments
[常驻内存大小]: https://en.wikipedia.org/wiki/Resident_set_size
[资源获取即初始化]: https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization
[对称多处理器]: https://en.wikipedia.org/wiki/Symmetric_multiprocessing
[JavaScript 引擎]: https://en.wikipedia.org/wiki/V8_\(JavaScript_engine\)
[Node.js VM 模块]: https://nodejs.org/api/vm.html
[Unicode 转换格式 - 8位]: https://en.wikipedia.org/wiki/UTF-8
[统一资源定位符]: https://en.wikipedia.org/wiki/URL
[用户界面]: https://en.wikipedia.org/wiki/User_interface
[Web 超文本应用技术工作组]: https://en.wikipedia.org/wiki/WHATWG
[WebAssembly 系统接口]: https://github.com/WebAssembly/WASI
[万维网联盟]: https://www.w3.org/
[ECMAScript 规范文档]: https://ecma-international.org/publications-and-standards/standards/ecma-262/
[web-platform-tests]: https://github.com/web-platform-tests/wpt