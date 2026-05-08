# Hermes Agent 文档

[TOC]

---

## 功能概述

Hermes Agent 包含丰富的功能集，远远超出基本的聊天能力。从持久化内存和文件感知上下文到浏览器自动化和语音对话，这些功能协同工作，使 Hermes 成为强大的自主助手。

### 核心功能

- **工具和工具集** — 工具是扩展代理能力的函数。它们被组织成逻辑工具集，可以按平台启用或禁用，涵盖 Web 搜索、终端执行、文件编辑、内存、委托等功能。
- **技能系统** — 按需加载的知识文档，代理可以在需要时调用。技能遵循渐进式披露模式以减少 token 使用量，并兼容 [agentskills.io](https://agentskills.io/specification) 开放标准。
- **持久化内存** — 跨会话持久的有界精选内存。Hermes 记住您的偏好、项目、环境以及通过 `MEMORY.md` 和 `USER.md` 学到的内容。
- **上下文文件** — Hermes 自动发现并加载项目上下文文件（`.hermes.md`、`AGENTS.md`、`CLAUDE.md`、`SOUL.md`、`.cursorrules`），这些文件塑造其在项目中的行为方式。
- **上下文引用** — 输入 `@` 跟引用名称可将文件、文件夹、git diff 和 URL 直接注入消息。Hermes 内联展开引用并自动追加内容。
- **检查点** — Hermes 在进行文件更改前自动快照工作目录，通过 `/rollback` 提供安全网回滚。

### 自动化

- **计划任务（Cron）** — 使用自然语言或 cron 表达式安排自动运行的任务。作业可以附加技能，交付结果到任何平台，支持暂停/恢复/编辑操作。
- **子代理委托** — `delegate_task` 工具生成具有隔离上下文、受限工具集和独立终端会话的子代理实例。默认运行 3 个并发子代理（可配置）用于并行工作流。
- **代码执行** — `execute_code` 工具让代理编写调用 Hermes 工具的 Python 脚本，通过沙箱 RPC 执行将多步骤工作流压缩为单个 LLM 调用。
- **事件钩子** — 在关键生命周期点运行自定义代码。网关钩子处理日志、警报和 webhook；插件钩子处理工具拦截、指标和防护栏。
- **批处理** — 跨数百或数千个提示并行运行 Hermes 代理，生成结构化的 ShareGPT 格式轨迹数据，用于训练数据生成或评估。

### 媒体与Web

- **语音模式** — 跨 CLI 和消息平台的完整语音交互。使用麦克风与代理对话，听取语音回复，在 Discord 语音频道中进行实时语音对话。
- **浏览器自动化** — 多后端完整浏览器自动化：Browserbase 云、Browser Use 云、本地 Chrome via CDP 或本地 Chromium。导航网站、填写表单和提取信息。
- **视觉与图像粘贴** — 多模态视觉支持。将图像从剪贴板粘贴到 CLI 中，使用任何支持视觉的模型让代理分析、描述或处理图像。
- **图像生成** — 使用 FAL.ai 从文本提示生成图像。支持 9 个模型；通过 `hermes tools` 选择。
- **语音与 TTS** — 跨所有消息平台的文本转语音输出和语音消息转录，提供 10 个原生提供商选项。

### 集成

- **MCP 集成** — 通过 stdio 或 HTTP 传输连接到任何 MCP 服务器。
- **提供商路由** — 精细控制哪些 AI 提供商处理您的请求。
- **后备提供商** — 当您的主要模型遇到错误时自动切换到备用 LLM 提供商。
- **凭证池** — 在同一提供商的多密钥间分配 API 调用。
- **内存提供商** — 插入外部内存后端用于跨会话用户建模和个性化。
- **API 服务器** — 将 Hermes 作为 OpenAI 兼容的 HTTP 端点暴露。
- **IDE 集成（ACP）** — 在 ACP 兼容的编辑器中使用 Hermes。
- **强化学习训练** — 从代理会话生成轨迹数据用于强化学习。

### 自定义

- **人格与 SOUL.md** — 完全可定制的代理人格。
- **皮肤与主题** — 自定义 CLI 的视觉呈现。
- **插件** — 无需修改核心代码即可添加自定义工具、钩子和集成。

---

## 工具与工具集

工具是扩展代理能力的函数。它们被组织成逻辑**工具集**，可以按平台启用或禁用。

### 可用工具

Hermes 附带了广泛的内置工具注册表，涵盖 Web 搜索、浏览器自动化、终端执行、文件编辑、内存、委托、强化学习训练、消息传递、Home Assistant 等。

                > **Honcho 跨会话内存** 作为内存提供商插件（`plugins/memory/honcho/`）提供，而非内置工具集。请参阅[插件](/docs/user-guide/features/plugins)了解安装方法。

### 终端后端

终端工具可在不同环境中执行命令：

| 后端 | 描述 | 用例 |
| --- | --- | --- |
| `local` | 在您本地机器上运行（默认） | 开发、可信任务 |
| `docker` | 隔离容器 | 安全、可重复性 |
| `ssh` | 远程服务器 | 沙箱、保持代理远离自己的代码 |
| `singularity` | HPC 容器 | 集群计算、无根 |
| `modal` | 云执行 | 无服务器、扩展 |
| `daytona` | 云沙箱工作区 | 持久远程开发环境 |
| `vercel_sandbox` | Vercel Sandbox 云微VM | 具有快照支持的文件系统持久化的云执行 |

#### Docker 后端

**一个持久容器，跨整个进程共享。** Hermes 在首次使用时启动一个长期容器（`docker run -d ... sleep 2h`），并通过 `docker exec` 将所有终端、文件和 `execute_code` 调用路由到同一容器。

### 后台进程管理

启动后台进程并管理它们：

```
terminal(command="pytest -v tests/", background=true)
# 返回: {"session_id": "proc_abc123", "pid": 12345}

# 然后使用 process 工具管理：
process(action="list")
process(action="poll", session_id="proc_abc123")
process(action="wait", session_id="proc_abc123")
process(action="log", session_id="proc_abc123")
process(action="kill", session_id="proc_abc123")
```

### Sudo 支持

如果命令需要 sudo，系统会提示您输入密码（会话期间缓存）。或在 `~/.hermes/.env` 中设置 `SUDO_PASSWORD`。

---

## 技能系统

技能是代理可以在需要时加载的按需知识文档。它们遵循**渐进式披露**模式以最小化 token 使用量，并兼容 [agentskills.io](https://agentskills.io/specification) 开放标准。

所有技能都位于 **`~/.hermes/skills/`** — 主目录和真实来源。新鲜安装时，捆绑的技能从仓库复制。Hub 安装和代理创建的技能也放在这里。代理可以修改或删除任何技能。

### 使用技能

每个已安装的技能都会自动作为斜杠命令可用：

```
# 在 CLI 或任何消息平台中：
/gif-search funny cats
/axolotl help me fine-tune Llama 3 on my dataset
/plan design a rollout for migrating our auth provider

# 仅使用技能名称加载它，让代理询问您需要什么：
/excalidraw
```

### 渐进式披露

技能使用 token 高效的加载模式：

```
Level 0: skills_list()           → [{name, description, category}, ...]   (~3k tokens)
Level 1: skill_view(name)        → Full content + metadata       (varies)
Level 2: skill_view(name, path)  → Specific reference file       (varies)
```

代理仅在实际需要时才加载完整技能内容。

### SKILL.md 格式

```
---
name: my-skill
description: Brief description of what this skill does
version: 1.0.0
platforms: [macos, linux]
metadata:
  hermes:
    tags: [python, automation]
    category: devops
---
# Skill Title

## When to Use
Trigger conditions for this skill.

## Procedure
1. Step one
2. Step two
```

### Skills Hub

浏览、搜索、安装和管理来自在线注册表、`skills.sh`、直接已知技能端点以及官方可选技能的技能。

```
hermes skills browse
hermes skills search kubernetes
hermes skills install openai/skills/k8s
hermes skills list --source hub
```

---

## 持久化内存

Hermes Agent 有跨会话持久的有界精选内存。这让它可以记住您的偏好、您的项目、您的环境以及它学到的内容。

### 工作原理

两个文件构成代理的内存：

| 文件 | 用途 | 字符限制 |
| --- | --- | --- |
| **MEMORY.md** | 代理的个人笔记 — 环境事实、约定、学到的内容 | 2,200 字符（约 800 tokens） |
| **USER.md** | 用户画像 — 您的偏好、沟通风格、期望 | 1,375 字符（约 500 tokens） |

两者都存储在 `~/.hermes/memories/` 中，在会话开始时作为冻结快照注入系统提示。代理通过 `memory` 工具管理自己的内存 — 可以添加、替换或删除条目。

### 内存工具操作

代理使用 `memory` 工具进行以下操作：

- **add** — 添加新的内存条目
- **replace** — 用更新后的内容替换现有条目（通过 `old_text` 子字符串匹配）
- **remove** — 移除不再相关的条目（通过 `old_text` 子字符串匹配）

没有 `read` 操作 — 内存在会话开始时自动注入系统提示。代理将记忆视为对话上下文的一部分。

### 容量管理

内存有严格的字符限制以保持系统提示 bounded：

| 存储 | 限制 | 典型条目 |
| --- | --- | --- |
| memory | 2,200 字符 | 8-15 条目 |
| user | 1,375 字符 | 5-10 条目 |

### 会话搜索

除了 MEMORY.md 和 USER.md，代理还可以使用 `session_search` 工具搜索过去的对话：

- 所有 CLI 和消息会话都存储在 SQLite（`~/.hermes/state.db`）中，具有 FTS5 全文搜索
- 搜索查询返回相关过去对话，并带有 Gemini Flash 摘要
- 代理可以找到几周前讨论的内容，即使它们不在活动内存中

---

## 上下文文件

Hermes 自动发现并加载项目上下文文件，这些文件塑造其在项目中的行为方式。

### 自动加载的上下文文件

- `.hermes.md` — 项目级指令和配置
- `AGENTS.md` — 代理行为规范
- `CLAUDE.md` — Claude 特定配置
- `SOUL.md` — 代理人格定义
- `.cursorrules` — Cursor IDE 规则

这些文件在会话开始时自动发现并注入系统提示。

---

## 上下文引用

输入 `@` 跟引用名称可将文件、文件夹、git diff 和 URL 直接注入消息。Hermes 内联展开引用并自动追加内容。

### 引用类型

- **@文件** — 注入文件内容
- **@文件夹** — 注入文件夹中的文件列表
- **@git** — 注入 git diff 或状态
- **@url** — 获取并注入 URL 内容

---

## 工具网关

付费 [Nous Portal](https://portal.nousresearch.com) 订阅者可以通过工具网关使用 Web 搜索、图像生成、TTS 和浏览器自动化 — 无需单独的 API 密钥。

### 启用

```
hermes model
```

或使用 `hermes tools` 配置各个工具。

---

## 计划任务（Cron）

使用自然语言或 cron 表达式安排自动运行的任务。作业可以附加技能，交付结果到任何平台，支持暂停/恢复/编辑操作。

### 基本使用

```
# 使用自然语言安排
/hermes schedule "每天早上8点发送天气摘要"

# 使用 cron 表达式
/hermes schedule "0 8 * * * weather summary"
```

---

## 子代理委托

`delegate_task` 工具生成具有隔离上下文、受限工具集和独立终端会话的子代理实例。

### 配置

默认运行 3 个并发子代理（可配置）用于并行工作流。

```
# 委托任务
delegate_task(task="分析销售数据", toolsets=["code_execution", "terminal"])
```

---

## 代码执行

`execute_code` 工具让代理编写调用 Hermes 工具的 Python 脚本，通过沙箱 RPC 执行将多步骤工作流压缩为单个 LLM 调用。

### 示例

```
execute_code(language="python", code="print('Hello, Hermes!')")
```

---

## 事件钩子

在关键生命周期点运行自定义代码。

- **网关钩子** — 处理日志、警报和 webhook
- **插件钩子** — 处理工具拦截、指标和防护栏

---

## 批处理

跨数百或数千个提示并行运行 Hermes 代理，生成结构化的 ShareGPT 格式轨迹数据，用于训练数据生成或评估。

---

## 语音模式

跨 CLI 和消息平台的完整语音交互。

- 使用麦克风与代理对话
- 听取语音回复
- 在 Discord 语音频道中进行实时语音对话

---

## 浏览器自动化

多后端完整浏览器自动化：

- **Browserbase 云**
- **Browser Use 云**
- **本地 Chrome via CDP**
- **本地 Chromium**

功能：导航网站、填写表单、提取信息。

---

## 视觉与图像粘贴

多模态视觉支持。将图像从剪贴板粘贴到 CLI 中，使用任何支持视觉的模型让代理分析、描述或处理图像。

---

## 图像生成

使用 FAL.ai 从文本提示生成图像。

### 支持的模型

- FLUX 2 Klein/Pro
- GPT-Image 1.5/2
- Nano Banana Pro
- Ideogram V3
- Recraft V4 Pro
- Qwen
- Z-Image Turbo

通过 `hermes tools` 选择模型。

---

## 语音与 TTS

跨所有消息平台的文本转语音输出和语音消息转录。

### 原生提供商

- Edge TTS（免费）
- ElevenLabs
- OpenAI TTS
- MiniMax
- Mistral Voxtral
- Google Gemini
- xAI
- NeuTTS
- KittenTTS
- Piper

以及任何本地 TTS CLI 的自定义命令提供商。

---

## MCP 集成

通过 stdio 或 HTTP 传输连接到任何 MCP 服务器。访问 GitHub、数据库、文件系统和内部 API 的外部工具，无需编写原生 Hermes 工具。

### 功能

- 每服务器工具过滤
- 采样支持
- 自动服务发现

---

## 提供商路由

精细控制哪些 AI 提供商处理您的请求。通过排序、白名单、黑名单和优先级排序优化成本、速度或质量。

---

## 后备提供商

当您的主要模型遇到错误时自动切换到备用 LLM 提供商，包括对视觉和压缩等辅助任务的独立后备。

---

## 凭证池

在同一提供商的多密钥间分配 API 调用。在速率限制或失败时自动轮换。

---

## 内存提供商

插入外部内存后端，超越内置 MEMORY.md 和 USER.md：

- Honcho
- OpenViking
- Mem0
- Hindsight
- Holographic
- RetainDB
- ByteRover
- Supermemory

外部提供商与内置内存**并行**运行（绝不替换），并增加知识图谱、语义搜索、自动事实提取和跨会话用户建模等能力。

---

## API 服务器

将 Hermes 作为 OpenAI 兼容的 HTTP 端点暴露。连接任何使用 OpenAI 格式的前端：

- Open WebUI
- LobeChat
- LibreChat
- 以及更多...

---

## IDE 集成（ACP）

在 ACP 兼容的编辑器中使用 Hermes：

- VS Code
- Zed
- JetBrains

聊天、工具活动、文件差异和终端命令在编辑器内呈现。

---

## 强化学习训练

从代理会话生成轨迹数据，用于强化学习和模型微调。

---

## 人格与 SOUL.md

完全可定制的代理人格。

`SOUL.md` 是主要的身份文件 — 系统提示中的第一项。您可以在每个会话中切换内置或自定义的 `/personality` 预设。

---

## 皮肤与主题

自定义 CLI 的视觉呈现：

- 横幅颜色
- 旋转器外观和动词
- 回复框标签
- 品牌文本
- 工具活动前缀

---

## 插件

无需修改核心代码即可添加自定义工具、钩子和集成。

### 插件类型

- **通用插件** — 工具/钩子
- **内存提供商** — 跨会话知识
- **上下文引擎** — 替代上下文管理

通过统一的 `hermes plugins` 交互式 UI 管理。

---

## Web 控制台

通过 Web 界面监控和管理 Hermes Agent。

---

## 内容策展人

自动化内容策展和工作流编排。

---

## 安装

使用一行安装命令，在两分钟内启动并运行 Hermes Agent。

### 快速安装

#### Linux / macOS / WSL2

```
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

```

#### Android / Termux

Hermes 现在也提供了 Termux 感知的安装路径。安装程序自动检测 Termux 并切换到经过测试的 Android 流程。

```
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

```

#### Windows

原生 Windows **不受支持**。请安装 [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install) 并从中运行 Hermes Agent。

### 安装程序的工作内容

安装程序自动处理所有内容 - 所有依赖项（Python、Node.js、ripgrep、ffmpeg）、仓库克隆、虚拟环境、全局 `hermes` 命令设置以及 LLM 提供商配置。

### 安装后

```
source ~/.bashrc
hermes

```

### 前提条件

唯一的前提条件是 **Git**。安装程序自动处理其他所有内容。

### 故障排除

| 问题 | 解决方案 |
| --- | --- |
| `hermes: command not found` | 重新加载 shell 或检查 PATH |
| `API key not set` | 运行 `hermes model` 配置您的提供商 |

---

## 快速入门

本指南帮助您从零到拥有一个能够经受真实使用考验的 Hermes 设置。安装、选择提供商、验证可用的聊天，并确切地知道当出现问题时该怎么做。

### 最快的路径

| 目标 | 首先这样做 | 然后这样做 |
| --- | --- | --- |
| 我只想让 Hermes 在我的机器上工作 | `hermes setup` | 运行一个真实的聊天并验证它是否响应 |
| 我已经知道我的提供商 | `hermes model` | 保存配置，然后开始聊天 |
| 我想要一个机器人或长期设置 | `hermes gateway setup` | 连接 Telegram、Discord、Slack 等 |

        > **经验法则：**如果 Hermes 无法完成普通聊天，请先不要添加更多功能。

### 1. 安装 Hermes Agent

```
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

```

### 2. 选择提供商

```
hermes model

```

| 提供商 | 这是什么 | 如何设置 |
| --- | --- | --- |
| **Nous Portal** | 基于订阅，零配置 | 通过 `hermes model` 进行 OAuth 登录 |
| **OpenAI Codex** | ChatGPT OAuth | 通过 `hermes model` 进行设备代码认证 |
| **Anthropic** | 直接使用 Claude 模型 | API 密钥 |
| **OpenRouter** | 跨多个模型的路由 | 输入您的 API 密钥 |

        > **最小上下文：64K tokens**Hermes Agent 需要至少 **64,000 tokens** 上下文的模型。

### 3. 运行您的第一个聊天

```
hermes
hermes --tui

```

### 4. 验证会话工作

```
hermes --continue
hermes -c

```

### 快速参考

| 命令 | 描述 |
| --- | --- |
| `hermes` | 开始聊天 |
| `hermes model` | 选择您的 LLM 提供商和模型 |
| `hermes setup` | 完整设置向导 |
| `hermes doctor` | 诊断问题 |
| `hermes update` | 更新到最新版本 |

---

## 学习路径

Hermes Agent 可以做很多事情 - CLI 助手、Telegram/Discord 机器人、任务自动化、RL 训练等等。

### 按经验水平

| 级别 | 目标 | 推荐阅读 | 时间 |
| --- | --- | --- | --- |
| **初学者** | 启动并运行，基本对话 | 安装 - 快速入门 - CLI - 配置 | 约1小时 |
| **中级** | 设置消息机器人，高级功能 | 会话 - 消息 - 工具 - 技能 - 记忆 - Cron | 约2-3小时 |
| **高级** | 构建自定义工具，RL训练 | 架构 - 添加工具 - 创建技能 - RL训练 | 约4-6小时 |

### 关键功能一览

| 功能 | 功能 |
| --- | --- |
| **工具** | 内置工具（文件I/O、搜索、shell等） |
| **技能** | 可安装的插件包 |
| **记忆** | 跨会话的持久记忆 |
| **MCP** | 连接外部工具服务器 |
| **Cron** | 安排重复的代理任务 |
| **RL训练** | 使用强化学习微调模型 |

---

## 更新与卸载

### 更新

```
hermes update

```

这会拉取最新代码、更新依赖项，并提示您配置新选项。

#### 完整预更新备份

```
hermes update --backup

```

#### 检查当前版本

```
hermes version

```

### 卸载

```
hermes uninstall

```

卸载程序让您选择保留配置文件（`~/.hermes/`）以供将来重新安装。

---

## 在 Android 上使用 Termux 运行 Hermes

这是通过 Termux 直接在 Android 手机上运行 Hermes Agent 的测试路径。

### 测试路径支持什么？

- Hermes CLI
- cron 支持
- PTY/后台终端支持
- Telegram 网关支持
- MCP 支持
- ACP 支持

### 一行安装程序

```
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

```

### 手动安装

```
pkg update
pkg install -y git python clang rust make pkg-config libffi openssl nodejs ripgrep ffmpeg
git clone --recurse-submodules https://github.com/NousResearch/hermes-agent.git
cd hermes-agent
python -m venv venv
source venv/bin/activate
python -m pip install -e '.[termux]' -c constraints-termux.txt
ln -sf "$PWD/venv/bin/hermes" "$PREFIX/bin/hermes"

```

---

## Nix 和 NixOS 设置

Hermes Agent 提供了一个 Nix flake，具有三个级别的集成：

| 级别 | 适用对象 | 获得的内容 |
| --- | --- | --- |
| **nix run / nix profile install** | 任何 Nix 用户 | 预构建二进制文件及所有依赖项 |
| **NixOS 模块（原生）** | NixOS 服务器部署 | 声明式配置，强化 systemd 服务 |
| **NixOS 模块（容器）** | 需要自我修改的代理 | 上述所有，外加持久化 Ubuntu 容器 |

### 快速开始

```
nix run github:NousResearch/hermes-agent -- setup
nix run github:NousResearch/hermes-agent -- chat
nix profile install github:NousResearch/hermes-agent

```

### NixOS 模块

```
# configuration.nix
{ config, ... }: {
  services.hermes-agent = {
    enable = true;
    settings.model.default = "anthropic/claude-sonnet-4";
    environmentFiles = [ config.sops.secrets."hermes-env".path ];
    addToSystemPackages = true;
  };
}

```

### 机密管理

        > **永远不要将 API 密钥放在 settings 或 environment 中。**

---

## CLI 界面

Hermes Agent 的 CLI 是一个完整的终端用户界面（TUI）- 不是 Web UI。它具有多行编辑、斜杠命令自动完成、对话历史、中断和重定向以及流式工具输出。

        > **提示：**Hermes 还附带了一个带有模态叠加、鼠标选择和非阻塞输入的现代 TUI。使用 `hermes --tui` 启动。

### 运行 CLI

```
hermes
hermes chat -q "Hello"
hermes chat --model "anthropic/claude-sonnet-4"
hermes chat --provider openrouter
hermes --continue
hermes --resume session_id
hermes chat --verbose

```

### 状态栏

```
⚕ claude-sonnet-4 │ 12.4K/200K │ [██████░░░░] 6% │ $0.06 │ 15m

```

| 元素 | 描述 |
| --- | --- |
| 模型名称 | 当前模型 |
| Token 计数 | 已用/最大上下文 |
| 上下文条 | 可视化填充指示器 |
| 成本 | 估计的会话成本 |
| 时长 | 已用会话时间 |

### 快捷键

| 按键 | 动作 |
| --- | --- |
| `Enter` | 发送消息 |
| `Alt+Enter` 或 `Ctrl+J` | 新行 |
| `Ctrl+B` | 开始/停止语音录制 |
| `Ctrl+C` | 中断代理 |
| `Ctrl+D` | 退出 |

### 斜杠命令

| 命令 | 功能 |
| --- | --- |
| `/help` | 显示所有可用命令 |
| `/tools` | 列出可用工具 |
| `/model` | 切换模型 |
| `/personality pirate` | 设置性格 |
| `/save` | 保存对话 |

### 后台会话

```
/background Analyze the logs in /var/log

```

---

## TUI

TUI 是 Hermes 的现代化前端 - 一个由与经典 CLI 相同的 Python 运行时支持的终端 UI。同样的代理，同样的会话，同样的斜杠命令。

这是推荐的方式来进行交互式运行 Hermes。

### 启动

```
hermes --tui
hermes --tui -c
hermes --tui --resume "session title"

```

### 为什么使用 TUI

- **即时首帧** - 横幅在应用加载完成前就绘制了
- **非阻塞输入** - 在会话准备好之前打字和排队消息
- **丰富的叠加** - 模型选择器、会话选择器渲染为模态面板
- **实时会话面板** - 工具和技能随着初始化逐步填充

### 斜杠命令

| 命令 | TUI 行为 |
| --- | --- |
| `/help` | 带分类命令的叠加 |
| `/sessions` | 模态会话选择器 |
| `/model` | 按提供商分组的模态模型选择器 |
| `/agents` | 带终止/暂停控制的实时子代理树 |

---

## 配置

所有设置都存储在 `~/.hermes/` 目录中以便轻松访问。

### 目录结构

```
~/.hermes/
├── config.yaml     # 设置
├── .env            # API 密钥和机密
├── auth.json       # OAuth 凭证
├── SOUL.md         # 主要代理身份
├── memories/       # 持久记忆
├── skills/        # 安装的技能
├── cron/          # 调度作业
└── sessions/      # 会话

```

### 管理配置

```
hermes config
hermes config edit
hermes config set KEY VAL
hermes config check
hermes config migrate

```

### 终端后端配置

Hermes 支持七种终端后端：

| 后端 | 命令运行位置 | 适用场景 |
| --- | --- | --- |
| **local** | 直接运行在您的机器上 | 开发、受信任的用户 |
| **docker** | 单个持久 Docker 容器 | 安全沙箱、CI/CD |
| **ssh** | 通过 SSH 的远程服务器 | 远程开发 |
| **modal** | Modal 云沙箱 | 临时云计算 |
| **daytona** | Daytona 工作区 | 托管云开发环境 |

### 上下文压缩

```
compression:
  enabled: true
  threshold: 0.50
auxiliary:
  compression:
    model: "google/gemini-3-flash-preview"

```

---

## 配置模型

Hermes 使用两种模型槽位：

- **主模型** - 代理思考用的模型
- **辅助模型** - 上下文压缩、视觉、网页摘要等任务

### 设置主模型

```
hermes model

```

| 提供商 | 这是什么 | 如何设置 |
| --- | --- | --- |
| **Nous Portal** | 基于订阅，零配置 | OAuth 登录 |
| **OpenAI Codex** | ChatGPT OAuth | 设备代码认证 |
| **Anthropic** | 直接使用 Claude 模型 | API 密钥 |
| **OpenRouter** | 跨多个模型路由 | API 密钥 |

        > **最小上下文：64K tokens**

### 设置辅助模型

点击 **Show auxiliary** 显示八个任务槽位：

| 任务 | 何时覆盖 |
| --- | --- |
| **Title Gen** | 几乎总是，使用便宜的模型 |
| **Vision** | 当主要模型没有视觉时 |
| **Compression** | 使用便宜的模型进行总结 |

---

## 会话

Hermes Agent 自动将每个对话保存为会话。会话支持对话恢复、跨会话搜索和完整的对话历史管理。

### 会话工作原理

每个对话——无论是来自 CLI、Telegram、Discord、Slack、WhatsApp、Signal、Matrix、Teams 还是任何其他消息平台——都作为具有完整消息历史的会话存储。两个互补系统跟踪会话：

1. **SQLite 数据库**（`~/.hermes/state.db`）——带 FTS5 全文搜索的结构化会话元数据
1. **JSONL 转录本**（`~/.hermes/sessions/`）——原始对话转录本，包括工具调用（网关）

SQLite 数据库存储：

- 会话 ID、源平台、用户 ID
- **会话标题**（唯一的、易读的名称）
- 模型名称和配置
- 系统提示快照
- 完整消息历史（角色、内容、工具调用、工具结果）
- Token 计数（输入/输出）
- 时间戳（started_at、ended_at）
- 父会话 ID（用于压缩触发的会话拆分）

### 会话来源

每个会话都标有其源平台：

| 来源 | 描述 |
| --- | --- |
| `cli` | 交互式 CLI（`hermes` 或 `hermes chat`） |
| `telegram` | Telegram Messenger |
| `discord` | Discord 服务器/私信 |
| `slack` | Slack 工作区 |
| `whatsapp` | WhatsApp Messenger |
| `signal` | Signal Messenger |
| `matrix` | Matrix 房间和私信 |
| `mattermost` | Mattermost 频道 |
| `email` | 电子邮件（IMAP/SMTP） |
| `sms` | 通过 Twilio 的 SMS |
| `dingtalk` | 钉钉 Messenger |
| `feishu` | 飞书/Lark Messenger |
| `wecom` | 企业微信 |
| `weixin` | 个人微信 |
| `bluebubbles` | 通过 BlueBubbles macOS 服务器的 Apple iMessage |
| `qqbot` | 通过官方 API v2 的 QQ 机器人 |
| `homeassistant` | Home Assistant 对话 |
| `webhook` | 传入的 Webhook |
| `api-server` | API 服务器请求 |
| `acp` | ACP 编辑器集成 |
| `cron` | 计划的 cron 作业 |
| `batch` | 批处理运行 |

### CLI 会话恢复

使用 `--continue` 或 `--resume` 从 CLI 恢复之前的对话：

#### 继续上次会话

```
# 恢复最近的 CLI 会话
hermes --continue
hermes -c
# 或使用 chat 子命令
hermes chat --continue
hermes chat -c

```

这会从 SQLite 数据库中查找最近的 `cli` 会话并加载其完整对话历史。

#### 按名称恢复

```
# 按名称恢复命名的会话
hermes -c "my project"
# 如果有世系变体（my project、my project #2、my project #3），
# 这会自动恢复最近的一个
hermes -c "my project"   # → 恢复 "my project #3"

```

#### 恢复特定会话

```
# 按 ID 恢复特定会话
hermes --resume 20250305_091523_a1b2c3d4
hermes -r 20250305_091523_a1b2c3d4
# 按标题恢复
hermes --resume "refactoring auth"
# 或使用 chat 子命令
hermes chat --resume 20250305_091523_a1b2c3d4

```

会话 ID 在退出 CLI 会话时显示，也可以通过 `hermes sessions list` 找到。

### 会话命名

为会话指定易读的标题，以便轻松查找和恢复。

#### 自动生成标题

Hermes 在第一次交换后自动为每个会话生成简短的描述性标题（3-7 个单词）。这在后台线程中使用快速辅助模型运行，因此不会增加延迟。当使用 `hermes sessions list` 或 `hermes sessions browse` 浏览会话时会看到自动生成的标题。

自动标题生成每个会话只会触发一次，如果您已经手动设置了标题，则会跳过。

#### 手动设置标题

```
/title my research project

```

标题会立即应用。如果会话尚未在数据库中创建（例如您在发送第一条消息之前运行 `/title`），它会被排队并在会话开始后应用。

#### 会话管理命令

```
# 列出最近的会话（默认：最近 20 个）
hermes sessions list
# 按平台过滤
hermes sessions list --source telegram
# 显示更多会话
hermes sessions list --limit 50

```

#### 导出会话

```
# 将所有会话导出为 JSONL 文件
hermes sessions export backup.jsonl
# 导出特定平台的会话
hermes sessions export telegram-history.jsonl --source telegram
# 导出单个会话
hermes sessions export session.jsonl --session-id 20250305_091523_a1b2c3d4

```

#### 删除会话

```
# 删除特定会话（带确认）
hermes sessions delete 20250305_091523_a1b2c3d4
# 无需确认删除
hermes sessions delete 20250305_091523_a1b2c3d4 --yes

```

#### 重命名会话

```
# 设置或更改会话标题
hermes sessions rename 20250305_091523_a1b2c3d4 "refactoring auth module"

```

#### 修剪旧会话

```
# 删除超过 90 天的已结束会话（默认）
hermes sessions prune
# 自定义年龄阈值
hermes sessions prune --older-than 30
# 只修剪特定平台的会话
hermes sessions prune --source telegram --older-than 60
# 跳过确认
hermes sessions prune --older-than 30 --yes

```

        > **注意：**修剪只删除**已结束**的会话（已明确结束或自动重置的会话）。活动会话永远不会被修剪。

### 会话统计

```
hermes sessions stats

```

### 会话搜索工具

代理有一个内置的 `session_search` 工具，使用 SQLite 的 FTS5 引擎对所有过去的对话执行全文搜索。

#### 工作原理

1. FTS5 搜索按相关性排序的匹配消息
1. 按会话分组结果，取前 N 个唯一会话（默认 3 个）
1. 加载每个会话的对话，截断到以匹配为中心的约 100K 字符
1. 发送到快速摘要模型以获得集中摘要
1. 返回每个会话的摘要及元数据和周围上下文

### 存储位置

| 内容 | 路径 | 描述 |
| --- | --- | --- |
| SQLite 数据库 | `~/.hermes/state.db` | 所有会话元数据 + 带 FTS5 的消息 |
| 网关转录本 | `~/.hermes/sessions/` | 每个会话的 JSONL 转录本 + sessions.json 索引 |
| 网关索引 | `~/.hermes/sessions/sessions.json` | 将会话键映射到活动会话 ID |

### 会话过期和清理

#### 自动清理

- 网关会话根据配置的 reset 策略自动重置
- 在重置之前，代理会从即将过期的会话中保存重要记忆或技能
- 选择加入自动修剪：当 `sessions.auto_prune` 为 `true` 时，超过 `sessions.retention_days`（默认 90）的已结束会话会在 CLI/网关启动时被修剪

---

## 多代理配置：运行多个 Agent

在同一台机器上运行多个独立的 Hermes Agent——每个都有自己独立的配置、API 密钥、记忆、会话、技能和网关状态。

### 什么是配置？

配置是一个独立的 Hermes 主目录。创建配置后，它会自动成为自己的命令。创建一个名为 `coder` 的配置，您立即拥有 `coder chat`、`coder setup`、`coder gateway start` 等命令。

### 快速开始

```
hermes profile create coder       # 创建配置 + "coder" 命令别名
coder setup                       # 配置 API 密钥和模型
coder chat                        # 开始聊天

```

就这样。`coder` 现在是一个独立的 Hermes 配置，有自己的配置、记忆和状态。

### 创建配置

#### 空白配置

```
hermes profile create mybot

```

#### 仅克隆配置（`--clone`）

```
hermes profile create work --clone

```

#### 克隆所有内容（`--clone-all`）

```
hermes profile create backup --clone-all

```

### 使用配置

#### 命令别名

```
coder chat                    # 与 coder 代理聊天
coder setup                   # 配置 coder 的设置
coder gateway start           # 启动 coder 的网关
coder doctor                  # 检查 coder 的健康状态
coder skills list             # 列出 coder 的技能

```

#### `-p` 标志

```
hermes -p coder chat
hermes --profile=coder doctor
hermes chat -p coder -q "hello"

```

#### 持久默认配置（`hermes profile use`）

```
hermes profile use coder
hermes chat                   # 现在指向 coder
hermes profile use default    # 切换回默认

```

### 配置与工作区与沙箱

配置通常与工作区或沙箱混淆，但它们是不同的：

- **配置**为 Hermes 提供自己的状态目录：`config.yaml`、`.env`、`SOUL.md`、会话、记忆、技能、cron 作业和网关状态。
- **工作区**或**工作目录**是终端命令启动的位置。这由 `terminal.cwd` 单独控制。
- **沙箱**是限制文件系统访问的东西。配置**不会**对代理进行沙箱处理。

### 运行网关

```
coder gateway start           # 启动 coder 的网关
assistant gateway start       # 启动 assistant 的网关（独立进程）

```

### 管理配置

```
hermes profile list           # 显示所有配置及状态
hermes profile show coder     # 显示一个配置的详细信息
hermes profile rename coder dev-bot   # 重命名（更新别名 + 服务）
hermes profile export coder   # 导出到 coder.tar.gz
hermes profile import coder.tar.gz   # 从存档导入

```

### 删除配置

```
hermes profile delete coder

```

        > **注意：**您不能删除默认配置（`~/.hermes`）。要删除所有内容，请使用 `hermes uninstall`。

---

## Docker

Docker 与 Hermes Agent 有两种不同的交互方式：

1. **在 Docker 中运行 Hermes**——代理本身在容器内运行（本章重点）
1. **Docker 作为终端后端**——代理在主机上运行，但在单个持久 Docker 沙箱容器内执行每个命令（请参阅[配置 → Docker 后端](#user-guide-configuration)）

本页面涵盖选项 1。容器将所有用户数据（配置、API 密钥、会话、技能、记忆）存储在从主机挂载到 `/opt/data` 的单个目录中。镜像本身是无状态的，可以通过拉取新版本进行升级而不会丢失任何配置。

### 快速开始

```
mkdir -p ~/.hermes
docker run -it --rm \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent setup

```

### 以网关模式运行

```
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/opt/data \
  -p 8642:8642 \
  nousresearch/hermes-agent gateway run

```

### 运行仪表板

```
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/opt/data \
  -p 8642:8642 \
  -p 9119:9119 \
  -e HERMES_DASHBOARD=1 \
  nousresearch/hermes-agent gateway run

```

### 持久卷

`/opt/data` 卷是所有 Hermes 状态的单一真相来源。它映射到主机的 `~/.hermes/` 目录，包含：

| 路径 | 内容 |
| --- | --- |
| `.env` | API 密钥和密钥 |
| `config.yaml` | 所有 Hermes 配置 |
| `SOUL.md` | 代理个性/身份 |
| `sessions/` | 对话历史 |
| `memories/` | 持久记忆存储 |
| `skills/` | 已安装的技能 |
| `cron/` | 计划作业定义 |

        > **警告：**永远不要同时对同一个数据目录运行两个 Hermes **网关**容器——会话文件和记忆存储不支持并发写入访问。

### 多配置支持

在 Docker 中运行时不推荐使用 Hermes 内置的多配置功能。建议的模式是**每个配置一个容器**，每个容器将自己的主机目录绑定挂载为 `/opt/data`。

### 资源限制

| 资源 | 最小 | 推荐 |
| --- | --- | --- |
| 内存 | 1 GB | 2-4 GB |
| CPU | 1 核 | 2 核 |
| 磁盘（数据卷） | 500 MB | 2+ GB（随会话/技能增长） |

### 升级

```
docker pull nousresearch/hermes-agent:latest
docker rm -f hermes
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent gateway run

```

### 故障排除

#### 容器立即退出

检查日志：`docker logs hermes`。常见原因：

- 缺少或无效的 `.env` 文件——首先以交互方式运行以完成设置
- 端口冲突（如果使用暴露的端口运行）

#### "权限被拒绝"错误

容器的入口点通过 `gosu` 将权限降级到非 root `hermes` 用户（UID 10000）。如果您主机的 `~/.hermes/` 由不同的 UID 拥有，请设置 `HERMES_UID`/`HERMES_GID` 来匹配您的主机用户，或确保数据目录可写。

#### 浏览器工具不工作

Playwright 需要共享内存。将 `--shm-size=1g` 添加到您的 Docker 运行命令。

---

## 安全

Hermes Agent 采用纵深防御安全模型设计。本页涵盖每个安全边界——从命令批准到容器隔离再到消息平台上的用户授权。

### 概述

安全模型有七层：

1. **用户授权**——谁可以与代理交谈（允许列表、DM 配对）
1. **危险命令批准**——破坏性操作的人机交互
1. **容器隔离**——使用强化设置的 Docker/Singularity/Modal 沙箱
1. **MCP 凭证过滤**——MCP 子进程的环境变量隔离
1. **上下文文件扫描**——项目文件中的提示注入检测
1. **跨会话隔离**——会话无法访问彼此的数据或状态；cron 作业存储路径经过强化以防止路径遍历攻击
1. **输入清理**——终端工具后端中的工作目录参数根据白名单进行验证以防止 shell 注入

### 危险命令批准

在执行任何命令之前，Hermes 会根据危险模式策划列表对其进行检查。如果找到匹配项，用户必须明确批准。

#### 批准模式

批准系统支持三种模式，通过 `~/.hermes/config.yaml` 中的 `approvals.mode` 进行配置：

| 模式 | 行为 |
| --- | --- |
| **manual**（默认） | 始终提示用户批准危险命令 |
| **smart** | 使用辅助 LLM 评估风险。低风险命令自动批准。真正危险的命令自动拒绝。不确定的情况升级到手动提示。 |
| **off** | 禁用所有批准检查——相当于使用 `--yolo` 运行。所有命令执行时不会有提示。 |

        > **警告：**设置 `approvals.mode: off` 会禁用所有安全提示。仅在可信环境（CI/CD、容器等）中使用。

#### YOLO 模式

YOLO 模式绕过当前会话的**所有**危险命令批准提示。可以通过三种方式激活：

1. **CLI 标志**：使用 `hermes --yolo` 或 `hermes chat --yolo` 启动会话
1. **斜杠命令**：在会话中输入 `/yolo` 来切换开/关
1. **环境变量**：设置 `HERMES_YOLO_MODE=1`

#### 硬性阻止列表（始终在线的底线）

有些命令如此灾难性——不可逆的文件系统擦除、fork 炸弹、直接块设备写入——无论以下情况如何，Hermes 都拒绝运行它们：

- `--yolo` / `/yolo` 切换开启
- `approvals.mode: off`
- 在无头 `approve` 模式下运行的 cron 作业
- 用户明确点击"始终允许"

阻止列表是 `--yolo` 之下的底线。它在批准层看到命令**之前**触发，并且没有覆盖标志。当前覆盖的模式（不完整；与 `tools/approval.py::UNRECOVERABLE_BLOCKLIST` 保持同步）：

| 模式 | 为什么是硬性的 |
| --- | --- |
| `rm -rf /` 及其明显变体 | 擦除文件系统根目录 |
| `:(){ :|:& };:`（bash fork 炸弹） | 使主机挂起直到重启 |
| `dd if=/dev/zero of=/dev/sd*` | 清零物理磁盘 |

### 用户授权（网关）

运行消息网关时，Hermes 通过分层授权系统控制谁可以与机器人交互。

#### 授权检查顺序

`_is_user_authorized()` 方法按此顺序检查：

1. **每个平台的全部允许标志**（例如 `DISCORD_ALLOW_ALL_USERS=true`）
1. **DM 配对批准列表**（通过配对代码批准的用户）
1. **平台特定的允许列表**（例如 `TELEGRAM_ALLOWED_USERS=12345,67890`）
1. **全局允许列表**（`GATEWAY_ALLOWED_USERS=12345,67890`）
1. **全局全部允许**（`GATEWAY_ALLOW_ALL_USERS=true`）
1. **默认：拒绝**

#### DM 配对系统

Hermes 包含一个基于代码的配对系统以获得更灵活的授权。不需要预先提供用户 ID，未知用户会收到一次性配对代码，机器人所有者通过 CLI 批准。

**工作原理：**

1. 未知用户向机器人发送 DM
1. 机器人回复 8 字符配对代码
1. 机器人所有者在 CLI 上运行 `hermes pairing approve &lt;platform&gt; &lt;code&gt;`
1. 用户永久批准在该平台上使用

### 容器隔离

使用 `docker` 终端后端时，Hermes 为每个容器应用严格的安全强化。

#### Docker 安全标志

每个容器都带有这些标志运行（定义在 `tools/environments/docker.py` 中）：

```
_SECURITY_ARGS = [
    "--cap-drop", "ALL",                          # 删除所有 Linux 权限
    "--cap-add", "DAC_OVERRIDE",                  # Root 可以写入绑定挂载的目录
    "--cap-add", "CHOWN",                         # 包管理器需要文件所有权
    "--cap-add", "FOWNER",                        # 包管理器需要文件所有权
    "--security-opt", "no-new-privileges",         # 阻止权限提升
    "--pids-limit", "256",                         # 限制进程数
    "--tmpfs", "/tmp:rw,nosuid,size=512m",         # 大小有限的 /tmp
    "--tmpfs", "/var/tmp:rw,noexec,nosuid,size=256m",  # 不可执行的 /var/tmp
    "--tmpfs", "/run:rw,noexec,nosuid,size=64m",   # 不可执行的 /run
]

```

### 终端后端安全比较

| 后端 | 隔离 | 危险命令检查 | 适合场景 |
| --- | --- | --- | --- |
| **local** | 无——在主机上运行 | ✅ 是 | 开发、可信用户 |
| **ssh** | 远程机器 | ✅ 是 | 在单独服务器上运行 |
| **docker** | 容器 | ❌ 跳过（容器是边界） | 生产网关 |
| **modal** | 云沙箱 | ❌ 跳过 | 可扩展云隔离 |

### SSRF 保护

所有支持 URL 的工具（网络搜索、网络提取、视觉、浏览器）在获取之前验证 URL，以防止服务端请求伪造（SSRF）攻击。阻止的地址包括：

- **私有网络**（RFC 1918）：`10.0.0.0/8`、`172.16.0.0/12`、`192.168.0.0/16`
- **回环**：`127.0.0.0/8`、`::1`
- **链路本地**：`169.254.0.0/16`

### 上下文文件注入保护

上下文文件（AGENTS.md、.cursorrules、SOUL.md）在包含在系统提示之前会被扫描提示注入。扫描程序检查：

- 忽略/忽视先前指令的指令
- 带有可疑关键字的隐藏 HTML 注释
- 尝试读取密钥（`.env`、`credentials`、`.netrc`）
- 通过 `curl` 窃取凭证
- 不可见的 Unicode 字符（零宽空格、双向覆盖）

---

## Windows (WSL2) 指南

Hermes Agent 在 **Linux** 和 **macOS** 上开发和测试。原生的 Windows 不受支持——在 Windows 上，您需要在 **WSL2**（Windows Subsystem for Linux，版本 2）中运行 Hermes。这意味着同时有两个计算机在起作用：您的 Windows 主机，以及由 WSL 管理的 Linux VM。

### 为什么是 WSL2（而不是"直接用 Windows"）

Hermes 假设 POSIX 环境：`fork`、`/tmp`、UNIX 套接字、信号语义、PTY 支持的终端、`bash`/`zsh` 等 shell，以及 `rg`、`git`、`ffmpeg` 等在 Linux 上行为方式的工具。为原生 Windows 重写需要完整的移植——WSL2 在轻量级 VM 中为您提供真正的 Linux 内核，而不是。

### 安装 WSL2

从**管理员 PowerShell** 或 Windows Terminal：

```
wsl --install

```

在全新的 Windows 10 22H2+ 或 Windows 11 电脑上，这会安装 WSL2 内核、虚拟机平台功能和默认的 Ubuntu 发行版。提示时重启。重启后 Ubuntu 会打开并要求提供 Linux 用户名 + 密码——这是一个**新的 Linux 用户**，与您的 Windows 账户无关。

### 验证 WSL2

```
wsl --list --verbose

```

您应该看到 `VERSION 2`。如果发行版显示 `VERSION 1`，请转换它：

```
wsl --set-version Ubuntu 2
wsl --set-default-version 2

```

        > **注意：**Hermes 在 WSL1 上不能可靠地工作——WSL1 动态转换 Linux 系统调用，某些行为（procfs、信号、网络）与真正的 Linux 不同。

### 启用 systemd（推荐）

```
sudo tee /etc/wsl.conf >/dev/null ——从网络角度来看，它们是两个独立的主机。

#### 情况 1——WSL 中的 Hermes 与 Windows 上的服务通信

最常见的情况：您在 Windows** 上运行 **Ollama、LM Studio 或 llama-server**，而 Hermes（在 WSL 内）需要访问它。

**Windows 11 22H2+：**启用镜像网络模式（`networkingMode=mirrored` 在 `%USERPROFILE%\.wslconfig` 中，然后 `wsl --shutdown`）。然后 `localhost` 在两个方向都可以工作。

**Windows 10 或更早版本：**使用 Windows 主机的 IP（WSL 虚拟网络的默认网关），并确保服务器在 Windows 上绑定到 `0.0.0.0`，而不仅仅是 `127.0.0.1`。Windows 防火墙通常还需要该端口的规则。

### 在 Windows 上长期运行 Hermes 服务

#### 在 WSL 内使用 systemd（推荐）

如果您按照设置部分启用了 systemd，`hermes gateway` 和 API 服务器的工作方式与在任何其他 Linux 机器上相同。使用网关设置向导：

```
hermes gateway setup

```

它会提供安装 systemd 用户单元的选项，以便网关在 WSL 启动时自动启动。

### 常见陷阱

| 问题 | 解决方案 |
| --- | --- |
| 连接到 Windows 托管的 Ollama/LM Studio "连接被拒绝" | 服务器可能绑定到 `127.0.0.1`，需要 `0.0.0.0`（Ollama：`OLLAMA_HOST=0.0.0.0`），或者您缺少防火墙规则。 |
| 在仓库中对 `git status`/`hermes chat` 极慢 | 您可能在 `/mnt/c/...` 下工作。将仓库移到 `~/code/...`（Linux 端）。速度快一个数量级。 |
| 脚本上 `bad interpreter: /bin/bash^M` | CRLF 行尾来自 Windows 编辑器。`dos2unix script.sh`，并在 WSL git 配置中设置 `core.autocrlf input`。 |
| 安装程序运行后找不到 `hermes` | 安装程序通过 `~/.bashrc` 将 `~/.local/bin` 添加到 shell 的 PATH。您需要 `source ~/.bashrc`（或打开一个新终端）才能在当前会话中生效。 |
| 磁盘空间不足 | WSL2 将其 VM 磁盘存储为稀疏 VHDX。它会增长，但删除文件时不会自动缩小。要回收空间：`wsl --shutdown`，然后从管理员 PowerShell 运行 `Optimize-VHD -Path &lt;path-to-ext4.vhdx&gt; -Mode Full`。 |

---

## Git Worktrees

Hermes Agent 常用于大型、长期存在的仓库。当您想要：

- 在同一项目上运行**多个并行代理**，或
- 将实验性重构与主分支隔离

Git **worktrees** 是为每个代理提供自己独立 checkout 而不复制整个仓库的最安全方式。

### 为什么将 Worktrees 与 Hermes 一起使用？

Hermes 将**当前工作目录**视为项目根目录：

- CLI：从运行 `hermes` 或 `hermes chat` 的目录
- 消息网关：由 `MESSAGING_CWD` 设置的目录

如果您在**同一个 checkout** 中运行多个代理，它们的更改可能会相互干扰。

使用 worktrees，每个代理获得：

- 自己的**分支和工作目录**
- 自己的**Checkpoint Manager 历史记录**用于 `/rollback`

### 快速开始：创建 Worktree

```
# 从主仓库根目录
cd /path/to/your/repo
# 在 ../repo-feature 中创建新分支和 worktree
git worktree add ../repo-feature feature/hermes-experiment

```

这会创建：

- 新目录：`../repo-feature`
- 新分支：`feature/hermes-experiment` 在该目录中检出

```
cd ../repo-feature
# 在 worktree 中启动 Hermes
hermes

```

### 并行运行多个代理

```
cd /path/to/your/repo
git worktree add ../repo-experiment-a feature/hermes-a
git worktree add ../repo-experiment-b feature/hermes-b

```

在单独的终端中：

```
# 终端 1
cd ../repo-experiment-a
hermes
# 终端 2
cd ../repo-experiment-b
hermes

```

### 安全清理 Worktrees

当您完成实验时：

1. 决定是保留还是丢弃工作。
1. 如果要保留：像往常一样将分支合并到主分支。
1. 删除 worktree：

```
cd /path/to/your/repo
git worktree remove ../repo-feature

```

### 最佳实践

- **每个 Hermes 实验一个 worktree**为每个实质性更改创建专用分支/worktree。这可以保持差异集中且 PR 小而可审查。
- **用实验名称命名分支**例如 `feature/hermes-checkpoints-docs`、`feature/hermes-refactor-tests`。
- **经常提交**使用 git 提交获取高级里程碑。使用检查点和 `/rollback` 作为工具驱动编辑之间的安全网。
- **当使用 worktrees 时，避免从裸仓库根目录运行 Hermes**请改用 worktree 目录，以便每个代理都有明确的范围。

### 使用 `hermes -w`（自动 Worktree 模式）

Hermes 有一个内置的 `-w` 标志，**自动创建一个带自己分支的 disposable git worktree**。您不需要手动设置 worktrees——只需进入您的仓库并运行：

```
cd /path/to/your/repo
hermes -w

```

Hermes 会：

- 在仓库内的 `.worktrees/` 下创建一个临时 worktree。
- 检出隔离分支（例如 `hermes/hermes-&lt;hash&gt;`）。
- 在该 worktree 内运行完整的 CLI 会话。

这也不需要手动设置 worktrees。只需进入您的仓库并运行：

---

## 技能中心 (Skills Hub)

从 **4 个注册表**中探索、搜索和安装共 **674** 个技能

            **89** 内置
            **64** 可选
            **521** 社区
            **17** 分类

注册表: All 674 | Built-in 89 | Optional 64 | Anthropic 16 | LobeHub 505

### 分类

            📋 全部技能 674
            📦 其他 347
            💻 软件开发 74
            🎨 创意 69
            🧪 MLOps 40
            🔍 研究 39
            🌍 翻译 24
            ✅ 生产力 17
            🎮 游戏 11
            ❤ 健康 8
            🎵 媒体 7
            📱 社交媒体 7
            🤖 AI 代理 6
            💻 GitHub 6
            🔒 安全 6
            ⚙ DevOps 5
            🍎 Apple 4
            📦 文案 4

### Apple

#### apple-notes

                ✓ 内置

通过 memo CLI 管理 Apple Notes：创建、搜索、编辑。

                Apple 🍎 macOS

#### apple-reminders

                ✓ 内置

通过 remindctl 管理 Apple 提醒事项：添加、列表、完成。

                Apple 🍎 macOS

#### findmy

                ✓ 内置

通过 macOS 上的 FindMy.app 追踪 Apple 设备/AirTags。

                Apple 🍎 macOS

#### imessage

                ✓ 内置

通过 macOS 上的 imsg CLI 发送和接收 iMessage/SMS。

                Apple 🍎 macOS

### AI 代理

#### claude-code

                ✓ 内置

通过 Claude Code CLI 委托编码任务（功能、PR）。

                AI 代理 🤖

#### codex

                ✓ 内置

通过 OpenAI Codex CLI 委托编码任务（功能、PR）。

                AI 代理 🤖

#### hermes-agent

                ✓ 内置

配置、扩展或贡献 Hermes Agent。

                AI 代理 🤖

#### opencode

                ✓ 内置

通过 OpenCode CLI 委托编码任务（功能、PR 审查）。

                AI 代理 🤖

### 创意

#### architecture-diagram

                ✓ 内置

深色主题的 SVG 架构/云/基础设施图作为 HTML。

                创意 🎨

#### ascii-art

                ✓ 内置

ASCII 艺术：pyfiglet、cowsay、boxes、image-to-ascii。

                创意 🎨

#### ascii-video

                ✓ 内置

ASCII 视频：将视频/音频转换为彩色 ASCII MP4/GIF。

                创意 🎨

#### baoyu-comic

                ✓ 内置

知识漫画：教育、传记、教程。

                创意 🎨

#### baoyu-infographic

                ✓ 内置

信息图：21 种布局 × 21 种样式。

                创意 🎨

#### claude-design

                ✓ 内置

设计一次性 HTML 产物（着陆页、演示稿、原型）。

                创意 🎨

#### comfyui

                ✓ 内置

使用 ComfyUI 生成图像、视频和音频 — 安装、启动、管理节点/模型、使用参数注入运行工作流。使用官方 comfy-cli 进行生命周期管理和直接 REST/WebSocket API 执行。

                创意 🎨 🍎 macOS 🐧 Linux 🪟 Windows

#### design-md

                ✓ 内置

编写/验证/导出 Google 的 DESIGN.md token 规范文件。

                创意 🎨

#### excalidraw

                ✓ 内置

手绘 Excalidraw JSON 图表（架构、流程、序列）。

                创意 🎨

#### humanizer

                ✓ 内置

人性化文本：去除 AI 痕迹，添加真实语气。

                创意 🎨

#### ideation

                ✓ 内置

通过创意约束生成项目创意。

                创意 🎨

#### manim-video

                ✓ 内置

Manim CE 动画：3Blue1Brown 数学/算法视频。

                创意 🎨

#### p5js

                ✓ 内置

p5.js 草图：生成艺术、着色器、交互、3D。

                创意 🎨

#### pixel-art

                ✓ 内置

像素艺术：各个时代的调色板（NES、Game Boy、PICO-8）。

                创意 🎨

#### popular-web-designs

                ✓ 内置

54 个真实设计系统（Stripe、Linear、Vercel）作为 HTML/CSS。

                创意 🎨

#### pretext

                ✓ 内置

使用 @chenglou/pretext 构建创意浏览器演示 — 用于 ASCII 艺术、围绕障碍物的字体流、几何文本游戏、动态排版和文本驱动的生成艺术的 DOM-free 文本布局。默认生成单文件 HTML 演示。

                创意 🎨

#### sketch

                ✓ 内置

一次性 HTML 模拟：2-3 个设计变体进行比较。

                创意 🎨

#### songwriting-and-ai-music

                ✓ 内置

歌曲创作技巧和 Suno AI 音乐提示词。

                创意 🎨

#### touchdesigner-mcp

                ✓ 内置

通过 twozero MCP 控制运行中的 TouchDesigner 实例 — 创建运算符、设置参数、连接线路、执行 Python、构建实时视觉效果。36 个原生工具。

                创意 🎨

### DevOps

#### kanban-orchestrator

                ✓ 内置

分解手册 + 专家名册约定 + 反诱惑规则，用于通过 Kanban 将工作路由的编排器配置文件。"不要自己做工作"规则和基本生命周期被自动注入每个 kanban 工作者的系统提示；当您专门扮演编排器角色时，此技能是更深入的手册。

                DevOps ⚙

#### kanban-worker

                ✓ 内置

Hermes Kanban 工作者的陷阱、示例和边缘情况。生命周期本身作为 KANBAN_GUIDANCE 自动注入每个工作者的系统提示（来自 agent/prompt_builder.py）；当您想要更深入了解特定场景时加载此技能。

                DevOps ⚙

#### webhook-subscriptions

                ✓ 内置

Webhook 订阅：事件驱动的代理运行。

                DevOps ⚙

### 游戏

#### minecraft-modpack-server

                ✓ 内置

托管带模组的 Minecraft 服务器（CurseForge、Modrinth）。

                游戏 🎮

#### pokemon-player

                ✓ 内置

通过无头模拟器 + RAM 读取玩宝可梦。

                游戏 🎮

### GitHub

#### codebase-inspection

                ✓ 内置

使用 pygount 检查代码库：代码行数、语言、比例。

                GitHub 💻

#### github-auth

                ✓ 内置

GitHub 身份验证设置：HTTPS 令牌、SSH 密钥、gh CLI 登录。

                GitHub 💻

#### github-code-review

                ✓ 内置

审查 PR：diff、内联评论通过 gh 或 REST。

                GitHub 💻

#### github-issues

                ✓ 内置

通过 gh 或 REST 创建、分流、标记、分配 GitHub issues。

                GitHub 💻

#### github-pr-workflow

                ✓ 内置

GitHub PR 生命周期：分支、提交、打开、CI、合并。

                GitHub 💻

#### github-repo-management

                ✓ 内置

克隆/创建/派生仓库；管理远程和发布。

                GitHub 💻

### 媒体

#### gif-search

                ✓ 内置

通过 curl + jq 从 Tenor 搜索/下载 GIF。

                媒体 🎵

#### heartmula

                ✓ 内置

HeartMuLa：从歌词 + 标签生成类 Suno 歌曲。

                媒体 🎵

#### songsee

                ✓ 内置

通过 CLI 获取音频频谱图/特征（mel、chroma、MFCC）。

                媒体 🎵

#### spotify

                ✓ 内置

Spotify：播放、搜索、队列、管理播放列表和设备。

                媒体 🎵

#### youtube-content

                ✓ 内置

YouTube 字幕转摘要、话题、博客。

                媒体 🎵

### MLOps

#### audiocraft-audio-generation

                ✓ 内置

AudioCraft：MusicGen 文本转音乐、AudioGen 文本转声音。

                MLOps 🧪

#### axolotl

                ✓ 内置

Axolotl：YAML LLM 微调（LoRA、DPO、GRPO）。

                MLOps 🧪

#### dspy

                ✓ 内置

DSPy：声明式 LM 程序、自动优化提示词、RAG。

                MLOps 🧪

#### evaluating-llms-harness

                ✓ 内置

lm-eval-harness：基准测试 LLM（MMLU、GSM8K 等）。

                MLOps 🧪

#### fine-tuning-with-trl

                ✓ 内置

TRL：SFT、DPO、PPO、GRPO、奖励建模用于 LLM RLHF。

                MLOps 🧪

#### huggingface-hub

                ✓ 内置

HuggingFace hf CLI：搜索/下载/上传模型、数据集。

                MLOps 🧪

#### llama-cpp

                ✓ 内置

llama.cpp 本地 GGUF 推理 + HF Hub 模型发现。

                MLOps 🧪

#### obliteratus

                ✓ 内置

OBLITERATUS：通过 diff-in-means 消除 LLM 拒绝行为。

                MLOps 🧪

#### outlines

                ✓ 内置

Outlines：结构化 JSON/regex/Pydantic LLM 生成。

                MLOps 🧪

#### segment-anything-model

                ✓ 内置

SAM：通过点、框、遮罩进行零样本图像分割。

                MLOps 🧪

#### serving-llms-vllm

                ✓ 内置

vLLM：高吞吐量 LLM 服务、OpenAI API、量化。

                MLOps 🧪

#### unsloth

                ✓ 内置

Unsloth：2-5 倍更快的 LoRA/QLoRA 微调，更少 VRAM。

                MLOps 🧪

#### weights-and-biases

                ✓ 内置

W&B：记录 ML 实验、扫描、模型注册表、仪表板。

                MLOps 🧪

### 生产力

#### airtable

                ✓ 内置

通过 curl 调用 Airtable REST API。记录 CRUD、过滤器、upsert。

                生产力 ✅

#### google-workspace

                ✓ 内置

通过 gws CLI 或 Python 调用 Gmail、日历、Drive、Docs、Sheets。

                生产力 ✅

#### linear

                ✓ 内置

Linear：通过 GraphQL + curl 管理 issues、项目、团队。

                生产力 ✅

#### maps

                ✓ 内置

通过 OpenStreetMap/OSRM 进行地理编码、POI、路线、时区查询。

                生产力 ✅

        显示更多（剩余 614 个技能）

### 安装和管理技能

```
# 浏览技能
hermes skills browse

# 搜索技能
hermes skills search &lt;keyword&gt;

# 安装技能
hermes skills install &lt;owner&gt;/&lt;skill-name&gt;

# 列出已安装的技能
hermes skills list

# 查看技能详情
hermes skills view &lt;skill-name&gt;

# 更新技能
hermes skills update &lt;skill-name&gt;

# 卸载技能
hermes skills uninstall &lt;skill-name&gt;
```

### 相关链接

- [Skills Hub](https://agentskills.io) - 官方技能中心
- [Discord](https://discord.gg/NousResearch) - 加入社区讨论
- [GitHub Discussions](https://github.com/NousResearch/hermes-agent/discussions) - 提出问题和分享想法
