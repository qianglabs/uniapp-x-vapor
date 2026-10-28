# 视频剪辑引擎配置

剪辑页面、素材选择、轨道交互、Canvas 波形和节拍标记使用项目自己的页面代码。厂商引擎只负责原生媒体编辑、预览和导出。

## 当前发布配置

默认 `none`：保留全部剪辑路由和入口，编辑页显示静态布局与“视频剪辑暂未开放”。不初始化编辑引擎，不轮询原生状态，不导入媒体或启动导出；工具点击给出未开放提示。智能混剪复用的素材选择、预览和生成流程保持原有行为。

唯一配置文件为 `pages/clip/engine/config/index.uts`。切换时只修改该文件末尾的静态导出路径，不执行脚本，不修改页面或预览组件。

无 SDK（当前默认）：

```uts
export { clipEngineSupported, clipEngineUnavailableMessage, initializeClipEngine, runClipEngineCommand, createClipEnginePreview } from "../providers/none.uts";
```

恢复美摄：保留上述五个导出名称，将唯一的 `from` 路径改成 `"../providers/meishe.uts"`。若此前移除了 `uni_modules/uts-meishe-basic`，先还原该插件。保存配置后按正常 HBuilderX 流程运行或打包即可。

不能同时启用两条导出，也不能先 import 全部厂商再用普通布尔值或 JSON 在运行时选择。这种静态配置同时控制引擎调用与预览创建；没有选中的 provider 不进入页面引用链。

## 引用边界

- `pages/clip/engine/types.uts`：直接声明与厂商无关的具名视频、音乐、波形、命令和状态类型；时间单位统一为毫秒。
- `pages/clip/engine/config/index.uts`：唯一的人工配置入口，只静态导出一个 provider。
- `pages/clip/engine/index.uts`：固定的公共入口，转发配置选中的接口，切换引擎不修改此文件。
- `pages/clip/engine/preview-types.uts`：预览释放行为契约，不包含厂商类型。
- `pages/clip/component/ClipEnginePreview/index.uvue`：固定的通用预览组件，未接入时是普通 view；接入时通过当前 provider 创建 native-view 的原生预览，并在卸载时释放。组件不直接引用美摄。
- `pages/clip/engine/providers/none.uts`：声明不可用；意外调用原生功能时明确报错，不伪造编辑、授权或导出成功。
- `pages/clip/engine/providers/meishe.uts`：唯一的美摄接口适配层，显式转换 SDK 模型与页面模型。美摄 License 仍在 `config/meishe.uts` 配置。
- `uni_modules/uts-meishe-basic`：保留原插件及二进制文件；`none` 模式的页面引用链不经过此目录。

当前 HBuilderX 的 `package.json` 自定义平台不支持扩展 App 打包，所以采用单个 UTS 文件的静态导出配置。配置保存后直接参与下一次编译，不依赖额外生成步骤。

## 后续新增引擎

1. 新增 provider 适配文件，实现 `clipEngineSupported`、`clipEngineUnavailableMessage`、`initializeClipEngine`、`runClipEngineCommand`、`createClipEnginePreview` 五个导出函数。预览句柄实现 `ClipEnginePreviewHandle`，负责释放厂商实例及 View 引用。
2. 使用 `engine/types.uts` 的页面契约，并在适配边界处理厂商的时间、音量、片段和转场差异。不将厂商类型传播到页面。
3. 将 `engine/config/index.uts` 的静态路径指向新 provider；页面、通用引擎入口和预览组件保持不变。
4. 完成导入、实时预览、转场、音频独立位置和尾段、导出及释放的真机验证后，再用于正式发布。

本轮没有接入阿里、FFmpeg 或其他替代引擎。

## 打包与验收

DCloud 的 uni-app x 按引用决定 UTS 插件是否参与打包。`none` 模式应使用新制作的正式安装包或自定义基座，旧基座和热更新不能证明原生 SDK 已移除。

发行前确认静态配置只指向本次需要的 provider，并对新安装包检查：

- 不存在 `libNvStreamingSdkCore.so` 及美摄原生类。
- 剪辑页能进入、返回、展示静态布局，无初始化、缺类、权限申请或轮询错误。
- 素材选择、素材预览和智能混剪仍正常。
- 恢复 `meishe` 后重新制作基座，检查原有剪辑功能。

实际安装包内容以打包后的检查结果为准，源码引用检查不能替代 App/Vapor 编译和真机验收。

参考：[模块摇树](https://doc.dcloud.net.cn/uni-app-x/collocation/manifest-modules.html)、[自定义平台的 App 限制](https://uniapp.dcloud.net.cn/collocation/package.html)。
