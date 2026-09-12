# Unity 文本源盘点

## 1. 构建形态

先判断脚本与资源形态：

| 形态 | 常见线索 | 影响 |
|---|---|---|
| Mono | `Assembly-CSharp.dll`、`Managed/` | 可反编译，适合静态/动态结合 |
| IL2CPP | `GameAssembly.dll`、`global-metadata.dat` | 需元数据恢复，运行时 Hook 更常见 |
| WebGL | `.data`、`.wasm`、压缩资源 | 平台限制多，常需专用工具 |
| Android/iOS | APK/IPA、StreamingAssets、AssetBundle | 注意签名与安装限制 |
| 编辑器工程 | `Assets/`、`ProjectSettings/` | 优先改源文件，不做二进制补丁 |

## 2. 常见文本源

### 原生 Unity Localization

- 线索：`Unity.Localization`、`SharedTableData`、`StringTable`、`AssetTableReference`、Addressables catalog
- 优先策略：新增目标 locale 或修改字符串表，而不是硬编码替换
- 工具：Unity 编辑器、UnityPy、Addressables 分析工具

### 外部数据表

- 线索：CSV、TSV、JSON、XML、YAML、PO、XLIFF
- 常见位置：`StreamingAssets/`、`Resources/`、AssetBundle 内 `TextAsset`
- 优先策略：保持格式不变，按字段级替换；不要把整个文件重排导致 diff 不可读

### 序列化对象

- 线索：`MonoBehaviour`、`ScriptableObject`、自定义 dialogue/story data
- 需要类型树；没有类型树时先用 AssetRipper 或 TypeTreeGenerator 辅助
- 记录 `file + path_id + field path`

### AssetBundle / Addressables

- 检查 catalog、依赖、压缩方式、Unity 版本
- 不要只改一个 bundle；依赖和哈希可能导致加载失败
- 无法安全重建时，改用运行时 Hook

### UI 组件

- 覆盖 `Text`、`TextMeshPro`、UGUI、自定义渲染器
- 记录组件默认文本、字体、行宽、自动缩放、富文本设置
- 动态赋值文本通常在代码或数据表中，不在组件默认值里

### 代码字符串

- Mono：ILSpy / dnSpy / dotPeek / `strings`
- IL2CPP：Il2CppDumper / Il2CppInspector / Cpp2IL
- 优先定位“生成最终显示文本”的函数，而不是盲目替换所有字符串

### 动态与服务端文本

- 拼接、格式化、随机文本、任务系统、网络返回值
- 若文本由服务器下发，离线补丁无法覆盖；应明确说明限制

### 媒体文本

- 图片字幕、烘焙 UI、语音、视频
- 不属于普通文本补丁；需要图像编辑、OCR 或音视频字幕流程

## 3. 发现流程

1. 生成文件清单与对象类型分布。
2. 识别本地化系统和资源加载器。
3. 用语言规则筛选候选字段，而不是只按字段名猜测。
4. 对候选对象导出结构化索引。
5. 对代码与资源做交叉引用，找出显示入口。
6. 标记不可达或调试文本，不默认翻译。
7. 输出覆盖率矩阵：已确认、待确认、不可达、媒体文本。

## 4. 编码与格式

- 不要假设 UTF-8；检测 UTF-8、UTF-16、平台默认编码或自定义编码。
- 保留 CRLF/LF、真实换行和字面 `\n` 的区别。
- 富文本标签、参数占位符、分隔符、转义序列都要进入校验清单。
- 对同一原文在不同上下文出现时，保留独立上下文而非强行合并含义。
