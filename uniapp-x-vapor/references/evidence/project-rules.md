# uni-app x Vapor 项目开发规则

> 本文保存详细的项目知识与执行检查；根目录 `AGENTS.md` 只保存必须始终遵守的硬规则。

## 1. 每次任务的阅读顺序

```text
AGENTS.md
→ docs/conclusion.md
→ 本文
→ 与任务相关的业务分册
→ 当前项目代码
→ 必要时只读分析 ../ai-commerce-app-view
→ 用户当前需求
```

不需要为一个小任务读取所有业务文档。首页、资产、积分、Socket 等任务分别读取相应分册；只有业务细节不足时才继续追踪参考源码。

## 2. 当前工程架构

| 层级         | 固定位置               | 职责                                           |
| ------------ | ---------------------- | ---------------------------------------------- |
| 页面         | `pages/`               | 路由参数、生命周期、数据加载、页面编排         |
| 公共组件     | `components/`          | 可复用 UI 与局部交互，通过 props/emits 通信    |
| 共用逻辑     | `composables/`         | 分页、缓存、请求状态、媒体与跨组件响应式逻辑   |
| 领域 API     | `api/`                 | URL、method、请求/响应类型和接口函数           |
| 请求基础设施 | `utils/request.uts`    | base URL、鉴权、刷新、错误、重复提交和响应归一 |
| 环境配置     | `config/`              | 环境与服务地址，不存放密钥                     |
| 组件库       | `uni_modules/rice-ui/` | 优先复用的 `rice-*` 组件                       |
| 项目规则     | `AGENTS.md`、`docs/`   | 硬规则、业务参考、迁移说明与复盘               |

页面只负责编排。两个以上页面稳定复用的有状态逻辑进入 `composables/`；可复用视觉进入 `components/`；无状态小函数进入 `utils/`。不要为了“分层”创建只有一次调用的空包装。

多个页面若仅向同一功能组件传递不同固定参数，应合并为一个参数化路由页面；页面必须先校验路由参数，再挂载依赖该参数初始化的子组件，避免子组件先按默认参数创建不可变状态。

## 3. Vapor/UTS 边界

当前 `manifest.json` 已启用：

```json
{
	"uni-app-x": {
		"styleIsolationVersion": "2",
		"vapor": true,
		"vapor-render-target": "bytecode"
	}
}
```

因此开发时必须遵守：

-   页面和组件使用 `<script setup lang="uts">` 与 Composition API。
-   不使用 Options API、mixin、`this`、VNode/render 函数或旧 VDOM 技巧。
-   按运行期字符串读取配置时，不把普通对象断言为字符串索引字典后动态取值；使用 `Map<string, XxxConfig>` 或显式分支，并让兜底直接返回具名默认配置，避免查找失败与兜底同时得到 `undefined`。
-   不把 App 当浏览器，不在共享代码使用 DOM、CSSOM、`window`、`document`。
-   原生能力采用“页面/组件 → composable/service → UTS 接口 → 平台实现”。
-   长列表优先评估 uni-app x 的 `list-view`、`waterflow`，不照搬 z-paging/renderjs/nvue 优化。
-   定时器、监听、Socket、播放器和原生资源在 `onHide`/`onUnload` 等正确时机释放。

平台和 HBuilderX 版本决定能力是否可用；静态检查或 Web 运行不能证明 Android/iOS 兼容。

### 3.1 内置组件事件兼容

-   `.uvue` 模板绑定内置组件事件前，必须核对 uni-app x 对应组件文档及目标 HBuilderX 版本；传统 uni-app、HTML 或小程序存在同名组件，不代表事件能通过 Vapor App 编译。
-   媒体类型优先按实际资源 URL 的扩展名判断，后台 `templateType`、输入 `dataType` 等声明只作为无扩展名时的兜底；请求 OSS 缩略图前必须再次确认对象是图片，视频源始终请求原文件签名，视频封面作为独立图片可请求缩略图。
-   `video` 的 `@loadedmetadata` 当前仅用于 Web、微信小程序，必须通过 `WEB || MP-WEIXIN` 条件编译隔离；App 使用已支持的 `@play`、`@timeupdate` 同步播放和加载状态，不得绑定 `@loadeddata`、`@canplay`、`@loadedmetadata`。
-   `uni.createVideoContext(id)` 只能在对应 `<video>` 节点已实际渲染时调用。节点受 `v-if` 控制时，失活、切换分支或卸载后不得再按 id 获取上下文；返回后判空不能防止 Android 原生层在查找到非视频节点时先发生类型转换异常。自定义组件内通过条件编译区分上下文查找：App 传当前 `ComponentPublicInstance` 限定组件作用域，Web/小程序保持 `uni.createVideoContext(id)`，禁止把 App 的作用域参数无差别带到 Web。
-   新增或修改媒体组件后，至少执行一次 `rg -n --glob '*.uvue' "@(loadeddata|loadedmetadata|canplay)" pages components`；命中项必须处于受支持的平台条件编译块内，或具备当前目标 HBuilderX 版本的官方兼容依据。

### 3.2 列表结构编译约束

-   `sticky-header` 必须是 `sticky-section` 的直接子节点；Vapor bytecode 下一个 `sticky-section` 内恰好保留一个直接 `sticky-header` 和一个直接带 `v-for` 的 `list-item`。
-   骨架项、空态、错误态、加载尾部和自定义 refresher 不放入同一个 `sticky-section`，继续作为 `list-view` 的直接子节点。
-   业务瀑布流统一使用 `components/WaterflowList`。本项目固定使用 Vapor，且 Rice UI 已记录 Vapor 下把原生 `waterflow` 封装进自定义组件存在问题（DCloud issue #29918），因此公共组件仅封装 Rice Waterflow，不保留非 Vapor 原生分支。
-   业务实体 ID 通过 `key` 或 `dataKey` 传给作用域插槽；所有插槽调用方都应按 `itemKey` 从当前数据映射中查找并处理不存在状态，禁止用可能短暂失配的 `index` 直接读取业务数组。接收可空数据的卡片必须保持安全根节点，不能依靠父级 `v-if` 与必填 Prop 在同一更新批次完成收窄。
-   `waterflow` 的双列骨架可以作为普通 `flow-item`/`rice-waterflow-item` 参与列宽和高度测量，也可以作为列表外覆盖层，但列表容器必须保持常驻。使用覆盖层时不能仅按请求 `loading` 撤下骨架，必须等待瀑布流完成当前业务项布局；`load-more`/`load` 插槽是整行尾部，只承载空态、错误态、结束态和分页反馈，不放双列骨架。
-   列表必须区分初始/切换加载、下拉刷新和触底分页：初始或数据集切换显示骨架，下拉刷新保留刷新反馈，只有触底分页请求显示“加载中”；公共列表根据请求来源和 `loading` 自动切换，不再增加独立骨架开关，骨架状态也不与尾部 `showLoadState` 开关绑定。
-   包含子项注册、尺寸缓存或异步测量的列表容器不得通过动态 `key` 强制重建来刷新布局；数据集整体替换时应保持滚动容器实例稳定，优先在页面提交边界触发组件已有的子项卸载、挂载和重新测量流程，只有确认组件自身缺少必要失效能力时才评估组件库修改。
-   多列数据按行放入 App 原生 `list-view` 时，行 `key` 不得拼接全部子项 ID；首部插入会造成整表换 key。行 key 应表达稳定位置和结构，子项继续使用业务 ID；上传行、普通满行和尾部不足行等不同子树必须使用不同 `type`。公共列表通过作用域插槽渲染业务行时，应同时传递行 key，业务数据按 key 解析，不只依赖可能与原生复用状态失配的下标。
-   `position: fixed` 的弹框、遮罩和上传来源选择器不得由可复用 `list-item` 持有。列表项只发出打开请求，弹框及其交互状态放在列表外的功能组件根部或页面末尾；禁止通过提高 `z-index` 或延时规避原生列表复用与根层命中区域错位。
-   列表外的隐藏功能组件若承载弹框，不得仅靠组件 `ref` 的公开方法切换内部显隐；调用方必须通过 `v-model` 持有弹框状态，并在所属页面或外层弹框关闭时同步复位，避免反复打开后残留不可见的待展示状态。
-   App 原生弹框为保留关闭动画而延迟卸载真实节点时，必须在 `show` 变为 `false` 的同一批次关闭弹框和遮罩的命中测试，并在重开时恢复；不能只通过 `transform`、`opacity` 或延迟 `v-if` 判断节点已经不可点击，否则不可见原生层仍可能消费下一次页面点击。
-   App + Vapor 中需要连续切换、返回拦截或多层展示的业务弹框优先使用内置 `page-container`；弹框组件统一声明在功能根组件末尾并由根组件持有受控状态，不嵌入滚动内容、列表项或随业务状态切换卸载的条件模板。Android 上包含标签、滚动区、原生列表或媒体节点的重型底部弹层，必须给 `page-container` 自身明确 `height`，内容再通过 `min-height: 0` 和 Flex 分配；只给子树 `height/min-height` 仍可能让原生容器首次测量为 0。公共 `Popup` 在 App 中保持原生实例和插槽常驻，通过 `show` 控制显隐，不在每次 `afterleave` 后销毁；非 App 仍保留按需挂载。打开、关闭和离场交接使用明确阶段，关闭期间的新打开请求在完整离场及 `closed` 调用方更新提交后按最新业务 `show` 执行；尚未提交的打开任务可直接取消，异步任务须核对请求版本和卸载状态。`nextTick` 仅用于提交业务更新，不代表原生内容已附着或绘制完成；不得把动态 key 或每次销毁重建当作已验证的空内容修复。App 开发环境记录实例、轮次、原生事件和面板边界，诊断不得反向控制显隐或自动重试，实际效果仍需真机确认。修改公共组件 Prop 后仍必须检查生成代码中的运行时 `props` 表，不能只看 UTS 类型类和 setup。嵌套弹框保持父层状态，父节点在前、子节点在后，子层使用更高层级。迁移时保留现有内容样式与事件契约，不为适配页面修改 `uni_modules/`。
-   公共 `Popup` 只负责弹层显示、关闭、尺寸和原生实例生命周期，不注册全局键盘监听，也不通过内容透明度等待键盘。需要跟随键盘的输入面板使用 `KeyboardPanel`，由实际输入组件的 `keyboardheightchange` 事件驱动。
-   原生 `list-item` 中的入口若需要打开列表外 fixed 弹框，出现命中或事件传递异常时不能继续修改弹框动画补偿入口问题。对无需分页的本地“最近项目”使用普通纵向 `scroll-view` 网格，让入口和弹框由同一功能组件直接控制；远程分页数据继续使用原生列表。
-   Android Vapor 的原生崩溃日志若确认空指针位于主线程的 `RenderEffect` 刷新链，检查是否在同一刷新批次把 keyed `v-for` 从一批非空 key 整体替换为另一批 key；已通过触发差异确认属于该路径时，在页面数据提交边界先卸载旧列表，等待 `nextTick` 并复查请求版本后再挂载新列表，不通过重建外层滚动容器规避。

## 4. CSS 与组件样式规范

### 4.1 固定原则

-   新增或修改样式前，先查 DCloud 当前 uni-app x CSS 文档，确认属性、属性值及关联组件在 App + Vapor 的兼容版本；Web 或传统 uni-app 可用不代表本项目可用。
-   布局以 Flex 为默认方案；使用明确的 `flex-direction`、对齐方式和尺寸。
-   App 原生端不能使用 `background-image: url(...)` 渲染装饰位图；改用真实 `<image>` 节点并通过局部 class 定位。只有官方明确支持的渐变场景才使用 `background-image`。
-   App 端 `min-height` 不使用百分比；需要固定下限时使用编译器支持的 `px`/`rpx`，内容自适应优先移除该声明并交给 Flex 布局。
-   App + Vapor 的 `border-radius`、`border-top-left-radius`、`border-top-right-radius`、`border-bottom-left-radius`、`border-bottom-right-radius` 不使用 `rpx`；CSS、动态样式以及 Rice UI 的圆角属性统一传明确的 `px`。普通圆角沿用项目 4px/8px/10px/12px 等视觉层级，圆形节点使用 `999px`，避免节点尺寸仍随 `rpx` 缩放时固定半径失真。
-   文字相关尺寸使用 `px`，包括 `font-size`、`line-height` 和新修改的 Rice UI 文字尺寸属性。
-   每个需要控制样式的 `<view>`、`<text>`、`<image>` 都添加明确 class。
-   父页面不修改子组件内部节点；子组件负责自己的全部视觉样式。
-   不依赖文字、颜色、行高等从父容器自动继承；需要什么就在对应 class 上明确设置。

### 4.2 子元素命名

项目使用扁平下划线命名来替代关系选择器：

```css
/* 禁止 */
.card .title {
	font-size: 16px;
}

.card > .description {
	color: #666666;
}

/* 正确 */
.card_title {
	font-size: 16px;
}

.card_description {
	color: #666666;
}
```

模板中也必须直接使用最终类名：

```vue
<view class="card">
  <text class="card_title">标题</text>
  <text class="card_description">说明</text>
</view>
```

不使用 `:deep()`、`>>>`、`.a .b`、`.a > .b`、标签选择器、伪元素或结构选择器完成关键 UI。装饰内容使用真实节点。

### 4.3 Flex 示例

```css
.card {
	display: flex;
	flex-direction: column;
}

.card_header {
	display: flex;
	flex-direction: row;
	align-items: center;
	justify-content: space-between;
}
```

不默认使用 Grid、复杂 `calc()`、浏览器 overflow 技巧或大 `z-index`。滚动使用官方滚动/列表组件，安全区使用项目或官方能力。

### 4.4 本地样式

当前项目不使用 UnoCSS、Tailwind 等原子样式生成器，避免生成器预设、默认提取和 preflight 向 App 样式链引入未确认的选择器与属性。

-   模板使用组件或业务语义明确的 class，不依赖构建期生成的 `flex`、`w-full`、`bg-*` 等工具类。
-   页面和组件样式优先写在对应 `.uvue` 的局部 CSS/SCSS；多个页面明确共享的视觉规则才放入公共样式文件。
-   Flex、尺寸、边距、颜色、边框和圆角等简单规则也直接归属到对应 class，不建立第二套工具类体系。
-   复杂动画、阴影、层叠、原生组件覆盖和平台差异继续使用已验证的局部 class/平台方案。
-   不引入 `virtual:uno.css`、`uno.config.*`、UnoCSS Vite 插件或相关依赖。

## 5. Rice UI 使用规则

实现按钮、输入框、导航、弹窗、图片、列表反馈等能力前，先搜索 `uni_modules/rice-ui/` 和现有页面中的 `rice-*` 用法。

1. 组件已有能力可以满足时直接复用。
2. 需要业务组合时在 `components/` 封装，不修改 Rice UI 内部样式来适配单页。
3. 确需修复 Rice UI 本身时，先确认所有调用方和跨平台影响，再单独修改并说明。
4. 不从参考项目引入 uView Pro 作为第二套组件体系。

## 6. 请求与 API 规则

-   所有领域接口放在 `api/<domain>/`，统一导入 `@/utils/request`。
-   页面、组件、composable 不直接使用 `uni.request`，不重复实现 token、刷新和错误处理。
-   先复用现有请求/响应类型；ID、可空字段、分页字段和状态枚举必须明确建模。
-   不因参考项目有字段就假设当前接口一致；核对 URL、method、query/body、响应层级和业务码。
-   快速切换、分页和重复提交必须考虑旧请求晚于新请求返回的竞态。
-   不在日志记录 token、支付内容、身份证件、上传签名或完整私有 URL。
-   流水线输入必须按视频、`AiSystemPrompt` 系统提示词、图片和其他参数（例如选项）四类契约处理：视频回填 `value` 与 `coverUrl`；系统提示词回填 `value`、`url`、`displayValue` 和视频展示所需的 `coverUrl`；图片和图片列表只修改 `value`；其他参数只修改对应的 `value`。分类依据是后台输入定义，不能用用户所选资源的媒体类型反向改变参数结构。禁止把封面地址写入文本语义的 `displayValue`，也不得用普通输入的 `url` 或 `displayValue` 兜底。

## 7. 参考项目的行为迁移方法

`../ai-commerce-app-view` 永远只读。不要做“旧文件 → 新文件”的机械转换，按以下模板先提取行为：

```text
入口与参数：从哪里进入，接收什么参数
初始化：onLoad/onShow 分别做什么
数据流：route → request → response transform → state → UI
用户操作：点击/切换/提交/返回/刷新/分页后的状态变化
页面状态：initial/loading/success/empty/error/refreshing/loadingMore
依赖：component/composable/store/API/cache/全局事件/Socket
异常：空参数、接口失败、重复点击、快速切换、回前台
```

随后建立迁移映射：

| 旧项目                       | 当前项目                           |
| ---------------------------- | ---------------------------------- |
| Options API / mixin / `this` | Composition API / composable       |
| `.vue` / `.nvue`             | `.uvue` + UTS                      |
| uView Pro                    | Rice UI 或项目组件                 |
| renderjs / DOM               | Vue 状态、uni-app x 组件/API       |
| `plus.*`                     | 支持的 `uni.*` API 或 UTS 插件     |
| z-paging                     | 按需求评估 `list-view`/`waterflow` |
| 旧请求封装                   | `utils/request.uts` + 当前 `api/`  |

保留初始化顺序、交互结果、接口时机、状态机和错误恢复；实现结构、组件和 CSS 必须服从当前项目。若目标 UI 已存在，禁止按旧页面重新生成 UI。

## 8. 格式化与最小改动

根目录 `.prettierrc` 已为 `.uvue` 使用 Vue parser、为 `.uts` 使用 TypeScript parser。只格式化本次修改文件，并保留 `.editorconfig` 的 UTF-8、LF、Tab 宽度 4 和末尾换行。项目未声明可用的 Prettier CLI 脚本时，使用已配置的编辑器/HBuilderX 格式化能力，不编造命令。

禁止顺带重排整个文件、更新依赖、改 manifest 权限、改应用标识或清理无关代码。

## 9. 完成前检查

-   [ ] 已读 `docs/conclusion.md`，没有重复已记录问题。
-   [ ] 复用了现有 Rice UI、API、类型、工具和 composable。
-   [ ] 没有 Options API、mixin、`this`、DOM 或浏览器 API。
-   [ ] 没有在业务代码直接调用 `uni.request`。
-   [ ] 页面状态包含 loading、empty、error 和重试/恢复路径。
-   [ ] 请求竞态、重复点击和页面返回已考虑。
-   [ ] 布局使用 Flex，文字尺寸使用 px。
-   [ ] 本次新增或修改的 CSS 已核对 App + Vapor 兼容性，装饰位图未使用 URL 型 `background-image`。
-   [ ] App 样式中的 `min-height` 未使用百分比。
-   [ ] App + Vapor 的 CSS、动态样式和组件圆角属性未使用 `rpx`，圆形节点使用 `999px`。
-   [ ] 每个子元素有独立 class；没有样式穿透和关系选择器。
-   [ ] 模板没有依赖构建期生成的原子工具类，样式均能定位到项目 CSS/SCSS 定义。
-   [ ] 定时器、监听、Socket、媒体资源按生命周期释放。
-   [ ] 只格式化和修改了任务相关文件。
-   [ ] HBuilderX/真机验证结果与未验证平台被如实说明。

可按改动范围使用 `rg` 做辅助检查，但必须人工复核命中项：

```powershell
rg -n "mixins:|this\.|window\.|document\.|:deep\(|>>>" pages components composables
rg -n "uni\.request" pages components composables api
```

`utils/request.uts` 内部使用底层请求属于预期；其它命中需要说明原因。

## 10. 复盘闭环

当用户指出结果不满意并要求继续修改时：

1. 先确认具体表现和根因，不把新需求误记为错误。
2. 完成修正并实际验证。
3. 按 `docs/conclusion.md` 模板追加一条可复用结论。
4. 将预防规则写成以后可执行的检查，而不是“下次注意”。
5. 后续相似任务开始前检索关键词并落实已有结论。
