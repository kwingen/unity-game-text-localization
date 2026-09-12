# 通用端到端工作流

## 0. 交付范围

先确认产物，不要默认“一步到位”：

| 用户请求 | 必要产物 | 禁止事项 |
|---|---|---|
| 只要文本 | 结构化导出 + 覆盖率报告 | 不改游戏文件 |
| 要翻译 | 双语语料 + QA 报告 | 不自动回填 |
| 要可玩补丁 | overlay/补丁包 + 验证记录 | 不覆盖原目录 |
| 要运行时方案 | 加载器/插件 + 隔离副本验证 | 不绕过保护机制 |

## 1. 构建盘点

产出 `inventory.json`，至少包含：

- 游戏路径、平台、构建时间或版本号
- Unity 版本
- Mono / IL2CPP / WebGL / 其他脚本后端
- 关键目录：`*_Data`、`StreamingAssets`、`Managed`、`Resources`、Addressables、AssetBundle
- 已有本地化系统、Mod 加载器、运行时补丁框架
- 字体系统和 UI 框架线索
- 初步风险：加密资源、反作弊、动态文本、图片文本

不要跳过盘点直接修改资源。若无法确定脚本后端或资源格式，先做只读检查并记录不确定项。

## 2. 文本源发现

按 [text-sources.md](text-sources.md) 枚举所有可能来源，至少覆盖：

- 原生 Unity Localization / Addressables
- CSV、JSON、XML、YAML、PO、XLIFF 等外部表
- `TextAsset`
- `MonoBehaviour` / `ScriptableObject`
- UI 组件中的默认文本
- 代码字符串和动态拼接
- AssetBundle、Resource、StreamingAssets
- 图片、音频、视频中的固定文本

每类来源都记录：文件、对象 ID、字段路径、文本语言、上下文、是否可写、回填难度。

## 3. 语料构建

推荐 JSONL，一行一条语料：

```json
{
  "id": "stable-hash",
  "source": "exact original text",
  "context": {
    "speaker": null,
    "scene": null,
    "component": null,
    "character_limit": null,
    "notes": null
  },
  "locations": [
    {
      "file": "path",
      "path_id": 123,
      "field": "dialogue.text",
      "container": "asset or code"
    }
  ],
  "placeholders": ["{0}", "<color>"],
  "status": "untranslated"
}
```

规则：

- 原文按精确字节保存；生成 ID 前不做 Unicode 归一化。
- 相同原文可合并，但必须保留全部来源位置。
- 保留标签、占位符、分隔符、换行、转义和大小写。
- 为 UI 文本记录长度、行数、自动换行和字体约束。
- 区分玩家可见文本、调试文本、资源名、着色器名和内部标识符。

## 4. 翻译与质量校验

按 [translation.md](translation.md) 执行：

1. 建术语表和风格说明。
2. 选择翻译提供方：人工、本地模型、在线模型或已有翻译记忆。
3. 小批量试译，校准风格和术语。
4. 全量翻译。
5. 自动校验占位符、行数、标签、术语一致性和目标语言规范。
6. 失败条目分类处理，不做无限重试。

## 5. 回填策略选择

优先级如下：

1. **原生本地化表**：游戏已有本地化系统时，优先新增或修改 locale。
2. **外部数据**：直接补丁 JSON/CSV/XML 等文本数据。
3. **序列化资产**：用 UnityPy、UABEA 或 AssetRipper 生成 overlay。
4. **运行时 Hook**：数据不可安全修改时，用 BepInEx/MelonLoader/Harmony 挂 UI 或本地化接口。
5. **媒体文本**：图片/音视频文本单独处理，不混入普通文本补丁。

详细做法见 [backfill.md](backfill.md)。

## 6. 验证

在隔离副本中验证：

- 原目录与交付前哈希一致
- overlay 清单与实际文件一致
- 未修改对象字节不变
- 关键场景截图或取证日志
- 字体缺字、UI 溢出、换行异常
- 占位符和标签未被破坏
- 游戏可启动且无新增致命错误

不要把冒烟验证表述为完整通关验证。

## 7. 交付物

至少提供：

- `inventory.json`
- `corpus.jsonl`
- `glossary.json`
- `translations.jsonl`
- `qa-report.json`
- `patch-manifest.json`
- 验证日志与截图索引
- 安装/回滚说明

交付说明必须列出未覆盖文本源、未验证场景和已知风险。
