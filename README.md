# MedicalRAGSystem

一个基于医疗知识图谱的中文问答系统，集成了：

- 医疗数据爬取与清洗
- Neo4j 知识图谱构建
- 规则型 KBQA 问答
- 基于子图检索的 GraphRAG 问答
- FastAPI 后端服务
- React + Vite 前端可视化界面

这是一个完整的医疗知识图谱问答系统，覆盖了从数据采集、知识图谱构建、规则问答、GraphRAG 检索生成，到后端服务和前端可视化的完整实现链路，也可以作为后续接入更大模型与更强检索策略的基础工程。


## 项目特点

- 同时支持两种问答模式
  - `基础问答`：规则解析 + Cypher 查询
  - `GraphRAG`：实体抽取 + 实体归一化 + 子图检索 + 上下文组装 + LLM 生成
- 提供前后端完整工程，不只是脚本示例
- 使用统一配置中心 `settings.py` 和 `.env`
- 支持 Ollama、本地兼容 OpenAI API、Anthropic 等多种 LLM 接入方式
- 提供流式 SSE 接口，便于前端实时展示生成过程
- 支持图谱邻居查询和图可视化调试

## 系统架构

### 1. 数据构建链路

1. `data_spider/` 从医疗站点抓取结构化信息
2. `knowledge_graph/` 将 `data/medical.json` 导入 Neo4j
3. `dict/` 中的实体词典用于实体识别、归一化和规则问答

### 2. 在线问答链路

#### 基础问答

1. 识别用户意图与实体
2. 生成 Cypher 查询
3. 查询 Neo4j
4. 模板化答案输出

#### GraphRAG

1. LLM 抽取问题中的医疗实体
2. 实体归一化到图谱节点
3. 多跳检索相关子图
4. 组装图谱上下文
5. 调用 LLM 生成自然语言答案

## 目录结构

```text
MedicalGraphRAGSystem/
├─ data/                 # 图谱数据，默认包含 medical.json
├─ data_spider/          # 爬虫与数据预处理
├─ dict/                 # 疾病、症状、药品等词典
├─ doc/                  # 设计文档与使用文档
├─ graphrag/             # GraphRAG 检索与生成链路
├─ KBQA/                 # 规则型知识图谱问答
├─ knowledge_graph/      # Neo4j 图谱构建脚本
├─ server/               # FastAPI 后端
├─ web/                  # React + Vite 前端
├─ img/                  # README 配图
├─ settings.py           # 全局配置入口
├─ .env.example          # 环境变量模板
└─ requirements.txt      # Python 依赖
```

## 技术栈

### 后端

- Python
- FastAPI
- py2neo
- LangChain
- Neo4j

### 前端

- React 19
- TypeScript
- Vite
- Tailwind CSS
- react-force-graph-2d

### 可选模型提供方

- Ollama
- OpenAI 兼容接口
- Anthropic

## 环境要求

- Python 3.10+
- Node.js 18+
- Neo4j 4.x/5.x
- 可选：Ollama 或其他 LLM 服务

## 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/ChailynCui/MedicalRAGSystem.git
cd MedicalRAGSystem
```

### 2. 安装 Python 依赖

```bash
python -m venv .venv

# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS / Linux
# source .venv/bin/activate

pip install -r requirements.txt
```

如果你要接 OpenAI 兼容接口或 Anthropic，可额外安装：

```bash
pip install langchain-openai
pip install langchain-anthropic
```

### 3. 启动 Neo4j

推荐使用 Docker：

```bash
docker run -d --name neo4j -p 7474:7474 -p 7687:7687 -e NEO4J_AUTH=neo4j/your_password neo4j:5-community
```

启动后访问：

- Neo4j Browser: `http://localhost:7474`
- Bolt 地址: `bolt://127.0.0.1:7687`

### 4. 配置环境变量

复制模板文件：

```bash
copy .env.example .env
```

或在 PowerShell 中执行：

```powershell
Copy-Item .env.example .env
```

示例配置：

```env
NEO4J_URI=bolt://127.0.0.1:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password

LLM_PROVIDER=ollama
LLM_MODEL=qwen3:8b
LLM_BASE_URL=http://localhost:11434
LLM_TEMPERATURE=0
LLM_MAX_TOKENS=512
```

如果使用 OpenAI 兼容接口：

```env
LLM_PROVIDER=openai
LLM_MODEL=gpt-4o
OPENAI_API_KEY=your_key
# OPENAI_BASE_URL=https://your-compatible-endpoint/v1
```

### 5. 导入医疗知识图谱

仓库默认包含 `data/medical.json`，执行下面命令将其导入 Neo4j：

```bash
python knowledge_graph/main.py --clear
```

常用参数：

```bash
python knowledge_graph/main.py --step nodes
python knowledge_graph/main.py --step rels
python knowledge_graph/main.py --uri bolt://127.0.0.1:7687 --user neo4j --password your_password
```

## 运行方式

### 1. CLI 问答

```bash
python -m KBQA.main
```

单次提问：

```bash
python -m KBQA.main --question "糖尿病有什么症状"
```

启用 LLM 润色：

```bash
python -m KBQA.main --answer-mode llm
```

### 2. 启动后端 API

```bash
python -m server.app
```

默认地址：

- API: `http://127.0.0.1:8000`
- 健康检查: `http://127.0.0.1:8000/api/health`

常用启动参数：

```bash
python -m server.app --host 0.0.0.0 --port 8000
python -m server.app --answer-mode llm
python -m server.app --llm-provider openai --llm-model gpt-4o --llm-api-key your_key
```

### 3. 启动前端

```bash
cd web
npm install
npm run dev
```

默认前端地址：

- `http://127.0.0.1:5173`

构建生产版本：

```bash
npm run build
```

当前端构建产物存在于 `web/dist` 时，后端会自动挂载静态页面。

## 核心接口

### 基础问答

- `POST /api/chat`
- `POST /api/chat/stream`

请求示例：

```json
{
  "question": "高血压怎么治疗"
}
```

### GraphRAG 问答

- `POST /api/graphrag/chat`
- `POST /api/graphrag/chat/stream`

### 图谱探索与健康检查

- `GET /api/graph/neighbors/{name}`
- `GET /api/health`

## 支持能力

### 规则型 KBQA

当前代码支持围绕疾病、症状、药品、检查、食物等实体进行问答，主要包括：

- 疾病有哪些症状
- 某症状可能对应什么疾病
- 某疾病的病因、并发症、预防方式、治疗方式、治愈概率
- 某疾病适合吃什么、不适合吃什么、常见药物、相关检查
- 某药物可以治疗什么疾病
- 某检查可以辅助发现什么疾病
- 某食物对哪些疾病有益或有禁忌

### GraphRAG

`graphrag/` 模块主要包含：

- `entity_extractor.py`：从问题抽取医疗实体
- `subgraph_retriever.py`：围绕实体做多跳子图检索
- `context_builder.py`：把图谱结构整理成可供 LLM 使用的上下文
- `generator.py`：生成最终自然语言答案
- `graphrag_bot.py`：整合整个 GraphRAG 流程

## 数据与图谱说明

图谱以疾病为中心，包含多类节点与关系，例如：

- 节点：`Disease`、`Symptom`、`Drug`、`Food`、`Check`、`Department`、`Producer`
- 关系：`has_symptom`、`common_drug`、`need_check`、`do_eat`、`no_eat`、`belongs_to` 等

图谱导入逻辑位于：

- `knowledge_graph/data_loader.py`
- `knowledge_graph/graph_builder.py`
- `knowledge_graph/main.py`

## 爬虫说明

如果你希望重新抓取或扩展数据源，可以使用：

```bash
python data_spider/main.py --start 1 --end 100
```

支持的常用参数：

- `--resume`：断点续爬
- `--test`：测试模式，仅打印不落盘
- `--delay`：控制请求间隔

## 配置说明

统一配置位于 `settings.py`，核心配置包括：

- Neo4j 地址、账号、密码
- LLM 提供方、模型名、服务地址
- 实体词典路径
- 模糊匹配阈值
- 默认回答与生成参数

项目优先级大致为：

- CLI 参数
- `.env`
- `settings.py` 默认值

- 增加 Docker Compose 一键启动方案

## 免责声明

- 医疗问答结果不能替代专业医生诊断
- 如将数据用于公开部署或商业用途，请先确认数据来源与授权合规性

## 致谢

本项目重点实现了：

- 更清晰的模块拆分
- GraphRAG 链路
- Web API
- 前端可视化界面
- 统一配置与更完整的运行方式
