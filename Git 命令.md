[TOC]

Gerrit 使用：https://epost.gitbooks.io/gerrit-user-guide/content/

## git 常用命令

### git 文件状态图

![输入图片说明](/imgs/2024-07-22/NlM47Qf9ejeLoUYw.png)

### git 提交代码流程
1. 拉取 tip code（最新代码）
	使用 repo 忽视本地修改，拉取最新代码
2. 修改相应 files 中的 code
3. git add files
4. git commit 进入文本界面根据 commit 规范填写
5. `git push origin HEAD:refs/for/dev_i6dw_bringup(对应分支名)` 提交代码
	`git push origin HEAD:refs/for/dev_i6dc_bringup`
	`git push origin HEAD:refs/for/Master_V5`
	`git push origin HEAD:refs/for/dev_i6cp_bringup`
6. 如果 push 后的代码需要返修，重复第2、3步，第4步中使用 `git commit --amend`在不创建新的提交的情况下，对最后一次提交进行修改。

撤回已 push 到 Gerrit 的 提交
git reset --soft HEAD^

### git blame -L：查看特定行号的最近一次修改历史
**查看特定行号的修改历史**：
```bash
git blame -L 10,20 main.py
```
这将显示 `main.py` 文件中第10行到第20行的修改历史。

### git log -L：查看特定行号的所有修改历史
```bash
git log -L 10,20:main.py
```
注意行号后面是冒号而不是空格


### 如何将代码 git cherry-pick 到其他分支
![输入图片说明](/imgs/2024-07-17/qhu2iUiZNS6NLp8s.png)
1. 点击 cherry pick，选择要合并的分支，没有冲突会直接成功，若有冲突会提示失败，见第2步
2. 复制链接到 shell，加上 -x 命令，将代码拉到本地，然后？

### git commit 自动补充 change-id 的设定
```bash
# 正确
git config --global core.hookspath /tools/.githooks
# 错误
git config --global core.hookspath=/tools/.githooks
# 查看是否设置成功
git config --list
```

### git commit --amend：修改 push 后的 commit 信息
```bash
git commit --amend
```

在 Gerrit 上也可以手动修改

### git add -u：只将所有修改过的（modified）文件添加到暂存区
```bash
git add -u
将所有已经被跟踪的、状态为modified的文件添加到暂存区，但是它不会包括新添加的文件（即untracked文件）

git add -u src/
将src/目录下所有已跟踪且被修改的文件添加到暂存区
```

### git revert commitID：将某一笔 commit 撤销（同时会生成新的 commit）

### git branch -vv：显示所有本地分支的详细信息

### git rebase -i HEAD～N：将最近的 N 笔 commit 合并成一笔

在使用git时经常会遇到多笔commit修改同一个问题的情况，在提交时，往往只希望将这些commit作为一笔commit提交，可以通过commit指令达到这个目的。假设我们希望将最近的N笔commit合并成一笔  

首先执行命令  
`git rebase -i HEAD～N`
例如，我们希望合并最近的3笔commit，则命令为 `git rebase -i HEAD～3`  

接下来，系统会打开一个编辑工具，内容如下  
  
```
pick 16212a5 main change  
pick 1343e22 remove log  
pick 2a21096 remove empty line  
  
# Rebase XXXXXXXXX  
# XXXXXXXXXXXXXXX  
# ......  
```

如果不希望branch的commit记录中有2，3笔commit的记录，我们可以将上面的文本修改成下面的样子，然后保存

```
pick 16212a5 main change  
squash 1343e22 remove log  
squash 2a21096 remove empty line  
  
# Rebase XXXXXXXXX  
# XXXXXXXXXXXXXXX  
# ......  
```

接下来，git会帮我们进行rebase的动作，rebase执行完后，git会启动一个编辑器让我们填写合并后的新的commit的记录，填写完成后保存。  

最后使用git log命令查看一下，可以看到，原先三笔commit中的改动都被合并到一笔新的commit中提交了。

### git tag：给仓库历史中的某一个提交打上标签，以示重要
[Git 基础 - 打标签](https://git-scm.com/book/zh/v2/Git-%E5%9F%BA%E7%A1%80-%E6%89%93%E6%A0%87%E7%AD%BE)
如何列出已有的标签、如何创建和删除新的标签、以及不同类型的标签分别是什么

[GIT中打标签（tag）的意义](https://blog.csdn.net/Jason_Lee155/article/details/115280687)
打tag就类似于我们看书放书签一样，以后可以直接用tag找到提交的位置，不然的话，就只有看commit的哈希值返回指定位置，比较繁琐

## repo 常用命令

### repo 忽视本地修改，强制恢复初始
**beal 给出的方案**
```bash
repo forall -c 'git clean -xfd'  // 强制清除当前工作目录中的所有未跟踪文件和目录，包括那些被 `.gitignore` 文件忽略的文件和目录

repo forall -c 'git reset --hard HEAD~1'

// repo forall -c -j8
repo sync -c -j16

repo forall -c 'git checkout $REPO_RREV'

```
**网上的方案**
```bash
# 将HEAD强制指向manifest的库，而忽略本地的改动。
repo sync -d

# Remove all working directory (and staged) changes.
repo forall -c 'git reset --hard'   

# Clean untracked files
repo forall -c 'git clean -f -d'   

# 拉代码
repo sync -c
```

#### 问题记录
![输入图片说明](/imgs/2024-07-29/AL4elOm9qVHIVisI.png)
```
cd sdk/
git rebase --abort
```

### repo manifest -r -o ./snapshot.xml：生成当前仓库状态的快照

```bash
repo manifest -r -o ./snapshot.xml
cat ./snapshot.xml

mv snapshot.xml .repo/manifests/
repo forall -c "git reset --hard HEAD"
repo forall -c "git clean -df"
repo init -m snapshot.xml
repo sync -c -j8
```


**参数解释**
- **`repo manifest`**:
  - 这个命令用于生成一个描述当前仓库状态的 XML 文件。

- **`-r`**:
  - 这个选项表示在生成的 manifest 文件中记录每个项目的当前修订版本（通常是某个分支或提交的哈希值）。
  - 如果不使用 `-r` 选项，生成的 manifest 文件只会包含项目的名称和 URL，而不包含具体的修订版本。

- **`-o ./snapshot.xml`**:
  - 这个选项用于指定生成的 manifest 文件的输出路径和文件名。
  - `./snapshot.xml` 表示将生成的 XML 文件保存到当前目录下的 `snapshot.xml` 文件中。

### 回到某天之前的最新提交
```
repo forall -c 'commitID=`git log --before="2024-02-20" --pretty="%H" | head -n 1`;git reset --hard $commitID'
```
作用：在每个子项目中找到2024 年 2 月 20 日之前的最近一次提交，并将当前分支重置到该提交

这条 `repo` 命令使用了 `repo forall` 来在所有子项目中执行一个复杂的 Git 命令。具体来说，这条命令会在每个子项目中找到某个日期之前的最近一次提交，并将当前分支重置到该提交。下面是详细的分析：

**复合命令解析**
1. 获取指定日期之前的最近一次提交
```sh
commitID=`git log --before="2024-02-20" --pretty="%H" | head -n 1`
```
- **`git log --before="2024-02-20"`**:
  - 这个命令会列出所有在 2024 年 2 月 20 日之前的提交。
- **`--pretty="%H"`**:
  - 这个选项用于格式化输出，只显示提交的哈希值（commit hash）。
- **`| head -n 1`**:
  - 这个管道命令会取上述输出的第一行，即最近的一次提交的哈希值。
- **`commitID=`**:
  - 将上述命令的结果赋值给变量 `commitID`。

2. 将当前分支重置到指定的提交
```sh
git reset --hard $commitID
```
- **`git reset --hard $commitID`**:
  - 这个命令会将当前分支重置到 `commitID` 指定的提交，并丢弃所有未提交的更改。

### 查找某段时间内的 commit 记录
```
repo forall -c "pwd;git log --oneline --after=2024-10-29"
```
作用：在每个子项目中打印当前目录路径，并显示从 2024 年 10 月 29 日之后的所有提交记录

### repo forall -c 'git config core.filemode false --global'：忽略文件权限的变化
```bash
repo forall -c 'git config core.filemode false --global'
```
**参数解释**
-   **`git config`**: 这是 Git 的配置命令，用于设置 Git 的配置选项。
-   **`core.filemode`**: 这是一个 Git 配置项，控制 Git 是否跟踪文件权限的变化（即文件的可执行位）。
-   **`false`**: 这是要设置给 `core.filemode` 的值。将其设置为 `false` 表示 Git 不会跟踪文件权限的变化。
-   **`--global`**: 这个选项表示该配置更改将应用于全局范围，即对当前用户的所有 Git 仓库生效。

**详细说明**
1.  **`core.filemode` 配置项**:
    -   默认情况下，Git 会跟踪文件的权限变化（例如，文件是否可执行）。这意味着如果你在一个 Unix 系统上修改了文件的可执行权限，Git 会记录这一变化，并在你提交时包含这些权限信息。
    -   设置 `core.filemode` 为 `false` 后，Git 将忽略文件权限的变化，只关注文件内容的变化



<!--stackedit_data:
eyJoaXN0b3J5IjpbMTYyMzAxNjA1N119
-->