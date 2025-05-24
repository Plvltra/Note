## 常用工具

- 使用git extensions在windows以界面的方式扩展
- tig工具: git的文本模式界面

## 命令原理

- [cherry-pick的原理](https://waynerv.com/posts/git-cherry-pick-intro/#contents:%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3-cherry-pick)

-   [merge与三路合并算法](https://waynerv.com/posts/git-merge-intro/#%E5%90%88%E5%B9%B6%E5%86%B2%E7%AA%81) 

## 技巧
- clone Unreal引擎: git clone -b 4.27.2-release --depth=1 https://github.com/EpicGames/UnrealEngine.git UE_27_2  --depth=1 不会checkout history, 看不了log但也会会大大节省空间
- [cherry-pick](https://www.ruanyifeng.com/blog/2020/04/git-cherry-pick.html)
- [fast-forward](https://waynerv.com/posts/git-merge-intro/#contents:%E5%BF%AB%E8%BF%9B%E5%90%88%E5%B9%B6): 想要合并的分支是master的直接后继 
- [fast-forward](https://waynerv.com/posts/git-merge-intro/#contents:%E5%BF%AB%E8%BF%9B%E5%90%88%E5%B9%B6): 想要合并的分支是master的直接后继 

### 常用示例
##### 增
``` bash
# 根据远程分支创建本地分支并切换
git checkout -b {本地分支名} origin/{远程分支名}

# 检出 origin/master 分支 
git checkout origin/master 

# 创建并切换到新的本地分支 
git checkout -b {new-branch-name}

# 推送新的本地分支到远程仓库
git push -u origin {本地分支名} 

# 删除远程仓库origin上的分支feature-branch
git push origin --delete {feature-branch}

# 查看当前分支的upstream分支
git rev-parse --abbrev-ref --symbolic-full-name @{u}

# 列出/压栈/弹出本地changes
git stash list/push/pop

# This creates an association between your local copy and repository on git(添加远程仓库并给url起个name别名)
git remote add <name> <url>
git remote add origin https://github.com/Plvltra/myproj

# 创建new分支追踪远程分支
git branch --track new origin/old
# 创建new分支track远程分支
git branch --set-upstream-to origin/old

```
##### 删
``` bash
```
##### 改
``` bash
# 修改全部的commit
git rebase -i --root
```
##### 查
``` bash
# List references in a remote repository
git ls-remote

# 查看diff tool
git config --list | grep diff.tool

# 查看两个提交的diff文件
git diff --name-status HEAD^ HEAD

# 查看某个提交的commit内容
git difftool 4718cd3^ 4718cd3

# 查看某个提交os.c的commit内容
git difftool 4718cd3^ 4718cd3 -- os.c

# 查看当前unstaged内容的diff
git difftool

# 列举远程仓库
git remote
```


### 配置
- 使用vscode作为difftool: https://code.visualstudio.com/docs/sourcecontrol/overview#_vs-code-as-git-editor
- github使用https协议的协议push password需要使用token(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 要配置权限)