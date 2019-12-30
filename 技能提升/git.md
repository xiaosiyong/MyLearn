## Git 操作

### Config 配置用户名和邮箱

1. git congfig --local/global user.name 'DomXiao'
2. git congfig --local/global user.email 'xiaosywy@163.com'

config的作用域![gitconfig](../images/gitconfig.png)

### 两种方式新建git项目

![init](../images/initGit.png)

### 命令

git 有缓冲区的概念，先添加到缓冲区，再提交![gitmanagement](../images/gitmanagement.png)

1. git mv 命令 给文件重命名  如：git mv readme readme.md，git rm readme git直接删除文件
2. git add -u 对已经添加到缓冲区的文件的更改
3. git log --oneline命令（一行显示commit信息，后边也可以加分支信息 ）  git log -n4 (查看最近4次的修改记录) 
4. git branch -v 查看本地分支版本
5. Git checkout -b 分支名 commitId   基于commit创建新的分支
6. git log --graph  图形化的方式查看log  
7. git cat-file -t xxxxx    Git 提供的接口查看文件类型（commit，blob，tree）  查看具体内容 -p   三种文件之间的关系![gitfilerelation](../images/gitfilerelation.png)
8. 当出现HEAD detached 时一定要注意关联到某分支，不然容易出现丢失
9. git checkout -b newBranchName commitId或者分支名  基于commit或者branch创建分支
10. Head 指针以及其父指针 Head^   git diff Head Head^
11. git branch -d 分支名  删除分支 -D强制删除 
12. git commit --amend 修改最近一次提交的commit信息
13. **git rebase -i commitId  基于某个历史的commit修改提交信息、合并commit，通常commitId选择需要变更的父级commitId  push到远程之前**
14. git diff --cached 对比暂存区和Head指向版本之间的差异，也可以指定分支中的文件去比较如：git diff version1 version2  --file1
15. git reset HEAD  如果不加文件名，则是把所有暂存区的文件恢复成跟HEAD指向的一样
16. git checkout --file  将工作区文件重置成暂存区一样
17. git reset --hard --comitId 回到某一个提交的目录
18. git stash 将某些暂时的改动存起来，下次要取出来重新编辑可用  git stash apply命令 apply命令不会删除stash里的记录，如果换pop会删除stash里的记录 

### Git协议

![gitxieyi](../images/gitxieyi.png)

![gitbak](../images/gitbak.png)