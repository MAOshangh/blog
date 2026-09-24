# L1 快速上手：跑通第一个 Agent · 预计 2~3 小时

**学完你能做到**：用官方 API 创建并运行第一个 ReActAgent，理解 Msg 消息与异步调用方式

**前置要求**：需先完成 L0（安装 + API Key 就绪）。

## 创建第一个 Agent（官方写法）
官方示例里创建 Agent 的标准写法是 ReActAgent + DashScopeChatModel + DashScopeMultiAgentFormatter。先看懂这三样东西：ReActAgent 是『会思考的智能体』（ReAct 即 Reasoning+Acting，先想再动）；DashScopeChatModel 是接入 qwen 模型的封装；formatter 决定对话历史怎么组织——多 Agent 场景必须用 DashScopeMultiAgentFormatter，它会把不同 Agent 的名字带进消息，让模型知道『谁在说话』。

下面这段代码是官方 pipeline 教程中创建 Agent 的原版写法（仅改了角色文案）。注意：Agent 的调用是异步的（async/await），这是 AgentScope 的惯例——模型调用是 IO 操作，异步能同时跑多个 Agent。

```python
import os
import asyncio
from agentscope.agent import ReActAgent
from agentscope.formatter import DashScopeMultiAgentFormatter
from agentscope.model import DashScopeChatModel


def create_agent(name: str, sys_prompt: str) -> ReActAgent:
    """按官方推荐方式创建一个 ReAct 智能体"""
    return ReActAgent(
        name=name,
        sys_prompt=sys_prompt,
        model=DashScopeChatModel(
            model_name="qwen-max",
            api_key=os.environ["DASHSCOPE_API_KEY"],
        ),
        formatter=DashScopeMultiAgentFormatter(),
    )


async def main() -> None:
    agent = create_agent(
        "assistant",
        "You're a helpful assistant. Reply in Chinese.",
    )
    # 直接向 Agent 发一条消息（字符串会被自动包装成 Msg）
    reply = await agent("你好，请用一句话介绍什么是多智能体系统")
    print(reply)


asyncio.run(main())
```

**预期输出**：
终端打印一条 Msg 对象：name='assistant'，content 里是模型回复『多智能体系统是……』的文本。如果看到 KeyError: DASHSCOPE_API_KEY，说明环境变量没配对（回 L0 检查）。

**操作步骤**
1. 把上方代码保存为 first_agent.py
2. 确认 DASHSCOPE_API_KEY 环境变量已设置（L0 已验证）
3. 运行 python first_agent.py，观察打印的 Msg
4. 改 sys_prompt 里的角色描述，再跑一次，感受提示词对回答风格的影响

**常见坑**
- 报 ModuleNotFoundError: agentscope 相关模块：先确认安装成功且解释器一致
- 报 KeyError: DASHSCOPE_API_KEY：环境变量没设或没在新终端生效
- 忘记 await：调用 Agent 不加 await 会打印 coroutine 对象而不是结果
- model_name 用错：qwen 系列用 qwen-max / qwen-plus 等官方模型名

**要点**
- 创建 Agent 三件套：ReActAgent + DashScopeChatModel + DashScopeMultiAgentFormatter
- Agent 调用是异步的：await agent(消息) 才能拿到回复
- api_key 一律从 os.environ 读，不要写死在代码里

## 理解消息 Msg：Agent 之间传什么
刚才你传字符串给 agent，AgentScope 内部会把它包装成 Msg。Msg 是框架里所有通信的统一载体，理解它等于理解了框架的『语言』。

Msg 的核心字段：name（谁发的消息）、content（内容，可以是纯文本，也可以是结构化内容）、role（消息角色：user/assistant/system）。构造方式有两种等价写法：Msg("user", "内容", "user") 位置参数，或 Msg(name="user", content="内容", role="user") 关键字参数。

为什么 Agent 之间传东西要用 Msg 而不是直接传字符串？因为协作场景需要『带身份的消息』：writer 收到一条消息，必须知道它是 planner 发来的大纲还是 user 的原始需求；role 字段让模型能区分『这是系统指令还是用户输入』。Msg 还统一了消息历史记录，方便调试时回看每一步传递。

```python
from agentscope.message import Msg

# 两种等价构造方式
msg1 = Msg("user", "帮我列三个学习 Python 的步骤", "user")
msg2 = Msg(name="user", content="帮我列三个学习 Python 的步骤", role="user")

# 读取字段
print(msg1.name)      # user
print(msg1.content)   # 帮我列三个学习 Python 的步骤
print(msg1.role)      # user
```

**预期输出**：
依次打印：user / 帮我列三个学习 Python 的步骤 / user

**常见坑**
- 把 content 误认为就是消息本身：调试时记住 name/content/role 三个字段都要看
- 位置参数顺序记错：Msg 的位置顺序是 (name, content, role)

**要点**
- Msg = name + content + role，是 AgentScope 里唯一的消息载体
- 字符串会被自动包装成 Msg；显式构造 Msg 能控制更多字段
- 多 Agent 协作时，消息带身份（name）是模型判断『谁在说话』的依据

## async 主函数：脚本为什么这样写
你可能奇怪：为什么所有示例都有 async def main() 和 asyncio.run(main()) 这两行？因为 AgentScope 的 Agent 调用是异步函数，异步函数必须在 async 上下文中 await 调用，而 asyncio.run() 是启动整个 async 程序的入口。

简单理解：async def 定义了一个『可以边等边干别的』的函数；await 表示『等这个异步操作完成』；asyncio.run(main()) 负责启动它。多智能体协作时，多个 Agent 的模型调用都是 IO 等待，异步让它们可以并行，这正是官方 fanout 管道能同时跑多个 Agent 的原因。

初学者最常见的错误是忘了 await 或忘了 asyncio.run。如果打印出来的是 <coroutine object ...>，就是漏了 await；如果报 RuntimeError: asyncio.run() cannot be called from a running event loop，则是你在 async 函数里又调了 asyncio.run。记住口诀：『定义用 async def，调用要 await，入口用 asyncio.run』。

**操作步骤**
1. 把上一节 first_agent.py 里 asyncio.run(main()) 临时删掉，运行看报什么错（学习诊断）
2. 把 await agent(...) 里的 await 删掉，运行观察打印的 coroutine 对象
3. 改回正确写法，确认程序恢复正常——理解这三行代码各自的作用

**常见坑**
- 漏 await：打印 <coroutine object main at ...>，加回 await 即可
- 在 async 函数内又写 asyncio.run：会报 running event loop 错误
- async def 与普通 def 混淆：被 await 的对象必须是 async def 定义的可等待对象

**要点**
- async def 定义异步函数，await 等待结果，asyncio.run 是启动入口
- AgentScope 的 Agent 调用都是异步的，统一用 await
- 用『删掉看报错』的方式理解语法，比死记规则更有效

## 延伸阅读（资源）
- [官方文档 · 创建 ReAct 智能体](https://doc.agentscope.io/tutorial/create_react_agent.html) · 官方 · 创建 Agent 的官方教程，含完整代码与解释
- [官方文档 · 创建消息 Create Message](https://doc.agentscope.io/tutorial/create_message.html) · 官方 · Msg 的官方用法，字段与构造方式以这里为准
- [官方文档 · 核心概念](https://doc.agentscope.io/tutorial/key_concepts.html) · 官方 · 把 Agent/Msg/模型/格式化器的关系再看一遍，巩固概念
- [Python 官方文档 · asyncio 入门](https://docs.python.org/zh-cn/3/library/asyncio.html) · 可靠 · 看不懂 async/await 时查这里，权威且免费

## 动手练习
- 创建 2 个不同角色的 Agent，分别问它们同一个问题，比较回答风格差异
- 显式构造 Msg 发给 Agent，打印返回 Msg 的 name/content/role 三个字段
- 故意制造一次『漏 await』错误并成功修复，把报错信息截图存进学习笔记

## 自测问题
- 创建 ReActAgent 需要哪三样东西？formatter 在多 Agent 场景为什么必须用 DashScopeMultiAgentFormatter？
- Msg 的三个核心字段是什么？为什么协作场景需要消息带 name？
- 打印出 <coroutine object> 说明代码犯了什么错？怎么修？
- 应用题：如果要把 sys_prompt 改成『你是一个严格的英文校对员，用英文回复』，代码里改哪一行？

## 能力自检
- 能从零手写一个创建 ReActAgent 并对话的脚本（不抄教程）
- 能解释 async def / await / asyncio.run 各自的作用
- 能构造 Msg 并读取其字段，知道 name/content/role 的含义