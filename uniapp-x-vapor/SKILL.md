---
name: uniapp-x-vapor
description: 为 uni-app x Vue 3 Vapor bytecode 工程开发、迁移和排错，覆盖 UVue/UTS 类型与模板、原生列表和滚动、弹层与键盘、媒体播放、请求状态及原生插件。适用于明确使用 Vapor 的项目；传统 uni-app、普通 Vue 和 uni-app x VDOM 需先识别模式，不能直接套用本技能的项目约定或历史绕过方案。
---

# uni-app x Vapor

把需求落到目标工程现有架构中，保持可感知行为、平台契约和资源生命周期一致。此技能提炼自实际工程，不把历史编译问题或 Rice UI 用法当成所有 Vapor 版本的规则。

## 先确定事实

1. 读取目标仓库的 AGENTS、复盘和平台规则；查看 manifest、页面入口、依赖与真实命令。确认是 Vapor bytecode，而非仅凭 `.uvue` 判断。
2. 记录目标平台、HBuilderX 版本/发布通道、基座版本和系统版本。缺少信息时继续可独立完成的工作，把版本相关结论标成待验证。
3. 在当前代码追踪页面 → 组件/composable → API/请求层 → 插件。先复用已有能力，再考虑扩展。
4. 与任务相关的组件、事件、CSS 属性和值，核对当前 DCloud 官方文档及目标工具链。在线最新兼容表不能证明旧基座支持。

## 按任务读取

| 任务                              | 必读参考                                                          |
| --------------------------------- | ----------------------------------------------------------------- |
| 新页面、目录组织、路由生命周期    | [architecture.md](references/architecture.md)                     |
| UTS、类型、Prop、插槽、编译报错   | [uts-and-template.md](references/uts-and-template.md)             |
| 样式、安全区、尺寸、裁切          | [css-and-layout.md](references/css-and-layout.md)                 |
| 列表、瀑布流、分页、吸顶和滚动    | [lists-and-scroll.md](references/lists-and-scroll.md)             |
| 弹层、跟随菜单、输入和键盘        | [popup-and-keyboard.md](references/popup-and-keyboard.md)         |
| 视频、封面、轮播、预览和下载      | [media.md](references/media.md)                                   |
| API、鉴权、请求竞态、Socket、表单 | [requests-and-state.md](references/requests-and-state.md)         |
| 相册、上传、原生插件、剪辑 SDK    | [native-and-engine.md](references/native-and-engine.md)           |
| 迁移、快速开发和验收模板          | [migration-and-workflow.md](references/migration-and-workflow.md) |
| 崩溃、运行产物、检查、平台验收    | [debug-and-validation.md](references/debug-and-validation.md)     |

需要核对某条经验时查 [case-catalog.md](references/case-catalog.md)，再读对应原始记录；需要区分当前实现与旧结论时读 [project-evidence.md](references/project-evidence.md)。不要默认加载全部历史档案。

## 实施约束

- 使用 Composition API。遵守目标项目的脚本语言；本来源工程采用 `.uvue`、`<script setup lang="uts">`、`.uts`，不是所有新版本 Vapor 都只能使用 UTS。
- 本经验包优先用模块级直接具名 `type` 表达数据与嵌套模型，避免用 `any`、连续断言、复杂交叉别名掩盖实际问题。
- App 共享代码不使用浏览器 DOM/CSSOM；需要节点测量时核对 uni-app x 官方元素 API。原生对象留在插件边界。
- 使用现有领域 API 和唯一请求层；只建模已确认字段，不猜字段别名，不通过 JSON 克隆充当普通业务类型转换。
- 先明确请求来源、状态归属、请求/会话版本和资源释放，再连接视图。失败必须可恢复，单纯复位 loading 用 `finally`。
- 样式以受支持的 Flex 和独立语义 class 实现。本项目 px 字号/行高、px 圆角、真实图片背景和独立安全区占位的具体约束见样式参考；新工程需确认采用范围。
- 原生列表按稳定业务 key 读取数据；fixed 弹层放在功能根部；播放器上下文只能在对应节点实际存在时获取。
- 不以动态 key、任意延时、超大 z-index 或反复销毁容器替代根因定位。历史分批提交方案只用于证据确认的同类刷新崩溃。
- 修改组件库前先证明漏洞发生在组件库，检查调用方与跨平台影响。页面适配优先在业务层完成。
- 迁移先提取行为与状态流，再选择 Vapor 实现；参考工程只读。保持用户指定的视觉基准，不擅自改版。

## 验证与交付

只运行实际存在且适合任务的检查。格式化本次文件，检查具名类型、空数据、迟到回调、重复操作和卸载清理；原生能力用相应工具链与设备验证。生成代码仅用于只读定位，不能手改。

说明具体行为变化、文件、保留/替换的旧行为、实际执行的验证及未验证的平台。区分静态、模拟、UTS/原生编译、打包与真机验证。临时验证脚本及产物在验证后删除，不删除已有工具或用户文件。

官方入口：[Vapor](https://doc.dcloud.net.cn/uni-app-x/app-vapor.html)、[CSS](https://doc.dcloud.net.cn/uni-app-x/css/)、[组件](https://doc.dcloud.net.cn/uni-app-x/component/)、[更新日志](https://doc.dcloud.net.cn/uni-app-x/release.html)。
