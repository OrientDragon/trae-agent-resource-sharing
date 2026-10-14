---
name: "agent-resource-sharing"
description: "Manages shared resource rules and directory structure across multiple TRAE agents (skills, MCPs, templates, knowledge, artifacts). Invoke when installing shared resources, setting up resource sharing, migrating resources between agents, or managing the global shared directory."
---

# 智能体资源共享管理 Skill

本 Skill 定义了 TRAE 多智能体环境下的资源共享规则、目录结构和操作规范，确保 Skill、MCP、模板、知识库、成果文件等资源在各智能体间互通复用、独立存续。

## 何时调用

- 用户要求安装/迁移/管理共享资源时
- 需要在多个智能体间共享 Skill、MCP 或其他资源时
- 搭建新的智能体环境，需要复用已有资源时
- 制定或调整资源共享策略时
- 卸载智能体但需要保留其安装的资源时

---

## 一、核心原则

### 规则1：资源独立存续原则

**Skill、MCP、成果文件等资源独立存在，不隶属于任何单一智能体。**

- 卸载/删除任一智能体时，不得删除其安装过的 skill、MCP 服务、生成的成果文件
- 智能体 A 安装的 skill，智能体 B 可直接发现并使用，无需重复安装
- 资源的生命周期独立于智能体的生命周期
- 智能体仅作为资源的"使用者"和"安装者"，而非"所有者"

### 规则2：共享资源集中存储原则

**所有可共享的资源集中存放于独立目录，各智能体均可读取，不重复存储。**

- 共享资源存放于全局独立目录（`~/.trae-cn/shared/`），不嵌入任何智能体的私有目录
- 同一资源（同名同版本）在系统中只保留一份
- 智能体对共享资源默认为只读，不得主动删除、修改、移动
- 资源的写入/更新需通过统一的安装/升级流程进行

---

## 二、完整规则体系

### 规则3：资源发现与注册机制

全局维护 `shared/registry/resource-registry.json`，记录所有已安装资源的元信息。新资源安装时自动注册，智能体启动时扫描注册表发现可用资源。

### 规则4：版本共存与兼容规则

资源目录按 `名称@版本号` 组织（如 `html-report@2.1.0`），支持多版本并存。每个智能体可锁定版本或使用最新版本。版本号遵循 SemVer 规范。

### 规则5：依赖自动解析规则

Skill、MCP 可声明依赖，安装时自动检查并安装缺失的依赖。依赖资源同样进入全局共享池。

### 规则6：命名空间与命名规范

- 内置资源：`builtin:<资源名>`
- 官方插件资源：`<插件名>:<资源名>`（如 `lark:lark-doc`）
- 用户自定义资源：`user:<资源名>`
- 第三方来源：`<来源标识>:<资源名>`（如 `github:xxx/yyy`）

### 规则7：成果文件共享规则

代码模板、配置文件、文档模板、数据集等可标记为共享，存放于 `shared/artifacts/`。使用时引用而非复制，确保单一数据源。

### 规则8：配置与环境共享规则

通用配置存放于 `shared/config/`，凭证类信息加密存储，需显式授权才能使用。共享配置按"配置集"组织。

### 规则9：知识库与记忆共享规则

通用知识（最佳实践、FAQ、领域知识等）可发布为共享 knowledge skill。私有对话记忆不共享。

### 规则10：引用而非复制原则

智能体使用共享资源时通过引用方式，不做本地副本。资源更新时所有引用方自动获得最新版本（除非锁定了版本）。

### 规则11：资源锁定与冲突避免

每个智能体维护自己的 `resource-lock.json`，声明锁定的资源版本。未锁定的默认使用最新兼容版本。

### 规则12：资源生命周期管理

- 安装：下载/复制到共享目录 → 注册到注册表
- 升级：安装新版本，旧版本保留
- 降级：通过锁定机制切换到旧版本
- 卸载：从注册表移除，文件默认保留以便回退
- 清理：定期扫描未引用的旧版本，用户确认后清理

### 规则13：安全与权限隔离

资源分安全等级：公开（所有智能体可用）、受限（需授权）、私有（仅创建者可用）。第三方资源安装前提示安全风险。

### 规则14：元数据规范

必选字段：`name`、`version`、`type`、`description`、`author`、`installTime`、`dependencies`、`compatibility`

### 规则15：MCP 服务共享规则

MCP 服务全局注册，同一服务只启动一个实例，多智能体共享使用。配置全局存储，智能体可按工具粒度授权。

### 规则16：模板与脚手架共享规则

项目模板、代码脚手架存放于 `shared/templates/`。智能体创建项目时可从共享模板库选择。

### 规则17：资源使用统计与反馈

匿名统计调用频次、成功率，智能体可对资源打分评价。长期未使用的资源标记为"可能过时"。

### 规则18：跨设备同步规则

共享资源目录可配置同步策略（本地 / 云端 / 混合）。云端同步时敏感信息加密处理。

### 规则19：插件级共享规则

插件安装到全局插件目录，所有智能体均可启用其中的能力。智能体可按需启用/禁用特定插件的特定能力。

### 规则20：故障隔离与降级

单个共享资源故障不影响其他资源和智能体。智能体可配置备选资源（fallback），主资源不可用时自动切换。

---

## 三、共享目录结构

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

---

## 四、操作指南

### 4.1 安装一个共享 Skill

1. 确定资源名称和命名空间（如 user:my-skill）
2. 在 shared/skills/<namespace>/ 下创建 <name>@<version>/ 目录
3. 放入 SKILL.md 和相关资源文件
4. 在 resource-registry.json 中注册该资源
5. 在各智能体的可用资源列表中即可发现

### 4.2 迁移已安装的 Skill 到共享池

1. 找到源 Skill 所在目录
2. 复制到 shared/skills/<对应命名空间>/<name>@<version>/
3. 在注册表中登记
4. 验证其他智能体能否正确发现和加载

### 4.3 锁定资源版本（智能体私有）

在智能体私有目录下创建 `resource-lock.json`：

```json
{
  "lockedResources": {
    "builtin:html-report": "2.1.0",
    "user:my-skill": "1.0.0"
  }
}
```

### 4.4 卸载智能体但保留资源

只需删除智能体私有目录，`~/.trae-cn/shared/` 下的所有共享资源不受影响。

---

## 五、注册表格式规范

`resource-registry.json` 结构：

```json
{
  "schemaVersion": "1.0",
  "lastUpdated": "ISO 8601 时间戳",
  "description": "全局共享资源注册表",
  "stats": {
    "totalResources": 0,
    "byType": {
      "skill": 0,
      "mcp": 0,
      "template": 0,
      "knowledge": 0,
      "artifact": 0,
      "config": 0
    }
  },
  "resources": [
    {
      "id": "命名空间:名称",
      "name": "资源名称",
      "namespace": "builtin | plugins | user | github",
      "type": "skill | mcp | template | knowledge | artifact | config",
      "version": "1.0.0",
      "description": "简短描述",
      "author": "作者/来源",
      "installTime": "ISO 8601 安装时间",
      "path": "相对于 shared/ 的路径",
      "dependencies": [],
      "compatibility": {
        "agentTypes": ["work", "code", "design"],
        "minVersion": "1.0.0"
      },
      "tags": [],
      "securityLevel": "public | restricted | private",
      "usageCount": 0
    }
  ]
}
```

---

## 六、执行优先级

1. **用户显式指令** > 规则约定
2. **数据安全与隐私保护** > 共享便利性
3. **系统稳定性** > 资源复用效率
4. **向后兼容** > 版本统一

---

## 七、注意事项

- 操作共享资源前确认用户意图，避免误改
- 敏感信息（凭证、Token 等）不得明文存储在共享目录
- 第三方来源的资源安装前需提示风险并获得用户确认
- 资源删除操作需二次确认，默认采用"软删除"（仅从注册表移除，文件保留）
