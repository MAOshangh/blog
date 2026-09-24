# AgentScope 学习路径 · 从入门到精通

> 目标：从零掌握 AgentScope 多智能体框架：能独立搭建、运行并扩展多智能体应用
> 生成日期：2026-09-23

## 主题简介
AgentScope 是阿里巴巴开源的多智能体（Multi-Agent）开发平台，提供高鲁棒、高易用的 Agent 编程框架，支持智能体构建、多智能体协作编排、消息通信、管道（Pipeline）与可视化调试，内置超时控制、错误处理等工程能力。本教程基于官方稳定版文档（doc.agentscope.io）的真实 API，带零基础用户从安装开始，一步步走到能独立搭建多智能体协作应用。

## 开始前检查：前置知识
- **Python 基础（函数、类、async/await）**（必备）：能独立写一个带参数和返回值的函数；能读懂 class 定义；知道 async def 函数要用 await 调用、asyncio.run() 入口
  - 不足时：Python 官方教程 https://docs.python.org/zh-cn/3/tutorial/ 前 10 章；异步部分看《Python Cookbook》第 15 章或廖雪峰 Python 教程的 async/await 小节
- **大模型与 API 基础概念**（建议）：知道什么是大语言模型（LLM）、什么是 API Key、什么是 token；明白『发一条请求带 prompt，模型返回文本』的调用模型
  - 不足时：随便读一篇大模型科普（如李宏毅《机器学习》LLM 讲座）或各大模型平台的『快速开始』文档（OpenAI/DashScope 皆可）
- **Python 3.11+ 与包管理工具**（必备）：python --version 显示 3.11 及以上；会用 pip 或 uv 安装 Python 包
  - 不足时：到 python.org 安装 3.11+；uv 安装见 https://docs.astral.sh/uv/ （一条命令即可）
- **一个大模型服务的 API Key**（必备）：拥有 DashScope（阿里云百炼）、OpenAI 等任一平台的有效 API Key，且余额充足；能放到环境变量中读取
  - 不足时：DashScope 注册与获取 Key：https://bailian.console.aliyun.com/ ；本教程示例默认使用 DashScope（qwen-max），其他平台替换 model 与 key 即可

## 术语表
- **Agent**（智能体）：能自主思考、调用工具并完成任务的对象；本教程用 ReActAgent 创建
- **ReAct**（思考-行动循环）：Reasoning + Acting：模型先思考，再决定是否调用工具，观察结果后继续
- **Msg**（消息）：AgentScope 中所有通信的载体，含 name（谁发的）、content（内容）、role（角色）
- **sys_prompt**（系统提示词）：创建 Agent 时设定的角色与行为约束，决定 Agent『是谁、怎么干活』
- **formatter**（格式化器）：把对话历史格式化成模型能理解的模板；多 Agent 场景用 DashScopeMultiAgentFormatter
- **Pipeline**（管道）：把多个 Agent 按固定逻辑串联/并联执行的语法糖：Sequential（顺序）与 Fanout（扇出）
- **MsgHub**（消息中心）：一个 Agent 发言会自动广播给其他参与者的消息集散地
- **DashScopeChatModel**（DashScope 对话模型）：AgentScope 内置的模型封装，接入阿里云百炼（qwen 系列）
- **async/await**（异步调用）：Python 异步语法；AgentScope 的 Agent 调用是异步的，必须 await
- **asyncio.run**（异步入口）：运行 async 主函数的入口，脚本以此启动
- **API Key**（接口密钥）：调用大模型服务的凭证，通过环境变量读取，不要写进代码
- **stream_printing_messages**（消息流打印）：把 Agent 运行中的中间消息转成异步生成器，用于观察过程
- **Agent Skill**（智能体技能）：Anthropic 提出的方法：把任务做法打包成含 SKILL.md 的文件夹，Agent 读取指引后行动；Skill 不是可调用工具
- **Toolkit**（技能工具箱）：AgentScope 管理 Agent Skill 的类：register_agent_skill / remove_agent_skill / get_agent_skill_prompt，可整体传给 ReActAgent

## 知识地图
- **Agent 基础**：什么是智能体、Agent 的核心能力、创建与调用
- **消息 Msg**：Agent 之间传什么：Msg 的结构与构造
- **多智能体协作**：MsgHub 广播、Sequential/Fanout 管道串联多个 Agent
- **模型与格式化器**：接入 LLM 模型（DashScopeChatModel）与对话格式化器
- **工具与技能**：@tool 工具调用，与 Agent Skill 技能包的设计、加载与使用
- **工程化与调试**：超时、日志、可视化调试与求助路径

## 学习路径
- **L0 认识与准备**（预计 1~2 小时）· 理解 AgentScope 是什么、能解决什么问题；在本机完成安装并成功运行官方快速开始 · [进入该阶段](phases/L0-认识与准备.md)
- **L1 快速上手：跑通第一个 Agent**（预计 2~3 小时）· 用官方 API 创建并运行第一个 ReActAgent，理解 Msg 消息与异步调用方式 · [进入该阶段](phases/L1-快速上手-跑通第一个-agent.md)
- **L2 核心概念：多智能体协作与编排**（预计 4~6 小时）· 掌握 MsgHub 广播与 Sequential/Fanout 管道，能搭建 2~3 个 Agent 的协作流水线 · [进入该阶段](phases/L2-核心概念-多智能体协作与编排.md)
- **L3 动手实践：搭建完整协作应用**（预计 8~12 小时）· 独立完成一个「策划-写作-评审」三 Agent 协作应用，掌握从设计到运行的完整流程 · [进入该阶段](phases/L3-动手实践-搭建完整协作应用.md)
- **L4 进阶深化：记忆、工具与工程化**（预计 6~10 小时）· 掌握记忆、工具调用、多模型接入与工程化调优，让应用具备记忆与行动能力，达到『熟练掌握』水平 · [进入该阶段](phases/L4-进阶深化-记忆-工具与工程化.md)

## 遇到问题怎么办
- 官方文档站（权威优先）：https://doc.agentscope.io/ —— 教程与 FAQ 页面覆盖绝大多数问题
- GitHub Issues（报错求助）：https://github.com/agentscope-ai/agentscope/issues —— 搜关键词或开新 Issue，附完整报错
- 官方社区：GitHub Discussions https://github.com/agentscope-ai/agentscope/discussions 与官方 Discord 群
- 关键口诀：报错先看消息流定位到具体 Agent，再对照官方 pipeline 教程的代码逐行核对

## 学习建议
- 先跑官方示例再读原理：AgentScope 官方文档的示例质量很高，动手优先
- 本教程所有代码基于官方稳定版文档（doc.agentscope.io）真实 API，可直接照敲；运行前确认你的 agentscope 版本与文档一致
- 术语对照见总览『术语表』：Agent=智能体，Msg=消息，Pipeline=管道，MsgHub=消息中心，ReAct=思考-行动循环
- L0-L1 一天内可完成，L2-L3 建议 1~2 周；卡住时先查求助路径，再回看对应阶段文档
- 学到 L3 后，把你的『写-评-改』应用扩展成自己的项目，比多学十篇教程更有用