# Agent Resource Sharing（跨智能体通用版）

> 一套与具体智能体无关的资源共享与复用规则体系，让 Skill、MCP/Connector、模板、知识库、成果文件在 **WorkBuddy、TRAE、Claude Code、ChatGPT、Codex、Gemini CLI、Cursor 等所有 AI 智能体**之间互通复用、独立存续。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform: Cross-Agent](https://img.shields.io/badge/Platform-Cross--Agent-blue.svg)]()
[![Version](https://img.shields.io/badge/version-2.0.0-green.svg)]()

## 项目简介

本项目定义了一套**不绑定任何单一智能体**的资源共享规则、固定目录约定与操作规范。核心思想是：所有可共享的资源统一存放在一个**固定的、中立的共享根目录**（`~/.agent-shared/`）中，任何支持「读取本地目录 + 加载 SKILL.md」机制的智能体都能直接套用，实现资源跨智能体互通、且卸载任一智能体都不会丢失资源。

> 设计哲学与「成果独立于工具、自包含不依赖某个载体」一致——资源独立于智能体而存续。

## 核心规则

1. **固定共享根目录（强制，规则0）** - 所有共享资源必须落在 `~/.agent-shared/`，禁止散落各智能体私有目录
2. **资源独立存续** - 资源不隶属于任何智能体，卸载智能体不删除资源
3. **共享资源集中存储** - 全局独立目录，不重复存储
4. **资源发现与注册** - 全局注册表，启动自动发现
5. **版本共存与兼容** - 多版本并存，SemVer 规范
6. **依赖自动解析** - 自动补全依赖，依赖也共享
7. **命名空间规范** - `builtin` / `user` / `project` / `remote`
8. **成果文件共享** - 模板/配置/数据集可标记共享
9. **配置与环境共享** - 通用配置全局存储，凭证加密
10. **知识库与记忆共享** - 通用知识沉淀，私有记忆不共享
11. **引用而非复制** - 单一数据源，自动获得更新
12. **安全审计（规则21）** - 远程资源强制审计，防恶意注入

更多规则（共 28 条 + 强制规则0）详见 [docs/rules.md](docs/rules.md)

## 智能体适配

本规范不依赖任何智能体的私有目录，各智能体只需最小适配即可接入同一份共享池：

| 智能体 | 私有目录（不存放共享资源） | 接入方式 |
|--------|--------------------------|---------|
| WorkBuddy | `~/.workbuddy/` | 直接读取 `~/.agent-shared/`；引用/软链，不复制 |
| TRAE | `~/.trae-cn/` | 同上；原 `~/.trae-cn/shared/` 可软链到 `~/.agent-shared/` 平滑迁移 |
| Claude Code | `~/.claude/` | 同上 |
| ChatGPT / Codex | 应用数据目录 | 同上；读取 `~/.agent-shared/` 加载共享 skill |
| Gemini CLI / Cursor / 其他 | 各自私有目录 | 同上 |

> 环境变量 `AGENT_SHARED_ROOT` 可覆盖默认路径（便携盘 / 云同步 / 团队统一）。一旦设定，所有智能体必须遵循同一路径。

## 目录结构

```
~/.agent-shared/                         # 固定共享根（AGENT_SHARED_ROOT 可覆盖）
├── skills/
│   ├── builtin/           # 内置 Skill
│   ├── user/              # 用户自定义 Skill
│   ├── project/           # 项目级 Skill
│   ├── remote/            # 远程/第三方 Skill
│   └── quarantine/        # 隔离区（待审计的远程资源）
├── mcps/                  # 共享 MCP / Connector 服务
├── templates/             # 共享模板与脚手架
├── knowledge/             # 共享知识库
├── artifacts/             # 共享成果文件
├── config/                # 共享配置
├── locks/                 # 各智能体版本锁定
├── registry/
│   ├── resource-registry.json   # 全局资源注册表（唯一真相源）
│   ├── blacklist.json           # 恶意资源黑名单
│   └── audit-log.json           # 审计日志
└── backups/               # 备份（按日期）
```

## 快速开始

1. 创建固定共享根目录结构（参考 [docs/setup-guide.md](docs/setup-guide.md)）
2. 初始化资源注册表
3. 安装共享 Skill
4. 迁移已有资源到共享池

## 文档

- [完整规则文档](docs/rules.md)
- [部署指南](docs/setup-guide.md)
- [注册表格式规范](docs/setup-guide.md#步骤二初始化资源注册表)（亦见 SKILL.md「五、注册表格式规范」）

## 许可证

MIT License - 详见 [LICENSE](LICENSE) 文件
