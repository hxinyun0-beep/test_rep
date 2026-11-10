# Git 与 VS Code 新手教程

本文根据本次 Git 相关对话整理，适用于 Windows、PowerShell 和 VS Code，覆盖仓库配置、SSH 克隆、文件跟踪与同步、分支合并及 Git Graph 操作。

示例测试仓库：<https://github.com/hxinyun0-beep/test_rep>

分支合并示例：将 `develop` 的更新合并到自己的 `feature/hxy` 分支。使用这些分支示例前，请确认实际项目中存在相应分支；测试仓库未必已经创建它们。

## 目录

1. [项目目的与基本概念](#1-项目目的与基本概念)
2. [检查环境与配置提交者](#2-检查环境与配置提交者)
3. [获取仓库与配置远程地址](#3-获取仓库与配置远程地址)
4. [使用 SSH 克隆和同步](#4-使用-ssh-克隆和同步)
5. [让 VS Code 显示未跟踪文件](#5-让-vs-code-显示未跟踪文件)
6. [在 VS Code 中添加文件并同步](#6-在-vs-code-中添加文件并同步)
7. [将 develop 合并到 feature/hxy](#7-将-develop-合并到-featurehxy)
8. [理解 Git Graph 的 Fetch、Merge 和 Pull](#8-理解-git-graph-的-fetchmerge-和-pull)
9. [处理合并冲突](#9-处理合并冲突)
10. [常见问题与日常操作清单](#10-常见问题与日常操作清单)
11. [参考资料](#11-参考资料)

## 1. 项目目的与基本概念

`test_rep` 是用于测试 Git 配置和练习版本控制的仓库，主要用途是：

- 验证 Git 安装、提交者姓名和邮箱配置。
- 练习查看修改、暂存文件和创建提交。
- 学习分支创建、切换与合并。
- 测试 HTTPS 或 SSH 远程连接，以及拉取和推送。

Git 的基本流程是：

```text
编辑并保存文件 → 工作区 → 暂存（Stage）→ 提交（Commit）→ 推送（Push）
                                        本地历史          远程仓库
```

| 概念 | 含义 |
| --- | --- |
| 工作区 | 当前在磁盘上编辑的项目文件 |
| 暂存区 | 为下一次提交选定的文件内容 |
| 提交（Commit） | 保存到本地 Git 历史中的一次项目快照 |
| 分支（Branch） | 指向一条开发历史最新提交的名称，例如 `develop` |
| 远程仓库（Remote） | 用于共享提交的仓库，常见名称是 `origin` |
| `develop` | 本地 develop 分支 |
| `origin/develop` | 本地记录的 origin 远程 develop 分支状态，通过 Fetch 更新 |
| 未跟踪文件（Untracked） | 尚未加入 Git 跟踪的新文件 |
| 被忽略的文件（Ignored） | 匹配 `.gitignore` 等忽略规则、通常不出现在待提交列表中的文件 |

**保存文件不等于提交，提交也不等于上传。** Git 同步的是提交记录；文件后续发生修改，仍需再次暂存、提交和推送。

## 2. 检查环境与配置提交者

在 PowerShell 中检查 Git 是否可用：

```powershell
git --version
```

若提示找不到命令，请安装 Git 并重新打开终端或 VS Code。

进入本地仓库目录后，将示例身份替换为自己的信息：

```powershell
git config --local user.name "Your Name"
git config --local user.email "you@example.com"
```

- `--local`：仅对当前仓库生效，适合测试。
- `--global`：设置当前用户所有仓库的默认身份。
- 同一个配置项中，仓库本地配置会覆盖全局配置。

检查生效值及其来源：

```powershell
git config --show-origin --get user.name
git config --show-origin --get user.email
```

姓名和邮箱用于标记提交者，不能代替 GitHub 登录、访问令牌或 SSH 密钥。

## 3. 获取仓库与配置远程地址

### 3.1 克隆已有仓库

在准备存放项目的父文件夹中执行：

```powershell
git clone https://github.com/hxinyun0-beep/test_rep.git
cd test_rep
```

克隆会下载已有历史，并通常自动创建名为 `origin` 的远程配置。已经有本地仓库时，不必为了添加文件或更换连接方式重新克隆。

### 3.2 从零创建本地仓库

仅在需要新建一个独立练习仓库时执行：

```powershell
mkdir test_rep
cd test_rep
git init
```

若远程仓库已经有提交，优先克隆远程仓库，再添加自己的文件，以保留共同历史。

### 3.3 检查或添加远程地址

```powershell
git remote -v
```

如果尚未配置 `origin`：

```powershell
git remote add origin https://github.com/hxinyun0-beep/test_rep.git
```

如果 `origin` 已存在且需要更改地址：

```powershell
git remote set-url origin https://github.com/hxinyun0-beep/test_rep.git
```

添加远程地址不会上传文件，也不会自动合并远程历史。推送需要对该远程仓库拥有写入权限。

## 4. 使用 SSH 克隆和同步

### 4.1 生成 SSH 密钥

已有可用 SSH 密钥时可以复用，不要覆盖原密钥。需要新建时，在 PowerShell 中执行：

```powershell
ssh-keygen -t ed25519 -C "你的邮箱"
```

根据提示选择保存位置并设置密钥口令。采用默认路径时：

| 文件 | 用途 |
| --- | --- |
| `$HOME\.ssh\id_ed25519` | 私钥，留在本机 |
| `$HOME\.ssh\id_ed25519.pub` | 公钥，添加到 GitHub |

### 4.2 将公钥添加到 GitHub

复制默认路径下的公钥：

```powershell
Get-Content "$HOME\.ssh\id_ed25519.pub" | Set-Clipboard
```

在 GitHub 打开 **Settings → SSH and GPG keys → New SSH key**，选择认证用途 **Authentication Key**，粘贴公钥并保存。若生成密钥时指定了其他路径，复制命令也要相应修改。

只上传 `.pub` 公钥，不要上传私钥。

### 4.3 测试连接

```powershell
ssh -T git@github.com
```

首次连接可能要求确认主机指纹，应与 GitHub 官方公布的 SSH 指纹核对后再接受。出现 `successfully authenticated` 表示认证成功；同时提示不提供 shell 访问是正常现象。

### 4.4 通过 SSH 克隆

```powershell
git clone git@github.com:hxinyun0-beep/test_rep.git
cd test_rep
```

### 4.5 将现有仓库从 HTTPS 改为 SSH

在已有本地仓库目录中执行：

```powershell
git remote set-url origin git@github.com:hxinyun0-beep/test_rep.git
git remote -v
```

此操作仅更改连接地址，不会重新下载仓库或修改提交记录。后续对 `origin` 的拉取、推送将使用 SSH。

## 5. 让 VS Code 显示未跟踪文件

先区分文件是没有出现在“源代码管理”面板，还是没有出现在“资源管理器”文件树中。

### 5.1 源代码管理面板中不显示

1. 按 `Ctrl + ,` 打开设置。
2. 搜索 `git.untrackedChanges`。
3. 将 **Git: Untracked Changes** 改为 `mixed` 或 `separate`。

| 值 | 显示效果 |
| --- | --- |
| `mixed` | 未跟踪文件与其他更改一起显示 |
| `separate` | 未跟踪文件显示在单独分组中 |
| `hidden` | 隐藏未跟踪文件 |

也可以在现有 `settings.json` 对象中加入或修改这一项，不要覆盖其他设置：

```json
"git.untrackedChanges": "mixed"
```

### 5.2 资源管理器文件树中不显示

- 搜索 `explorer.excludeGitIgnore`，取消勾选，以显示因 Git 忽略规则而被隐藏的文件。
- 搜索 `files.exclude`，检查是否有匹配目标文件的隐藏规则。
- 同时检查“用户”和“工作区”设置，因为工作区配置可能覆盖用户配置。

### 5.3 被忽略的文件仍不会进入更改列表

未跟踪与被忽略不是同一种状态。更改显示设置不会取消 `.gitignore` 规则。

如需确认某个文件为何被忽略，在仓库终端执行，将路径替换为实际文件：

```powershell
git check-ignore -v "路径/文件名"
```

确认该文件应该受版本控制后，再调整对应规则。忽略规则也可能来自 `.git/info/exclude` 或全局配置。普通目录若完全为空，Git 不会单独跟踪它。

## 6. 在 VS Code 中添加文件并同步

以添加 `README.md` 为例：

1. **打开仓库目录**，确保文件位于仓库内，保存编辑内容。
2. 点击左侧分叉形状的 **源代码管理** 图标，或按 `Ctrl + Shift + G`。
3. 在“更改”列表找到文件，点击文件右侧 **`+`**，将它加入暂存区。
4. 检查“暂存的更改”，在输入框填写说明，例如 `docs: 完善 README 新手教程`。
5. 点击 **提交（Commit）**，将暂存内容记录到本地历史。
6. 点击源代码管理面板的 **`…` → 推送（Push）**，上传提交。

若分支没有关联远程分支，VS Code 可能提示 **发布分支（Publish Branch）**，按提示发布到正确的远程仓库。

文件状态常见标记：

| 标记 | 含义 |
| --- | --- |
| `U` | Untracked，未跟踪 |
| `M` | Modified，已修改 |
| `A` | Added，已暂存的新文件 |
| `D` | Deleted，已删除 |

**同步更改（Sync Changes）** 会先拉取再推送，可能需要处理远程更新；**推送（Push）** 只上传本地提交。

对应的终端操作如下，在仓库已配置正确远程地址的前提下执行：

```powershell
git status
git add README.md
git diff --cached
git commit -m "docs: 完善 README 新手教程"
git push -u origin HEAD
```

这里的 `HEAD` 表示当前分支。首次建立跟踪关系后，后续通常可以直接执行 `git push`。

暂存保存的是当时的文件内容。如果暂存后又编辑了文件，需要再次暂存，才能让新修改进入这次提交。

## 7. 将 develop 合并到 feature/hxy

目标方向为：

```text
来源：develop 的更新 → 接收方：feature/hxy
```

**执行合并时，当前分支必须是 `feature/hxy`。** 合并会更新当前分支，不会把自己的功能代码反向合并到远程 develop。

### 7.1 合并前的准备

- 保存当前文件。
- 检查未提交改动，提交应该保留的内容，或使用 Stash 临时存储。
- 普通 Stash 可能不包含未跟踪文件；若需要存储它们，应选择包含未跟踪文件的选项。
- 若出现大量未提交文件，先检查是否混入生成文件、缓存或依赖目录，不要直接全部提交。

### 7.2 使用 VS Code 自带菜单

1. 点击 **左下角分支名**，选择 `feature/hxy`。
2. 打开 **源代码管理**，点击 **`…` → 提取（Fetch）**，获取远程最新分支信息。
3. 点击 **`…` → 分支（Branch）→ 合并分支（Merge Branch…）**。
4. 选择 `origin/develop`，合并刚获取的远程 develop。
5. 如有冲突，按下一节的方法解决并完成提交。
6. 检查项目内容与运行结果，再使用 **推送（Push）** 上传 `feature/hxy`。

菜单文字可能因界面语言和版本略有不同。如果菜单不易找到，可以按 `Ctrl + Shift + P`，执行 `Git: Merge...`。

选择来源时注意：

- `origin/develop`：最近一次 Fetch 获取的远程 develop 状态。
- `develop`：本地 develop，可能落后于远程，也可能包含未推送提交。

### 7.3 使用 Git Graph 图形界面

Git Graph 是扩展提供的视图，与 VS Code 自带的源代码管理视图入口不同。

1. 确认当前分支为 `feature/hxy`。
2. 点击工具栏中提示为 **Fetch From All Remotes** 的按钮。
3. 找到并右键 **`origin/develop` 分支标签**。
4. 选择 **Merge into current branch…**，检查弹窗中的来源和目标后执行。
5. 解决可能的冲突，确认结果，再推送自己的分支。

不要只根据某条提交的信息判断分支，应确认右键的是需要合并的分支标签。

### 7.4 对应终端操作

工作区已整理好，且本地存在 `feature/hxy` 时：

```powershell
git switch feature/hxy
git fetch origin
git merge origin/develop
```

合并完成并检查结果后：

```powershell
git push -u origin feature/hxy
```

如果提示 `Already up to date`，说明所选来源的提交已包含在当前分支历史中，无需重复合并。

## 8. 理解 Git Graph 的 Fetch、Merge 和 Pull

假设当前分支是 `feature/hxy`，右键操作的远程分支为 `origin/develop`：

| 操作 | 含义 | 对当前 feature/hxy 的影响 |
| --- | --- | --- |
| Fetch From All Remotes | 获取所有已配置远程仓库的最新信息，更新远程跟踪引用 | 不自动合并，不修改当前工作文件 |
| Fetch into local branch… | 获取选中远程分支的提交，并更新本地同名分支 | 更新本地 develop，当前分支保持不变 |
| Merge into current branch… | 将选中分支当前已获取的提交合并到当前分支 | 将 develop 内容合并进 feature/hxy |
| Pull into current branch… | 获取所选远程分支最新提交，再整合进当前分支 | 会更新 feature/hxy；整合方式可能受设置影响 |

### 8.1 Fetch From All Remotes

这是从所有远程仓库获取更新。如果只有一个 `origin`，它就会从该远程获取。它不是合并按钮，执行后还需要单独选择 Merge 才会把 develop 整合到功能分支。

### 8.2 Fetch into local branch

这个菜单项用于直接更新本地同名分支，通常要求该本地分支存在且没有被检出。例如当前在 `feature/hxy`，可以获取 `origin/develop` 的提交来更新本地 `develop`，无需先切换过去。

正常情况下，这种更新需要能够快进；如果本地分支有独有提交而形成分叉，操作可能被拒绝。不要随意启用 **Force Fetch**：它可以强制重置目标本地分支指向，使本地独有提交不再属于该分支。

对于“把 develop 合并到自己的 feature/hxy”这一需求，使用 **Fetch → Merge into current branch…** 即可，不需要先执行 Fetch into local branch。

## 9. 处理合并冲突

当两个分支修改了同一区域且 Git 无法自动整合时，会出现冲突。

在本教程的普通合并场景中：

| 编辑器名称 | 对应内容 |
| --- | --- |
| Current / 当前 | 当前 `feature/hxy` 的版本 |
| Incoming / 传入 | 被合并的 develop 版本 |
| Result / 结果 | 最终要保存的内容 |

处理步骤：

1. 在源代码管理的“合并更改”中打开冲突文件。
2. 对照两边内容，选择需要的部分，必要时手动编辑结果。
3. 不要一律选择“接受全部当前”或“接受全部传入”；按实际需求检查每处冲突。
4. 保存文件，确认没有残留的 `<<<<<<<`、`=======`、`>>>>>>>` 冲突标记。
5. 使用合并编辑器的 **Complete Merge / 完成合并** 或手动暂存解决后的文件。
6. 所有文件都解决后，在源代码管理中提交，完成整个仓库的合并。
7. 检查程序或文档结果，再推送。

合并编辑器中的“完成合并”通常只完成并暂存当前文件，不代表整个仓库的合并操作已经结束。

如果本次合并尚未完成提交，想放弃这次合并，可以在终端执行：

```powershell
git merge --abort
```

这也是合并前整理工作区的原因：已有未提交改动会增加恢复和辨别内容的难度。

## 10. 常见问题与日常操作清单

### 10.1 常见问题

| 现象 | 检查方法 |
| --- | --- |
| `Author identity unknown` | 配置 `user.name` 和 `user.email` |
| `nothing to commit` | 确认文件已保存、有实际改动，并已暂存 |
| `remote origin already exists` | 先用 `git remote -v` 检查；需改地址时用 `set-url` |
| SSH 出现 `Permission denied (publickey)` | 检查公钥是否添加到正确账号，以及 SSH 是否使用了对应私钥 |
| 认证成功但不能推送 | 检查账号对目标仓库是否有写入权限 |
| 推送出现 `non-fast-forward` | 先 Fetch 并检查历史，整合远程更新后再推送；不要直接强制推送 |
| 文件未出现在更改列表 | 检查是否位于仓库内、是否被隐藏或忽略、是否实际发生变化 |
| 左下角分支名带 `*` | 通常表示存在未提交改动，到源代码管理中确认 |
| `Uncommitted Changes` 数量异常大 | 先检查文件来源和忽略规则，不要直接全部暂存提交 |

### 10.2 常用命令速查

```powershell
# 当前分支和文件状态
git status
git branch --show-current

# 查看未暂存与已暂存的改动
git diff
git diff --cached

# 查看历史和远程配置
git log --oneline --graph --all -15
git remote -v

# 获取远程最新信息
git fetch origin
```

`git diff` 默认不会显示尚未跟踪的新文件内容；先使用 `git status` 找到它们，暂存后可通过 `git diff --cached` 检查。

### 10.3 每次提交前

- [ ] 文件已经保存。
- [ ] 暂存区只包含这次需要提交的内容。
- [ ] 没有混入密码、访问令牌、私钥或无关生成文件。
- [ ] 提交说明能解释这次改动。

### 10.4 每次合并 develop 前后

- [ ] 当前分支确实是 `feature/hxy`。
- [ ] 未提交改动已妥善处理。
- [ ] 已 Fetch，选中的来源是需要合并的 `origin/develop` 或本地 `develop`。
- [ ] 冲突已逐项解决，必要的合并提交已完成。
- [ ] 已检查最终内容，并推送到自己的功能分支。

## 11. 参考资料

- [VS Code：暂存与提交](https://code.visualstudio.com/docs/sourcecontrol/staging-commits)
- [VS Code：仓库与远程同步](https://code.visualstudio.com/docs/sourcecontrol/repos-remotes)
- [VS Code：分支与工作树](https://code.visualstudio.com/docs/sourcecontrol/branches-worktrees)
- [VS Code：图形界面版本控制操作](https://code.visualstudio.com/docs/introvideos/versioncontrol)
- [VS Code：处理合并冲突](https://code.visualstudio.com/docs/sourcecontrol/merge-conflicts)
- [VS Code：资源管理器与隐藏文件设置](https://code.visualstudio.com/docs/editing/getting-started/userinterface)
- [VS Code：Git 问题排查](https://code.visualstudio.com/docs/sourcecontrol/troubleshooting)
- [Git Graph：右键菜单说明](https://github.com/mhutchie/vscode-git-graph/wiki/Context-Menus)
- [Git Graph：版本说明与 Force Fetch](https://github.com/mhutchie/vscode-git-graph/releases)
