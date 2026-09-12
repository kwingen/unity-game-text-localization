# 端到端工作流

## 0. 环境与工具

- Python 3.11+ 虚拟环境；`UnityPy`（1.25.x）+ `TypeTreeGeneratorAPI` 用于资产解析与类型树。
- `llama-server`（CUDA 构建）+ Sakura 模型（如 `sakura-14b-qwen2.5-v1.0-iq4xs.gguf`），OpenAI 兼容端点仅限 `127.0.0.1` loopback；不用代理、不下发模型、不连第三方服务。
- 游戏隔离副本用于验证；原游戏目录只读。

## 1. 结构与文本定位

1. 列出 `*_Data` 资产与根目录文件（`globalgamemanagers`、`level0`、`sharedassets0.assets`、`resources.assets`、`StreamingAssets`、`Managed` DLL、BepInEx 残留）。
2. 用 UnityPy 枚举对象类型分布（MonoBehaviour/TextAsset/GameObject/Utage 相关），统计含日文字段的资产文件与对象数，确定文本载体。
3. 确认 Unity 版本；类型树与运行时不匹配时用 `TypeTreeGeneratorAPI` 生成树。

**类型树兼容坑（Unity 2021.1.x + UnityPy 1.25.3）**：
- 生成器的 MonoBehaviour `m_Enabled` 字段缺四字节对齐标记；
- `List<string>` 被标记成标量 `string`。
- 对策：只在内存中修正这两处；用「原样字节往返」测试验证真实对象无损。

## 2. 提取与建语料

- 导出结构化 JSON：每个对象 → `{file, path_id, class, name, fields}`；另建 `index.json`（对象清单）与 `coverage.json`。
- 语料行：`{id: sha256(text), text: 精确原文, category, locations: [{file, path_id, field 路径, mode}]}`。**id 必须由原文重算**，不做空白/转义/Unicode 归一化。
- 去重：相同文本合并，`locations` 聚合全部来源位置，支持逐处回填与追踪。
- 保留格式控制：富文本标签 `<color=...>`、参数占位符 `<param=...>`、`{...}` 花括号、`\n` 字面与真实换行、`/` 分隔符——翻译时必须原样保留（占位符保护）。

## 3. 本地 Sakura 翻译管线

### 启动与探活
- 启动参数建议：`--ctx-size 4096 --parallel 1 --n-gpu-layers <按显存> --flash-attn on --offline`。
- **先探测真实延迟再规划批量**：`--parallel 1` 单槽，一个慢请求（如 90s+）阻塞后续全部；测速要把排队算进去，别把排队误判成模型限速/网络故障。

### 请求与校验（不可省略）
- 单条或小批量（≤4）请求，`temperature 0.1` 附近，显式 loopback endpoint。
- 占位符机制：格式控制与术语表词替换为 `[[Pxxxx_xxxx]]`；译文必须原样保留全部占位符（顺序、多重性、行数、行序）。
- 校验拒绝：占位符丢失/乱序/增删、行数不一致、代码块、JSON 包裹、解释性前缀、prompt 回显（leakage，如「将下面的日文文本翻译成中文：」）。
- 降级路径：整条校验失败时，仅对「无保护的纯文本段」逐段重译（segment fallback），**绝不接受丢弃标记的整条响应**；fallback 段再次校验术语身份/顺序/多重性。
- 断点续跑：输出 append-only JSONL，逐条 fsync；已有记录校验后跳过；`glossary_sha256` 绑定每条记录（**改术语表会让旧断点全部失效**，改表前先规划重跑范围与合并策略）。

### 术语表
- JSON 对象 `{日文词: 中文词}`；**按词长降序匹配**，最长优先保护。
- 短词必须谨慎：`ロード→读取` 会截断 `ダウンロード/プリロード/リロード`。把这些长词显式入表（`ダウンロード→下载`、`プリロード→预加载`、`リロード→重新加载`）。
- 校验：空词、含控制语法的词拒绝；术语命中计数用于质量审计。

## 4. 校验与收尾

- 合并多轮输出按优先级覆盖（人工校订 > 最新模型轮 > 旧模型轮），每条做结构与来源校验（标签/占位符/分隔符/行数）。
- 质量审计项：未翻译（原文==译文）、残留假名、prompt 泄漏、人名不一致、术语命中一致性、结构标记数量。
- **收尾策略**：剩余个位数到十几条顽固失败直接人工/Agent 校订（curated entries），结构校验照旧。分类决策：术语污染类 → 改术语表后重跑该子集；话痨/短串/降级丢术语类 → 直接校订。

## 5. 回填与加载器

### 资源回填（独立目录）
- 只从原游戏目录读取，输出到独立 overlay 目录；**禁止输出指向原游戏**。
- 每个改动位置先验证原文匹配再写译文；写后重新读回校验；未修改对象保持原字节一致（SHA256）。
- 生成 `patch-manifest.json`（文件级 source/patched SHA256、对象改动数、替换位置数）与运行时静态字典（base64(原文)\tbase64(译文)）。

### BepInEx 插件
- 职责：运行时 UGUI 文本精确替换（字典按原文精确匹配，无正则/无部分匹配/无插值）+ 字体替换 + 可选取证。
- Utage 剧情文本：资源已烘入译文，插件**只替换字体、不做运行时文本替换**（避免 typewriter 状态冲突）。
- 字典格式：UTF-8，首行版本头（如 `# krone-offline-zh-v1`），数据行 `base64(UTF8(原文))\tbase64(UTF8(译文))`；整文件 fail-closed（坏行/重复冲突源拒载，不部分导入）；16 MiB 上限。
- 配置项独立（General/Enabled、Fonts/UguiFontName、Fonts/RoundedTmpFontName、Evidence/DumpVisibleText）。

## 6. 验证（隔离副本）

- 复制完整游戏到隔离目录，部署 overlay + 插件 + 字典 + 字体包（保持原目录未动）。
- 冒烟：启动后查 BepInEx 日志（字典条数、UGUI/TMP 字体加载、钩子数、缺字/错误 0）、插件取证 `visible-ui.json`（可见中文文本）、截屏确认风格。
- 部署前核对隔离副本资产与补丁基线哈希一致（可回滚）；先关游戏再替换 DLL。
- 验收语言：冒烟通过 ≠ 全流程通关。交付状态写 `machine_translated_overlay_pending_review` / `pending_full_playthrough`。
