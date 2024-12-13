# github使用

在GitHub上为一个项目提交修改，通常需要遵循以下步骤：

1. **Fork仓库**：
   在GitHub上找到该项目的仓库，点击页面右上角的"Fork"按钮，将项目仓库复制到您的GitHub账户下。
2. **克隆仓库**：
   使用 `git clone` 命令将Fork后的仓库克隆到您的本地计算机上。

```sh
   git clone https://github.com/your-username/project-repo.git
```

1. **创建分支**：
   在本地仓库中，进入项目目录，然后创建一个新的分支用于修改。

```sh
   cd project-repo
   git checkout -b my-fix-branch
```

1. **修改代码**：
   在您创建的分支上进行代码修改或添加新功能。
2. **测试代码**：
   确保您的修改不会破坏现有的功能，并符合项目的代码规范。
3. **提交更改**：
   将您的更改提交到本地仓库。

```sh
   git add .
   git commit -m "Fixed issue with XYZ, added feature ABC"
```

1. **推送更改**：
   将本地分支的更改推送到您在GitHub上的Fork仓库。

```sh
   git push origin my-fix-branch
```

1. **创建Pull Request (PR)**：
   在GitHub上，转到您的Fork仓库页面，会看到一个提示您创建Pull Request的选项。点击它，然后按照提示填写相关信息，包括选择您的 `my-fix-branch` 分支作为源分支，选择原项目的相应分支（如 `main` 或 `master`）作为目标分支。
2. **等待审查**：
   提交Pull Request后，项目的维护者会审查您的修改。他们可能会提出问题、要求更改或直接合并您的提交。
3. **讨论和修改**：
   根据审查者的反馈，您可能需要进一步讨论或修改代码。可以在GitHub的PR界面上进行讨论，并根据需要更新您的PR。
4. **合并**：
   一旦得到批准，维护者会合并您的更改到项目的主要分支中。如果您是项目的贡献者，有时也可以自己合并。
5. **保持您的Fork同步**：
   如果您计划继续为该项目贡献代码，记得定期将上游仓库（原项目仓库）的更改拉取到您的Fork中，以保持同步。

提交修改是一个协作的过程，需要与项目维护者和社区成员进行沟通。确保遵循项目的文档和贡献指南，这通常可以在项目的 `CONTRIBUTING.md` 文件中找到。



执行 `git remote -v` 命令会显示当前 Git 仓库的远程仓库信息。这个命令列出了所有远程仓库的名称和对应的 URL，以及它们分别用于 `fetch`（拉取）和 `push`（推送）操作



```plaintext
origin  git@github.com:Whuichenggong/7days-golang.git (fetch)
origin  git@github.com:Whuichenggong/7days-golang.git (push)
```



这段输出告诉你，当前本地 Git 仓库配置了名为 `origin` 的远程仓库，其地址是 `git@github.com:Whuichenggong/7days-golang.git`，你可以从这个地址拉取代码，也可以向这个地址推送代码。这是 Git 与远程仓库交互的标准方式。







要在你所 fork 的仓库上做修改并提交到原始仓库，你可以按照以下步骤操作：

### 1. Fork 仓库

首先，确保你已经在 GitHub 上 fork 了目标仓库。这会在你的 GitHub 账号下创建该仓库的一个副本。

### 2. 克隆你的 Fork

在你的本地计算机上克隆你的 fork 版本。打开终端并运行

```bash
git clone https://github.com/your-username/repository-name.git
```

将`your-username`和`repository-name`替换成你的 GitHub 用户名和仓库名。

### 3. 创建一个新的分支

为了保持工作的组织性，最好在一个新的分支上进行修改：

```bash
cd repository-name
git checkout -b your-branch-name
```

将`your-branch-name`替换成你想要的分支名。

### 4. 进行修改

在本地编辑、添加或删除文件，完成你需要的修改。

### 5. 提交更改

提交你的更改到你的本地仓库：

```bash
git add .
git commit -m "Add your commit message"
```

确保你的提交消息清晰、具体，描述了你所做的更改。

### 6. 推送到 GitHub

将更改推送到你的 GitHub fork：

```bash
git push origin your-branch-name
```

### 7. 创建 Pull Request

- 打开你的 GitHub 仓库页面。
- 你通常会看到一个 “Compare & pull request” 按钮。点击这个按钮。
- 确保 pull request 是从你的新分支（在你的 fork 中）到原始仓库的主分支。
- 填写 pull request 的标题和描述，清楚地说明你的更改和动机。
- 提交 pull request。

### 8. 等待审核

仓库的维护者将会查看你的 pull request，并可能请求进一步的修改或直接合并它。这可能涉及一些讨论，所以保持通知开启以便跟进。

### 9. 同步 Fork

在一段时间后，原始仓库可能会有新的更改，你的 fork 需要同步这些更改。你可以通过设置上游（原始仓库）并拉取它的更改来做到这一点：

```bash
git remote add upstream https://github.com/original-owner/repository-name.git
git fetch upstream
git checkout main
git merge upstream/main
```

替换`original-owner`和`repository-name`为原始仓库的所有者和仓库名。这样你的 fork 就保持了更新。

通过这些步骤，你可以在 fork 的仓库上做出修改并成功地向原始仓库提交 pull request。