# 仓库指南

## 项目身份与规则优先级

本项目是 **uni-app x + Vue 3 + Vapor + bytecode** 工程，不是传统 uni-app，也不是 uni-app x VDOM 工程。页面和组件使用 `.uvue` 与 `<script setup lang="uts">`，业务类型和工具使用 UTS。

发生冲突时按以下顺序执行：

1. 用户当前需求。
2. `jinnin-app` 当前代码与已确认的后端契约。
3. 本文件和 `docs/uniapp-x-vapor-rules.md`。
4. `docs/` 中的业务参考文档。
5. 关联项目 `../ai-commerce-app-view` 的实现。

开始开发或修复前，必须先阅读 `docs/conclusion.md`，再阅读与任务相关的 `docs/` 分册，避免重复已记录的问题。

## 固定目录职责

-   `pages/`：页面编排、路由参数、页面生命周期和页面级状态。
-   `components/`：本项目公共组件；组件目录使用大写字母开头的 PascalCase，禁止使用 `-` 连接，目录内主组件默认命名为 `index.uvue`；页面私有小组件可放在对应功能目录。
-   `composables/`：跨页面/组件复用的响应式逻辑、分页、缓存和交互状态；文件名统一以 `use` 开头并使用 camelCase，例如 `useVerificationCountdown.uts`。
-   `api/`：接口地址、请求/响应类型和领域 API；禁止在页面散写接口。
-   `utils/request.uts`：全项目唯一通用请求入口；业务代码不得直接调用 `uni.request`。
-   `config/`：环境和服务地址配置。
-   `uni_modules/rice-ui/`：Rice UI 组件库；新增公共组件前先检索并复用 `rice-*`。
-   `static/`、`styles/`：静态资源和全局样式。
-   `docs/`：业务、架构、迁移规则及复盘记录。
-   `unpackage/`：HBuilderX 生成物，禁止手改或提交。

除非调用链、错误日志或最小复现已经确认问题源于组件库自身漏洞，否则禁止修改 `uni_modules/`。页面适配、业务状态和调用时序问题必须优先在 `pages/`、`components/` 或 `composables/` 处理；确需修复 `uni_modules/` 时，修改前必须说明漏洞证据、受影响调用方和跨平台风险，并进行最小范围修改。

## API 契约与响应建模

-   `docs/api-map.md` 是后端完整接口说明。实现或修改接口功能时，必须先定位当前端点的请求和返回字段，再结合已确认的实际响应与关联项目行为确定本次需要的契约；禁止把文档中该模型的全部字段无差别搬入业务类型。
-   API 类型使用后端实际字段名、大小写、可空性和数据类型。不得自行增加 `posterUrl`/`thumbnailUrl`/大小写别名等未确认字段，也不得为猜测后端返回而编写多字段兜底链。已确认的领域字段类型（例如 `workflowData`）不得在无新契约依据时重新建模。
-   禁止仅为类型转换或“保险兼容”在业务层使用 `JSON.stringify` + `JSON.parse` 克隆响应。优先在 API/响应边界标明具名返回类型并直接使用；只有确认存在 UTS 运行时结构转换需求时才允许转换，且必须在代码附近说明原因。

## 逻辑组织与封装规则

-   页面和组件只保留视图编排、生命周期接入与少量直接状态；独立的页面/组件交互逻辑放在对应功能目录的 `composables/`，跨功能复用逻辑放在根目录 `composables/`，无响应式状态的工具函数放在 `utils/`。
-   页面组件拆分以实际职责为准，只有跨处复用、逻辑或排版已经复杂到能明显降低页面理解成本，或承载弹框、下拉、选择器等独立外部交互时，才拆到对应功能目录的 `component/`；仅在单个页面直接展示的静态区块、普通表单和简单信息列表应直接写在页面内，禁止为形式上的组件化额外拆分。
-   简单且只使用一次的逻辑直接写在调用处，不创建只转发一次调用、只返回一个值或没有明确职责的包装函数。
-   仅在逻辑稳定重复、需要独立状态/生命周期或能明显降低调用方复杂度时封装；禁止复制相同倒计时、请求状态、监听清理等逻辑。
-   简单的固定值多分支转换优先使用具名对象映射；需要保存处理函数、动态增删或复杂键值关系时使用 `Map`，禁止用冗长 `switch`/`if` 链表达静态映射。
-   UTS bytecode 中按运行期字符串读取配置时，禁止把普通对象断言为字符串索引字典后动态取值；使用 `Map<string, XxxConfig>` 或显式分支，并保留不依赖动态索引的具名默认配置。
-   禁止空 `catch` 和无操作的 `.catch(() => {})`。仅打印日志不视为业务处理：本层没有状态恢复、降级、错误转换、验证码重置或流程回退等业务动作时不得捕获错误，交给统一请求层或上层调用方；仅需复位 loading/锁状态时使用 `try/finally`。

## Vapor 与 UTS 硬规则

-   只使用 Composition API；禁止 Options API、mixin、`this.xxx` 和 Vue 2 写法。
-   新增或修改的对象类型必须先声明为模块级具名 `type`，类型名必须显式且表达业务语义；禁止匿名 Object Literal Type。
-   `ref`、`reactive`、`Promise`、`Array`、`Set`、`Map`、函数参数/返回值和嵌套对象中都不得直接写 `{ ... }` 类型，嵌套对象也必须拆成独立具名 `type`。
-   API/JSON、页面数据、表单、列表和状态模型优先使用 `type`；按 `XxxParams`、`XxxResponse`、`XxxData`、`XxxItem`、`XxxState`、`XxxForm` 命名。`interface` 主要用于行为接口和 `class implements`。
-   禁止用 `any` 或连续类型断言掩盖类型问题；标准 TypeScript 写法与 UTS 冲突时，以 UTS 编译约束为准。
-   App 共享代码不得使用 `window`、`document`、DOM、CSSOM 或浏览器专属库。
-   原生 Android/iOS 能力优先使用受支持的 `uni.*` API；不足时封装到 UTS 插件，不得散落在页面中。
-   遵循 `onLoad`、`onShow`、`onHide`、`onUnload` 等 uni-app 页面生命周期，不把 `onMounted` 当作等价替代。
-   不照搬传统 uni-app、nvue、renderjs、`plus.*` 或 VDOM 优化方式。

## CSS 与布局硬规则

-   每次新增或修改 CSS 前，必须先核对 DCloud 当前 uni-app x 官方文档中对应属性、属性值和组件在 App + Vapor 下的兼容性；不能仅凭 Web、传统 uni-app 或关联项目写法判断可用。
-   App 样式按受限 CSS 处理；布局统一优先使用 Flex，禁止默认引入 Grid 或浏览器布局技巧。
-   App 原生端不支持 `background-image: url(...)`；本地或远程装饰位图必须使用真实 `<image>` 节点配合绝对定位实现。`background-image` 仅用于官方已确认支持的渐变能力，不得照搬关联项目的 URL 背景写法。
-   App 端 `min-height` 禁止使用百分比；使用编译器支持的固定 `px`/`rpx` 值，或移除最小高度并通过 Flex 完成自适应。
-   App + Vapor 的 `height`、`min-height`、`max-height` 禁止使用 `vh`/`vw` 等视口单位；全屏根容器优先使用 `height: 100%` 或 Flex，占位和上限使用编译器支持的 `px`、`rpx`、百分比或 `auto`。
-   App 端禁止用 `calc()` 将固定间距与安全区相加，也不再使用 `env(safe-area-inset-bottom)`；固定 `padding`/`margin` 使用明确的 `px`/`rpx`，额外安全距离使用独立占位节点并设置 `height: var(--uni-safe-area-inset-bottom)`。
-   App + Vapor 的 `border-radius` 及四角圆角属性不支持 `rpx`，页面样式、动态样式和 Rice UI 圆角属性统一使用明确的 `px`；普通卡片按项目已有的 4px/8px/10px/12px 等层级取值，圆形节点使用 `999px`，不得机械地把圆形半径换成会随屏宽失真的固定半径。
-   横向 `scroll-view` 的直接内容容器必须显式设置 `display: flex` 和 `flex-direction: row`，不得依赖平台默认排列方向。
-   新增或修改的 `scroll-view` 必须使用 `direction="horizontal"` 或 `direction="vertical"` 明确滚动方向，禁止继续使用 `scroll-x`、`scroll-y` 布尔属性；横向容器仍须遵守直接内容容器显式横排规则。
-   文字尺寸统一使用明确的 `px`；修改 Rice UI 文字尺寸属性时也优先传 `px` 值。
-   所有需要样式的节点都使用明确 class，不依赖标签选择器、父容器继承或样式穿透。
-   禁止依赖后代/子代选择器：将 `.parent .child `、`.parent > .child` 改为独立类 `.parent_child`，例如 `.card_title`。
-   禁止默认使用 `:deep()`、`>>>`、伪元素、结构选择器或复杂组合选择器；组件样式必须自包含。
-   `.uvue` 允许使用 `<style scoped lang="scss">` 组织样式；可使用 `.order-card { &__header { ... } &--disabled { ... } }` 这类 BEM 嵌套编写，编译后的选择器仍必须是独立扁平 class，不得借助 SCSS 产生后代、子代或复杂组合选择器。
-   当前项目不使用 UnoCSS、Tailwind 等原子样式生成器；模板使用具有组件或业务语义的明确 class，具体样式写入对应 `.uvue` 的局部 CSS/SCSS 或已有公共样式文件。
-   禁止重新引入 `virtual:uno.css`、`uno.config.*`、UnoCSS Vite 插件及相关依赖；除非用户明确要求重新评估样式体系，不得新增 `flex`、`w-full`、`bg-*` 等依赖构建期生成的工具类。
-   复杂动画、层叠、原生组件覆盖和平台敏感样式使用独立局部 class 并真机验证。
-   不用 `!important`、超大 `z-index` 或任意延时掩盖结构问题。

## 关联项目只读参考规则

`../ai-commerce-app-view` 是传统 uni-app Vue 3/TypeScript 项目，只用于参考业务逻辑、接口字段、页面视觉、用户交互、Socket、状态机和功能流水线。

-   除非用户明确要求，否则禁止在参考项目中新增、修改、删除、格式化文件或处理其未提交改动。
-   迁移目标是“行为一致”，不是“代码一致”；所有实现必须落在当前仓库并符合 Vapor 架构。
-   不按文件复制。先追踪旧页面依赖，再整理初始化、用户操作、数据流、生命周期和 loading/empty/error 等状态。
-   禁止复制 `.vue`、`.nvue`、renderjs、`plus.*`、uView Pro、z-paging、旧 UnoCSS 产物或生成配置。
-   默认保留当前项目已有 UI；当用户明确指定关联项目为视觉基准时，必须保持对应页面的结构层级、间距、字号、颜色、资源、控件尺寸和交互位置在可感知结果上一致，禁止自行增加标题、说明、卡片或重新编排视觉层级。
-   “展示一致”不等于复制源码：实现仍须使用当前项目的 `.uvue`、UTS、Rice UI、Pinia、请求层和 Vapor 兼容 CSS；因平台或组件能力产生差异时，优先保证可感知结果一致并记录真机验证点。
-   API URL、method、参数、响应、分页和状态枚举必须与当前 API/后端契约复核，不因旧项目存在某字段就直接采用。

## 实施流程

1. 阅读 `docs/conclusion.md`、`docs/uniapp-x-vapor-rules.md` 和相关业务分册；涉及接口时同时查阅 `docs/api-map.md` 中对应端点。
2. 搜索当前页面、组件、composable、API、类型、工具和 Rice UI，明确可复用项。
3. 涉及参考项目时先形成简短行为映射，再确定目标文件与 Vapor 替代方案。
4. 进行最小范围修改，不重构无关代码、不新增重复能力、不修改依赖或生成文件。
5. 使用根目录 `.prettierrc` 格式化本次修改文件，禁止顺手格式化全仓库。
6. 检查 UTS 具名类型、请求竞态、生命周期释放、CSS 兼容和平台条件。
7. 在 HBuilderX 运行受影响平台；原生、媒体、权限和平台敏感样式必须真机验证，Web 通过不代表 App 通过。

项目当前只有 `pnpm install` 可用于安装依赖；`pnpm test` 是失败占位脚本，且没有受支持的命令行构建脚本。运行和发行使用 HBuilderX，不得虚构验证结果。

本次任务临时新增的验证脚本，在本轮验证执行结束后必须立即删除，无论验证通过还是失败；仅在完成报告或复盘文档中记录实际执行结果，不把临时脚本留在仓库或提交。清理范围仅限本次自行创建的临时验证脚本及其临时产物，不得删除项目原有测试、工具脚本或用户文件；用户明确要求保留时除外。

`.uvue` 模板遵循根目录格式化配置：在 `printWidth` 范围内属性保持同一行，不启用单属性强制换行；必须换行时属性按同一缩进对齐，开始标签的 `>` 或 `/>` 不单独拆成一行。禁止手工排版出与格式化配置冲突的标签结构。

## 复盘记录

当用户明确表示本次结果不满意并要求继续修改时，在问题修正后、最终回复前，将本次问题追加到 `docs/conclusion.md`。记录任务、表现、根因、正确处理、预防规则和验证结果；普通迭代、需求新增或尚未确认根因时不要机械记录。遇到类似任务必须先检索已有结论并落实其预防规则。

## 完成报告

最终说明：改了什么文件、行为如何变化、参考项目哪些行为被保留、哪些旧实现被替换、实际执行了哪些检查、哪些平台仍需真机确认。若目录无 Git 元数据，应明确无法提供 Git diff 统计，不得编造。
