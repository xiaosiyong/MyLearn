## Git 操作

### Config 配置用户名和邮箱

1. git congfig --local/global user.name 'DomXiao'
2. git congfig --local/global user.email 'xiaosywy@163.com'

config的作用域![gitconfig](../images/gitconfig.png)

### 两种方式新建git项目

![init](../images/initGit.png)

### 命令

git 有缓冲区的概念，先添加到缓冲区，再提交

1. git mv 命令 给文件重命名
2. git add -u 对已经添加到缓冲区的文件的更改
3. git log --oneline命令（一行显示git commit）  git log -n4 (查看最近4次的修改记录)

