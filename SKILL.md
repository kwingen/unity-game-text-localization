---
name: unity-game-text-localization
description: Generic, agent-neutral workflow for extracting, translating, and back-filling text in Unity games across Mono and IL2CPP builds, localization tables, asset bundles, Addressables, UI components, code strings, and fonts. Use for any Unity text-localization task; not for non-Unity games or unrelated modding.
metadata:
  short-description: Unity 游戏文本提取与本地化通用流程
  keywords:
    - unity text extraction
    - unity localization
    - unity game translation
    - unity asset patching
    - unity il2cpp text
---

# Unity 游戏文本本地化（通用版）

适用于任意 Unity 游戏的文本提取、翻译与回填。不要假设特定引擎版本、平台、语言对、文本存储方式或运行时方案；先盘点构建，再按证据选择策略。

## 必守约束

- 只处理用户明确授权的游戏副本；不得绕过 DRM、反作弊、加密或混淆。
- 原游戏目录只读。所有修改写入独立 overlay、补丁包或隔离副本。
- 不要只按 `TextAsset`、某个插件或某个本地化系统猜测；先做完整文本源盘点。
- 保留格式控制、占位符、换行、富文本标签、参数和行序；译文不得破坏结构。
- 任务范围要收敛：只要提取就不回填，只要译文就不改游戏，只要补丁就不额外发布。
- 游戏文本包含敏感内容时，未经用户明确许可不得发送到在线模型或第三方服务。

## 最小输入

能从上下文推断就不要反复询问；只确认缺失且影响结果的信息：

- 游戏路径或构建目录
- 源语言与目标语言
- 期望产物：文本导出、双语语料、可安装补丁、隔离副本验证
- 是否允许运行时 Hook、是否允许在线翻译

## 主流程

1. **构建盘点**：确认 Unity 版本、平台、Mono/IL2CPP、资源布局、已有本地化系统、字体与加载器。
2. **文本发现**：系统枚举本地化表、外部数据、序列化资产、AssetBundle/Addressables、UI 组件、代码字符串、动态文本和图片/音视频文本。
3. **语料构建**：导出原文、上下文、精确位置、格式标记和 UI 约束；去重但保留所有来源。
4. **翻译与校验**：使用用户许可的翻译方式，建立术语表、翻译记忆和结构校验。
5. **回填策略**：优先原生本地化表，其次数据/资产补丁，最后运行时 Hook；字体和 UI 布局同步适配。
6. **验证交付**：在隔离副本中验证哈希、日志、截图和覆盖率；明确未验证范围。

## 资料路由

- 开始任何修改前读 [references/workflow.md](references/workflow.md)。
- 做文本源盘点时读 [references/text-sources.md](references/text-sources.md)。
- 做翻译、术语或质量校验时读 [references/translation.md](references/translation.md)。
- 做补丁、字体、运行时 Hook 或打包时读 [references/backfill.md](references/backfill.md)。
- 遇到反直觉问题、构建差异或验证失败时读 [references/pitfalls.md](references/pitfalls.md)。

## 跨 Agent 兼容

本技能只依赖通用文件读写、Shell、代码编辑和 Git 能力，不绑定 Codex、Claude 或 Hermes 的私有 API。不同宿主应按自身工具执行同一流程；引用文件使用相对路径。除 `name`、`description` 外的 frontmatter 元数据可被不支持的宿主忽略。
