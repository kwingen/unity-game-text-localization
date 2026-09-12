---
name: unity-game-text-localization
description: Extract Japanese text from Unity games (including adult titles) and produce a playable Chinese localization: structure analysis, corpus extraction, local Sakura translation with strict validation, asset back-fill into an independent overlay, a BepInEx runtime loader, and style-matched font adaptation. Use for end-to-end Unity text extraction/localization work, not for ordinary game modding.
metadata:
  short-description: Unity 游戏文本提取与本地化（提取→本地翻译→回填→字体）
---

# Unity 游戏文本提取与本地化

面向 Unity 引擎游戏（含 DLsite 成人作品）的端到端本地化：结构分析 → 文本提取 → 本地 Sakura 翻译 → 严格校验 → 资源回填（独立 overlay）→ BepInEx 加载器 → 隔离副本验证。

## 核心约束

- **原游戏目录只读**：所有产出写入独立目录；回填只生成 overlay，不覆盖原文件。
- **离线预翻译**：游戏运行时不调用模型；译文全部提前烘焙进资源 + 运行时静态字典。
- **成人内容只对用户明确授权的目标操作**；不把特定作品信息写成通用规则。

## 工作流概要

1. **结构与文本定位**：UnityPy 枚举资产对象（MonoBehaviour/TextAsset/Utage），确认 Unity 版本与类型树兼容性，定位日文文本字段。
2. **提取与建语料**：导出结构化 JSON；语料行 = `{id: sha256(text), text, category, locations[]}`，**id 由原文重算、不做归一化**；去重并聚合全部来源位置；保留格式控制（标签/参数/`/`分隔符/换行）。
3. **本地翻译**：llama-server + Sakura 模型，严格校验管线（占位符保护、行数/顺序、prompt 泄漏拒绝、segment fallback）。**先小批试译校准术语表再全量**。
4. **校验与收尾**：结构/质量审计；剩余顽固条目**直接人工/Agent 收尾**，不重复烧模型时间。
5. **回填与加载器**：独立 overlay（源只读、哈希校验、未改对象字节不变）；BepInEx 插件做 UGUI/TMP 字体 + 运行时精确字典；Utage 剧情只改字体不替换文本。
6. **验证**：隔离副本冒烟（日志/取证/截图），原游戏未动；状态写 `pending_full_playthrough`。

## 关键决策（踩坑结论）

- **模型分工**：规划/结构分析/提取用强推理模型效果好；但**成人内容会触发在线模型过滤**，正文翻译必须走本地 Sakura。
- **术语表**：长词优先匹配，且把 `ダウンロード/プリロード/リロード` 等长词显式入表，否则短词（如 `ロード→读取`）污染长词。
- **翻译收尾**：剩余十几条顽固条目（话痨截断、短串边界、降级丢术语）人工/Agent 收尾，别浪费 Sakura 时间与 token。
- **字体**：先识别原字体风格（常见丸ゴシック→中文圆体如幼圆），GDI 做覆盖率预检防 □；TMP 运行时创建字体用三参 `CreateFontAsset(familyName, styleName, pointSize)` 重载（8 参/DynamicOS 在 Player 返回 null）。

## 详细资料

- [端到端工作流](references/workflow.md)：提取、语料、翻译管线、回填、加载器、验证的完整做法与代码模式。
- [踩坑记录](references/pitfalls.md)：模型选择与过滤、Sakura 问题、收尾策略、字体适配的具体教训。
