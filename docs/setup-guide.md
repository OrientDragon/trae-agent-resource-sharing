# 部署指南

本指南将帮助你从零开始搭建 TRAE 多智能体共享资源体系。

## 前置条件

- 已安装 TRAE 客户端
- 至少有一个可用的智能体（WorkBuddy、CodeBuddy 等）
- 拥有 `~/.trae-cn/` 目录的读写权限

## 步骤一：创建共享目录结构

在 `~/.trae-cn/` 下创建 `shared/` 目录及其子目录：

```
~/.trae-cn/shared/
├── skills/
│   ├── builtin/
│   ├── plugins/
│   └── user/
├── mcps/
├── templates/
│   ├── project/
│   ├── component/
│   └── document/
├── knowledge/
│   ├── best-practices/
│   ├── faq/
│   └── domain-knowledge/
├── artifacts/
│   ├── code-snippets/
│   ├── configs/
│   └── datasets/
├── config/
└── registry/
```

**Windows PowerShell 创建命令：**

```powershell
$baseDir = "$env:USERPROFILE\.trae-cn\shared"
$dirs = @(
    "skills\builtin",
    "skills\plugins",
    "skills\user",
    "mcps",
    "templates\project",
    "templates\component",
    "templates\document",
    "knowledge\best-practices",
    "knowledge\faq",
    "knowledge\domain-knowledge",
    "artifacts\code-snippets",
    "artifacts\configs",
    "artifacts\datasets",
    "config",
    "registry"
)
foreach ($d in $dirs) {
    New-Item -ItemType Directory -Force -Path (Join-Path $baseDir $d) | Out-Null
}
Write-Host "Shared directory structure created at $baseDir"
```

## 步骤二：初始化资源注册表

在 `shared/registry/` 目录下创建 `resource-registry.json`：

```json
{
  "schemaVersion": "1.0",
  "lastUpdated": "2026-10-14T00:00:00+08:00",
  "description": "TRAE 全局共享资源注册表",
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
  "resources": []
}
```

## 步骤三：安装共享 Skill

将需要共享的 Skill 放入对应命名空间的目录中。

例如，安装用户自定义 skill：

```bash
# 创建 skill 目录（使用 @version 格式）
mkdir -p ~/.trae-cn/shared/skills/user/my-skill@1.0.0

# 放入 SKILL.md
cp /path/to/SKILL.md ~/.trae-cn/shared/skills/user/my-skill@1.0.0/
```

然后在注册表中添加该资源的元数据。

## 步骤四：迁移已有资源到共享池

如果你已有在某智能体中安装的 skill，想迁移到共享池：

1. 找到源 skill 目录位置
2. 复制到 `shared/skills/<namespace>/<name>@<version>/`
3. 在注册表中登记
4. 验证其他智能体能否正确发现和加载

## 步骤五：配置智能体资源锁定（可选）

如果某个智能体需要锁定特定版本的资源，在智能体私有目录创建 `resource-lock.json`：

```json
{
  "lockedResources": {
    "user:my-skill": "1.0.0",
    "builtin:html-report": "2.1.0"
  }
}
```

## 步骤六：验证

1. 重启 TRAE 或智能体
2. 检查智能体是否能发现共享资源
3. 尝试调用一个共享 skill，确认可正常使用
