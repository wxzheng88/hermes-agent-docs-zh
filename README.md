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

---

## 贡献指南

感谢您为 Hermes Agent 做出贡献！本指南涵盖设置开发环境、了解代码库以及让您的 PR 合并。

### 贡献优先级

我们按以下顺序重视贡献：

1. **Bug 修复** — 崩溃、错误行为、数据丢失
1. **跨平台兼容性** — macOS、不同 Linux 发行版、WSL2
1. **安全加固** — shell 注入、提示注入、路径遍历
1. **性能和健壮性** — 重试逻辑、错误处理、优雅降级
1. **新技能** — 广泛有用的技能（参见[创建技能](/docs/developer-guide/creating-skills)）
1. **新工具** — 很少需要；大多数功能应该是技能
1. **文档** — 修复、澄清、新示例

### 常见贡献路径

- 想要构建自定义/本地工具而不修改 Hermes 核心？从[构建 Hermes 插件](/docs/guides/build-a-hermes-plugin)开始
- 想要为 Hermes 本身构建新的内置核心工具？从[添加工具](/docs/developer-guide/adding-tools)开始
- 想要构建新技能？从[创建技能](/docs/developer-guide/creating-skills)开始
- 想要构建新的推理提供者？从[添加提供者](/docs/developer-guide/adding-providers)开始

### 开发环境设置

#### 前提条件

| 要求 | 说明 |
| --- | --- |
| **Git** | 支持 `--recurse-submodules`，已安装 `git-lfs` 扩展 |
| **Python 3.11+** | uv 会在缺失时安装 |
| **uv** | 快速 Python 包管理器（[安装](https://docs.astral.sh/uv/)） |
| **Node.js 20+** | 可选 — 需要浏览器工具和 WhatsApp 桥接（与根 `package.json` engines 匹配） |

#### 克隆和安装

```
git clone --recurse-submodules https://github.com/NousResearch/hermes-agent.git
cd hermes-agent
# 使用 Python 3.11 创建 venv
uv venv venv --python 3.11
export VIRTUAL_ENV="$(pwd)/venv"
# 安装所有额外依赖（消息、cron、CLI 菜单、开发工具）
uv pip install -e ".[all,dev]"
uv pip install -e "./tinker-atropos"
# 可选：浏览器工具
npm install

```

#### 配置开发环境

```
mkdir -p ~/.hermes/{cron,sessions,logs,memories,skills}
cp cli-config.yaml.example ~/.hermes/config.yaml
touch ~/.hermes/.env
# 至少添加一个 LLM 提供者密钥：
echo 'OPENROUTER_API_KEY=sk-or-v1-your-key' >> ~/.hermes/.env

```

#### 运行

```
# 创建全局访问符号链接
mkdir -p ~/.local/bin
ln -sf "$(pwd)/venv/bin/hermes" ~/.local/bin/hermes
# 验证
hermes doctor
hermes chat -q "Hello"

```

#### 运行测试

```
pytest tests/ -v

```

### 代码风格

- **PEP 8**，有实际例外（不严格限制行长度）
- **注释**：仅在解释非显而易见的意图、权衡或 API 怪癖时使用
- **错误处理**：捕获特定异常。对意外错误使用 `logger.warning()`/`logger.error()` 并带 `exc_info=True`
- **跨平台**：永远不要假设 Unix（见下文）
- **配置安全路径**：永远不要硬编码 `~/.hermes` — 对代码路径使用 `hermes_constants` 中的 `get_hermes_home()`，对用户面向消息使用 `display_hermes_home()`。有关完整规则，请参见 [AGENTS.md](https://github.com/NousResearch/hermes-agent/blob/main/AGENTS.md#profiles-multi-instance-support)。

### 跨平台兼容性

Hermes 正式支持 Linux、macOS 和 WSL2。本地 Windows **不受支持**，但代码库包含一些防御性编码模式以避免边缘情况下的硬崩溃。关键规则：

#### 1. `termios` 和 `fcntl` 仅限 Unix

始终同时捕获 `ImportError` 和 `NotImplementedError`：

```
try:
    from simple_term_menu import TerminalMenu
    menu = TerminalMenu(options)
    idx = menu.show()
except (ImportError, NotImplementedError):
    # 后备：编号菜单
    for i, opt in enumerate(options):
        print(f"  {i+1}. {opt}")
    idx = int(input("Choice: ")) - 1

```

#### 2. 文件编码

某些环境可能以非 UTF-8 编码保存 `.env` 文件：

```
try:
    load_dotenv(env_path)
except UnicodeDecodeError:
    load_dotenv(env_path, encoding="latin-1")

```

#### 3. 进程管理

`os.setsid()`、`os.killpg()` 和信号处理在不同平台间存在差异：

```
import platform
if platform.system() != "Windows":
    kwargs["preexec_fn"] = os.setsid

```

#### 4. 路径分隔符

使用 `pathlib.Path` 而不是使用 `/` 的字符串连接。

### 安全注意事项

Hermes 有终端访问权限。安全很重要。

#### 现有保护

| 层 | 实现 |
| --- | --- |
| **Sudo 密码管道** | 使用 `shlex.quote()` 防止 shell 注入 |
| **危险命令检测** | `tools/approval.py` 中的正则表达式模式，带用户批准流程 |
| **Cron 提示注入** | 扫描器阻止指令覆盖模式 |
| **写入拒绝列表** | 通过 `os.path.realpath()` 解析受保护路径以防止符号链接绕过 |
| **技能防护** | Hub 安装技能的安全扫描器 |
| **代码执行沙箱** | 子进程运行时会剥离 API 密钥 |
| **容器加固** | Docker：丢弃所有能力、无特权升级、PID 限制 |

#### 贡献安全敏感代码

- 将用户输入插入 shell 命令时始终使用 `shlex.quote()`
- 在访问控制检查之前使用 `os.path.realpath()` 解析符号链接
- 不要记录 secrets
- 在工具执行周围捕获广泛异常
- 如果您的更改涉及文件路径或进程，请在所有平台上测试

### Pull Request 流程

#### 分支命名

```
fix/description        # Bug 修复
feat/description       # 新功能
docs/description       # 文档
test/description       # 测试
refactor/description   # 代码重构

```

#### 提交前

1. **运行测试**：`pytest tests/ -v`
1. **手动测试**：运行 `hermes` 并练习您更改的代码路径
1. **检查跨平台影响**：考虑 macOS 和不同 Linux 发行版
1. **保持 PR 专注**：每个 PR 一个逻辑更改

#### PR 描述

包括：

- **什么**改变了以及**为什么**
- **如何测试**它
- **在什么平台上**测试过
- 引用任何相关 issues

#### 提交消息

我们使用 [Conventional Commits](https://www.conventionalcommits.org/)：

```
&lt;type&gt;(&lt;scope&gt;): &lt;description&gt;

```

| 类型 | 用于 |
| --- | --- |
| `fix` | Bug 修复 |
| `feat` | 新功能 |
| `docs` | 文档 |
| `test` | 测试 |
| `refactor` | 代码重构 |
| `chore` | 构建、CI、依赖更新 |

范围：`cli`、`gateway`、`tools`、`skills`、`agent`、`install`、`whatsapp`、`security`

示例：

```
fix(cli): prevent crash in save_config_value when model is a string
feat(gateway): add WhatsApp multi-user session isolation
fix(security): prevent shell injection in sudo password piping

```

### 报告问题

- 使用 [GitHub Issues](https://github.com/NousResearch/hermes-agent/issues)
- 包括：操作系统、Python 版本、Hermes 版本（`hermes version`）、完整错误追溯
- 包括复现步骤
- 创建重复项之前检查现有问题
- 对于安全漏洞，请私下报告

### 社区

- **Discord**：[discord.gg/NousResearch](https://discord.gg/NousResearch)
- **GitHub Discussions**：用于设计提案和架构讨论
- **技能中心**：上传专业技能并与社区分享

### 许可证

通过贡献，您同意您的贡献将根据 [MIT 许可证](https://github.com/NousResearch/hermes-agent/blob/main/LICENSE)获得许可。

---

## 架构

本页是 Hermes Agent 内部结构的顶层地图。使用它来在代码库中定位自己，然后深入阅读特定子系统的文档以获取实现细节。

### 系统概述

```
┌─────────────────────────────────────────────────────────────────────┐
│                        入口点                                        │
│                                                                      │
│  CLI (cli.py)    Gateway (gateway/run.py)    ACP (acp_adapter/)     │
│  Batch Runner    API Server                  Python Library          │
└──────────┬──────────────┬───────────────────────┬───────────────────┘
           │              │                       │
           ▼              ▼                       ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     AIAgent (run_agent.py)                          │
│                                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │ Prompt       │  │ Provider     │  │ Tool         │               │
│  │ Builder      │  │ Resolution   │  │ Dispatch     │               │
│  │ (prompt_     │  │ (runtime_    │  │ (model_      │               │
│  │  builder.py) │  │  provider.py)│  │  tools.py)   │               │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘               │
│         │                 │                 │                       │
│  ┌──────┴───────┐  ┌──────┴───────┐  ┌──────┴───────┐               │
│  │ Compression  │  │ 3 API Modes  │  │ Tool Registry│               │
│  │ & Caching    │  │ chat_compl.  │  │ (registry.py)│               │
│  │              │  │ codex_resp.  │  │ 61 tools     │               │
│  │              │  │ anthropic    │  │ 52 toolsets  │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
└─────────┴─────────────────┴─────────────────┴───────────────────────┘
          │                                    │
          ▼                                    ▼
┌───────────────────┐              ┌──────────────────────┐
│ Session Storage   │              │ Tool Backends         │
│ (SQLite + FTS5)  │              │ Terminal (7 backends) │
│ hermes_state.py   │              │ Browser (5 backends)  │
│ gateway/session.py│              │ Web (4 backends)      │
└───────────────────┘              │ MCP (dynamic)         │
                                    │ File, Vision, etc.    │
                                    └──────────────────────┘

```

### 目录结构

```
hermes-agent/
├── run_agent.py              # AIAgent — 核心对话循环（约 13,700 行）
├── cli.py                    # HermesCLI — 交互式终端 UI（约 11,500 行）
├── model_tools.py            # 工具发现、模式收集、分派
├── toolsets.py               # 工具分组和平台预设
├── hermes_state.py           # 带 FTS5 的 SQLite 会话/状态数据库
├── hermes_constants.py       # HERMES_HOME、配置感知路径
├── batch_runner.py           # 批量轨迹生成
│
├── agent/                    # Agent 内部结构
│   ├── prompt_builder.py     # 系统提示组装
│   ├── context_engine.py     # ContextEngine ABC（可插拔）
│   ├── context_compressor.py # 默认引擎 — 有损摘要
│   ├── prompt_caching.py     # Anthropic 提示缓存
│   ├── auxiliary_client.py   # 辅助 LLM 用于辅助任务（视觉、摘要）
│   ├── model_metadata.py     # 模型上下文长度、token 估算
│   ├── models_dev.py         # models.dev 注册集成
│   ├── anthropic_adapter.py  # Anthropic Messages API 格式转换
│   ├── display.py            # KawaiiSpinner、工具预览格式化
│   ├── skill_commands.py     # 技能斜杠命令
│   ├── memory_manager.py    # 记忆管理器编排
│   ├── memory_provider.py   # 记忆提供者 ABC
│   └── trajectory.py         # 轨迹保存助手
│
├── hermes_cli/               # CLI 子命令和设置
│   ├── main.py               # 入口点 — 所有 `hermes` 子命令（约 10,400 行）
│   ├── config.py             # DEFAULT_CONFIG、OPTIONAL_ENV_VARS、迁移
│   ├── commands.py           # COMMAND_REGISTRY — 中央斜杠命令定义
│   ├── auth.py               # PROVIDER_REGISTRY、凭证解析
│   ├── runtime_provider.py   # Provider → api_mode + 凭证
│   ├── models.py             # 模型目录、提供者模型列表
│   ├── model_switch.py       # /model 命令逻辑（CLI 和网关共享）
│   ├── setup.py              # 交互式设置向导（约 3,500 行）
│   ├── skin_engine.py        # CLI 主题引擎
│   ├── skills_config.py      # hermes skills — 按平台启用/禁用
│   ├── skills_hub.py         # /skills 斜杠命令
│   ├── tools_config.py       # hermes tools — 按平台启用/禁用
│   ├── plugins.py            # PluginManager — 发现、加载、钩子
│   ├── callbacks.py          # 终端回调（澄清、sudo、批准）
│   └── gateway.py            # hermes gateway 启动/停止
│
├── tools/                    # 工具实现（每个工具一个文件）
│   ├── registry.py           # 中央工具注册表
│   ├── approval.py           # 危险命令检测
│   ├── terminal_tool.py      # 终端编排
│   ├── process_registry.py   # 后台进程管理
│   ├── file_tools.py         # read_file、write_file、patch、search_files
│   ├── web_tools.py          # web_search、web_extract
│   ├── browser_tool.py       # 10 个浏览器自动化工具
│   ├── code_execution_tool.py # execute_code 沙箱
│   ├── delegate_tool.py      # 子代理委托
│   ├── mcp_tool.py           # MCP 客户端（约 3,100 行）
│   ├── credential_files.py   # 基于文件的凭证传递
│   ├── env_passthrough.py    # 沙箱环境变量传递
│   ├── ansi_strip.py         # ANSI 转义序列剥离
│   └── environments/         # 终端后端（local、docker、ssh、modal、daytona、singularity）
│
├── gateway/                  # 消息平台网关
│   ├── run.py                # GatewayRunner — 消息分派（约 12,200 行）
│   ├── session.py            # SessionStore — 对话持久化
│   ├── delivery.py           # 出站消息传递
│   ├── pairing.py            # DM 配对授权
│   ├── hooks.py              # 钩子发现和生命周期事件
│   ├── mirror.py            # 跨会话消息镜像
│   ├── status.py             # Token 锁、配置作用域进程跟踪
│   ├── builtin_hooks/        # 始终注册钩子的扩展点（无发货）
│   └── platforms/            # 20 个适配器：telegram、discord、slack、whatsapp、
│                             #   signal、matrix、mattermost、email、sms、
│                             #   dingtalk、feishu、wecom、wecom_callback、weixin、
│                             #   bluebubbles、qqbot、homeassistant、webhook、api_server、
│                             #   yuanbao
│
├── acp_adapter/              # ACP 服务器（VS Code / Zed / JetBrains）
├── cron/                     # 调度器（jobs.py、scheduler.py）
├── plugins/memory/           # 记忆提供者插件
├── plugins/context_engine/   # 上下文引擎插件
├── environments/             # RL 训练环境（Atropos）
├── skills/                   # 捆绑技能（始终可用）
├── optional-skills/          # 官方可选技能（显式安装）
├── website/                  # Docusaurus 文档站点
└── tests/                    # Pytest 测试套件（3,000+ 测试）

```

### 数据流

#### CLI 会话

```
User input → HermesCLI.process_input()  → AIAgent.run_conversation()
    → prompt_builder.build_system_prompt()
    → runtime_provider.resolve_runtime_provider()
    → API call (chat_completions / codex_responses / anthropic_messages)
    → tool_calls? → model_tools.handle_function_call() → loop
    → final response → display → save to SessionDB

```

#### 网关消息

```
Platform event → Adapter.on_message() → MessageEvent
  → GatewayRunner._handle_message()
    → authorize user
    → resolve session key
    → create AIAgent with session history
    → AIAgent.run_conversation()
    → deliver response back through adapter

```

#### Cron 任务

```
Scheduler tick → load due jobs from jobs.json
  → create fresh AIAgent (no history)
  → inject attached skills as context
  → run job prompt
  → deliver response to target platform
  → update job state and next_run

```

### 推荐阅读顺序

如果您是代码库新手：

1. **本页** — 定位自己
1. **[Agent 循环内部原理](/docs/developer-guide/agent-loop)** — AIAgent 如何工作
1. **[提示组装](/docs/developer-guide/prompt-assembly)** — 系统提示构建
1. **[提供者运行时解析](/docs/developer-guide/provider-runtime)** — 如何选择提供者
1. **[添加提供者](/docs/developer-guide/adding-providers)** — 添加新提供者的实用指南
1. **[工具运行时](/docs/developer-guide/tools-runtime)** — 工具注册表、分派、环境
1. **[会话存储](/docs/developer-guide/session-storage)** — SQLite 模式、FTS5、会话世系
1. **[网关内部原理](/docs/developer-guide/gateway-internals)** — 消息平台网关
1. **[上下文压缩和提示缓存](/docs/developer-guide/context-compression-and-caching)** — 压缩和缓存
1. **[ACP 内部原理](/docs/developer-guide/acp-internals)** — IDE 集成
1. **[环境、基准测试和数据生成](/docs/developer-guide/environments)** — RL 训练

### 主要子系统

#### Agent 循环

同步编排引擎（`run_agent.py` 中的 `AIAgent`）。处理提供者选择、提示构建、工具执行、重试、回退、回调、压缩和持久化。支持三种 API 模式用于不同提供者后端。

→ [Agent 循环内部原理](/docs/developer-guide/agent-loop)

#### 提示系统

对话生命周期中的提示构建和维护：

- **`prompt_builder.py`** — 从以下内容组装系统提示：人格（SOUL.md）、记忆（MEMORY.md、USER.md）、技能、上下文文件（AGENTS.md、.hermes.md）、工具使用指导，和模型特定指令
- **`prompt_caching.py`** — 应用 Anthropic 缓存断点以进行前缀缓存
- **`context_compressor.py`** — 当上下文超过阈值时对中间对话轮次进行摘要

→ [提示组装](/docs/developer-guide/prompt-assembly)、[上下文压缩和提示缓存](/docs/developer-guide/context-compression-and-caching)

#### 提供者解析

CLI、网关、cron、ACP 和辅助调用使用的共享运行时解析器。将 `(provider, model)` 元组映射到 `(api_mode, api_key, base_url)`。处理 18+ 提供者、OAuth 流程、凭证池和别名解析。

→ [提供者运行时解析](/docs/developer-guide/provider-runtime)

#### 工具系统

中央工具注册表（`tools/registry.py`），包含 52 个工具集中注册的 61 个工具。每个工具文件在导入时自我注册。注册表处理模式收集、分派、可用性检查和错误包装。终端工具支持 7 个后端（local、Docker、SSH、Daytona、Modal、Singularity、Vercel Sandbox）。

→ [工具运行时](/docs/developer-guide/tools-runtime)

#### 会话持久化

基于 SQLite 的会话存储，带 FTS5 全文搜索。会话具有世系跟踪（压缩时的父/子）、每个平台隔离和原子写入与竞争处理。

→ [会话存储](/docs/developer-guide/session-storage)

#### 消息网关

长期运行进程，具有 20 个平台适配器、统一会话路由、用户授权（允许列表 + DM 配对）、斜杠命令分派、钩子系统、轮询 ticking 和后台维护。

→ [网关内部原理](/docs/developer-guide/gateway-internals)

#### 插件系统

三个发现来源：`~/.hermes/plugins/`（用户）、`.hermes/plugins/`（项目）和 pip 入口点。插件通过上下文 API 注册工具、钩子和 CLI 命令。存在两种专业插件类型：记忆提供者（`plugins/memory/`）和上下文引擎（`plugins/context_engine/`）。两者都是单选 — 每次只能激活其中一个，通过 `hermes plugins` 或 `config.yaml` 配置。

→ [插件指南](/docs/guides/build-a-hermes-plugin)、[记忆提供者插件](/docs/developer-guide/memory-provider-plugin)

#### Cron

一级代理任务（不是 shell 任务）。作业存储在 JSON 中，支持多种调度格式，可以附加技能和脚本，并交付到任何平台。

→ [Cron 内部原理](/docs/developer-guide/cron-internals)

#### ACP 集成

通过 stdio/JSON-RPC 为 VS Code、Zed 和 JetBrains 暴露 Hermes 作为编辑器原生代理。

→ [ACP 内部原理](/docs/developer-guide/acp-internals)

#### RL / 环境 / 轨迹

用于评估和 RL 训练的完整环境框架。与 Atropos 集成，支持多种工具调用解析器，并生成 ShareGPT 格式轨迹。

→ [环境、基准测试和数据生成](/docs/developer-guide/environments)、[轨迹和训练格式](/docs/developer-guide/trajectory-format)

### 设计原则

| 原则 | 在实践中意味着什么 |
| --- | --- |
| **提示稳定性** | 系统提示不会在对话中途改变。除了显式用户操作（`/model`）外，没有缓存破坏性变更。 |
| **可观测执行** | 每个工具调用都通过回调对用户可见。CLI 中的进度更新（spinner）和网关中的聊天消息。 |
| **可中断** | API 调用和工具执行可以被用户输入或信号在飞行中取消。 |
| **平台无关核心** | 一个 AIAgent 类服务于 CLI、网关、ACP、批处理和 API 服务器。平台差异存在于入口点，而不是代理中。 |
| **松耦合** | 可选子系统（MCP、插件、记忆提供者、RL 环境）使用注册表模式和 check_fn 门控，而不是硬依赖。 |
| **配置隔离** | 每个配置（`hermes -p &lt;name&gt;`）获得自己的 HERMES_HOME、配置、记忆、会话和网关 PID。多个配置可以并发运行。 |

### 文件依赖链

```
tools/registry.py  (无依赖 — 被所有工具文件导入)
       ↑
tools/*.py  (每个在导入时调用 registry.register())
       ↑
model_tools.py  (导入 tools/registry + 触发工具发现)
       ↑
run_agent.py, cli.py, batch_runner.py, environments/

```

此链意味着工具注册发生在导入时，在创建任何代理实例之前。任何带有顶层 `registry.register()` 调用的 `tools/*.py` 文件都会被自动发现 — 不需要手动导入列表。

---

## 添加工具

在编写工具之前，问问自己：**这应该是一个[技能](/docs/developer-guide/creating-skills)吗？**

### 概述

添加工具涉及**2 个文件**：

1. **`tools/your_tool.py`** — 处理器、模式、检查函数、`registry.register()` 调用
1. **`toolsets.py`** — 将工具名称添加到 `_HERMES_CORE_TOOLS`（或特定工具集）

任何带有顶层 `registry.register()` 调用的 `tools/*.py` 文件都会在启动时自动发现 — 不需要手动导入列表。

### 步骤 1：创建内置工具文件

每个工具文件遵循相同的结构：

```
# tools/weather_tool.py
"""Weather Tool -- look up current weather for a location."""
import json
import os
import logging

logger = logging.getLogger(__name__)

# --- Availability check ---
def check_weather_requirements() -> bool:
    """Return True if the tool's dependencies are available."""
    return bool(os.getenv("WEATHER_API_KEY"))

# --- Handler ---
def weather_tool(location: str, units: str = "metric") -> str:
    """Fetch weather for a location. Returns JSON string."""
    api_key = os.getenv("WEATHER_API_KEY")
    if not api_key:
        return json.dumps({"error": "WEATHER_API_KEY not configured"})
    try:
        # ... call weather API ...
        return json.dumps({"location": location, "temp": 22, "units": units})
    except Exception as e:
        return json.dumps({"error": str(e)})

# --- Schema ---
WEATHER_SCHEMA = {
    "name": "weather",
    "description": "Get current weather for a location.",
    "parameters": {
        "type": "object",
        "properties": {
            "location": {
                "type": "string",
                "description": "City name or coordinates (e.g. 'London' or '51.5,-0.1')"
            },
            "units": {
                "type": "string",
                "enum": ["metric", "imperial"],
                "description": "Temperature units (default: metric)",
                "default": "metric"
            }
        },
        "required": ["location"]
    }
}

# --- Registration ---
from tools.registry import registry
registry.register(
    name="weather",
    toolset="weather",
    schema=WEATHER_SCHEMA,
    handler=lambda args, **kw: weather_tool(
        location=args.get("location", ""),
        units=args.get("units", "metric")),
    check_fn=check_weather_requirements,
    requires_env=["WEATHER_API_KEY"],
)

```

#### 关键规则

        > **重要：**

                处理器**必须**返回 JSON 字符串（通过 `json.dumps()`），而不是原始字典
                错误**必须**作为 `{"error": "message"}` 返回，而不是作为异常抛出
                `check_fn` 在构建工具定义时调用 — 如果返回 `False`，工具会被静默排除
                处理器接收 `(args: dict, **kwargs)`，其中 `args` 是 LLM 的工具调用参数

### 步骤 2：将内置工具添加到工具集

在 `toolsets.py` 中，添加工具名称：

```
# 如果它应该在所有平台（CLI + 消息）上可用：
_HERMES_CORE_TOOLS = [
    ...
    "weather",  #  str:
    async with aiohttp.ClientSession() as session:
        ...
    return json.dumps(result)

registry.register(
    name="weather",
    toolset="weather",
    schema=WEATHER_SCHEMA,
    handler=lambda args, **kw: weather_tool_async(args.get("location", "")),
    check_fn=check_weather_requirements,
    is_async=True,  # registry 自动调用 _run_async()
)

```

注册表透明地处理异步桥接 — 您永远不需要自己调用 `asyncio.run()`。

### 需要 task_id 的处理器

管理每会话状态的工具通过 `**kwargs` 接收 `task_id`：

```
def _handle_weather(args, **kw):
    task_id = kw.get("task_id")
    return weather_tool(args.get("location", ""), task_id=task_id)

registry.register(
    name="weather",
    ...
    handler=_handle_weather,
)

```

### Agent 循环拦截的工具

某些工具（`todo`、`memory`、`session_search`、`delegate_task`）需要访问每会话代理状态。这些在到达注册表之前被 `run_agent.py` 拦截。注册表仍然保存它们的模式，但如果绕过拦截，`dispatch()` 会返回后备错误。

### 可选：设置向导集成

如果您的工具需要 API 密钥，请将其添加到 `hermes_cli/config.py`：

```
OPTIONAL_ENV_VARS = {
    ...
    "WEATHER_API_KEY": {
        "description": "Weather API key for weather lookup",
        "prompt": "Weather API key",
        "url": "https://weatherapi.com/",
        "tools": ["weather"],
        "password": True,
    },
}

```

### 检查清单

- ✅ 工具文件已创建，包含处理器、模式、检查函数和注册
- ✅ 已添加到 `toolsets.py` 中的适当工具集
- ✅ 确认这确实应该是内置/核心工具而不是插件
- ✅ 处理器返回 JSON 字符串，错误作为 `{"error": "..."}` 返回
- ✅ 可选：API 密钥已添加到 `hermes_cli/config.py` 中的 `OPTIONAL_ENV_VARS`
- ✅ 可选：添加到 `toolset_distributions.py` 用于批处理
- ✅ 使用 `hermes chat -q "Use the weather tool for London"` 测试

---

## 工具运行时

Hermes 工具是自注册函数的分组，组成工具集并通过中央注册表/分派系统执行。

主要文件：

- `tools/registry.py`
- `model_tools.py`
- `toolsets.py`
- `tools/terminal_tool.py`
- `tools/environments/*`

### 工具注册模型

每个工具模块在导入时调用 `registry.register(...)`。

`model_tools.py` 负责导入/发现工具模块并构建模型使用的模式列表。

#### `registry.register()` 如何工作

`tools/` 中的每个工具文件在模块级别调用 `registry.register()` 来声明自己。函数签名是：

```
registry.register(
    name="terminal",               # 唯一工具名称（在 API 模式中使用）
    toolset="terminal",            # 此工具所属的工具集
    schema={...},                  # OpenAI 函数调用模式（描述、参数）
    handler=handle_terminal,       # 调用工具时执行的函数
    check_fn=check_terminal,       # 可选：返回可用性 True/False
    requires_env=["SOME_VAR"],     # 可选：需要的 env 变量（用于 UI 显示）
    is_async=False,                 # 处理器是否是异步协程
    description="Run commands",    # 人类可读描述
    emoji="💻",                    # 用于 spinner/进度显示的 emoji
)

```

每次调用创建一个 `ToolEntry`，存储在单例 `ToolRegistry._tools` 字典中，以工具名称为键。如果跨工具集发生名称冲突，会记录警告，后注册的生效。

#### 发现：`discover_builtin_tools()`

当 `model_tools.py` 被导入时，它从 `tools/registry.py` 调用 `discover_builtin_tools()`。此函数使用 AST 解析扫描每个 `tools/*.py` 文件，以找到包含顶层 `registry.register()` 调用的模块，然后导入它们：

```
# tools/registry.py（简化版）
def discover_builtin_tools(tools_dir=None):
    tools_path = Path(tools_dir) if tools_dir else Path(__file__).parent
    for path in sorted(tools_path.glob("*.py")):
        if path.name in {"__init__.py", "registry.py", "mcp_tool.py"}:
            continue
        if _module_registers_tools(path):  # AST 检查顶层 registry.register()
            importlib.import_module(f"tools.{path.stem}")

```

此自动发现意味着新的工具文件会被自动拾取 — 无需维护手动列表。AST 检查仅匹配顶层 `registry.register()` 调用（不是函数内部的调用），因此 `tools/` 中的辅助模块不会被导入。

核心工具发现后，也会发现 MCP 工具和插件工具：

1. **MCP 工具** — `tools.mcp_tool.discover_mcp_tools()` 读取 MCP 服务器配置并从外部服务器注册工具。
1. **插件工具** — `hermes_cli.plugins.discover_plugins()` 加载可能注册额外工具的用户/项目/pip 插件。

### 工具可用性检查（`check_fn`）

每个工具可以选择提供一个 `check_fn` — 一个在工具可用时返回 `True`、否则返回 `False` 的可调用对象。典型检查包括：

- **API 密钥存在** — 例如 `lambda: bool(os.environ.get("SERP_API_KEY"))` 用于网络搜索
- **服务运行中** — 例如检查 Honcho 服务器是否已配置
- **二进制文件已安装** — 例如验证浏览器工具的 `playwright` 是否可用

当 `registry.get_definitions()` 为模型构建模式列表时，它运行每个工具的 `check_fn()`：

```
# 来自 registry.py 的简化版本
if entry.check_fn:
    try:
        available = bool(entry.check_fn())
    except Exception:
        available = False   # 异常 = 不可用
    if not available:
        continue            # 跳过此工具

```

关键行为：

- 检查结果**按调用缓存** — 如果多个工具共享相同的 `check_fn`，它只运行一次。
- `check_fn()` 中的异常被视为"不可用"（fail-safe）。
- `is_toolset_available()` 方法检查工具集的 `check_fn` 是否通过，用于 UI 显示和工具集解析。

### 工具集解析

工具集是命名的工具捆绑包。Hermes 通过以下方式解析它们：

- 显式启用/禁用工具集列表
- 平台预设（`hermes-cli`、`hermes-telegram` 等）
- 动态 MCP 工具集
- 策划的专用工具集，如 `hermes-acp`

#### `get_tool_definitions()` 如何过滤工具

主要入口点是 `model_tools.get_tool_definitions(enabled_toolsets, disabled_toolsets, quiet_mode)`：

1. **如果提供了 `enabled_toolsets`** — 仅包含来自这些工具集的工具。每个工具集名称通过 `resolve_toolset()` 解析，该函数将复合工具集扩展为单个工具名称。
1. **如果提供了 `disabled_toolsets`** — 从所有工具集开始，然后减去禁用的工具集。
1. **如果两者都没有** — 包含所有已知工具集。
1. **注册表过滤** — 解析的工具名称集传递给 `registry.get_definitions()`，它应用 `check_fn` 过滤并返回 OpenAI 格式模式。
1. **动态模式修补** — 过滤后，`execute_code` 和 `browser_navigate` 模式会动态调整，仅引用实际通过过滤的工具（防止模型产生不可用工具的幻觉）。

#### 旧工具集名称

带有 `_tools` 后缀的旧工具集名称（例如 `web_tools`、`terminal_tools`）通过 `_LEGACY_TOOLSET_MAP` 映射到其现代工具名称以保持向后兼容。

### 分派

在运行时，工具通过中央注册表分派，代理循环对某些代理级工具（如 memory/todo/session-search 处理）有例外。

#### 分派流程：模型 tool_call → 处理器执行

当模型返回 `tool_call` 时，流程是：

```
Model response with tool_call
    ↓
run_agent.py agent loop
    ↓
model_tools.handle_function_call(name, args, task_id, user_task)
    ↓
[Agent-loop tools?] → handled directly by agent loop (todo, memory, session_search, delegate_task)
    ↓
[Plugin pre-hook] → invoke_hook("pre_tool_call", ...)
    ↓
registry.dispatch(name, args, **kwargs)
    ↓
Look up ToolEntry by name
    ↓
[Async handler?] → bridge via _run_async()
[Sync handler?]  → call directly
    ↓
Return result string (or JSON error)
    ↓
[Plugin post-hook] → invoke_hook("post_tool_call", ...)

```

#### 错误包装

所有工具执行都在两个级别包装错误处理：

1. **`registry.dispatch()`** — 捕获处理器的任何异常并返回 JSON 格式的 `{"error": "Tool execution failed: ExceptionType: message"}`。
1. **`handle_function_call()`** — 在整个分派上包装辅助 try/except，返回 `{"error": "Error executing tool_name: message"}`。

这确保模型始终收到格式良好的 JSON 字符串，而不是未处理的异常。

#### 代理循环工具

四个工具在注册表分派之前被拦截，因为它们需要代理级状态（TodoStore、MemoryStore 等）：

- `todo` — 计划/任务跟踪
- `memory` — 持久化内存写入
- `session_search` — 跨会话召回
- `delegate_task` — 生成子代理会话

这些工具的模式仍然在注册表中注册（用于 `get_tool_definitions`），但如果分派以某种方式直接到达它们，处理器会返回存根错误。

#### 异步桥接

当工具处理器是异步的时，`_run_async()` 将其桥接到同步分派路径：

- **CLI 路径（无运行循环）** — 使用持久事件循环来保持缓存的异步客户端存活
- **网关路径（运行中的循环）** — 使用 `asyncio.run()` 生成一次性线程
- **工作线程（并行工具）** — 使用存储在线程本地存储中的每线程持久循环

### DANGEROUS_PATTERNS 批准流程

终端工具集成了在 `tools/approval.py` 中定义的危险命令批准系统：

1. **模式检测** — `DANGEROUS_PATTERNS` 是一个 `(regex, description)` 元组列表，涵盖破坏性操作：

- 递归删除（`rm -rf`）
- 文件系统格式化（`mkfs`、`dd`）
- SQL 破坏性操作（无 `WHERE` 的 `DROP TABLE`、`DELETE FROM`）
- 系统配置覆盖（`&gt; /etc/`）
- 服务操作（`systemctl stop`）
- 远程代码执行（`curl | sh`）
- Fork 炸弹、进程终止等
1. **检测** — 在执行任何终端命令之前，`detect_dangerous_command(command)` 会检查所有模式。
1. **批准提示** — 如果找到匹配项：

- **CLI 模式** — 交互式提示询问用户批准、拒绝或永久允许
- **网关模式** — 异步批准回调将请求发送到消息平台
- **智能批准** — 可选地，辅助 LLM 可以自动批准与模式匹配的低风险命令（例如 `rm -rf node_modules/` 是安全的但匹配"递归删除"）
1. **会话状态** — 批准按会话跟踪。一旦您批准了会话的"递归删除"，后续的 `rm -rf` 命令不会再提示。
1. **永久允许列表** — "永久允许"选项将模式写入 `config.yaml` 的 `command_allowlist`，跨会话持久化。

### 终端/运行时环境

终端系统支持多个后端：

- local
- docker
- ssh
- singularity
- modal
- daytona
- vercel_sandbox

它还支持：

- 每任务 cwd 覆盖
- 后台进程管理
- PTY 模式
- 危险命令的批准回调

### 并发

工具调用可以根据工具组合和交互要求顺序或并发执行。

### 相关文档

- [工具集参考](/docs/reference/toolsets-reference)
- [内置工具参考](/docs/reference/tools-reference)
- [Agent 循环内部原理](/docs/developer-guide/agent-loop)
- [ACP 内部原理](/docs/developer-guide/acp-internals)

---

## 创建技能

技能是将新能力添加到 Hermes Agent 的首选方式。它们比工具更容易创建，不需要对代理进行代码更改，并且可以与社区共享。

### 应该是技能还是工具？

在以下情况下将其设为**技能**：

- 该功能可以表达为指令 + shell 命令 + 现有工具
- 它包装了一个外部 CLI 或 API，代理可以通过 `terminal` 或 `web_extract` 调用
- 它不需要内置到代理中的自定义 Python 集成或 API 密钥管理
- 示例：arXiv 搜索、git 工作流、Docker 管理、PDF 处理、通过 CLI 工具发送电子邮件

在以下情况下将其设为**工具**：

- 它需要与 API 密钥、auth 流程或多组件配置进行端到端集成
- 它需要每次精确执行的自定义处理逻辑
- 它处理二进制数据、流或实时事件
- 示例：浏览器自动化、TTS、视觉分析

### 技能目录结构

捆绑技能位于按类别组织的 `skills/` 中。官方可选技能在 `optional-skills/` 中使用相同的结构：

```
skills/
├── research/
│   └── arxiv/
│       ├── SKILL.md              # 必需：主要指令
│       └── scripts/              # 可选：辅助脚本
│           └── search_arxiv.py
├── productivity/
│   └── ocr-and-documents/
│       ├── SKILL.md
│       ├── scripts/
│       └── references/
└── ...

```

### SKILL.md 格式

```
---
name: my-skill
description: Brief description (shown in skill search results)
version: 1.0.0
author: Your Name
license: MIT
platforms: [macos, linux]          # 可选 — 限制在特定操作系统平台
                                   #   有效值：macos、linux、windows
                                   #   省略则在所有平台加载（默认）
metadata:
  hermes:
    tags: [Category, Subcategory, Keywords]
    related_skills: [other-skill-name]
    requires_toolsets: [web]            # 可选 — 仅在这些工具集激活时显示
    requires_tools: [web_search]        # 可选 — 仅在这些工具可用时显示
    fallback_for_toolsets: [browser]    # 可选 — 在这些工具集激活时隐藏
    fallback_for_tools: [browser_navigate]  # 可选 — 在这些工具存在时隐藏
    config:                              # 可选 — 技能需要的 config.yaml 设置
      - key: my.setting
        description: "What this setting controls"
        default: "sensible-default"
        prompt: "Display prompt for setup"
required_environment_variables:          # 可选 — 技能需要的环境变量
  - name: MY_API_KEY
    prompt: "Enter your API key"
    help: "Get one at https://example.com"
    required_for: "API access"
---

# Skill Title

Brief intro.

## When to Use

Trigger conditions — when should the agent load this skill?

## Quick Reference

Table of common commands or API calls.

## Procedure

Step-by-step instructions the agent follows.

## Pitfalls

Known failure modes and how to handle them.

## Verification

How the agent confirms it worked.

```

### 平台特定技能

技能可以使用 `platforms` 字段将自己限制在特定操作系统：

```
platforms: [macos]            # 仅 macOS（例如 iMessage、Apple Reminders）
platforms: [macos, linux]     # macOS 和 Linux
platforms: [windows]          # 仅 Windows

```

设置后，技能会在不兼容平台上从系统提示、`skills_list()` 和斜杠命令中自动隐藏。如果省略或为空，技能会在所有平台上加载（向后兼容）。

### 条件技能激活

技能可以声明对特定工具或工具集的依赖。这控制技能是否出现在给定会话的系统提示中。

```
metadata:
  hermes:
    requires_toolsets: [web]           # 如果 web 工具集不可用，则隐藏
    requires_tools: [web_search]       # 如果 web_search 工具不可用，则隐藏
    fallback_for_toolsets: [browser]   # 如果 browser 工具集可用，则隐藏
    fallback_for_tools: [browser_navigate]  # 如果 browser_navigate 可用，则隐藏

```

| 字段 | 行为 |
| --- | --- |
| `requires_toolsets` | 当**任何**列出的工具集**不可用**时，技能被**隐藏** |
| `requires_tools` | 当**任何**列出的工具**不可用**时，技能被**隐藏** |
| `fallback_for_toolsets` | 当**任何**列出的工具集**可用**时，技能被**隐藏** |
| `fallback_for_tools` | 当**任何**列出的工具**可用**时，技能被**隐藏** |

`fallback_for_*` 的用例：当主工具不可用时，创建作为 workaround 的技能。例如，带有 `fallback_for_tools: [web_search]` 的 `duckduckgo-search` 技能仅在网络搜索工具（需要 API 密钥）未配置时显示。

`requires_*` 的用例：创建仅在某些工具存在时有意义的技能。例如，带有 `requires_toolsets: [web]` 的网络抓取工作流技能在 Web 工具被禁用时不会使提示混乱。

### 环境变量要求

技能可以声明它们需要的环境变量。当技能通过 `skill_view` 加载时，其必需的变量会自动注册以传递到沙箱执行环境（终端、`execute_code`）。

```
required_environment_variables:
  - name: TENOR_API_KEY
    prompt: "Tenor API key"               # 提示用户时显示
    help: "Get your key at https://tenor.com"  # 获取帮助文本或 URL
    required_for: "GIF search functionality"   # 需要此变量的功能

```

每个条目支持：

- `name`（必需）— 环境变量名称
- `prompt`（可选）— 询问用户值时的提示文本
- `help`（可选）— 帮助文本或获取值的 URL
- `required_for`（可选）— 描述哪个功能需要此变量

用户也可以在 `config.yaml` 中手动配置传递变量：

```
terminal:
  env_passthrough:
    - MY_CUSTOM_VAR
    - ANOTHER_VAR

```

有关 macOS 专用技能的示例，请参见 `skills/apple/`。

### 安全设置加载

当技能需要 API 密钥或令牌时使用 `required_environment_variables`。缺失值**不会**从发现中隐藏技能。相反，Hermes 在本地 CLI 中加载技能时会安全地提示输入。

```
required_environment_variables:
  - name: TENOR_API_KEY
    prompt: Tenor API key
    help: Get a key from https://developers.google.com/tenor
    required_for: full functionality

```

用户可以跳过设置并继续加载技能。Hermes 永远不会向模型暴露原始 secret 值。网关和消息会话显示本地设置指导，而不是收集带内 secrets。

        > **注意：****沙箱传递**：当您的技能加载时，任何已设置的声明 `required_environment_variables` 都会**自动传递**到 `execute_code` 和 `terminal` 沙箱 — 包括 Docker 和 Modal 等远程后端。您的技能的脚本可以访问 `$TENOR_API_KEY`（或在 Python 中使用 `os.environ["TENOR_API_KEY"]`），而无需用户配置任何额外内容。

旧版 `prerequisites.env_vars` 仍然支持作为向后兼容别名。

### 配置设置（config.yaml）

技能可以声明存储在 `config.yaml` 中 `skills.config` 命名空间下的非秘密设置。与环境变量（存储在 `.env` 中的 secrets）不同，配置设置用于路径、首选项和其他非敏感值。

```
metadata:
  hermes:
    config:
      - key: myplugin.path
        description: Path to the plugin data directory
        default: "~/myplugin-data"
        prompt: Plugin data directory path
      - key: myplugin.domain
        description: Domain the plugin operates on
        default: ""
        prompt: Plugin domain (e.g., AI/ML research)

```

每个条目支持：

- `key`（必需）— 设置的点路径（例如 `myplugin.path`）
- `description`（必需）— 解释设置控制什么
- `default`（可选）— 如果用户未配置，则使用默认值
- `prompt`（可选）— 在 `hermes config migrate` 期间显示的提示文本；回退到 `description`

**工作原理：**

1. **存储：**值写入 `config.yaml` 的 `skills.config.&lt;key&gt;` 下：

```
skills:
  config:
    myplugin:
      path: ~/my-data

```
1. **发现：**`hermes config migrate` 扫描所有已启用技能，找到未配置设置并提示用户。设置也显示在 `hermes config show` 下的"技能设置"中。
1. **运行时注入：**当技能加载时，其配置值被解析并附加到技能消息：

```
[Skill config (from ~/.hermes/config.yaml):  myplugin.path = /home/user/my-data]

```

                代理看到配置值，而无需自己读取 `config.yaml`。
1. **手动设置：**用户也可以直接设置值：

```
hermes config set skills.config.myplugin.path ~/my-data

```

**何时使用哪个**

| 使用 `required_environment_variables` 用于 | 使用 `config` 用于 |
| --- | --- |
| API 密钥、令牌和其他**secrets**（存储在 `~/.hermes/.env` 中，从不向模型显示） | 路径、首选项和非敏感设置（存储在 `config.yaml` 中，可在配置显示中查看） |

### 凭证文件要求（OAuth 令牌等）

使用 OAuth 或基于文件的凭证的技能可以声明需要挂载到远程沙箱的文件。这适用于存储为**文件**的凭证（不是 env 变量）— 通常是设置脚本生成的 OAuth 令牌文件。

```
required_credential_files:
  - path: google_token.json
    description: Google OAuth2 token (created by setup script)
  - path: google_client_secret.json
    description: Google OAuth2 client credentials

```

每个条目支持：

- `path`（必需）— 相对于 `~/.hermes/` 的文件路径
- `description`（可选）— 解释文件是什么以及如何创建

加载时，Hermes 检查这些文件是否存在。缺失文件触发 `setup_needed`。现有文件自动：

- **挂载到 Docker** 容器作为只读绑定挂载
- **同步到 Modal** 沙箱（在创建 + 每个命令之前，因此会话中 OAuth 有效）
- 在**本地**后端上可用，无需任何特殊处理

**何时使用哪个**

| 使用 `required_environment_variables` 用于 | 使用 `required_credential_files` 用于 |
| --- | --- |
| 简单的 API 密钥和令牌（存储在 `~/.hermes/.env` 中的字符串） | OAuth 令牌文件、客户端 secrets、服务账户 JSON、证书或任何磁盘上的凭证文件 |

有关同时使用两者的完整示例，请参见 `skills/productivity/google-workspace/SKILL.md`。

### 技能指南

#### 无外部依赖

优先使用 stdlib Python、curl 和现有 Hermes 工具（`web_extract`、`terminal`、`read_file`）。如果需要依赖，在技能中记录安装步骤。

#### 渐进式披露

将最常见的工作流放在首位。边缘情况和高级用法放在底部。这可以保持常用任务的 token 使用量低。

#### 包含辅助脚本

对于 XML/JSON 解析或复杂逻辑，在 `scripts/` 中包含辅助脚本 — 不要期望 LLM 每次都内联编写解析器。

        从 SKILL.md 引用捆绑脚本

当技能加载时，激活消息将绝对技能目录公开为 `[Skill directory: /abs/path]`，还在 SKILL.md 正文中替换两个模板令牌：

| 令牌 | 替换为 |
| --- | --- |
| `${HERMES_SKILL_DIR}` | 技能目录的绝对路径 |
| `${HERMES_SESSION_ID}` | 活动会话 ID（如果没有会话则保留原位） |

因此，SKILL.md 可以告诉代理直接运行捆绑脚本：

```
To analyse the input, run:
    node ${HERMES_SKILL_DIR}/scripts/analyse.js &lt;input&gt;

```

代理看到替换后的绝对路径，并使用准备好的命令调用 `terminal` 工具 — 无需路径计算，无需额外的 `skill_view` 往返。使用 `skills.template_vars: false` 在 `config.yaml` 中全局禁用替换。

        内联 shell 代码片段（选择性加入）

技能还可以在 SKILL.md 正文中嵌入内联 shell 代码片段，写为 `!`cmd``。启用后，每个代码片段的 stdout 在代理读取之前内联到消息中，因此技能可以注入动态上下文：

```
Current date: !`date -u +%Y-%m-%d`
Git branch: !`git -C ${HERMES_SKILL_DIR} rev-parse --abbrev-ref HEAD`

```

这**默认关闭** — SKILL.md 中的任何代码片段都会在主机上运行，无需批准，因此仅为您信任的技能来源启用：

```
# config.yaml
skills:
  inline_shell: true
  inline_shell_timeout: 10   # 每个代码片段的秒数

```

代码片段使用技能目录作为其工作目录运行，输出上限为 4000 字符。失败（超时、非零退出）显示为简短的 `[inline-shell error: ...]` 标记，而不是破坏整个技能。

#### 测试它

运行技能并验证代理正确遵循指令：

```
hermes chat --toolsets skills -q "Use the X skill to do Y"

```

### 技能应该放在哪里？

捆绑技能（在 `skills/` 中）随每个 Hermes 安装发货。它们应该**对大多数用户广泛有用**：

- 文档处理、网络研究、常见开发工作流、系统管理
- 被广泛范围的人 regular 使用

如果您的技能是官方的且有用，但不是普遍需要的（例如付费服务集成、重量级依赖），请将其放入**`optional-skills/`** — 它随仓库发货，可通过 `hermes skills browse` 发现（标记为"official"），并以 builtin trust 安装。

如果您的技能是专业的、社区贡献的或小众的，它更适合**技能中心** — 上传到注册表并通过 `hermes skills install` 分享。

### 发布技能

#### 到技能中心

```
hermes skills publish skills/my-skill --to github --repo owner/repo

```

#### 到自定义仓库

将您的仓库添加为 tap：

```
hermes skills tap add owner/repo

```

用户可以从此仓库搜索和安装。

### 安全扫描

所有 hub 安装的技能都要经过安全扫描，检查：

- 数据泄露模式
- 提示注入尝试
- 破坏性命令
- Shell 注入

信任级别：

- `builtin` — 随 Hermes 发货（始终受信任）
- `official` — 来自仓库中的 `optional-skills/`（builtin trust，无第三方警告）
- `trusted` — 来自 openai/skills、anthropics/skills
- `community` — 非危险发现可以用 `--force` 覆盖；`dangerous` 裁决保持阻止

Hermes 现在可以消费来自多个外部发现模型的第三方技能：

- 直接 GitHub 标识符（例如 `openai/skills/k8s`）
- `skills.sh` 标识符（例如 `skills-sh/vercel-labs/json-render/json-render-react`）
- 从 `/.well-known/skills/index.json` 提供服务的已知端点

如果您希望技能无需 GitHub 特定安装程序即可被发现，请考虑除了在仓库或市场中发布外，还从已知端点提供服务。

---

## 添加提供者

Hermes 已经可以通过自定义提供者路径与任何 OpenAI 兼容端点通信。除非您想要该服务的一流 UX，否则不要添加内置提供者：

- 提供者特定的 auth 或令牌刷新
- 策划的模型目录
- 设置 / `hermes model` 菜单条目
- 提供者别名用于 `provider:model` 语法
- 需要适配器的非 OpenAI API 形状

如果提供者只是"另一个 OpenAI 兼容 base URL 和 API 密钥"，命名自定义提供者可能就足够了。

### 思维模型

内置提供者需要在几个层面协调：

1. `hermes_cli/auth.py` 决定如何找到凭证。
1. `hermes_cli/runtime_provider.py` 将其转换为运行时数据：

- `provider`
- `api_mode`
- `base_url`
- `api_key`
- `source`
1. `run_agent.py` 使用 `api_mode` 决定如何构建和发送请求。
1. `hermes_cli/models.py` 和 `hermes_cli/main.py` 使提供者在 CLI 中显示。（`hermes_cli/setup.py` 自动委托给 `main.py` — 无需在那里更改。）
1. `agent/auxiliary_client.py` 和 `agent/model_metadata.py` 保持辅助任务和 token 预算工作。

重要的抽象是 `api_mode`。

- 大多数提供者使用 `chat_completions`。
- Codex 使用 `codex_responses`。
- Anthropic 使用 `anthropic_messages`。
- 新的非 OpenAI 协议通常意味着添加新的适配器和新的 `api_mode` 分支。

### 首先选择实现路径

#### 路径 A — OpenAI 兼容提供者

当提供者接受标准 chat-completions 样式请求时使用此路径。

典型工作：

- 添加 auth 元数据
- 添加模型目录 / 别名
- 添加运行时解析
- 添加 CLI 菜单布线
- 添加 aux-model 默认值
- 添加测试和用户文档

您通常不需要新的适配器或新的 `api_mode`。

#### 路径 B — 本机提供者

当提供者不像 OpenAI chat completions 那样工作时使用此路径。

树中当前的示例：

- `codex_responses`
- `anthropic_messages`

此路径包括路径 A 的所有内容 plus：

- `agent/` 中的提供者适配器
- `run_agent.py` 分支用于请求构建、分派、usage 提取、interrupt 处理和响应规范化
- 适配器测试

### 文件检查清单

#### 每个内置提供者必需

1. `hermes_cli/auth.py`
1. `hermes_cli/models.py`
1. `hermes_cli/runtime_provider.py`
1. `hermes_cli/main.py`
1. `agent/auxiliary_client.py`
1. `agent/model_metadata.py`
1. 测试
1. `website/docs/` 下的用户面向文档

        > **提示：**`hermes_cli/setup.py`**不需要**更改。设置向导将提供者/模型选择委托给 `main.py` 中的 `select_provider_and_model()` — 在那里添加的任何提供者会自动在 `hermes setup` 中可用。

#### 本机 / 非 OpenAI 提供者的附加项

1. `agent/&lt;provider&gt;_adapter.py`
1. `run_agent.py`
1. 如果需要提供者 SDK，则为 `pyproject.toml`

### 快速路径：简单的 API 密钥提供者

如果您的提供者只是一个带有单个 API 密钥认证的 OpenAI 兼容端点，您不需要触及 `auth.py`、`runtime_provider.py`、`main.py` 或完整检查清单中的任何其他文件。

您只需要：

1. 在 `plugins/model-providers/&lt;your-provider&gt;/` 下创建一个插件目录，包含：

- `__init__.py` — 在模块级别调用 `register_provider(profile)`
- `plugin.yaml` — 清单（name、kind: model-provider、version、description）
1. 就这样。提供者插件在第一次调用 `get_provider_profile()` 或 `list_providers()` 时自动加载 — 捆绑插件（此仓库）和位于 `$HERMES_HOME/plugins/model-providers/` 的用户插件都会被拾取。

当您添加插件并调用 `register_provider()` 时，以下内容自动连接：

1. `auth.py` 中的 `PROVIDER_REGISTRY` 条目（凭证解析、env-var 查找）
1. `api_mode` 设置为 `chat_completions`
1. `base_url` 从配置或声明的 env 变量获取
1. `env_vars` 按优先级顺序检查 API 密钥
1. `fallback_models` 列表为提供者注册
1. `--provider` CLI 标志接受提供者 ID
1. `hermes model` 菜单包含提供者
1. `hermes setup` 向导自动委托给 `main.py`
1. `provider:model` 别名语法有效
1. 运行时解析器返回正确的 `base_url` 和 `api_key`
1. `HERMES_INFERENCE_PROVIDER` env-var 覆盖接受提供者 ID
1. 后备模型激活可以干净地切换到提供者

`$HERMES_HOME/plugins/model-providers/&lt;name&gt;/` 处的用户插件覆盖同名捆绑插件（`register_provider()` 中的后写者胜出）— 因此第三方可以在不编辑仓库的情况下替换或修补任何内置配置。

参见 `plugins/model-providers/nvidia/` 或 `plugins/model-providers/gmi/` 作为模板，以及完整的[模型提供者插件指南](/docs/developer-guide/model-provider-plugin)，了解字段参考、钩子习惯用法和端到端示例。

### 完整路径：OAuth 和复杂提供者

当您的提供者需要以下任何内容时，使用完整检查清单：

- OAuth 或令牌刷新（Nous Portal、Codex、Google Gemini、Qwen Portal、Copilot）
- 需要新适配器的非 OpenAI API 形状（Anthropic Messages、Codex Responses）
- 自定义端点检测或多区域探测（z.ai、Kimi）
- 策划的静态模型目录或实时 `/models` 获取
- 具有自定义 auth 流的提供者特定的 `hermes model` 菜单条目

### 步骤 1：选择一个规范提供者 ID

选择一个提供者 ID 并在各处使用它。

来自仓库的示例：

- `openai-codex`
- `kimi-coding`
- `minimax-cn`

相同的 ID 应该出现在：

- `hermes_cli/auth.py` 中的 `PROVIDER_REGISTRY`
- `hermes_cli/models.py` 中的 `_PROVIDER_LABELS`
- `hermes_cli/auth.py` 和 `hermes_cli/models.py` 中的 `_PROVIDER_ALIASES`
- `hermes_cli/main.py` 中的 CLI `--provider` 选项
- 设置 / 模型选择分支
- 辅助模型默认值
- 测试

如果 ID 在这些文件之间不同，提供者会感觉半连接：auth 可能工作，而 `/model`、setup 或运行时解析静默错过它。

### 步骤 2：在 `hermes_cli/auth.py` 中添加 auth 元数据

对于 API 密钥提供者，在 `PROVIDER_REGISTRY` 中添加 `ProviderConfig` 条目，包含：

- `id`
- `name`
- `auth_type="api_key"`
- `inference_base_url`
- `api_key_env_vars`
- 可选的 `base_url_env_var`

还将别名添加到 `_PROVIDER_ALIASES`。

使用现有提供者作为模板：

- 简单 API 密钥路径：Z.AI、MiniMax
- 带端点检测的 API 密钥路径：Kimi、Z.AI
- 本机令牌解析：Anthropic
- OAuth / auth-store 路径：Nous、OpenAI Codex

在此回答的问题：

- Hermes 应该检查哪些 env 变量，优先级顺序是什么？
- 提供者需要 base-URL 覆盖吗？
- 需要端点探测或令牌刷新吗？
- 缺少凭证时 auth 错误应该说什么？

如果提供者需要比"查找 API 密钥"更复杂的东西，添加专用凭证解析器，而不是将逻辑塞进无关分支。

### 步骤 3：在 `hermes_cli/models.py` 中添加模型目录和别名

更新提供者目录以使提供者在菜单和 `provider:model` 语法中工作。

典型编辑：

- `_PROVIDER_MODELS`
- `_PROVIDER_LABELS`
- `_PROVIDER_ALIASES`
- `list_available_providers()` 内的提供者显示顺序
- 如果提供者支持实时 `/models` 获取，则为 `provider_model_ids()`

如果提供者暴露实时模型列表，优先使用它，并将 `_PROVIDER_MODELS` 保持为静态后备。

此文件还使以下输入工作：

```
anthropic:claude-sonnet-4-6
kimi:model-name

```

如果别名在这里缺失，提供者可能正确认证但在 `/model` 解析中仍然失败。

### 步骤 4：在 `hermes_cli/runtime_provider.py` 中解析运行时数据

`resolve_runtime_provider()` 是 CLI、网关、cron、ACP 和辅助客户端使用的共享路径。

添加返回至少包含以下内容的 dict 的分支：

```
{
    "provider": "your-provider",
    "api_mode": "chat_completions",  # 或您的本机模式
    "base_url": "https://...",
    "api_key": "...",
    "source": "env|portal|auth-store|explicit",
    "requested_provider": requested_provider,
}

```

如果提供者是 OpenAI 兼容的，`api_mode` 通常应该保持 `chat_completions`。

注意 API 密钥优先级。Hermes 已经包含逻辑以避免将 OpenRouter 密钥泄露到无关端点。新提供者应该在哪个密钥发送到哪个 base URL 方面同样明确。

### 步骤 5：在 `hermes_cli/main.py` 中连接 CLI

提供者在显示在交互式 `hermes model` 流程中之前是不可发现的。

在 `hermes_cli/main.py` 中更新：

- `provider_labels` 字典
- `select_provider_and_model()` 中的 `providers` 列表
- 提供者分派（`if selected_provider == ...`）
- `--provider` 参数选项
- 如果提供者支持这些流程，则为 login/logout 选项
- `_model_flow_&lt;provider&gt;()` 函数，或者如果适合则重用 `_model_flow_api_key_provider()`

        > **提示：**`hermes_cli/setup.py` 不需要更改 — 它从 `main.py` 调用 `select_provider_and_model()`，因此您的新提供者在 `hermes model` 和 `hermes setup` 中自动显示。

### 步骤 6：保持辅助调用工作

这里有两个文件重要：

#### `agent/auxiliary_client.py`

如果这是直接 API 密钥提供者，在 `_API_KEY_PROVIDER_AUX_MODELS` 中添加便宜的/快速的默认 aux 模型。

辅助任务包括：

- 视觉摘要
- 网页提取摘要
- 上下文压缩摘要
- 会话搜索摘要
- 内存刷新

如果提供者没有合理的 aux 默认值，辅助任务可能会糟糕地回退或意外使用昂贵的主模型。

#### `agent/model_metadata.py`

为提供者的模型添加上下文长度，以便 token 预算、压缩阈值和限制保持正常。

### 步骤 7：如果提供者是本机的，添加适配器和 `run_agent.py` 支持

如果提供者不是普通 chat completions，在 `agent/&lt;provider&gt;_adapter.py` 中隔离提供者特定的逻辑。

保持 `run_agent.py` 专注于编排。它应该调用适配器助手，而不是在文件各处内联构建提供者负载。

本机提供者通常需要在这些地方工作：

#### 新适配器文件

典型职责：

- 构建 SDK / HTTP 客户端
- 解析令牌
- 将 OpenAI 风格的对话消息转换为提供者的请求格式
- 如需要，转换工具模式
- 将提供者响应规范化为 `run_agent.py` 期望的内容
- 提取 usage 和 finish-reason 数据

#### `run_agent.py`

搜索 `api_mode` 并审计每个切换点。至少验证：

- `__init__` 选择新的 `api_mode`
- 客户端构建适用于提供者
- `_build_api_kwargs()` 知道如何格式化请求
- `_interruptible_api_call()` 分派到正确的客户端调用
- interrupt / 客户端重建路径工作
- 响应验证接受提供者的形状
- finish-reason 提取正确
- token-usage 提取正确
- 后备模型激活可以干净地切换到新提供者
- 摘要生成和内存刷新路径仍然工作

还要搜索 `run_agent.py` 中的 `self.client.`。任何假设标准 OpenAI 客户端存在的代码路径在本地提供者使用不同客户端对象或 `self.client = None` 时可能会破坏。

#### 提示缓存和提供者特定请求字段

提示缓存和提供者特定旋钮很容易回归。

树中已有的示例：

- Anthropic 有本机提示缓存路径
- OpenRouter 获取提供者路由字段
- 并非每个提供者都应该接收每个请求端选项

当您添加本机提供者时，仔细检查 Hermes 是否仅发送该提供者实际理解的字段。

### 步骤 8：测试

至少触及守卫提供者连接的测试。

常见位置：

- `tests/test_runtime_provider_resolution.py`
- `tests/test_cli_provider_resolution.py`
- `tests/test_cli_model_command.py`
- `tests/test_setup_model_selection.py`
- `tests/test_provider_parity.py`
- `tests/test_run_agent.py`
- 对于本机提供者，为 `tests/test_&lt;provider&gt;_adapter.py`

对于仅文档的示例，确切的文件集可能不同。关键是覆盖：

- auth 解析
- CLI 菜单 / 提供者选择
- 运行时提供者解析
- 代理执行路径
- provider:model 解析
- 任何适配器特定的 message 转换

使用禁用 xdist 的方式运行测试：

```
source venv/bin/activate
python -m pytest tests/test_runtime_provider_resolution.py tests/test_cli_provider_resolution.py tests/test_cli_model_command.py tests/test_setup_model_selection.py -n0 -q

```

对于更深层次的更改，推送前运行完整套件：

```
source venv/bin/activate
python -m pytest tests/ -n0 -q

```

### 步骤 9：实时验证

测试后，运行真实冒烟测试。

```
source venv/bin/activate
python -m hermes_cli.main chat -q "Say hello" --provider your-provider --model your-model

```

如果您更改了菜单，也测试交互式流程：

```
source venv/bin/activate
python -m hermes_cli.main model
python -m hermes_cli.main setup

```

对于本机提供者，也验证至少一个工具调用，而不仅仅是纯文本响应。

### 步骤 10：更新用户面向文档

如果提供者旨在作为一流选项发货，也要更新用户文档：

- `website/docs/getting-started/quickstart.md`
- `website/docs/user-guide/configuration.md`
- `website/docs/reference/environment-variables.md`

开发者可以完美地连接提供者，但仍可能使用户无法发现所需的 env 变量或设置流程。

### OpenAI 兼容提供者检查清单

如果提供者是标准 chat completions，请使用此检查清单。

- ✅ 在 `hermes_cli/auth.py` 中添加了 `ProviderConfig`
- ✅ 在 `hermes_cli/auth.py` 和 `hermes_cli/models.py` 中添加了别名
- ✅ 在 `hermes_cli/models.py` 中添加了模型目录
- ✅ 在 `hermes_cli/runtime_provider.py` 中添加了运行时分支
- ✅ 在 `hermes_cli/main.py` 中添加了 CLI 连接（setup.py 自动继承）
- ✅ 在 `agent/auxiliary_client.py` 中添加了 aux 模型
- ✅ 在 `agent/model_metadata.py` 中添加了上下文长度
- ✅ 更新了运行时 / CLI 测试
- ✅ 更新了用户文档

### 本机提供者检查清单

当提供者需要新协议路径时使用。

- ✅ OpenAI 兼容检查清单中的所有内容
- ✅ 在 `agent/&lt;provider&gt;_adapter.py` 中添加了适配器
- ✅ 在 `run_agent.py` 中支持新的 `api_mode`
- ✅ interrupt / 重建路径工作
- ✅ usage 和 finish-reason 提取工作
- ✅ 后备路径工作
- ✅ 添加了适配器测试
- ✅ 实时冒烟测试通过

### 常见陷阱

#### 1. 将提供者添加到 auth 但不添加到模型解析

这使得凭证正确解析，而 `/model` 和 `provider:model` 输入失败。

#### 2. 忘记 `config["model"]` 可以是字符串或字典

很多提供者选择代码必须规范化两种形式。

#### 3. 假设需要内置提供者

如果服务只是 OpenAI 兼容的，自定义提供者可能已经以更少的维护解决用户问题。

#### 4. 忘记辅助路径

主聊天路径可能工作，而摘要、内存刷新或视觉助手失败，因为从未更新 aux 路由。

#### 5. 本机提供者分支隐藏在 `run_agent.py` 中

搜索 `api_mode` 和 `self.client.`。不要假设明显的请求路径是唯一的。

#### 6. 向其他提供者发送仅 OpenRouter 的旋钮

像提供者路由这样的字段仅适用于支持它们的提供者。

#### 7. 更新 `hermes model` 但不更新 `hermes setup`

两个流程都需要知道提供者。

### 实现时的好搜索目标

如果您在寻找提供者触及的所有位置，请搜索这些符号：

- `PROVIDER_REGISTRY`
- `_PROVIDER_ALIASES`
- `_PROVIDER_MODELS`
- `resolve_runtime_provider`
- `_model_flow_`
- `select_provider_and_model`
- `api_mode`
- `_API_KEY_PROVIDER_AUX_MODELS`
- `self.client.`

### 相关文档

- [提供者运行时解析](/docs/developer-guide/provider-runtime)
- [架构](/docs/developer-guide/architecture)
- [贡献指南](/docs/developer-guide/contributing)

---

## ACP 内部原理

ACP 适配器将 Hermes 的同步 `AIAgent` 包装在异步 JSON-RPC stdio 服务器中。

关键实现文件：

- `acp_adapter/entry.py`
- `acp_adapter/server.py`
- `acp_adapter/session.py`
- `acp_adapter/events.py`
- `acp_adapter/permissions.py`
- `acp_adapter/tools.py`
- `acp_adapter/auth.py`
- `acp_registry/agent.json`

### 启动流程

```
hermes acp / hermes-acp / python -m acp_adapter
  -> acp_adapter.entry.main()
  -> load ~/.hermes/.env
  -> configure stderr logging
  -> construct HermesACPAgent
  -> acp.run_agent(agent, use_unstable_protocol=True)

```

stdout 保留用于 ACP JSON-RPC 传输。人类可读的日志发送到 stderr。

### 主要组件

#### `HermesACPAgent`

`acp_adapter/server.py` 实现 ACP 代理协议。

职责：

- 初始化 / 认证
- new/load/resume/fork/list/cancel 会话方法
- 提示执行
- 会话模型切换
- 将同步 AIAgent 回调连接到 ACP 异步通知的布线

#### `SessionManager`

`acp_adapter/session.py` 跟踪活动 ACP 会话。

每个会话存储：

- `session_id`
- `agent`
- `cwd`
- `model`
- `history`
- `cancel_event`

管理器是线程安全的，支持：

- create
- get
- remove
- fork
- list
- cleanup
- cwd updates

#### 事件桥接

`acp_adapter/events.py` 将 AIAgent 回调转换为 ACP `session_update` 事件。

桥接的回调：

- `tool_progress_callback`
- `thinking_callback`
- `step_callback`
- `message_callback`

因为 `AIAgent` 在工作线程中运行，而 ACP I/O 位于主事件循环上，桥接使用：

```
asyncio.run_coroutine_threadsafe(...)

```

#### 权限桥接

`acp_adapter/permissions.py` 将危险终端批准提示适配为 ACP 权限请求。

映射：

- `allow_once` -> Hermes `once`
- `allow_always` -> Hermes `always`
- reject 选项 -> Hermes `deny`

超时和桥接失败默认拒绝。

#### 工具渲染助手

`acp_adapter/tools.py` 将 Hermes 工具映射到 ACP 工具类型并构建面向编辑器的内容。

示例：

- `patch` / `write_file` -> 文件 diff
- `terminal` -> shell 命令文本
- `read_file` / `search_files` -> 文本预览
- 大结果 -> 用于 UI 安全的截断文本块

### 会话生命周期

```
new_session(cwd)
  -> create SessionState
  -> create AIAgent(platform="acp", enabled_toolsets=["hermes-acp"])
  -> bind task_id/session_id to cwd override

prompt(..., session_id)
  -> extract text from ACP content blocks
  -> reset cancel event
  -> install callbacks + approval bridge
  -> run AIAgent in ThreadPoolExecutor
  -> update session history
  -> emit final agent message chunk

```

#### 取消

`cancel(session_id)`：

- 设置会话取消事件
- 在可用时调用 `agent.interrupt()`
- 导致提示响应返回 `stop_reason="cancelled"`

#### 分叉

`fork_session()` 将消息历史深拷贝到新的活动会话，保留对话状态，同时给分叉自己的会话 ID 和 cwd。

### 提供者/auth 行为

ACP 不实现自己的 auth store。

相反，它重用 Hermes 的运行时解析器：

- `acp_adapter/auth.py`
- `hermes_cli/runtime_provider.py`

因此 ACP 广告并使用当前配置的 Hermes 提供者/凭证。

### 工作目录绑定

ACP 会话携带编辑器 cwd。

会话管理器通过任务作用域的终端/文件覆盖将 cwd 绑定到 ACP 会话 ID，以便文件和终端工具相对于编辑器工作区操作。

### 重复同名工具调用

事件桥接按工具名称跟踪工具 ID FIFO，而不只是每个名称一个 ID。这对于以下情况很重要：

- 并行同名调用
- 一步中重复同名调用

没有 FIFO 队列，完成事件会附加到错误的工具调用。

### 批准回调恢复

ACP 在提示执行期间在终端工具上临时安装批准回调，然后恢复之前的回调。这避免了将 ACP 会话特定的批准处理器全局永久安装。

### 当前限制

- ACP 会话持久化到共享 `~/.hermes/state.db`（SessionDB）并跨进程重启透明恢复；它们出现在 `session_search` 中
- 非文本提示块当前被忽略以进行请求文本提取
- 编辑器特定的 UX 因 ACP 客户端实现而异

### 相关文件

- `tests/acp/` — ACP 测试套件
- `toolsets.py` — `hermes-acp` 工具集定义
- `hermes_cli/main.py` — `hermes acp` CLI 子命令
- `pyproject.toml` — `[acp]` 可选依赖 + `hermes-acp` 脚本

---

## CLI 命令参考

本页面涵盖您从终端运行的**命令行命令**。

有关聊天内斜杠命令，请参阅[斜杠命令参考](/docs/reference/slash-commands)。

### 全局入口点

```
hermes [global-options] &lt;command&gt; [subcommand/options]
```

#### 全局选项

| 选项 | 描述 |
| --- | --- |
| `--version`, `-V` | 显示版本并退出。 |
| `--profile &lt;name&gt;`, `-p &lt;name&gt;` | 选择用于此次调用的 Hermes profile。 |
| `--resume &lt;session&gt;`, `-r &lt;session&gt;` | 通过 ID 或标题恢复之前的会话。 |
| `--continue [name]`, `-c [name]` | 恢复最近的会话。 |
| `--worktree`, `-w` | 在隔离的 git worktree 中启动。 |
| `--yolo` | 跳过危险命令批准提示。 |
| `--tui` | 启动 TUI 而不是经典 CLI。 |

### 顶级命令

| 命令 | 用途 |
| --- | --- |
| `hermes chat` | 与 agent 进行交互式或一次性聊天。 |
| `hermes model` | 交互式选择默认 provider 和模型。 |
| `hermes gateway` | 运行或管理消息网关服务。 |
| `hermes setup` | 交互式设置向导。 |
| `hermes auth` | 管理凭证。 |
| `hermes status` | 显示 agent、认证和平台状态。 |
| `hermes cron` | 检查和触发 cron 调度器。 |
| `hermes kanban` | 多 profile 协作看板。 |
| `hermes webhook` | 管理动态 webhook 订阅。 |
| `hermes doctor` | 诊断配置和依赖问题。 |
| `hermes backup` | 将 Hermes 主目录备份为 zip 文件。 |
| `hermes logs` | 查看、跟踪和过滤日志文件。 |
| `hermes config` | 显示、编辑、迁移和查询配置文件。 |
| `hermes skills` | 浏览、安装、发布、审计和配置技能。 |
| `hermes mcp` | 管理 MCP server 配置。 |
| `hermes plugins` | 管理插件。 |
| `hermes sessions` | 浏览、导出、清理和删除会话。 |
| `hermes profile` | 管理 profiles。 |
| `hermes update` | 拉取最新代码并重新安装依赖。 |

### hermes chat

```
hermes chat [options]
```

常用选项：`-q` 一次性查询、`-m` 覆盖模型、`-t` 启用工具集、`--provider` 强制 provider、`--yolo` 跳过批准、`--ignore-user-config` 忽略配置。

### hermes -z &lt;prompt&gt;

程序化一次性调用：单次提示输入，最终响应文本输出，无其他输出。

```
hermes -z "What's the capital of France?"
# → Paris.
```

### hermes model

交互式 provider + 模型选择器。用于添加新 provider、设置 API 密钥和运行 OAuth 流程。

### hermes gateway

子命令：`run`（前台运行）、`start/stop/restart`（服务管理）、`install/uninstall`（安装服务）、`setup`（交互式设置）。

### hermes setup

`hermes setup [model|tts|terminal|gateway|tools|agent]` 跳转到特定部分。使用 `--non-interactive` 使用默认值、`--reset` 重置配置。

### hermes auth

管理凭证池。`hermes auth list` 显示、`add` 添加、`remove` 删除、`reset` 重置。

### hermes cron

统一计划任务管理器。`list` 显示、`create` 创建、`edit` 更新、`pause/resume` 暂停恢复、`remove` 删除。

### hermes kanban

多 profile、多项目协作看板。`boards` 子命令管理看板，`create/show/assign/complete/block/dispatch` 等操作管理任务。

### hermes webhook

管理动态 webhook 订阅。`subscribe` 创建、`list` 显示、`remove` 删除、`test` 测试。

### hermes doctor

诊断配置和依赖问题。`--fix` 尝试自动修复。

### hermes dump

输出完整的 Hermes 设置摘要，用于支持请求时复制粘贴。

### hermes logs

查看日志。`agent`（默认）、`errors`、`gateway`。使用 `-f` 跟踪、`--level` 过滤级别、`--since` 时间过滤。

### hermes config

`show` 显示、`edit` 编辑、`set` 设置值、`path` 路径、`migrate` 迁移。

### hermes skills

`browse` 浏览、`search` 搜索、`install` 安装、`inspect` 预览、`list` 列出、`check/update` 检查更新、`config` 配置。

### hermes curator

策展人定期审查技能。`status` 状态、`run` 触发审查、`run --dry-run` 预览、`backup/rollback` 快照恢复、`pause/resume` 暂停恢复。

### hermes mcp

管理 MCP server。`serve` 运行、`add/remove` 添加移除、`list` 列出、`test` 测试、`configure` 配置工具选择。

### hermes plugins

`install` 安装、`update/remove` 更新移除、`enable/disable` 启用禁用、`list` 列出。

### hermes sessions

`list` 列出、`browse` 交互选择、`export` 导出、`delete/prune` 删除清理、`rename` 重命名。

### hermes dashboard

启动 Web 仪表板。`--port` 端口（默认 9119）、`--host` 绑定地址、`--no-open` 不打开浏览器。

### hermes profile

`list` 列出、`use` 设置活动、`create` 创建、`delete` 删除、`show` 显示、`export/import` 导出导入。

---

## 斜杠命令参考

Hermes 有两个斜杠命令界面：交互式 CLI 斜杠命令和消息斜杠命令（在 Telegram、Discord 等平台可用）。

### 会话命令

| 命令 | 描述 |
| --- | --- |
| `/new` / `/reset` | 开始新会话 |
| `/clear` | 清屏并开始新会话 |
| `/history` | 显示对话历史 |
| `/save` | 保存当前对话 |
| `/retry` | 重试上一条消息 |
| `/undo` | 移除最后一个交换 |
| `/title` | 设置会话标题 |
| `/compress` | 手动压缩对话上下文 |
| `/rollback` | 列出或恢复检查点 |
| `/snapshot` | 创建或恢复状态快照 |
| `/stop` | 终止所有后台进程 |
| `/queue &lt;prompt&gt;` | 为下一轮排队提示 |
| `/steer &lt;prompt&gt;` | 在中途运行后注入笔记 |
| `/goal &lt;text&gt;` | 设定持续目标 |
| `/resume [name]` | 恢复之前命名的会话 |
| `/background &lt;prompt&gt;` | 在后台会话中运行提示 |
| `/branch` | 分支当前会话 |

### 配置命令

| 命令 | 描述 |
| --- | --- |
| `/config` | 显示当前配置 |
| `/model [model-name]` | 显示或更改当前模型 |
| `/personality` | 设置预定义人格 |
| `/verbose` | 循环工具进度显示 |
| `/fast [normal|fast|status]` | 切换快速模式 |
| `/reasoning` | 管理推理努力和显示 |
| `/skin` | 显示或更改显示皮肤 |
| `/voice [on|off|tts|status]` | 切换语音模式 |
| `/yolo` | 切换 YOLO 模式 |
| `/footer [on|off|status]` | 切换运行时元数据页脚 |

### 工具和技能命令

| 命令 | 描述 |
| --- | --- |
| `/tools [list|disable|enable]` | 管理工具 |
| `/toolsets` | 列出可用工具集 |
| `/browser [connect|disconnect|status]` | 管理浏览器连接 |
| `/skills` | 管理技能 |
| `/cron` | 管理计划任务 |
| `/kanban &lt;action&gt;` | 驱动协作看板 |
| `/reload-mcp` | 重新加载 MCP server |
| `/plugins` | 列出插件 |

### 信息命令

| 命令 | 描述 |
| --- | --- |
| `/help` | 显示帮助 |
| `/usage` | 显示 token 使用量和费用 |
| `/insights` | 显示使用分析 |
| `/platforms` | 显示网关状态 |
| `/debug` | 上传调试报告 |
| `/profile` | 显示活动 profile |

### 退出命令

`/quit` 或 `/exit` 退出 CLI。

### 消息平台命令

在 Telegram、Discord、Slack、WhatsApp 等平台中可用：

| 命令 | 描述 |
| --- | --- |
| `/new` | 开始新对话 |
| `/reset` | 重置对话历史 |
| `/status` | 显示会话信息 |
| `/stop` | 终止后台进程 |
| `/model [provider:model]` | 切换模型 |
| `/retry` | 重试消息 |
| `/undo` | 撤销 |
| `/compress` | 压缩上下文 |
| `/usage` | 显示使用统计 |
| `/insights [days]` | 显示分析 |
| `/voice [on|off]` | 控制语音 |
| `/background &lt;prompt&gt;` | 后台运行 |
| `/queue &lt;prompt&gt;` | 排队提示 |
| `/goal &lt;text&gt;` | 设定目标 |
| `/update` | 更新 Hermes |
| `/restart` | 重启网关 |
| `/approve` | 批准危险命令 |
| `/deny` | 拒绝危险命令 |

### 注意事项

`/skin`、`/snapshot`、`/gquota`、`/reload`、`/tools`、`/toolsets`、`/browser`、`/config`、`/cron`、`/skills`、`/platforms`、`/paste`、`/image`、`/statusbar`、`/plugins`、`/busy`、`/indicator`、`/redraw`、`/clear`、`/history`、`/save`、`/copy` 和 `/quit` 是**仅 CLI**命令。

`/sethome`、`/update`、`/restart`、`/approve`、`/deny`、`/topic` 和 `/commands` 是**仅消息**命令。

---

## Profile 命令参考

本页面涵盖与 Hermes profiles 相关的所有命令。

### hermes profile

```
hermes profile &lt;subcommand&gt;
```

| 子命令 | 描述 |
| --- | --- |
| `list` | 列出所有 profiles |
| `use &lt;name&gt;` | 设置活动 profile |
| `create &lt;name&gt;` | 创建新 profile |
| `delete &lt;name&gt;` | 删除 profile |
| `show &lt;name&gt;` | 显示 profile 详情 |
| `alias` | 重新生成 shell 别名 |
| `rename` | 重命名 profile |
| `export` | 导出为 tar.gz |
| `import` | 从 tar.gz 导入 |

### hermes profile create

```
hermes profile create &lt;name&gt; [options]
```

`--clone` 复制配置、`--clone-all` 复制所有、`--clone-from` 从指定 profile 克隆、`--no-alias` 跳过别名创建。

### hermes -p / --profile

```
hermes -p &lt;name&gt; &lt;command&gt; [options]
```

在特定 profile 下运行任何 Hermes 命令。

### hermes completion

```
hermes completion &lt;shell&gt;
```

生成 shell 补全脚本（bash 或 zsh）。

---

## 环境变量参考

所有变量放在 `~/.hermes/.env` 中。

### LLM Providers

| 变量 | 描述 |
| --- | --- |
| `OPENROUTER_API_KEY` | OpenRouter API 密钥 |
| `OPENAI_API_KEY` | OpenAI API 密钥 |
| `ANTHROPIC_API_KEY` | Anthropic API 密钥 |
| `GOOGLE_API_KEY` | Google AI Studio API 密钥 |
| `DEEPSEEK_API_KEY` | DeepSeek API 密钥 |
| `HERMES_INFERENCE_PROVIDER` | 覆盖 provider 选择 |
| `HERMES_MODEL` | 覆盖模型名称 |

### Tool APIs

| 变量 | 描述 |
| --- | --- |
| `PARALLEL_API_KEY` | AI 网络搜索 |
| `FIRECRAWL_API_KEY` | Web 抓取 |
| `TAVILY_API_KEY` | Tavily 搜索 |
| `EXA_API_KEY` | Exa 搜索 |
| `FAL_KEY` | 图像生成 |
| `ELEVENLABS_API_KEY` | 语音合成 |
| `GITHUB_TOKEN` | GitHub 令牌 |
| `WANDB_API_KEY` | W&B 指标 |

### Terminal Backend

| 变量 | 描述 |
| --- | --- |
| `TERMINAL_ENV` | 后端类型：local、docker、ssh 等 |
| `TERMINAL_DOCKER_IMAGE` | Docker 镜像 |
| `TERMINAL_TIMEOUT` | 命令超时秒数 |
| `TERMINAL_CWD` | 工作目录 |

### Messaging

| 变量 | 描述 |
| --- | --- |
| `TELEGRAM_BOT_TOKEN` | Telegram bot 令牌 |
| `TELEGRAM_ALLOWED_USERS` | 允许的用户 ID |
| `DISCORD_BOT_TOKEN` | Discord bot 令牌 |
| `SLACK_BOT_TOKEN` | Slack bot 令牌 |
| `WHATSAPP_ENABLED` | 启用 WhatsApp |
| `HASS_TOKEN` | Home Assistant 令牌 |

### Agent Behavior

| 变量 | 描述 |
| --- | --- |
| `HERMES_MAX_ITERATIONS` | 最大迭代次数（默认：90） |
| `HERMES_INFERENCE_MODEL` | 覆盖模型名称 |
| `HERMES_YOLO_MODE` | 绕过危险命令提示 |
| `HERMES_API_TIMEOUT` | API 超时秒数 |
| `HERMES_STREAM_READ_TIMEOUT` | 流式读取超时 |
| `HERMES_AGENT_TIMEOUT` | Agent 非活动超时 |

### Interface

| 变量 | 描述 |
| --- | --- |
| `HERMES_TUI` | 启动 TUI 界面 |
| `HERMES_TUI_THEME` | TUI 颜色主题 |

### Session Settings

| 变量 | 描述 |
| --- | --- |
| `SESSION_IDLE_MINUTES` | 非活动后重置会话（默认：1440） |
| `SESSION_RESET_HOUR` | 每日重置小时（默认：4） |

---

## 内置工具参考

Hermes 工具注册表中的所有内置工具，按工具集分组。

### browser 工具集

| 工具 | 描述 |
| --- | --- |
| `browser_back` | 导航回上一页 |
| `browser_click` | 点击元素 |
| `browser_console` | 获取控制台输出 |
| `browser_get_images` | 获取页面图像 |
| `browser_navigate` | 导航到 URL |
| `browser_press` | 按键 |
| `browser_scroll` | 滚动页面 |
| `browser_snapshot` | 获取页面快照 |
| `browser_type` | 输入文本 |
| `browser_vision` | 截图分析 |

### file 工具集

| 工具 | 描述 |
| --- | --- |
| `patch` | 查找替换编辑 |
| `read_file` | 读取文件 |
| `search_files` | 搜索文件内容 |
| `write_file` | 写入文件 |

### 其他工具集

| 工具集 | 主要工具 | 描述 |
| --- | --- | --- |
| `clarify` | clarify | 向用户提问 |
| `code_execution` | execute_code | 运行 Python 脚本 |
| `cronjob` | cronjob | 管理计划任务 |
| `delegation` | delegate_task | 生成 subagent |
| `homeassistant` | ha_* | 智能家居控制 |
| `image_gen` | image_generate | 生成图像 |
| `memory` | memory | 记忆管理 |
| `messaging` | send_message | 发送消息 |
| `moa` | mixture_of_agents | 多模型共识 |
| `rl` | rl_* | 强化学习训练 |
| `session_search` | session_search | 搜索过去对话 |
| `skills` | skill_* | 技能管理 |
| `terminal` | process, terminal | shell 命令 |
| `todo` | todo | 任务列表 |
| `tts` | text_to_speech | 语音合成 |
| `vision` | vision_analyze | 图像分析 |
| `web` | web_search, web_extract | 网页搜索和提取 |

---

## 工具集参考

工具集是工具的命名捆绑，控制 agent 可以做什么。

### 工具集类型

- **核心** — 相关工具组
- **复合** — 组合多个核心工具集
- **平台** — 特定部署上下文的完整工具配置

### 配置工具集

```
hermes chat --toolsets web,file,terminal
hermes chat --toolsets debugging
hermes chat --toolsets all
```

### 核心工具集

| 工具集 | 包含 | 用途 |
| --- | --- | --- |
| `browser` | 10 个浏览器工具 | 浏览器自动化 |
| `clarify` | clarify | 向用户提问 |
| `debugging` | file + terminal + web | 调试捆绑 |
| `delegation` | delegate_task | subagent |
| `file` | patch, read_file, search_files, write_file | 文件操作 |
| `homeassistant` | 4 个 HA 工具 | 智能家居控制 |
| `memory` | memory | 记忆管理 |
| `messaging` | send_message | 发送消息 |
| `rl` | 10 个 RL 工具 | 强化学习训练 |
| `safe` | 4 个只读工具 | 只读研究 |
| `terminal` | process, terminal | shell 命令 |
| `todo` | todo | 任务列表 |
| `tts` | text_to_speech | 语音合成 |
| `vision` | vision_analyze | 图像分析 |
| `web` | web_extract, web_search | 网页搜索和提取 |

### 平台工具集

| 工具集 | 描述 |
| --- | --- |
| `hermes-cli` | 完整工具集（38 个工具） |
| `hermes-acp` | IDE 上下文，专注编码 |
| `hermes-api-server` | 程序化访问 |
| `hermes-discord` | 添加 Discord 工具 |
| `hermes-feishu` | 添加 Feishu 工具 |
| `hermes-yuanbao` | 添加 Yuanbao 工具 |

---

## MCP 配置参考

MCP（Model Context Protocol）server 配置参考。

### 根配置形状

```
mcp_servers:
  &lt;server_name&gt;:
    command: "..."
    args: []
    env: {}
    # OR
    url: "..."
    headers: {}
    enabled: true
    timeout: 120
    tools:
      include: []
      exclude: []
      resources: true
      prompts: true
```

### Server 键

| 键 | 类型 | 含义 |
| --- | --- | --- |
| `command` | string | 可执行文件（stdio） |
| `args` | list | 参数（stdio） |
| `env` | mapping | 环境变量（stdio） |
| `url` | string | 端点 URL（HTTP） |
| `headers` | mapping | 请求头（HTTP） |
| `enabled` | bool | 是否启用 |
| `timeout` | number | 工具调用超时 |
| `auth` | string | 设为 oauth 启用 OAuth |

### tools 策略键

| 键 | 含义 |
| --- | --- |
| `include` | 白名单工具 |
| `exclude` | 黑名单工具 |
| `resources` | 启用 list_resources + read_resource |
| `prompts` | 启用 list_prompts + get_prompt |

### 过滤语义

`include` 和 `exclude` 同时设置时，`include` 优先。

### 工具命名

Server 原生 MCP 工具变为：`mcp_&lt;server&gt;_&lt;tool&gt;`

---

## 模型目录

Hermes 从 JSON 清单获取 OpenRouter 和 Nous Portal 的精选模型列表。

### 实时清单 URL

```
https://hermes-agent.nousresearch.com/docs/api/model-catalog.json
```

### 架构

```
{
  "version": 1,
  "updated_at": "2026-04-25T22:00:00Z",
  "providers": {
    "openrouter": { "models": [...] },
    "nous": { "models": [...] }
  }
}
```

### 获取行为

| 情况 | 行为 |
| --- | --- |
| /model 或 hermes model | 缓存过期时获取 |
| 缓存新鲜 | 无网络请求 |
| 网络失败 + 缓存 | 静默回退到缓存 |
| 网络失败 + 无缓存 | 静默回退到内置快照 |

### 配置

```
model_catalog:
  enabled: true
  url: https://hermes-agent.nousresearch.com/docs/api/model-catalog.json
  ttl_hours: 24
```

---

## 捆绑技能目录

Hermes 附带大量内置技能库。

### 主要技能分类

| 分类 | 技能示例 | 描述 |
| --- | --- | --- |
| apple | apple-notes, findmy, imessage | Apple 生态集成 |
| autonomous-ai-agents | claude-code, codex, hermes-agent | AI agent 委托 |
| creative | architecture-diagram, excalidraw, p5js | 创意工具 |
| devops | kanban-orchestrator, webhook-subscriptions | DevOps 工具 |
| github | github-code-review, github-pr-workflow | GitHub 集成 |
| mlops | axolotl, dspy, huggingface-hub, vllm | MLOps 工具 |
| productivity | airtable, google-workspace, notion | 生产力工具 |
| software-development | plan, systematic-debugging, TDD | 软件开发 |

---

## 可选技能目录

可选技能**默认不激活**，需要显式安装：

```
hermes skills install official/&lt;category&gt;/&lt;skill&gt;
```

### 主要可选技能

| 分类 | 技能 | 描述 |
| --- | --- | --- |
| autonomous-ai-agents | blackbox, honcho | AI agent 委托、记忆 |
| blockchain | base, solana | 区块链查询 |
| creative | blender-mcp, kanban-video-orchestrator | 创意工具 |
| devops | docker-management, inference-sh-cli | DevOps 工具 |
| mlops | chroma, faiss, pinecone, whisper | 向量数据库、语音 |
| productivity | canvas, memento-flashcards, telephony | 生产力工具 |
| research | bioinformatics, domain-intel, duckduckgo-search | 研究工具 |
| security | 1password, sherlock | 安全工具 |

---

## 常见问题与故障排除

### 常见问题

#### Hermes 支持哪些 LLM provider？

支持 OpenAI 兼容 API，包括：OpenRouter、Nous Portal、OpenAI、Anthropic、Google、z.ai/ZhipuAI、Kimi/Moonshot AI、MiniMax、本地模型（Ollama、vLLM、llama.cpp）等。

#### 支持 Windows 吗？

**不原生支持。** 需要 WSL2。在 Windows 上安装 WSL2 并在其中运行 Hermes。

#### 数据发送到哪？

API 调用仅发送到您配置的 provider。Hermes 不收集遥测数据。对话、记忆和技能存储在本地 `~/.hermes/`。

#### 可以使用本地模型吗？

可以。运行 `hermes model`，选择"自定义端点"，输入服务器 URL。

#### 费用是多少？

Hermes Agent **免费开源**。您只需支付 LLM API 费用。本地模型完全免费。

### 故障排除

#### 安装问题

**command not found** — 运行 `source ~/.bashrc` 或 `source ~/.zshrc`。

**Python 版本过旧** — Hermes 需要 Python 3.11+。

#### Provider 问题

**/model 只显示一个 provider** — 退出并从终端运行 `hermes model` 添加新 provider。

**API 密钥不工作** — 检查配置：`hermes config show`。

#### 终端问题

**命令被阻止** — 这是安全功能。按 y 批准。

**Docker 无法连接** — 检查 Docker 是否运行：`docker info`。

#### 消息问题

**Bot 不响应** — 检查网关：`hermes gateway status`。

**WSL 网关断开** — 使用前台模式：`hermes gateway run`。

#### 性能问题

**响应慢** — 尝试更小更快的模型，减少活动工具集。

**Token 使用量高** — 使用 `/compress` 压缩对话。

#### MCP 问题

**MCP server 无法连接** — 验证配置和依赖。

**工具未显示** — 检查工具过滤配置。

### Profiles

**Profiles 和 HERMES_HOME 区别？** Profiles 是托管层，处理目录、别名、profile 跟踪和技能同步。

**两个 profiles 共享 token？** 不能，每个消息平台需要独占 token。

### 仍有问题？

- 搜索 GitHub Issues
- 加入 Nous Research Discord
- 提交 bug 报告（包含 OS、Python 版本、Hermes 版本）

---
