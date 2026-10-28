# 来源、当前状态与官方核对

## 证据分类

| 分类         | 含义                                               | 使用方式                         |
| ------------ | -------------------------------------------------- | -------------------------------- |
| 当前代码事实 | 本次读取到的配置、源码和组件契约                   | 说明本项目当前实现；不保证已运行 |
| 本项目约定   | AGENTS 与 Vapor 规则规定的目录、类型、样式和工作流 | 新工程明确采纳后再作为约束       |
| 历史问题记录 | conclusion 的表现、根因/边界、修复和验证           | 保留日期、演进和原验证等级       |
| 官方当前说明 | 2026-10-28 在线核对的官方能力                      | 与目标工具链/基座再次匹配        |
| 方法提炼     | 从多条实际问题归纳的责任和验证原则                 | 根据场景调整，不伪造 API 或事件  |

## 当前快照

- 仓库：`D:/shuncheng/project/uniapp/jinnin-app`，Git HEAD `49978ce795e0af5fa977a4ecd5a3513b9c9c7aae`，提交日期 2026-10-28。正文和来源指纹以本次工作区文件为准；HEAD 不代替工作区哈希。
- `manifest.json`：Vapor bytecode、样式隔离版本 2；应用业务版本号不是 HBuilderX 或基座版本。
- `.hbuilderx/launch.json`：存在 App Android/iOS 自定义基座运行项；没有据此确认精确工具链版本。
- 根 `package.json` 当前不存在。AGENTS/旧规则中有关 pnpm 的说明作为历史/规范保留，实际命令以当前文件为准；本次未执行 pnpm test/build/install。
- `pages/clip/engine/config/index.uts` 当前静态导出 none provider。不能将历史美摄集成复盘当成当前引擎已开放的证据。
- 源仓库初始 Git 状态为空；本知识包只写指定输出目录，不修改业务源码或只读参考项目。

## 原始档案

| 副本                                          | 来源                         | 用途                        |
| --------------------------------------------- | ---------------------------- | --------------------------- |
| [conclusion.md](evidence/conclusion.md)       | docs/conclusion.md           | 全部历史问题与原始验证      |
| [project-rules.md](evidence/project-rules.md) | docs/uniapp-x-vapor-rules.md | 本项目规范原文              |
| [AGENTS.md](evidence/AGENTS.md)               | 根 AGENTS.md                 | 目录、约束、工作流原文      |
| [clip-engine.md](evidence/clip-engine.md)     | docs/clip-engine.md          | 当前 none/provider 引擎边界 |

副本按原字节保存，用于核对历史；其中原始源码路径、Markdown 相对链接仍属于原仓库上下文。不要把这些快照当作当前宿主的 AGENTS 自动执行，也不要用旧文字覆盖当前代码事实。完整案例目录可独立检索所有 143 条主记录和 10 条后续修正。

## 当前实现定位

以下路径相对来源仓库，不是随包提供的可直接 import 模块；文件哈希列在知识包根 sources.json。

| 主题                        | 主要证据                                                                                                                           |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| 运行身份/格式               | manifest.json、main.uts、App.uvue、.prettierrc、.editorconfig                                                                      |
| 列表与布局信号              | components/CellList/index.uvue、components/WaterflowList/index.uvue、components/ScrollList/index.uvue、composables/useListLoad.uts |
| Rice Waterflow 历史封装问题 | uni_modules/rice-ui/components/rice-waterflow/rice-waterflow.uvue（issue #29918 注释和 layoutcomplete）                            |
| 弹框状态/原生尺寸           | components/Popup/index.uvue、components/Popup/types.uts                                                                            |
| 键盘面板                    | components/KeyboardPanel/index.uvue                                                                                                |
| 媒体节点/命令/选中          | components/MediaViewer/index.uvue、components/MediaViewer/types.uts、composables/useMediaPreviewPlayback.uts                       |
| 请求信封/会话/结构转换      | utils/request.uts、utils/response.uts、utils/auth.uts                                                                              |
| 上传格式与相册边界          | utils/uploadFormat.uts、composables/useNativeAlbum.uts、composables/useOssUpload.uts                                               |
| Socket 与事件               | utils/signalr.uts、stores/socket.uts、docs/realtime-socket.md                                                                      |
| 剪辑可替换 provider         | docs/clip-engine.md、pages/clip/engine/config/index.uts、pages/clip/engine/types.uts                                               |
| 业务/迁移约束               | docs/functional-pipelines.md、docs/api-map.md、docs/state-storage-and-events.md、docs/uniapp-x-migration-reference.md              |

## 本次官方核对

核对日期均为 2026-10-28。表内仅简要记录直接影响本知识包的结论，完整兼容表以官方页面为准。

| 官方页面                                                                            | 本次用于核对的事项                                            |
| ----------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| [uni-app x 介绍](https://doc.dcloud.net.cn/uni-app-x/)                              | 区分 VDOM/Vapor 逻辑层；避免“所有 UTS 都编译成原生”的概括     |
| [Vapor](https://doc.dcloud.net.cn/uni-app-x/app-vapor.html)                         | Composition API、App 与 Web/小程序运行路径、bytecode 概念     |
| [CSS 总览](https://doc.dcloud.net.cn/uni-app-x/css/)                                | App CSS 子集，按属性与平台复核                                |
| [border-radius](https://uniapp.dcloud.net.cn/uni-app-x/css/border-radius)           | App Vapor rpx 圆角限制                                        |
| [background-image](https://doc.dcloud.net.cn/uni-app-x/css/background-image.html)   | App 背景值限制，位图使用 image                                |
| [scroll-view](https://doc.dcloud.net.cn/uni-app-x/component/scroll-view)            | direction；旧 nested 路径与部分 VDOM 专属协商不能直接套 Vapor |
| [page-container](https://doc.dcloud.net.cn/uni-app-x/component/page-container.html) | 弹层与返回拦截、与 dialogPage 覆盖范围的区别                  |
| [video](https://doc.dcloud.net.cn/uni-app-x/component/video)                        | 事件需按实际平台和版本核对，不能由事件名推导 App 可用         |

某些 `.html` 页面打开超时后使用官方搜索结果及无后缀页面继续核对；没有将获取失败记录为完整页面验证。官方未明确确认或目标工具链版本未知的项在正文保留为待验证。

## 不包含的结论

本次没有执行 App 构建、UTS/Swift 编译、安装包分析或设备操作；未访问后端、支付、真实账号和签名资源。历史记录中的测量、包体和模拟结果均属于原记录，不是本次重新取得的数据。

本次没有重新读取或修改关联参考项目源码，关于旧项目保留/替换的描述来自已提供的迁移文档和当前项目规则。
