# 架构、环境与生命周期

## 技术身份

来源工程在 `manifest.json` 明确配置 `vapor: true`、`vapor-render-target: bytecode`、`styleIsolationVersion: 2`。页面与组件为 `.uvue`，脚本采用 UTS，入口为 `App.uvue` 和 `main.uts`。

必须区分四件事：Vue 的响应式逻辑、Vapor 视图编译、App 原生渲染和 UTS 原生插件编译。不能因为用了 UTS 就断言整个 App 逻辑都编译成 Kotlin/Swift。当前官方介绍已区分 VDOM 与 Vapor 的逻辑层执行方式；新项目遵循目标版本支持的语言，本工程继续遵守已有 UTS 约定。[官方介绍](https://doc.dcloud.net.cn/uni-app-x/)

官方 Vapor 文档说明 App 各平台支持起点不同，Web/小程序目前的运行路径也不同。配置启用 Vapor 不能证明所有目标都走同一渲染路径；本知识包不将官方最新最低版本冒充本项目实际版本。[Vapor 文档](https://doc.dcloud.net.cn/uni-app-x/app-vapor.html)

## 目录职责

| 层级                               | 放什么                                         | 避免什么                             |
| ---------------------------------- | ---------------------------------------------- | ------------------------------------ |
| `pages/`                           | 页面编排、路由参数、页面生命周期与少量直接状态 | 大量媒体控制、请求基础设施、原生细节 |
| 功能目录 `composables/`            | 该功能独立交互状态、请求编排、资源管理         | 提取只转发一次的空包装               |
| 根 `composables/use*.uts`          | 跨功能复用的响应式能力                         | 为形式统一复制已有逻辑               |
| `components/PascalCase/index.uvue` | 跨处复用的 UI/交互                             | 把普通静态区块都拆成组件             |
| `api/<domain>/`                    | 端点、参数、响应与领域转换                     | 在页面散写 URL                       |
| `utils/`                           | 无响应式状态的纯函数、协议与基础服务           | 页面专属响应式状态                   |
| `stores/`                          | 登录会话、跨页面状态、领域事件分发             | 所有弹框/loading 都全局化            |
| `config/`                          | 环境配置、静态能力选择                         | 页面散落平台开关                     |
| `uni_modules/`                     | 第三方组件、UTS API/原生组件插件               | 为单页外观修改组件库                 |
| `unpackage/`                       | HBuilderX 生成物                               | 手改或把生成物当源码提交             |

这些目录和 PascalCase 命名是来源项目约定，不是框架唯一目录结构。

## 生命周期归属

- `onLoad`：读取并校验路由参数；决定一次性初始化输入。
- `onReady`：需要真实节点的初始化/测量入口之一，仍须核对具体节点是否已挂载。
- `onShow`：恢复可见状态、按需回查数据；不机械重复初始化、发验证码或启动导出。
- `onHide`：暂停页面媒体、暂停 UI 时钟、使不应继续影响 UI 的回调失效；保留业务要求保留的草稿/会话。
- `onUnload`：清理订阅、定时器、未完成任务和原生句柄，增加版本号或销毁标志。
- 组件 mounted/unmounted 管理组件自身资源，不能等同于页面显示/隐藏。

路由参数决定不可变配置时，先完成参数校验再挂载功能组件，避免组件按默认参数创建状态后无法纠正。

## 先复用的能力

来源项目已有 `CellList`、`ScrollList`、`WaterflowList`、`Popup`、`KeyboardPanel`、`MediaViewer`、`SwiperViewer`、`UploadFile`。列表触发门禁是 `useListLoad`，媒体播放控制包括 `useVideoPlayback`、`useMediaPreviewPlayback`；请求入口为 `utils/request.uts`。

这些组件不随知识包复制；新工程可以复用设计与契约思想，但调用之前必须存在实际实现，不能让 AI 生成不存在的 import。

## 启动一个新功能

先确定入口、路由参数、API 字段、页面状态和可复用组件。再画出数据从接口到视图的路径及隐藏/返回时的去向；复杂功能补充“谁拥有状态、谁释放资源”。最后实现最小完整链路，包括失败恢复和空态，而不是先搭所有组件壳。
