# UTS 类型、模板与运行产物

以下是来源项目的保守写法及真实故障；兼容性绕过点需按目标编译器复核。

## 直接具名模型

数据、表单、Prop、请求和响应优先用模块级直接具名 `type`。嵌套对象也独立命名，函数参数/返回值、集合和 Promise 中不内嵌匿名对象类型。

```uts
export type MediaSourceData = {
  url: string;
  coverUrl: string | null;
};

export type AssetItem = {
  id: string;
  source: MediaSourceData;
};

const items = ref<AssetItem[]>([]);
```

上例只展示类型组织，字段是假设的示例契约，不能据此给实际后端增加 `source` 或 `coverUrl`。

对于初始化数组，优先 `const options: XxxItem[] = [...]`，避免每个字面量重复 `as XxxItem`。本项目曾出现 import type 引入的泛型派生类型在运行时未定义，以及交叉别名存在 export 但调用方 UTS 导出失败；修复为直接声明类型，不以更多断言掩盖。

## 动态配置

bytecode 历史案例中，将普通对象断言为字符串字典后动态索引可能读不到配置；兜底又用相同索引也会为空。采用 `Map<string, ModeConfig>` 并保留具名默认常量。

```uts
type ModeConfig = {
  title: string;
};

const defaultMode: ModeConfig = { title: "默认模式" };
const modes = new Map<string, ModeConfig>();
modes.set("default", defaultMode);

function getMode(key: string): ModeConfig {
  return modes.get(key) ?? defaultMode;
}
```

简单、固定、编译期已知的字段映射可使用具名对象；这里的 Map 是为运行期字符串检索选择，不要求把所有对象改成 Map。

## 空数据与桥接边界

类型声明不能保证运行时响应具有该字段。对外部数据，在 API 或原生状态入口识别 `null`、缺失、空数组、字符串布尔和合法零值，再向视图提供稳定模型。

- `amount.toFixed` 前校验空值和有限数；零值不能被 `||` 吞掉。
- 输出对象数组先验证地址为字符串，再读 `.length`。
- `for...of`、`map`、`reduce` 的源必须为已初始化数组。
- Prop 为数组时设置工厂默认值；默认值不能替代对桥接显式 null 的检查。
- `unknown` 用在边界，收窄后再进入领域；不要让全页面变成动态 JSON 访问器。
- 已确认的旧基座缺字段兼容只能写在转换入口，并说明版本和删除条件；不写猜测性的多字段兜底链。

## Prop 不是只有类型

出现“父组件已传、类型有字段、模板却 undefined”时，依次检查：

1. 模板是否读的是已定义绑定；新增字段优先明确读取 `props.xxx`。
2. 默认值、父层传参和中间组件转发是否完整。
3. 当前运行产物的 runtime `props` 表是否真的包含字段。
4. import type、跨文件类型展开或增量缓存是否造成运行时声明缺失。

历史 `selected` 字段问题通过把单组件 Prop 类型移到组件内直接声明修复。它是该问题的有证据修复，不代表所有共享 Prop 都应内联。生成代码只读，不手补 `props` 表。

## 插槽与结构

本项目 bytecode 曾在命名插槽外壳同节点组合 `v-if` 时降级失败。保守写法是静态插槽外壳，条件放真实内容节点。

```vue
<template #right>
  <view v-if="managing" class="toolbar_actions">
    <text class="toolbar_label">完成</text>
  </view>
</template>
```

不要为一个装饰箭头生成依赖未就绪数量的 `v-for`；可见数量有限时使用单节点和明确位置类。复杂循环内局部变量在 bytecode 运行时异常时，先复现作用域路径，再考虑抽成具名纯函数；不要未经证据归因于 SDK。

## 原生插件编译

UTS 插件公共代码可能生成 Kotlin、Swift 等目标代码，与 Vapor 页面逻辑层的行为不是同一问题。循环显式写块；异常传播要核对生成 Swift 的 `throws`。已有具名 error 返回契约时可在该边界返回错误并清空成功字段，不以删除校验解决编译错误。

典型检索：`交叉类型`、`运行时 props`、`条件命名插槽`、`frames`、`throws`，详见案例目录。
