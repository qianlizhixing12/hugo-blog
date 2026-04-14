---
title: "AI相关概念整理和本地部署"
date: 2026-04-14
bookHidden: true
---

> AI越来越火，火到连楼下煎饼摊的大爷都能跟你聊两句大模型。但火归火，隐私和安全的隐忧也像影子一样甩不掉——你的聊天记录、公司的内部文档，到底是在‘云端裸奔’，还是真能锁进自己的硬盘里？再看满屏飞的AGI、AIGC、LLM、RAG、微调……概念混杂得像一锅粥。想搞本地部署，结果第一步就被术语劝退。
>
> 今天不追热只做两件事：把这团乱麻按层次捋清楚，动手制作一张看得懂的‘本地部署入场券’。

## AI生态概念分层

### 硬件基础设施

模型运行的物理基础，最主要的两种资源是算力(GPU或CPU)和内存资源。

算力决定了模型运行的快不快和能源消耗，模型文件占的是硬盘空间，而运行时占的是内存空间，模型必须完整装入内存才能运行。服务器集群和个人PC都算硬件基础设施，有些模型参数很多个人PC能下载到本地磁盘却能内存不够不能运行。

### 模型

模型相当于训练好的专家知识库，训练什么类型的数据决定了擅长什么，能成为什么样的专家 。模型参数相当于专家的知识和训练量，训练读越多，脑子越好使，脑容量越大，但工资（基础设施）要求也越高。

这也就是为什么大家听到有的模型擅长编程，聊天，写作等等的原因。且一般通用型的专家，几个领域都能涉及，但回答质量可能不够理想。

常见的模型有Llama、Qwen、DeepSeek、Doubao、GPT、Gemini、Claude等。其中一些是开源的可以下载到个人PC本地运行，有些是闭源的只能通过厂商提供的api或app调用，并有一定费用。

### 推理引擎

模型这个专家知识库，是一个屏蔽了硬件差异的静态数据，或者说一份菜谱，推理引擎的作用是做“翻译官”和“厨师”：它读取模型菜谱，然后根据当前厨房的实际炊具（硬件），生成最高效的运行指令。

个人本地推理引擎代表有Ollama、llama.cpp、LM Studio等，厂商的推理引擎一般都是魔改或自研的，有极致吞吐量，对硬件要求也高。

### 智能体

上面三层都是只运行模型的，没有记忆和上下文，这层就要管理用户会话历史或编排任务执行了，主要关注的安全问题集中在这一层出现。

1. 个人数据安全，智能体存储用户会话历史，比如要存储到某个地方(云端数据库/本地数据库等)，有可能被厂商用于分析，本地部署没有这个问题，但是小心下载的智能体某个模块有后门或第三方集成，传送本地数据到某个地方，另外本地或远端也可能被黑客攻击导致数据泄密
   1. 关闭数据分析授权协议
   2. 数据脱敏，个人信息秘密等用xx代替
2. 执行任务安全，智能体根据用户需求生成的任务被执行时，有可能删除本地文件(本地部署)或通过第三方接口触发误判的动作(删邮件，发红包等等)
   1. 执行隔离，用户分级低权限用户运行或docker环境运行智能体等
   2. 测试账号，比如邮件微信等弄个测试账号

常见智能体：Hermes Agent(自进化、有记忆) 、 OpenClaw 、 龙虾/LobsterAI 、 AutoGen 、 CrewAI等

### 用户交互

用户交互入口这个比较容易理解，微信ClawBot(消息入口) 、 ChatBox 、 OpenWebUI(聊天UI) 、 各厂商app和网页交互入口

## 本地部署

### 推理引擎部署

```bash
# 1 创建低权限用户
# 1.1 创建一个普通用户，个人数据物理隔离
sudo useradd -s /bin/bash -m -d /home/ai-agent ai-agent
# 1.2 （可选）将当前的账户加入ai-agent组，方便日后共享文件
sudo usermod -aG ai-agent $USER
# 1.3. 设置该用户的密码
sudo passwd ai-agent
# 2 用低权限用户安装运行ollama
# 2.1 切换到新用户
su - ai-agent
# 2.2 安装Ollama，先下载ollama-linux-amd64.tar.zst（https://www.modelscope.cn/models/Lixiang/ollama-release/files）
tar -C .local -I zstd -xvf ollama-linux-amd64.tar.zst
# 2.3 检查是否安装成功，需要重启下终端
ollama --version
# 2.4 启动Ollama服务
ollama serve &
```

### 模型部署

模型可以从推理引擎拉去和管理，有些模型训练参数较多，本地硬件基础设施可能拉不起来

```bash
# 3 拉取一个测试模型
ollama pull *（模型名称）
# 3.1 验证Ollama是否正常工作
ollama run *（模型名称）
# 输入 "你好，请回复：Ollama 运行成功"，看到回复后，按Ctrl+D退出对话
```

### Hermes Agent部署

```bash
# 4 安装Hermes Agent
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

Hermes Agent配置项有的多，直接修改配置文件

.hermes/.env

```ini
API_SERVER_ENABLED=true
API_SERVER_HOST=127.0.0.1
API_SERVER_PORT=8642
API_SERVER_KEY=you-key
GATEWAY_ALLOW_ALL_USERS=true
```

.hermes/config.yaml

```yaml
# 1. 模型连接配置（必填）
model:
  provider: custom
  default: llama3.1:8b
  model_name: llama3.1:8b
  base_url: http://127.0.0.1:11434/v1

custom_providers:
- name: local-ollama
  base_url: http://127.0.0.1:11434/v1
  model: llama3.1:8b

# 2. 工具集配置
toolsets:
- terminal                              # 允许执行终端命令
- file                                  # 允许读写文件
- web                                   # 允许联网搜索
- memory                                # 允许读写记忆

# 3. 记忆系统配置
memory:
  memory_enabled: true
  user_profile_enabled: true

# 4. 终端执行配置
terminal:
  backend: local
  cwd: .                                # 命令执行时的工作目录
  timeout: 180                          # 命令超时秒数

```

会话方式启动hermes

```bash
hermes
```

启动api

```bash
hermes gateway run
```
