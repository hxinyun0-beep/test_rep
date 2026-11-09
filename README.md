# test_rep

用于测试 Git 配置、熟悉版本控制流程的练习仓库。你可以在这里练习提交、分支和远程同步，逐步掌握 Git 的基本用法。

## 项目目的

- **验证环境**：检查 Git 是否安装成功，以及提交者姓名、邮箱是否配置正确。
- **学习版本控制**：理解工作区、暂存区和提交记录的关系，学会查看和保存改动。
- **练习团队协作流程**：创建分支、合并修改，并使用远程仓库同步代码。
- **积累操作经验**：在测试项目中熟悉常见问题的排查方法。

本项目以学习和测试为主，不包含需要运行的业务程序。请使用演示内容练习，不要提交密码、访问令牌、私钥或其他敏感信息。

## Git 新手教程

### 1. 准备环境与本地仓库

在终端中确认 Git 已安装：

```bash
git --version
```

如果提示找不到命令，请先安装 Git，再重新打开终端。

如果你已经打开本仓库的本地目录，可以直接进入第 2 步。否则，根据情况选择下面一种方式。

**方式 A：下载已有的远程仓库**

将下面的示例地址替换为实际仓库地址，再执行：

```bash
git clone [https://github.com/YOUR_NAME/test_rep.git](https://github.com/hxinyun0-beep/test_rep.git)
cd test_rep
```

**方式 B：从零创建本地练习仓库**

```bash
mkdir test_rep
cd test_rep
git init
```

后续命令都在本地仓库目录中执行。

### 2. 配置提交者信息

将示例姓名和邮箱替换为你希望出现在提交记录中的信息：

```bash
git config --local user.name "Your Name"
git config --local user.email "you@example.com"
```

`--local` 表示仅对当前仓库生效，适合测试配置。如果希望为所有本地仓库设置默认身份，可以改用 `--global`；仓库中的本地配置会覆盖全局配置。

检查实际生效的配置及其来源：

```bash
git config --show-origin --get user.name
git config --show-origin --get user.email
```

姓名和邮箱用于标识提交者，并不等于远程平台的登录凭据。

### 3. 理解一次提交的流程

```text
编辑文件 → 工作区 → git add → 暂存区 → git commit → 本地提交记录
```

- **工作区**：你正在编辑的文件。
- **暂存区**：准备放入下一次提交的改动。
- **提交记录**：Git 保存的项目快照，可以用于查看历史和比较变化。

`git commit` 只保存到本地；执行 `git push` 才会上传到远程仓库。

### 4. 完成第一次练习提交

使用编辑器在仓库根目录新建 `practice.md`，写入以下内容并保存：

```markdown
# Git 练习记录

- 已完成 Git 安装检查。
- 已配置提交者姓名和邮箱。
```

检查文件状态，将练习文件加入暂存区，然后检查待提交内容：

```bash
git status
git add practice.md
git diff --cached
```

确认内容正确后提交：

```bash
git commit -m "docs: 添加 Git 练习记录"
git log --oneline -5
git status
```

如果还修改了 README，可以另行执行 `git add README.md`，将它加入下一次提交。Git 只提交已经暂存的改动。

接着在 `practice.md` 中增加一行练习心得，保存后运行：

```bash
git diff
git add practice.md
git diff --cached
git commit -m "docs: 更新 Git 练习心得"
```

`git diff` 查看尚未暂存的已跟踪文件改动，`git diff --cached` 查看已经暂存的改动。新建且尚未跟踪的文件不会出现在普通的 `git diff` 中，可以先通过 `git status` 查看。

### 5. 练习分支与合并

先确认当前分支名称，并记下输出，合并时需要切回它：

```bash
git branch --show-current
git status
```

确认待保存的改动已提交后，创建练习分支：

```bash
git switch -c practice/readme
```

在 `practice.md` 中增加一条“已完成分支练习”，保存并提交：

```bash
git add practice.md
git commit -m "docs: 记录分支练习"
```

假设之前记下的原分支名为 `main`，执行以下命令合并；如果实际名称为 `master` 或其他名称，请相应替换第一条命令中的 `main`：

```bash
git switch main
git merge practice/readme
git log --oneline --graph --all -10
```

合并成功后，可以删除已经合并的练习分支：

```bash
git branch -d practice/readme
```

如果出现合并冲突，使用编辑器处理文件中的 `<<<<<<<`、`=======`、`>>>>>>>` 标记，保留正确内容并移除标记，然后对解决后的文件执行 `git add 文件名` 和 `git commit`。如果暂时不想继续本次合并，可以在合并进行中执行 `git merge --abort`。

### 6. 连接并推送到远程仓库（可选）

先查看是否已经配置远程地址：

```bash
git remote -v
```

如果通过 `git clone` 下载仓库，通常已经存在名为 `origin` 的远程配置，可以跳过添加地址。

如果从本地新建仓库，请先在 GitHub、GitLab 等平台创建一个空仓库，不要自动添加 README、许可证或 `.gitignore`。然后用实际地址替换示例地址：

```bash
git remote add origin https://github.com/YOUR_NAME/test_rep.git
```

完成至少一次本地提交后，将当前分支推送到远程，并建立跟踪关系：

```bash
git push -u origin HEAD
```

这里的 `HEAD` 代表当前分支，不要求分支必须叫 `main`。推送需要远程仓库的写入权限，以及平台支持的身份认证方式，例如 HTTPS 访问令牌或 SSH 密钥。

建立跟踪关系后，日常同步可以使用：

```bash
git pull --ff-only
git push
```

建议在开始编辑前拉取远程更新，并保持工作区干净。`git pull --ff-only` 只允许快进更新；如果本地和远程各自出现了新提交，它会停止，留给你检查和处理分叉历史的机会。

## 常用命令速查

| 命令 | 用途 |
| --- | --- |
| `git status` | 查看当前分支和文件状态 |
| `git diff` | 查看已跟踪文件中尚未暂存的改动 |
| `git diff --cached` | 查看暂存区中的改动 |
| `git add 文件名` | 将指定文件的改动加入暂存区 |
| `git commit -m "说明"` | 保存暂存区内容为一次提交 |
| `git log --oneline -5` | 查看最近五条提交 |
| `git branch` | 查看本地分支 |
| `git switch 分支名` | 切换到已有分支 |
| `git remote -v` | 查看远程仓库地址 |
| `git restore --staged 文件名` | 取消暂存，保留工作区内容；需要已有提交 |

## 常见问题

**提交时提示 `Author identity unknown`**

Git 没有找到有效的提交者信息。按照第 2 步设置姓名和邮箱后重新提交。

**提交时提示 `nothing to commit`**

先执行 `git status`。可能文件没有保存、没有实际变化，或者改动尚未通过 `git add` 加入暂存区。

**添加远程地址时提示 `remote origin already exists`**

先用 `git remote -v` 检查已有地址。如果确实需要改为另一个仓库，使用 `git remote set-url origin 实际仓库地址`，不要重复添加。

**推送时提示认证失败或没有权限**

检查远程地址、登录账号、仓库写入权限，以及访问令牌或 SSH 密钥配置。修改 `user.name` 和 `user.email` 不能解决登录认证问题。

**推送被拒绝，提示 `non-fast-forward`**

远程分支存在本地尚未包含的历史。先执行 `git fetch origin`，再使用 `git log --oneline --graph --all -15` 检查历史。若仅落后于远程且已建立跟踪关系，可以执行 `git pull --ff-only`；若历史已分叉，需要先合并并解决可能的冲突。不要为了跳过错误直接强制推送。

## 学习完成检查

- [ ] 能检查 Git 版本和提交者配置。
- [ ] 能解释工作区、暂存区和提交记录的区别。
- [ ] 能查看改动并完成一次提交。
- [ ] 能创建分支、提交修改并合并回原分支。
- [ ] 有远程仓库时，能完成一次推送和拉取。
