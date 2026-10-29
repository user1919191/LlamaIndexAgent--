# CNKI Multi-Domain Full-Text Test Corpus (Annotated Index)

计算机领域多方向**全文测试集候选文献索引**。本仓库仅包含文献**题录、检索链接与自定义标注**，**不包含知网全文 / PDF**。适用于科学声明验证、全文检索、证据定位等系统的工程测试与内部评估。

---

## 1. 仓库结构

```text
.
├── README.md
└── datasets/
    ├── agent.json              # Agent（智能体）
    ├── rag.json                # RAG（检索增强生成）
    ├── vector_database.json    # 向量数据库
    ├── multimodal.json         # 多模态
    └── neural_network.json     # 神经网络
```

| 文件 | 领域 | `domain` 字段值 | `domain_cn` |
|---|---|---|---|
| `agent.json` | Agent / 智能体 | `Agent` | 智能体 |
| `rag.json` | 检索增强生成 | `RAG` | 检索增强生成 |
| `vector_database.json` | 向量数据库 | `Vector Database` | 向量数据库 |
| `multimodal.json` | 多模态 | `Multimodal` | 多模态 |
| `neural_network.json` | 神经网络 | `Neural Network` | 神经网络 |

每个 JSON 对应 **1 篇标准参考文献（`standard`）+ 5——10 篇测试文献（`papers`）**。

---

## 2. 通用 JSON 结构

五个文件共享同一套顶层结构与字段语义：

```json
{
  "domain": "Neural Network",
  "domain_cn": "神经网络",
  "description": "……",
  "standard": { /* 单篇文献对象 */ },
  "papers": [ /*  多个文献对象 */ ]
}
```

### 2.1 顶层字段

| 字段 | 类型 | 说明 |
|---|---|---|
| `domain` | `string` | 领域英文标识，用于程序过滤与分组。 |
| `domain_cn` | `string` | 领域中文名称，便于人工阅读。 |
| `description` | `string` | 该文件的简要说明，注明“仅含标注、标题与链接”。 |
| `standard` | `object` | **标准参考文献**。每个领域选定 1 篇，作为该领域测试基准或对照文献。 |
| `papers` | `array<object>` | **候选文献列表**。每个领域 5-10 篇，与 `standard`。 |

> `standard` 与 `papers` 中的对象结构完全一致，区别仅在语义：`standard` 用于锚定基准，`papers` 用于扩充测试样本。

### 2.2 单篇文献对象字段

```json
{
  "id": "nn-10",
  "annotation": {
    "domain": "Neural Network",
    "doc_type": "网络首发",
    "intended_use": "标准参考文献/全文测试集基准"
  },
  "title": "轻量化目标检测模型探究",
  "link": "https://kns.cnki.net/kns8s/defaultresult/index?kw=轻量化目标检测模型探究"
}
```

| 字段 | 类型 | 说明 |
|---|---|---|
| `id` | `string` | **文献唯一标识**。格式为 `领域前缀-两位序号`，如 `nn-10`、`rag-01`、`vecdb-04`。前缀含义见下表。序号在同一文件内唯一，**不因 `standard`/`papers` 位置而变化**，可用于跨文件引用与去重。 |
| `annotation` | `object` | **自定义标注块**。描述该文献在测试集中的角色与元信息，供程序读取。详见 2.3。 |
| `title` | `string` | **文献标题**。中文标题保持原样；英文标题或中英混排标题保留原文，便于在知网检索。 |
| `link` | `string` | **检索链接**。当前为知网标题检索 URL 占位，用于定位文献详情页。**建议发布前替换为实际详情页 URL 或 DOI**。 |

#### `id` 前缀对照

| 前缀 | 对应文件 | 示例 |
|---|---|---|
| `agent-` | `agent.json` | `agent-01` ~ `agent-11` |
| `rag-` | `rag.json` | `rag-01` ~ `rag-11` |
| `vecdb-` | `vector_database.json` | `vecdb-01` ~ `vecdb-11` |
| `mm-` | `multimodal.json` | `mm-01` ~ `mm-11` |
| `nn-` | `neural_network.json` | `nn-01` ~ `nn-11` |

### 2.3 `annotation` 字段说明

| 字段 | 类型 | 取值示例 | 说明 |
|---|---|---|---|
| `domain` | `string` | `"Neural Network"` | 与顶层 `domain` 一致，冗余存储，方便单条记录独立使用。 |
| `doc_type` | `string` | `"期刊论文"`、`"网络首发"`、`"会议论文"`、`"期刊专题"` | 文献类型。`网络首发` 表示知网网络首发，卷期信息可能尚未最终确定；`期刊专题` 表示来自期刊专题栏目（如《软件学报》向量数据库专题）。 |
| `intended_use` | `string` | `"标准参考文献/全文测试集基准"` 或 `"全文测试集候选"` | 该文献在测试集中的预期用途。`standard` 中固定为“标准参考文献/全文测试集基准”；`papers` 中固定为“全文测试集候选”。 |

> `annotation` 是**可扩展字段**。后续如需增加如 `language`、`year`、`doi`、`evidence_span`、`label`（`SUPPORT`/`CONTRADICT`）等，建议新增键而不修改既有语义。

---

## 4. 数据构建说明

- **文献来源**：中国知网（CNKI）题录信息。
- **领域划分**：计算机领域 5 个方向，每个方向 5-10 篇，优先选取 2024–2025 年综述或系统性梳理类文献，便于覆盖技术脉络。
- **标准文献**：每个领域人工选定 1 篇作为 `standard`，用于锚定该领域的测试基准。
- **未包含内容**：本仓库**不包含**摘要、正文、PDF 或任何知网全文内容。
- **链接说明**：`link` 字段当前为知网标题检索链接占位。知网详情页 URL 可能随会话变化。

---

## 5. 版权与使用限制

- 本仓库仅分发**题录信息（标题、类型、检索链接）与自定义标注**，不分发知网全文或 PDF。
- 知网文献的著作权归原作者及出版方所有。使用者如需获取全文，应通过**合法授权渠道**（如机构订阅、知网个人账号）自行下载。
- 若将本索引用于公开评测或论文，建议补充 DOI，并注明“全文未随仓库分发，需自行获取”。
- 标注字段（`id`、`annotation`、`standard`/`papers` 角色划分）为本仓库自定义，采用 MIT 或 CC BY 4.0 等宽松许可发布即可；题录本身不受此许可约束。

---

## 8. 引用

如使用本索引，请注明：

```text
CNKI Multi-Domain Full-Text Test Corpus (Annotated Index), 2025.
Contains bibliographic metadata and custom annotations only; full texts are not distributed.
```
