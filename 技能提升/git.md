## Git 操作

### Config 配置用户名和邮箱

1. git congfig --local/global user.name 'DomXiao'
2. git congfig --local/global user.email 'xiaosywy@163.com'

config的作用域![gitconfig](../images/gitconfig.png)

### 两种方式新建git项目

![init](../images/initGit.png)

### 命令

git 有缓冲区的概念，先添加到缓冲区，再提交![gitmanagement](../images/gitmanagement.png)

1. git mv 命令 给文件重命名  如：git mv readme readme.md
2. git add -u 对已经添加到缓冲区的文件的更改
3. git log --oneline命令（一行显示commit信息，后边也可以加分支信息 ）  git log -n4 (查看最近4次的修改记录) 
4. git branch -v 查看本地分支版本
5. Git checkout -b 分支名 commitId   基于commit创建新的分支
6. git log --graph  图形化的方式查看log  
7. git cat-file -t xxxxx    Git 提供的接口查看文件类型（commit，blob，tree）  查看具体内容 -p   三种文件之间的关系![gitfilerelation](../images/gitfilerelation.png)
8. 

