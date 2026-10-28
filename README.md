# uni-app x Vapor 开发知识包

整理日期：2026-10-28。来源：核对 DCloud 官方文档。面向后续使用 **uni-app x + Vue 3 + Vapor + bytecode** 快速开发、迁移和排错。

## 从哪里开始

| 使用目的                                 | 阅读入口                                                                                                                |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| 了解技术栈、收益、风险与实施顺序         | [总结文档](总结文档.md)                                                                                                 |
| 开发前查经验、遇到问题按现象定位         | [经验文档](经验文档.md)                                                                                                 |
| 给 Codex 或其他支持 skills 的 AI 使用    | [uniapp-x-vapor/SKILL.md](uniapp-x-vapor/SKILL.md)                                                                      |
| AI 按需检索，节省上下文                  | [llms.txt](llms.txt)                                                                                                    |
| 一次性导入整理后的完整正文               | [llms-full.txt](llms-full.txt)                                                                                          |
| 搜索全部历史问题、日期、验证结果与源文件 | [完整案例目录](uniapp-x-vapor/references/case-catalog.md) 与 [knowledge-index.json](knowledge-index.json)               |
| 复核来源、文件指纹及本次检查             | [来源说明](uniapp-x-vapor/references/project-evidence.md)、[sources.json](sources.json)、[VALIDATION.md](VALIDATION.md) |

## Skill 使用

`uniapp-x-vapor/` 是可独立复制的 skill 文件夹，包含入口、AI 展示元数据、按场景拆分的参考资料及原始证据快照；不依赖本机已有的其他 skill。

在支持 Codex skills 的环境，将整个文件夹复制到用户的 skills 目录（例如 Windows 的 `%USERPROFILE%/.codex/skills/uniapp-x-vapor`），按宿主要求刷新技能发现。此知识包仅输出到指定目录，没有自动安装或覆盖当前已有技能。

调用示例：

```text
使用 $uniapp-x-vapor，在当前项目实现带筛选、刷新、分页的素材列表。
先确认渲染模式、工具链与目标平台，读取对应经验；复用现有请求层和列表组件。
```

也可以直接给 AI 读取本目录的 `llms.txt`。相对链接都以所在文件为基准；离线工具应读取文件路径，而不是将它们当公网 URL。

## 适用边界

- “通用方法”用于其他 Vapor 工程；“本项目约定”仅在目标工程采纳后生效。Rice UI、目录名、全局鉴权和后端业务字段不是 DCloud 标准。
- “历史问题”保留原验证边界。记录中的“模拟通过”“静态通过”“待真机”没有在本次整理中升级为平台通过。
- 最新官方能力与旧项目避坑可能存在差异。先核对 HBuilderX、基座、系统、渲染模式，再决定是否继续使用绕过方案。
- 当前工作区没有根 `package.json` 或可确认的 CLI 构建/测试脚本；不能从旧说明推导出现在可执行 `pnpm install/test/build`。
- 当前剪辑 provider 为 `none`。美摄接入经验不代表本次快照已启用美摄，也不代表 iOS/鸿蒙具备相同 SDK 实现。

## 维护

保持分场景参考资料为正文事实来源，经验文档负责定位，总结文档负责整体判断。新问题先明确根因与验证结果，再补充案例。升级工具链时复测历史绕过点并记录替代状态。

`llms-full.txt` 是正文聚合快照，`knowledge-index.json` 是原始复盘的结构化索引；更新正文或原始记录后同步更新，不能只改聚合文件。原始证据保留历史，不直接作为新工程的强制指令。
