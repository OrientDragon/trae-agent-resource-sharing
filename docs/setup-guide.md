# 部署指南（跨智能体通用版）

本指南将帮助你从零开始搭建跨智能体的共享资源体系。该体系与具体智能体无关，可被 WorkBuddy、TRAE、Claude Code、ChatGPT、Codex、Gemini CLI、Cursor 等所有支持「读取本地目录 + 加载 SKILL.md」机制的智能体共用。

## 前置条件

- 拥有 `<root>`（默认 `~/.agent-shared/`，可用环境变量 `AGENT_SHARED_ROOT` 覆盖）目录的读写权限
- 至少有一个可用的 AI 智能体环境
- 如需自定义路径，先设置环境变量 `AGENT_SHARED_ROOT` 并让所有智能体引用同一值

---

## 步骤一：创建固定共享根目录结构

在 `<root>`（默认 `~/.agent-shared/`）下创建目录结构：

```
~/.agent-shared/
├── skills/
│   ├── builtin/
│   ├── user/
│   ├── project/
│   ├── remote/
│   └── quarantine/
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
├── locks/
├── registry/
└── backups/
```

**Windows PowerShell 创建命令：**

```powershell
$baseDir = "$env:USERPROFILE\.agent-shared"
$dirs = @(
    "skills\builtin",
    "skills\user",
    "skills\project",
    "skills\remote",
    "skills\quarantine",
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
    "locks",
    "registry",
    "backups"
)
foreach ($d in $dirs) {
    New-Item -ItemType Directory -Force -Path (Join-Path $baseDir $d) | Out-Null
}
Write-Host "Shared root directory structure created at $baseDir"
```

**macOS / Linux 创建命令：**

```bash
BASE="$HOME/.agent-shared"
mkdir -p "$BASE/skills/builtin" "$BASE/skills/user" "$BASE/skills/project" \
         "$BASE/skills/remote" "$BASE/skills/quarantine" "$BASE/mcps" \
         "$BASE/templates/project" "$BASE/templates/component" "$BASE/templates/document" \
         "$BASE/knowledge/best-practices" "$BASE/knowledge/faq" "$BASE/knowledge/domain-knowledge" \
         "$BASE/artifacts/code-snippets" "$BASE/artifacts/configs" "$BASE/artifacts/datasets" \
         "$BASE/config" "$BASE/locks" "$BASE/registry" "$BASE/backups"
echo "Shared root directory structure created at $BASE"
```

---

## 步骤二：初始化资源注册表

在 `<root>/registry/` 目录下创建 `resource-registry.json`：

```json
{
  "schemaVersion": "1.0",
  "lastUpdated": "2026-10-15T00:00:00+08:00",
  "description": "跨智能体共享资源注册表",
  "root": "~/.agent-shared/",
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

---

## 步骤三：安装共享 Skill

将需要共享的 Skill 放入对应命名空间的目录中。命名空间包括：`builtin`（内置）、`user`（用户自定义）、`project`（项目级）、`remote`（远程/第三方）。

例如，安装用户自定义 skill：

```bash
# 创建 skill 目录（使用 @version 格式）
mkdir -p ~/.agent-shared/skills/user/my-skill@1.0.0

# 放入 SKILL.md
cp /path/to/SKILL.md ~/.agent-shared/skills/user/my-skill@1.0.0/
```

然后在注册表中添加该资源的元数据。

---

## 步骤四：迁移已有资源到共享池

如果你已在某智能体的私有目录中安装了 skill，想迁移到共享池：

1. 找到源 skill 目录位置（如 `~/.workbuddy/skills/`、`~/.trae-cn/...`、`~/.claude/...`）
2. 复制到 `<root>/skills/<namespace>/<name>@<version>/`
3. 在注册表中登记
4. 验证其他智能体能否正确发现和加载
5. 验证无误后，可将原私有副本改为指向共享根的符号链接（可选）

---

## 步骤五：配置智能体资源锁定（可选）

如果某个智能体需要锁定特定版本的资源，在 `<root>/locks/<agent-id>/` 或智能体私有目录创建 `resource-lock.json`：

```json
{
  "lockedResources": {
    "user:my-skill": "1.0.0",
    "builtin:html-report": "2.1.0"
  }
}
```

---

## 步骤六：验证

1. 各智能体启动时扫描 `<root>/registry/resource-registry.json`，发现共享资源
2. 检查智能体是否能发现共享 skill
3. 尝试调用一个共享 skill，确认可正常使用

---

## 步骤七：从远程安装资源（带安全审计，可选）

```bash
# 示例：从 GitHub 安装 Skill（先隔离审计，再移入）
1. 克隆/下载资源到 ~/.agent-shared/skills/quarantine/<name>/   # 临时隔离区
2. 执行安全审计（静态扫描 + 依赖检查，见规则21）
3. 审计通过 → 移动到 ~/.agent-shared/skills/remote/github/<name>@<version>/
4. 审计可疑 → 提示用户确认
5. 审计恶意 → 自动删除，加入 ~/.agent-shared/registry/blacklist.json
```
