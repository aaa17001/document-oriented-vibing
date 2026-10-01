# Document Oriented Vibing 中文增强版（DOV）

> 面向 VS Code + OpenAI Codex 的代码修改审核工具。

本仓库 Fork 自 [ethanitovitch/document-oriented-vibing](https://github.com/ethanitovitch/document-oriented-vibing)，保留原项目的 DOV 审核能力，并作为后续 **中文界面、SVN 隔离工作区、Step Review、DSE 企业透明加密适配** 的开发基线。

![VS Code](https://img.shields.io/badge/VS%20Code-Extension-blue)
![Reviews](https://img.shields.io/badge/Review-Diff%20Hunks-green)
![License](https://img.shields.io/badge/license-MIT-green)

![Document Oriented Vibing review workflow hero](assets/dov-hero.png)

## 这个插件解决什么问题？

Codex 可以快速修改大量代码，但正式项目开发通常还需要明确知道：

- Codex 这一轮到底改了哪些文件；
- 每个代码块具体增加、删除了什么；
- 哪些修改已经审核通过；
- 哪些修改需要拒绝并恢复；
- 下一轮修改是否建立在已确认代码之上。

DOV 把 **代码审核** 放到 AI 编程流程中心：

```text
Codex 修改代码
      ↓
生成 Review Diff
      ↓
按文件 / Hunk 审核
      ↓
Approve / Reject / Undo
      ↓
继续下一轮修改
```

## 当前已有能力

### 1. 捕获 Codex 修改

在 Codex 完成代码修改后输入：

```text
+review
```

DOV 会读取当前 Codex 本地会话，捕获支持的 `apply_patch` 修改并生成 Review。

也可以从 VS Code 命令面板执行：

```text
DOV: Capture Codex Review
```

### 2. 文件级和 Hunk 级审核

可以逐文件、逐代码块查看修改，并分别执行：

- **Approve**：接受修改；
- **Reject**：拒绝修改并把对应代码恢复为修改前内容；
- **Undo**：撤销 Reject，重新恢复 Codex 的修改。

### 3. VS Code 原生 Diff

点击 Review 中的文件或 Hunk，会直接打开 VS Code 原生 Diff Editor，对比：

```text
修改前代码  ↔  修改后代码
```

可以查看新增、删除、修改行和语法高亮。

### 4. 多轮 Review

审核数据默认保存在：

```text
.reviews/
├─ example-review.diff
├─ example-review.diff.state.json
└─ .rounds/
```

可以连续执行多轮：

```text
Round 1
   ↓
提出审核意见
   ↓
Codex 修改
   ↓
Round 2
   ↓
继续审核
```

### 5. Review Comments

可以针对具体修改留下评论，再把反馈交给 Codex 继续调整。

### 6. Codex Agent / Thread 列表

插件读取本机：

```text
CODEX_HOME/sessions
```

默认位置通常为：

```text
~/.codex/sessions
```

并在 Codex 侧边栏显示当前 Workspace 相关会话。

## 与官方 Codex 插件的关系

本插件不是 Codex 的替代品。

它依赖官方 VS Code 扩展：

```text
openai.chatgpt
```

整体结构：

```text
VS Code
│
├─ OpenAI Codex Extension
│      └─ 读取和修改代码
│
└─ DOV
       ├─ 读取 Codex Session
       ├─ 捕获 apply_patch
       ├─ 生成 Diff
       ├─ Approve
       ├─ Reject
       └─ 保存 Review 状态
```

## 快速使用

1. 安装 OpenAI Codex VS Code 扩展。
2. 安装本插件。
3. 用 VS Code 打开项目。
4. 运行 `DOV: Home`。
5. 按提示完成 DOV 项目级配置。
6. 让 Codex 正常修改代码。
7. 修改完成后输入 `+review`。
8. 在 DOV 中逐项审核。

## 当前限制

当前 DOV 主要依据 Codex Session 中记录的 `apply_patch` 捕获代码变化，因此以下情况可能无法完整捕获：

- Shell/PowerShell 直接写文件；
- Python、Node 等脚本生成或覆盖文件；
- 动态构造的 Patch；
- 部分删除文件场景。

因此本 Fork 后续会增加 **真实 Workspace Snapshot**，用实际文件状态作为最终审核依据，而不是只依赖 Codex 日志。

## 本 Fork 的后续方向

本仓库计划重点增加：

### SVN AI Workspace

目标是在 SVN 项目中实现类似 Git Worktree 的 AI 隔离开发体验：

```text
SVN Repository
      │
      ├──────────────┐
      │              │
正式 Working Copy   AI Working Copy
      │              │
      │              └─ Codex 修改
      │                    ↓
      │                Step Review
      │                    ↓
      │              Approve / Reject
      │                    ↓
      │                最终 SVN Diff
      │                    ↓
      └──────────── SVN Commit
```

AI 不直接修改正式工作目录。

### Step Review / Baseline

计划支持：

```text
Step #001
   ↓
Review
   ↓
Approve
   ↓
成为下一步 Baseline

Step #002
   ↓
只审核 #001 → #002 的实际变化
```

### SVN Final Review

正式提交前再进行：

```text
svn status
svn diff
```

形成两层审核：

```text
AI Step Review
+
SVN Final Review
```

### DSE 企业透明加密兼容

优先保持源码读取、写入和 Diff 操作位于 VS Code 环境内，适配已经允许 VS Code / Codex 正常访问源码的企业 DSE 透明加密场景。

不会要求现有 SVN 项目迁移到 Git。

## 安装本 Fork

### 方法一：安装 GitHub Actions 生成的 VSIX

本仓库合并到 `master` 后会自动运行 VSIX 构建流程。

在 GitHub 仓库的 **Actions** 页面找到最新的 **Build VSIX**，下载构建产物，然后在 VS Code 中：

```text
Extensions
→ ...
→ Install from VSIX...
```

或者使用命令行：

```bash
code --install-extension document-oriented-vibing-cn-0.1.0.vsix --force
```

### 方法二：本地构建

```bash
git clone https://github.com/aaa17001/document-oriented-vibing.git
cd document-oriented-vibing

pnpm install
pnpm run package

pnpm dlx @vscode/vsce package --no-dependencies --out document-oriented-vibing-cn-0.1.0.vsix
code --install-extension document-oriented-vibing-cn-0.1.0.vsix --force
```

安装完成后重新加载 VS Code。

## 常用命令

| 命令 | 作用 |
|---|---|
| `DOV: Home` | 打开 DOV 首页 |
| `DOV: +review` | 打开最新 Review |
| `DOV: Capture Codex Review` | 从 Codex Thread 捕获修改 |
| `DOV: Copy Selection for LLM` | 复制带文件路径/行号的上下文 |
| `DOV: Quick New Feature` | 新建 Feature Diagram |
| `DOV: Open Feature` | 打开 Feature Diagram |

## 上游项目

原项目：

- [ethanitovitch/document-oriented-vibing](https://github.com/ethanitovitch/document-oriented-vibing)

感谢原作者 Ethan Itovitch 对 DOV 的开发。

## License

MIT License。

本 Fork 继续保留并遵循原项目的版权声明和 MIT License。
