# git使用

**用git操纵远程仓库删除文件等**



要从 Git 远程仓库中彻底删除博客（或博客中的特定文件，如照片），您需要执行一系列步骤。请注意，这些步骤将更改您的提交历史，并可能影响其他协作者。以下是详细步骤：

1. **删除本地文件**：
   首先，在本地仓库中删除您想要删除的文件或博客。



```sh
   git rm -r --cached path/to/your/blog
```

这里 `path/to/your/blog` 是您想要删除的文件或目录的路径。

1. **提交更改**：
   将删除操作提交到本地仓库。



```sh
   git commit -m "Remove sensitive blog files"
```

1. **强制推送到远程仓库**：
   将更改强制推送到远程仓库。这会覆盖远程仓库的历史记录。



```sh
   git push origin <branch-name> --force
```

替换 `<branch-name>` 为您当前的分支名，通常是 `main` 或 `master`。

1. **检查远程仓库**：
   确认远程仓库的文件已经被删除。
2. **使用 BFG Repo-Cleaner（可选）**：
   如果文件已经被推送到远程仓库，并且您想要从历史记录中彻底删除它们，可以使用 BFG Repo-Cleaner。首先，下载 BFG Repo-Cleaner：



```sh
   wget https://repo1.maven.org/maven2/com/madgag/bfg/1.14.0/bfg-1.14.0.jar -O bfg.jar
```

然后，在您的 Git 仓库目录中运行 BFG Repo-Cleaner：



```sh
   java -jar bfg.jar --delete-files <file-to-delete>.git
```

替换 `<file-to-delete>` 为您想要删除的文件。

1. **使用 Git 重新写历史（可选）**：
   如果您不想使用 BFG Repo-Cleaner，可以使用 `git filter-branch` 重新写历史记录。这将会删除整个历史记录中的指定文件：



```sh
   git filter-branch --force --index-filter \
   "git rm --cached --ignore-unmatch <file>" --prune-empty --tag-name-filter cat -- --all
```

替换 `<file>` 为您想要从历史记录中删除的文件。

1. **推送更改**：
   使用 BFG Repo-Cleaner 或 `git filter-branch` 后，您需要将新的、更新后的历史记录推送到远程仓库：



```sh
   git push origin <branch-name> --force
```

1. **通知协作者**：
   如果多人正在协作您的仓库，请通知他们您已经重写了项目历史。这将要求他们采取特定的步骤来同步他们的本地仓库。
2. **删除远程标签**：
   如果您删除的文件在远程标签中存在，强制推送可能不会影响它们。使用以下命令删除特定的远程标签：



```sh
   git push origin --delete tag <tag-name>
```

1. **删除旧的远程分支**：
   如果您删除的文件在旧的远程分支中，可能需要删除整个远程分支：



```sh
    git push origin --delete <branch-name>
```

请谨慎执行这些操作，因为它们会更改仓库的历史记录。如果其他人已经基于旧的历史记录进行了工作，他们将需要重新克隆仓库或使用 `git fetch` 和 `git reset` 来同步更改。



1. **强制推送（谨慎使用）**：
   如果您了解强制推送的影响，并且确实需要覆盖远程分支上的更改，可以使用 `--force` 选项。但请务必谨慎，因为这会丢失远程分支上的提交历史。



```sh
   git push origin master --force
```



~~~
git branch //查看本地分支   -r 查看远程分支  -a 查看全部分支
~~~



~~~
  origin/HEAD -> origin/master
  origin/master
~~~

这意味着你的本地仓库跟踪的远程默认分支是 `origin/master`。如果你想要切换到 `master` 分支，可以使用命令 `git checkout master`，或者如果你想要跟踪远程的 `master` 分支，确保你的本地 `master` 分支已经设置好了正确的上游（upstream）配置。





**如何配置上游分支**

配置本地分支的上游（upstream）是Git中的一个重要功能，它可以帮助您明确本地分支与其对应的远程分支之间的关系。这样做的原因包括：

1. **简化操作**：让您可以通过简单的命令（如 `git pull` 或 `git push`）直接与远程分支交互。
2. **避免错误**：确保您的提交被推送到正确的远程分支上，避免数据丢失或推送到错误的分支。
3. **增强协作**：当多人协作时，清楚地知道每个人的工作是基于哪个远程分支，可以减少集成问题。

以下是配置本地分支上游的基本步骤：

1. **查看当前配置**：
   使用命令 `git branch -vv` 查看本地分支及其上游的配置情况。
2. **设置上游分支**：
   为本地分支设置上游分支，使用以下命令：

```sh
   git branch --set-upstream-to=origin/master master
```

这里将本地的 `master` 分支设置为跟踪远程 `origin` 仓库的 `master` 分支。

1. **后续操作**：
   设置完成后，您可以使用 `git push` 直接推送到远程分支，使用 `git pull` 从远程分支拉取更改。

如果您已经克隆了远程仓库，但尚未配置上游分支，执行上述设置上游分支的命令即可。如果您想更改现有的上游分支设置，可以使用相同的命令，只需更改 `origin/master` 为其他远程分支的名称。







**添加你想上传的文件**

1. **添加多个文件**：
   您可以同时添加多个文件，只需在命令后面列出所有文件路径，并用空格分隔。例如：

```sh
   git add path/to/file1.txt path/to/file2.txt path/to/another/directory/
```

实现在本地Git项目中仅上传重要的更改到GitHub的原作者仓库，您可以按照以下具体步骤操作：

1. **创建新分支**：
   在命令行中，使用 `git checkout -b important-changes` 命令创建并切换到一个名为 "important-changes" 的新分支。
2. **选择重要文件**：
   在Git中，您可以使用 `git add` 命令仅添加您认为是重要的文件。例如：

```sh
   git add path/to/important-file1 path/to/important-directory/
```

1. **提交更改**：
   使用 `git commit -m "Commit message describing important changes"` 命令提交您刚才添加的文件。
2. **推送到远程仓库**：
   使用 `git push origin important-changes` 命令将您的新分支推送到GitHub上的远程仓库。
3. **创建Pull Request**：
   在GitHub上，切换到您的“important-changes”分支，然后点击“New pull request”按钮，填写Pull Request的标题和描述，然后提交。
4. **等待原作者审查和合并**：
   提交Pull Request后，原作者将会收到通知，并审查您提交的更改。如果更改被接受，原作者会将您的更改合并到主分支中。

以上步骤可以帮助您只上传您认为重要的更改，同时保持项目的整洁和专注性。





1. **如果您还没有添加远程fork仓库**：
   您可以使用以下命令添加：

```sh
   git remote add origin https://github.com/your-username/repository.git
```

1. **设置上游分支并推送**：
   如果您想设置上游分支并推送您的更改，可以使用以下命令：



```sh
   git push --set-upstream origin water
```

~~~
git remote set-url origin "'https://github.com/xietongMe/ginEssential.git"
~~~

~~~
 git remote set-url origin "https://github.com/Whuichenggong/ginEssential.git"
~~~

# 用ssh连接仓库



要使用SSH代替HTTPS进行Git操作，您需要完成以下步骤：

1. **生成SSH密钥对**（如果您还没有）：
   打开终端并运行以下命令，按提示操作即可生成新的SSH密钥对。

```sh
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

这会生成两个文件：一个公钥（`id_rsa.pub`）和一个私钥（`id_rsa`）。

1. **添加SSH公钥到GitHub账户**：
   - 打开生成的公钥文件（例如`cat ~/.ssh/id_rsa.pub`）。
   - 复制公钥内容。
   - 登录到您的GitHub账户，进入“Settings”（设置），然后点击“SSH and GPG keys”（SSH 和 GPG 密钥）。
   - 点击“New SSH key”（新建SSH密钥），在“Title”（标题）栏输入描述（例如`Personal MacBook`），然后在“Key”（密钥）栏粘贴您的公钥。
2. **测试SSH连接**：

```sh
   ssh -T git@github.com
```

如果连接成功，您会看到一条欢迎消息。

1. **更改远程仓库的URL**：
   将远程仓库的URL从HTTPS更改为SSH格式：

```sh
   git remote set-url origin git@github.com:Whuichenggong/ginEssential.git
```

确保替换`Whuichenggong`和`ginEssential.git`为您的GitHub用户名和仓库名。

1. **推送更改**：
   现在，您可以尝试再次使用`git push`命令推送您的更改：

```sh
   git push -u origin <创建的新分支>
```

如果一切设置正确，您应该能够成功推送到GitHub。

这些步骤将帮助您通过SSH与GitHub通信，这通常比HTTPS更稳定，特别是在某些网络环境下。如果在过程中遇到任何问题，请检查SSH密钥是否正确添加到GitHub账户，以及是否有任何SSH连接错误提示。

## 创建仓库并上传步骤

echo "# -typro-" >> README.md 
git init 
git add README.md 
git commit -m "首次提交" 
git branch -M main 
git remote add origin git@github.com:Whuichenggong/-typro-.git
 git push -u origin main







```bash
 git remote remove origin
   git remote add origin git@github.com:Whuichenggong/-typro-.git
```

1. **检查并推送 `main` 分支**

```bash
   git push origin main
```

1. **检查本地分支**：
   如果您不确定本地有哪些分支，可以使用以下命令查看：



```bash
   git branch -a
```

1. **如果需要推送 `dev` 分支**：
   确保 `dev` 分支存在并且您已经切换到该分支：



```bash
   git checkout dev
   git push origin dev
```

1. **验证 URL**：
   使用 `git remote -v` 检查更新后的远程 URL 是否正确：



```bash
   git remote -v
```

确保您使用的 URL 是正确的，并且没有包含任何不必要的字符，如波浪号 (`~`)。这样可以避免远程仓库识别错误和推送问题。







30413@xiaoxinxiaohao MINGW64 /e/实验室软件管理 (master)
$      git config --global core.autocrlf true

30413@xiaoxinxiaohao MINGW64 /e/实验室软件管理 (master)
$      git config --global core.autocrlf input

30413@xiaoxinxiaohao MINGW64 /e/实验室软件管理 (master)







```bash
git checkout -b main
git push -u origin main
# 或者
git checkout -b master
git push -u origin master
```

```bash
git checkout main
# 或者
git checkout master
```

```bash
git push -u origin <branch_name>:<branch_name>
```



**安装大文件**

Git LFS（Large File Storage）是一种用于管理大文件的工具，主要与 Git 版本控制系统配合使用。

**一、作用**

当在使用 Git 进行版本控制时，如果项目中包含一些大型文件，如视频、音频、大型数据集等，会带来以下问题：

1. 存储和传输效率低下：Git 仓库会变得非常大，克隆和推送操作会变得很慢，占用大量的存储空间和网络带宽。
2. 性能问题：会影响 Git 的整体性能，特别是在处理历史记录和分支合并时。

Git LFS 通过将大文件存储在单独的服务器上，并在 Git 仓库中只保留对这些大文件的指针，从而解决了这些问题。这样可以大大减小 Git 仓库的大小，提高存储和传输效率，同时不影响 Git 的正常使用。

**二、工作原理**

1. 安装和配置 Git LFS 后，当你添加一个大文件到 Git 仓库时，Git LFS 会自动识别该文件，并将其上传到指定的 LFS 服务器。
2. 在 Git 仓库中，只会存储该文件的指针信息，而不是实际的文件内容。
3. 当其他人克隆这个仓库时，Git LFS 会自动从 LFS 服务器下载大文件，确保他们能够获取完整的项目。

**三、使用步骤**

1. 安装：
   - 在命令行中运行 `git lfs install` 来安装 Git LFS。
2. 追踪大文件类型：
   - 使用 `git lfs track "<file pattern>"` 命令来告诉 Git LFS 哪些文件类型需要被追踪。例如，`git lfs track "*.mp4"` 会追踪所有的 MP4 文件。
3. 添加和提交：
   - 像平常一样使用 `git add` 和 `git commit` 命令来添加和提交文件。Git LFS 会自动处理被追踪的大文件。
4. 推送和拉取：
   - 使用 `git push` 和 `git pull` 命令来推送和拉取仓库。Git LFS 会自动处理大文件的上传和下载。

**四、适用场景**

1. 软件开发项目中包含大型二进制文件，如游戏开发中的资源文件、软件开发中的安装包等。
2. 数据科学项目中包含大型数据集，如图像数据集、文本数据集等。
3. 多媒体项目中包含视频、音频文件等。



```bash
git add <文件名>
```

或者添加所有修改过的文件：

bash



Copy

```bash
   git add .
```

1. **提交更改**：
   使用 `git commit` 命令将暂存区的更改提交到本地仓库：

bash



Copy

```bash
   git commit -m "您的提交信息"
```

如果您只是快速提交，可以使用 `-a` 参数自动暂存所有已修改的文件，并提交：

bash



Copy

```bash
   git commit -a -m "您的提交信息"
```

1. **推送到远程仓库**：
   使用 `git push` 命令将提交的更改推送到远程的 `main` 分支：

bash



Copy

```bash
   git push origin main
```

如果远程仓库有新的提交，您可能需要先拉取这些更改并合并到您的本地仓库中，然后再推送：

bash



Copy

```bash
   git pull origin main
   git push origin main
```

