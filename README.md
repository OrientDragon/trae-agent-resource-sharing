# TRAE Agent Resource Sharing

> 一套完整的 TRAE 多智能体资源共享与复用规则体系，让 Skill、MCP、模板、知识库、成果文件在各智能体间互通复用、独立存续。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform: TRAE](https://img.shields.io/badge/Platform-TRAE-blue.svg)]()
[![Version](https://img.shields.io/badge/version-1.0.0-green.svg)]()

## 项目简介

本项目定义了 TRAE 多智能体环境下的资源共享规则、目录结构和操作规范，确保 Skill、MCP、模板、知识库、成果文件等资源在各智能体间互通复用、独立存续。

## 核心规则

1. **资源独立存续** - 资源不隶属于任何智能体，卸载智能体不删除资源
2. **共享资源集中存储** - 全局独立目录，不重复存储
3. **资源发现与注册** - 全局注册表，启动自动发现
4. **版本共存与兼容** - 多版本并存，SemVer 规范
5. **依赖自动解析** - 自动补全依赖，依赖也共享
6. **命名空间规范** - builtin/plugins/user/github
7. **成果文件共享** - 模板/配置/数据集可标记共享
8. **配置与环境共享** - 通用配置全局存储，凭证加密
9. **知识库与记忆共享** - 通用知识沉淀，私有记忆不共享
10. **引用而非复制** - 单一数据源，自动获得更新

更多规则详见 [docs/rules.md](docs/rules.md)

## 目录结构

```
~/.trae-cn/shared/
├── skills/
│   ├── builtin/           # 内置 Skill
│   ├── plugins/           # 插件提供的 Skill
│   └── user/              # 用户自定义 Skill
├── mcps/                  # 共享 MCP 服务
├── templates/             # 共享模板与脚手架
├── knowledge/             # 共享知识库
├── artifacts/             # 共享成果文件
├── config/                # 共享配置
└── registry/
    └── resource-registry.json   # 全局资源注册表
```

## 快速开始

1. 创建共享目录结构（参考 docs/setup-guide.md）
2. 初始化资源注册表
3. 安装共享 Skill
4. 迁移已有资源到共享池

## 文档

- [完整规则文档](docs/rules.md)
- [部署指南](docs/setup-guide.md)
- [注册表规范](docs/registry-spec.md)
- [常见问题](docs/faq.md)

## 许可证

MIT License - 详见 [LICENSE](LICENSE) 文件
