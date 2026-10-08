# 基于 LLM 的车载智能语音任务型对话 Agent Demo

---

## 1. 项目简介

本项目面向车载语音助手场景，构建了一个任务型对话 Agent Demo。系统接收用户自然语言指令后，可以完成任务仲裁、拒识判断、意图识别、槽位抽取、工具调用和自然语言回复生成。

项目支持天气查询、导航搜索、音乐搜索、车辆控制等典型车载语音交互场景，能够模拟智能座舱语音助手从“用户输入”到“任务执行结果”的完整流程。

---

## 2. 项目核心功能

- 支持车载语音指令理解
- 支持任务型对话和闲聊百科区分
- 支持拒识无效请求
- 支持多轮上下文处理
- 支持意图识别和槽位抽取
- 支持 Function Calling 工具调用
- 支持 MCP 调用地图、天气、音乐等外部能力
- 支持 NLG 自然语言回复生成
- 支持基础测试与端到端评估

---

## 3. 技术栈

- Python
- FastAPI
- SocketIO
- Redis
- BERT / RoBERTa
- LLM Function Calling
- MCP
- 高德地图 API
- 音乐搜索接口
- Locust 压测

---

## 4. 系统流程

用户输入语音文本后，系统大致流程如下：

```text
用户输入
→ 入口服务
→ 多轮上下文处理
→ 任务仲裁
→ 拒识判断
→ BERT / RoBERTa 意图召回
→ LLM Function Calling 槽位抽取
→ DM 模块分发
→ MCP 工具调用
→ NLG 回复生成
→ 返回最终结果
```

---

## 5. 项目结构

```text
demo/
├── client/              # 对话客户端核心模块
│   ├── arbitration.py   # 任务仲裁
│   ├── correlation.py   # 多轮相关性判断
│   ├── nlg.py           # 自然语言回复生成
│   ├── nlu.py           # NLU 调用
│   ├── reject.py        # 拒识判断
│   ├── rewrite.py       # 多轮改写
│   └── stream_chat.py   # 流式闲聊回复
│
├── config/              # 配置文件
│   ├── class.txt        # 意图类别映射
│   ├── config.ini       # 项目配置
│   ├── new_map.json     # 意图映射文件
│   └── slot_intent.json # 槽位与意图配置
│
├── function_call/       # Function Calling 与任务分发模块
│   ├── dm/              # 领域任务处理
│   │   ├── factory.py   # DM 工厂类
│   │   ├── maps.py      # 地图任务
│   │   ├── music.py     # 音乐任务
│   │   └── weather.py   # 天气任务
│   ├── chatnlu_infer.py # NLU 服务入口
│   ├── slot_process.py  # 槽位处理
│   └── function.py      # Function Calling 工具定义
│
├── mcp_core/            # MCP 工具调用模块
│   ├── amp_server.py    # 高德地图 / 天气 MCP 服务
│   ├── mcp_client.py    # MCP 客户端
│   └── music_server.py  # 音乐搜索服务
│
├── test/                # 测试与评估脚本
│   ├── intent_client.py
│   ├── nlu_client.py
│   ├── reject_client.py
│   ├── intent_benchmark.py
│   ├── nlu_benchmark.py
│   └── result/
│
├── train/               # 模型训练与推理模块
│   ├── data/            # 训练数据
│   ├── models/          # 模型结构
│   ├── pretrained/      # 预训练模型
│   ├── saved/           # 保存的模型权重
│   ├── intent_infer.py  # 意图识别推理
│   ├── reject_infer.py  # 拒识模型推理
│   └── run.py           # 训练入口
│
├── utils/               # 工具模块
│   ├── logger.py        # 日志工具
│   └── redis_tool.py    # Redis 工具
│
├── dialog.py            # 命令行对话测试入口
├── e2e_score.py         # 端到端准确率统计
├── prompts.py           # Prompt 配置
├── requirements.txt     # 项目依赖
├── server.sh            # 服务启动脚本
├── start.py             # 主服务启动入口
└── test.py              # 测试入口
```

---

## 6. Demo 示例

### 示例 1：天气查询

用户输入：

```text
今天北京天气怎么样？
```

系统输出示例：

```json
{
  "query": "今天北京天气怎么样？",
  "intent": "实时查询天气",
  "function": "Query_Timely_Weather",
  "slots": {
    "city": "北京",
    "date": "今天"
  },
  "nlg": "北京今天天气为晴，白天温度约 26 度。"
}
```

说明：

该示例展示了系统对天气查询类指令的理解能力。系统能够识别用户的查询意图，提取城市、日期等槽位，并通过工具模块生成自然语言回复。

---

### 示例 2：导航搜索

用户输入：

```text
帮我导航到火车站
```

系统输出示例：

```json
{
  "query": "帮我导航到火车站",
  "intent": "导航搜索",
  "function": "Go_POI",
  "slots": {
    "POI": "火车站"
  },
  "nlg": "已为你查询附近的火车站，可以选择一个地点开始导航。"
}
```

说明：

该示例展示了系统对导航类指令的识别能力。系统能够识别用户想要进行地点搜索或导航，并提取目的地信息。

---

### 示例 3：音乐搜索

用户输入：

```text
播放周杰伦的歌
```

系统输出示例：

```json
{
  "query": "播放周杰伦的歌",
  "intent": "音乐搜索",
  "function": "Search_Music",
  "slots": {
    "singer": "周杰伦"
  },
  "nlg": "已为你搜索周杰伦相关歌曲。"
}
```

说明：

该示例展示了系统对音乐搜索类指令的处理能力。系统能够识别音乐搜索意图，并提取歌手、歌曲名或音乐类型等关键信息。

---

### 示例 4：车辆控制

用户输入：

```text
把空调温度调低一点
```

系统输出示例：

```json
{
  "query": "把空调温度调低一点",
  "intent": "降低空调温度",
  "function": "Dec_Air_Condition_Temperature",
  "slots": {},
  "nlg": "已为你调低空调温度。"
}
```

说明：

该示例展示了系统对车辆控制类指令的识别能力。系统能够将用户的自然语言表达映射到对应的车控功能。

---

## 7. 项目亮点

1. **模块化 Agent 架构清晰**

   项目将入口服务、任务仲裁、拒识判断、NLU、工具调用和 NLG 拆分为多个模块，整体结构较清晰，便于后续维护和扩展。

2. **BERT / RoBERTa + LLM Function Calling 结合**

   系统先使用意图识别模型召回候选意图，再结合 LLM Function Calling 完成槽位抽取，使传统 NLU 与大模型工具调用能力结合起来。

3. **支持 MCP 工具调用**

   项目通过 MCP 封装地图、天气、音乐等外部工具能力，能够模拟真实车载语音助手中的技能调用流程。

4. **支持多轮上下文处理**

   系统使用 Redis 缓存对话历史，并通过改写、相关性判断等方式处理多轮对话场景，能够支持一定程度的上下文理解。

5. **具备测试和评估闭环**

   项目中包含意图识别测试、NLU 测试、拒识测试、端到端评估和压测脚本，能够对系统效果进行基础验证。

---

## 8. 个人负责内容

- 负责车载语音助手任务型对话整体流程设计
- 负责意图分类、槽位配置和 Function Calling Schema 整理
- 负责意图识别、槽位抽取和任务分发流程实现
- 负责 MCP 工具调用模块接入与 Demo 场景验证
- 负责编写 Demo 测试样例和结果评估脚本
- 负责项目结构梳理、README 文档和展示材料制作

---

## 9. 运行说明

### 9.1 安装依赖

建议使用 Python 3.8 或 Python 3.9。

```bash
pip install -r requirements.txt
```

### 9.2 配置环境变量

项目中涉及大模型接口、意图识别服务、NLU 服务、Redis 和地图接口等配置。为了安全起见，真实 API Key 不应上传到公开仓库或直接发送给他人。

可以参考以下格式创建 `.env.example`：

```env
API_KEY=your_api_key_here
BASE_URL=your_llm_base_url_here

INTENT_URL=http://127.0.0.1:8008/intent-server/v1
NLU_URL=http://127.0.0.1:8009/chatnlu-server/v1
REJECT_URL=http://127.0.0.1:8010/reject-server/v1
ENTRY_URL=http://127.0.0.1:8000

REDIS_HOST=127.0.0.1
REDIS_PORT=6379

AMAP_MAPS_API_KEY=your_amap_key_here
```

### 9.3 启动 Redis

项目使用 Redis 存储多轮对话历史，需要先启动 Redis 服务。

```bash
redis-server
```

### 9.4 启动服务

启动意图识别服务：

```bash
python train/intent_infer.py
```

启动 NLU 服务：

```bash
python function_call/chatnlu_infer.py
```

启动主入口服务：

```bash
python start.py
```

### 9.5 运行 Demo

可以使用 `dialog.py` 进行命令行交互测试：

```bash
python dialog.py
```

输入示例：

```text
今天北京天气怎么样？
帮我导航到火车站
播放周杰伦的歌
```

---

## 10. 测试说明

项目中包含基础测试和评估脚本，主要包括：

| 测试类型 | 作用 | 对应文件 |
|---|---|---|
| 意图识别测试 | 验证用户指令是否能正确分类 | test/intent_client.py |
| NLU 测试 | 验证意图和槽位抽取是否正确 | test/nlu_client.py |
| 拒识测试 | 验证系统能否识别不支持请求 | test/reject_client.py |
| 端到端测试 | 验证完整对话链路是否正确 | e2e_score.py |
| 接口压测 | 验证接口并发性能 | test/intent_benchmark.py、test/nlu_benchmark.py |

意图识别测试示例：

```bash
python test/intent_client.py
```

NLU 测试示例：

```bash
python test/nlu_client.py
```

端到端评估示例：

```bash
python e2e_score.py
```

压测示例：

```bash
locust -f intent_benchmark.py --host http://127.0.0.1:8008 --headless -u 1000 -r 100 -t 60s
```

---

## 11. 项目展示文件说明

本 Demo 建议包含以下展示文件：

```text
01_车载语音Agent_Demo/
├── README.md                 # 项目总说明
├── demo输入输出样例.md        # 典型输入输出展示
├── 项目流程图.png             # 系统流程图
├── core_code/                # 核心代码
└── .env.example              # 环境变量示例
```

其中：

- `README.md`：用于让 HR 或面试官快速了解项目整体情况。
- `demo输入输出样例.md`：用于展示项目能够处理哪些输入，以及返回什么结果。
- `项目流程图.png`：用于快速说明系统从输入到输出的处理链路。
- `core_code/`：用于存放项目核心代码。
- `.env.example`：用于说明项目需要哪些配置，但不包含真实密钥。

---

## 12. 注意事项

1. 真实 API Key、接口密钥和账号信息不要上传或发送给他人。
2. 如果项目中包含大模型权重、预训练模型或缓存文件，可以根据文件大小选择不放入展示压缩包。
3. 如果日志文件中包含真实接口地址、密钥或敏感返回信息，建议删除或脱敏后再发送。
4. Demo 示例中的输出结果可以根据实际运行结果进行替换。

---

## 13. 适用场景

本项目适用于以下场景：

- 智能座舱语音助手
- 车载任务型对话系统
- LLM Agent 工程实践
- Function Calling 应用 Demo
- MCP 工具调用 Demo
- 多轮对话与意图识别系统展示

---

## 14. 项目归属声明

项目内容围绕车载智能语音助手中的任务型对话场景展开，包含意图识别、槽位抽取、Function Calling、MCP 工具调用、多轮上下文处理和自然语言回复生成等模块。

为保证项目安全性，Demo 文件中涉及的真实 API Key、接口密钥、账号信息、部分服务地址和模型运行环境配置均已做脱敏处理。项目展示材料仅用于说明系统设计思路、核心功能流程和工程实现能力。


---

## 15. 总结

本项目围绕车载语音助手中的任务型对话需求，构建了从用户输入、上下文处理、意图识别、槽位抽取、工具调用到自然语言回复生成的完整链路。项目结合了传统 NLU 模型和 LLM Function Calling 能力，并通过 MCP 接入外部工具，能够较完整地展示智能座舱语音助手的核心工程流程。
