进入Terminal终端：

1、快捷键： Ctrl+Alt+T

2、点击Dash Home 输入Te

root用户切换到普通用户：su - username

普通用户切换到root用户：su -

切换到root用户，然后启动 启用，禁用防火墙

systemctl start/enable/status/stop/disable/ firewalld

systemctl mask/unmask firewalld

VIM

定位到目录  vim 文件名   进入编辑模式

输入 i  进入编辑模式 可以输入，输入完成之后按esc  退出编辑 Shift+：，然后wq  保存

cat 文件名 打开文件、

1， cat :由第一行开始显示文件内容；

2，tac：从最后一行开始显示，可以看出tac与cat字母顺序相反；

3，nl:显示的时候输出行号；

4，mv  xxx 目的地址

mv xxx ~/.Trash  y移到回收站

cat filename | tail -n 100 显示文件最后100行

cat filename | head -n 100 显示文件前面100行

cat filename | tail -n +100 从100行开始显示，显示100行以后的所有行

显示100行到500行

cat filename | head -n 500 | tail -n +100

Mac退出全屏：Command +Control+F


查看端口：

netstat -lnpt 查看监听端口

netstat -an | grep 3306

lsof -i tcp:portnet

ps aux | grep nsq

brew services start /stop   servicename

通过端口号查看进程id：

```shell
netstat -vanp tcp | grep 3000
```

```shell
sudo lsof -i tcp:3000 
```



![permission](../images/clipboard.png)

当为[ d ]则是目录

当为[ - ]则是文件；

若是[ l ]则表示为链接文档(link file)；

若是[ b ]则表示为装置文件里面的可供储存的接口设备(可随机存取装置)；

若是[ c ]则表示为装置文件里面的串行端口设备，例如键盘、鼠标(一次性读取装置)。

读   写  执行  对应  4,2,1

systemctl is-active application.service

 安装完Tomcat后

开放8080端口给外边访问

iptables -I INPUT -p tcp --dport 80 -j ACCEPT

增加端口后重启一下系统

firewall-cmd --list-ports

查看已经开放的端口

Curl 连接Websocket：

~~~shell
curl --include \
     --no-buffer \
     --header "Connection: Upgrade" \
     --header "Upgrade: websocket" \
     --header "Host: echo.websocket.org" \
     --header "Origin: https://echo.websocket.org" \
     --header "Sec-WebSocket-Key: NVwjmQUcWCenfWu98asDmg==" \
     --header "Sec-WebSocket-Version: 13" \
     http://echo.websocket.org
~~~



ifconfig 网卡名称 down/up 关闭 启用网卡

ifconfig 网卡名称 192.x.x.x netmask x.x.x.x 设置IP  立即生效

nslookup 查询网址解析情况  exit退出

也可用dig 查询（Linux）

刷新dns： windows:ipconfig /flushdns

Linux：sudo /etc/init.d/nscd restart

#### VIM

##### 退出vim的快捷键，不需要进入命令编辑模式

按住shift

zz    保存退出

zq    不保存退出，q表示放弃

之所以按住shift，其实是切换大小写

#####  在命令编辑模式下：

:q 不保存退出

:q! 不保存强制退出

:wq 保存退出，w表示写入，不论是否修改，都会更改时间戳

:x     保存退出，如果内容未改，不会更改时间戳

:w !sudo tee % > /dev/null

##### :x" 和 ":wq" 的区别如下：

(1) :wq 强制性写入文件并退出（存盘并退出 write and quite）。即使文件没有被修改也强制写入，并更新文件的修改时间。

(2) :x 写入文件并退出。仅当文件被修改时才写入，并更新文件修改时间；否则不会更新文件修改时间。

rm  -参数 文件   参数R表示递归删除   -rf表示强制递归删除

tail -f xxxxx.log|grep "find"        只输出符合过滤条件的

\# tail -n20 /var/log/mail/info |head -n1  tail 重后往前 默认10条 

tail -n20 /var/log/mail/info |tee results.txt |head -n1

想要选择最后 20 行，将其保存到 results.txt，但是只在屏幕上显示这 20 行中的第一行

##### Mongo相关

命令行MongoDB：

rs.status()

show

mongo

show dbs

use dbname

db.createColldection("name")

db.collection.insert("xxxxx")

db.getCollectionNames();获取所有的Collection

db.getMongo();

然后：

db.getMongo().setSlaveOk();

Mongo 备份与还原：进入到mongo安装目录:

mongodump/mongorestore   -h 127.0.0.1:27017 -d push -o /home/xiaosiyong/mongo

 mongo --host=192.168.0.104

db.getMongo().setSlaveOk()

then show dbs

{age: {$gt: 22}}查询大于22的   排序  .sort({age:1/-1}) 顺序或者倒叙

使文件可执行：chmod +x filename

#### 查看系统配置

**1、查看CPU**

1）top 后输入1

2）cat /proc/cpuinfo|grep processor

**2、查看内存**

1）top

top 运行中可以通过 top 的内部命令对进程的显示方式进行控制。内部命令如下表：
s – 改变画面更新频率
l – 关闭或开启第一部分第一行 top 信息的表示
t – 关闭或开启第一部分第二行 Tasks 和第三行 Cpus 信息的表示
m – 关闭或开启第一部分第四行 Mem 和 第五行 Swap 信息的表示
N – 以 PID 的大小的顺序排列表示进程列表
P – 以 CPU 占用率大小的顺序排列进程列表
M – 以内存占用率大小的顺序排列进程列表
h – 显示帮助
n – 设置在进程列表所显示进程的数量
q – 退出 top
s – 改变画面更新周期

2）free或free -h

3）head /proc/meminfo

1、查看所有的用户信息cat /etc/passwd

2、查看所有的用户信息cat /etc/passwd|grep 用户名 查找某个用户

3、cat  /etc/group 查看所有组信息  

4、cat /etc/group|grep 组名

查看Linux内核版本命令：

cat     /proc/version

uname  -a

cat /etc/issue

**3、用户和组常用命令**

groups 查看当前登录用户的组内成员

groups test 查看test用户所在的组,以及组内成员

whoami 查看当前登录用户名

**4、环境变量**

环境变量：

/etc/profile       /etc/bashrc

 ~/.bash_profile文件 ~/.bashrc

host文件：

/etc/hosts

**5、重启命令：**

1、reboot

2、shutdown -r now 立刻重启(root用户使用)

3、shutdown -r 10 过10分钟自动重启(root用户使用) 

4、shutdown -r 20:35 在时间为20:35时候重启(root用户使用)

如果是通过shutdown命令设置重启的话，可以用shutdown -c命令取消重启

**6、关机命令：**

1、halt   立刻关机

2、poweroff  立刻关机

3、shutdown -h now 立刻关机(root用户使用)

4、shutdown -h 10 10分钟后自动关机

如果是通过shutdown命令设置关机的话，可以用shutdown -c命令取消重启

mkdir -p xxx/{xx,xx}  建多层目录

df -h 查看磁盘各分区大小、已用空间等信息

du -sh foo 查看foo目录的大小

sudo yum install lrzsz

查看zookeeper 版本：echo stat|nc localhost 2181 

grep -C  10  --color=auto "DH2018121320217453-109853450"  flie   符合条件的前、后10行

grep -B  10   前10行

grep -A  10   后10行

cd ~/.ssh/ 查看rsa 公钥及私钥

**:%s/foo/bar/g** 全文替换

**:s/foo/bar/g 单行替换**

command >out.file 2>&1 &



**Close与Shutdown的区别**

close-----关闭本进程的socket id，但链接还是开着的，用这个socket id的其它进程还能用这个链接，能读或写这个socket id

shutdown--则破坏了socket 链接，读的时候可能侦探到EOF结束符，写的时候可能会收到一个SIGPIPE信号，这个信号可能直到socket buffer被填充了才收到，shutdown还有一个关闭方式的参数，0 不能再读，1不能再写，2 读写都不能。所有数据操作结束以后可以调用Close，从而停止在该socket上的所有操作。

SHutdown可以允许只停止某个方向上的数据传输，如终止读或者终止写。使用close中止一个连接，但它只是减少描述符的参考数，并不直接关闭连接，只有当描述符的参考数为0时才关闭连接。shutdown可直接关闭描述符，不考虑描述符的参考数，可选择中止一个方向的连接。

注意:

​    1>. 如果有多个进程共享一个套接字，close每被调用一次，计数减1，直到计数为0时，也就是所用进程都调用了close，套接字将被释放。

​    2>. 在多进程中如果一个进程中shutdown(sfd, SHUT_RDWR)后其它的进程将无法进行通信. 如果一个进程close(sfd)将不会影响到其它进程

**以数字的格式查看权限：stat -c %a**

root用户开启免密登录检查是否禁用了root登录， /etc/ssh/sshd_config

No Space的时候：

1.首先确定是否是磁盘空间不足  输入命令：df –h 查看磁盘信息

2.输入命令：du -h --max-depth=1 寻找当前目录，哪个文件夹占用空间最大

3.进入logs文件夹 输入命令：ls –lhS 将文件以从大到小顺序展现

#### 查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
#### 查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo| grep "cpu cores"| uniq
#### 查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l
#### 查看线程数      
 grep 'processor'        /proc/cpuinfo | sort -u | wc -l  


#### 日志查找、去重
原内容`{“err_code”： 200，"err_msg"： "this is no error"， “status”： 1， “dev_name”： “mylinux”， “dev_id”： 123}`

如果想要获取dev_id的值，可以用如下命令：

cat example.txt | sed 's/,/\n/g' | grep "dev_id" | sed 's/:/\n/g' | sed '1d' | sed 's/}//g'

1）第一个sed命令的意思是将json数据中的“，”替换为换行符“\n”，这样该串数据就变为每一行一个字段的内容，即按逗号分隔数据串。

2）第二个grep命令的意思是查找“dev_id”关键字，并单列出来。

3）第三个sed命令的意思是将（2）中的结果再次按冒号“：”进行分隔。

4）第四个sed命令的意思是将（3）中的结果，删除第一行内容，即删除“dev_id”行。

5）最后一个sed命令的意思是将最后的花括号“}”用空字符替换，最终得到我们想要的值。

tail -10000  data-sync.log | grep error |sed 's/,/\n/g' |grep 'resource.id'|uniq

### 系统操作篇

Mac 无法打开应用：sudo xattr -d com.apple.quarantine /Applications/xxxx.app

1. man帮助  一共9章，例如：man ls 可以获取相关帮助，不知道哪一章时可以-a 

2. help空格+命令（内部命令）     或者命令空格--help （外部命令）

3. Type +command 显示是内部命令还是外部命令

4. info帮助比help更详细，info ls

5. ls详解![linux-ls](../images/linux-ls.png)

6. cd - 回到之前操作的目录

7. 文件通配符![linux-tongpeifu](../images/linux-tongpeifu.png)

8. 查看文件![linux-text](../images/linux-text.png)

9. 修改权限![linux-ch](../images/linux-ch.png)![linux-file](../images/linux-file.png)

10. touch filename 创建空白文件

11. w 或者uptime查看负载， Linux的负载高，主要是由于CPU使用、内存使用、IO消耗三部分构成。任意一项使用过多，都将导致服务器负载的急剧攀升。查看服务器负载有多种命令，w或者uptime都可以直接展示负载，**load average分别对应于过去1分钟，5分钟，15分钟的负载平均值。**

    $ uptime
     12:20:30 up 44 days, 21:46,  2 users,  load average: 8.99, 7.55, 5.40

    $ w
     12:22:02 up 44 days, 21:48,  2 users,  load average: 3.96, 6.28, 5.16

12. Top 命令查看负载 Tasks行展示了目前的进程总数及所处状态，要注意zombie，表示僵尸进程，不为0则表示有进程出现问题。Cpu(s)行展示了当前CPU的状态，us表示用户进程占用CPU比例，sy表示内核进程占用CPU比例，id表示空闲CPU百分比，wa表示IO等待所占用的CPU时间的百分比。wa占用超过30%则表示IO压力很大。Mem行展示了当前内存的状态，total是总的内存大小，userd是已使用的，free是剩余的，buffers是目录缓存。Swap行同Mem行，cached表示缓存，用户已打开的文件。如果Swap(交换分区，相当于windows中的虚拟内存)的used很高，则表示系统内存不足。

13. **io 监控**   iostat -x 1 10命令，表示开始监控输入输出状态，-x表示显示所有参数信息，1表示每隔1秒监控一次，10表示共监控10次。  

### 进程间通信（Inter-Process Communication）

inux下的进程通信手段基本上是从Unix平台上的进程通信手段继承而来的。而对Unix发展做出重大贡献的两大主力AT&T的贝尔实验室及BSD（加州大学伯克利分校的伯克利软件发布中心）在进程间通信方面的侧重点有所不同。前者对Unix早期的进程间通信手段进行了系统的改进和扩充，形成了“system V IPC”，通信进程局限在单个计算机内；后者则跳过了该限制，形成了基于套接口（socket）的进程间通信机制。Linux则把两者继承了下来，如图示：![linux-ipc](../images/linux-ipc.gif)

进程通信有如下一些目的：

- 数据传输：一个进程需要将它的数据发送给另一个进程，发送的数据量在一个字节到几M字节之间
- 共享数据：多个进程想要操作共享数据，一个进程对共享数据的修改，别的进程应该立刻看到。
- 通知事件：一个进程需要向另一个或一组进程发送消息，通知它（它们）发生了某种事件（如进程终止时要通知父进程）。
- 资源共享：多个进程之间共享同样的资源。为了作到这一点，需要内核提供锁和同步机制。
- 进程控制：有些进程希望完全控制另一个进程的执行（如Debug进程），此时控制进程希望能够拦截另一个进程的所有陷入和异常，并能够及时知道它的状态改变。

Linux 进程间通信（IPC）以下以几部分发展而来：早期UNIX进程间通信（管道、FIFO、信号）、基于System V进程间通信（System V消息队列、System V信号灯、System V共享内存）、基于Socket进程间通信和POSIX进程间通信（posix消息队列、posix信号灯、posix共享内存）。

1、无名管道

管道是单向的、先进先出的、无结构的、固定大小的字节流，它把一个进程的标准输出和另一个进程的标准输入连接在一起。写进程在管道的尾端写入数据，读进程在管道的道端读出数据。数据读出后将从管道中移走，其它读进程都不能再读到这些数据。管道提供了简单的流控制机制。进程试图读空管道时，在有数据写入管道前，进程将一直阻塞。同样，管道已经满时，进程再试图写管道，在其它进程从管道中移走数据之前，写进程将一直阻塞。管道主要用于不同进程（只能用于父子进程或者兄弟进程之间<**具有亲缘关系的进程**>）间通信。管道单独构成一种独立的文件系统：对于管道两端的进程而言，就是一个文件，但它不是普通的文件，它不属于某种文件系统，而是自立门户，单独构成一种文件系 统，并且只存在与内存中。管道两端可 分别用描述字fd[0]以及fd[1]来描述，需要注意的是，管道的两端是固定了任务的。即一端只能用于读，由描述字fd[0]表示，称其为管道读端；另 一端则只能用于写，由描述字fd[1]来表示，称其为管道写端。如果试图从管道写端读取数据，或者向管道读端写入数据都将导致错误发生。

创建一个简单的管道，可以使用系统调用pipe()。它接受一个参数，也就是一个包括两个整数的数组。如果系统调用成功，此数组将包括管道使用的两个文件描述符。创建一个管道之后，一般情况下进程将产生一个新的进程。代码如下所示：

~~~c
#include<stdio.h>
#include<stdlib.h>
#include<errno.h>
#include<string.h>

#define N 10
#define MAX 100

int child_read_pipe(int fd){
    char buf[N];
    int n =0;
    while(1){
        n = read(fd,buf,sizeof(buf));
        buf[n]='\0';
        printf("Read %d bytes: %s.\n",n,buf);
        if(strncmp(buf,"quit",4)==0)
            break;
    }
    return 0;
}

int father_write_pipe(int fd){
    char buf[MAX]={0};
    while(1){
        printf(">");
        fgets(buf,sizeof(buf),stdin);
        buf[strlen(buf)-1]='\0';
        write(fd,buf,strlen(buf));
        usleep(500);
        if(strncmp(buf,"quit",4)==0)
            break;
    }
    return 0;
}

int main(){
    int pid;
    int fd[2];
    if(pipe(fd)<0){//父进程创建管道
        perror("Fail to pipe");
        exit(EXIT_FAILURE);
    }
    if((pid=fork())<0){
        perror("Fail to fork");
        exit(EXIT_FAILURE);
    }else if(pid == 0){
        close(fd[1]);
        child_read_pipe(fd[0]);//子进程读取管道
    }else{
        close(fd[0]);
        father_write_pipe(fd[1]);//父进程写入管道
    }
}
~~~

2、 有名管道

有名管道的使用方式与无名管道不同。有名管道可被任何知道名字的进程打开和使用，为了使用有名管道，进程药建立它，并与他的一段相连。创建有名管道的进程叫做服务器进程，存取管道的其他进程叫客户进程。通信双方必须首先创建有名管道后，才能打开进行读写。当文件不再需要时，要显示删除。

在Linux系统下，有名管道可由两种方式创建：命令行方式mknod系统调用和函数mkfifo。下面的两种途径都在当前目录下生成了一个名为myfifo的有名管道：

- 方式一：mkfifo("myfifo","rw");
- 方式二：mknod myfifo p

生成了有名管道后，就可以使用一般的文件I/O函数如open、close、read、write等来对它进行操作

```c
/*writeFIFO.c*/
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<errno.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>

#define MAX 655360

int main(int argc,char * argv[]){
    int n,fd;
    char buf[MAX];
    if(argc<2){
        fprintf(stderr,"usage : %s argv[1].\n",argv[0]);
        exit(EXIT_FAILURE);    
    }
    if(mkfifo(argv[1],0666)<0 && errno!=EEXIST){//创建有名管道，名称是命令行第一个参数
        fprintf(stderr,"Fail to mkfifo %s : %s.\n",argv[1],strerror(errno));
        exit(EXIT_FAILURE);
    }
    if((fd=open(argv[1],O_WRONLY))<0){
        fprintf(stderr,"Fail to open %s : %s.\n",argv[1],strerror(errno));
        exit(EXIT_FAILURE);
    }
    printf("open for write success.\n");

    while(1){
        printf(">");
        scanf("%d",&n);
        n = write(fd,buf,n);//写入管道
        printf("write %d bytes.\n",n);
    }
    exit(EXIT_SUCCESS);
}
```

~~~c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
#include<errno.h>
#include<sys/types.h>
#include<sys/stat.h>
#include<fcntl.h>

#define MAX 655360

int main(int argc,char * argv[]){
    int n,fd;
    char buf[MAX];
    if(argc<2){
        fprintf(stderr,"usage : %s argv[1].\n",argv[0]);
        exit(EXIT_FAILURE);    
    }
    if(mkfifo(argv[1],0666)<0 && errno!=EEXIST){//创建有名管道，名称是命令行第一个参数
        fprintf(stderr,"Fail to mkfifo %s : %s.\n",argv[1],strerror(errno));
        exit(EXIT_FAILURE);
    }
    if((fd=open(argv[1],O_RDONLY))<0){
        fprintf(stderr,"Fail to open %s : %s.\n",argv[1],strerror(errno));
        exit(EXIT_FAILURE);
    }
    printf("open for read success.\n");

    while(1){
        printf(">");
        scanf("%d",&n);
        n = read(fd,buf,n);//读入有名管道
        printf("Read %d bytes.\n",n);
    }
    exit(EXIT_SUCCESS);
}
~~~

3、信号

进程A实现了对某个信号的处理，然后别的进程B来向进程A发送相应的信号，则A会执行相关的内容，有关信号的详细内容：**[Linux 信号通信](https://blog.csdn.net/ljianhui/article/details/10128731)**

4、消息队列

消息队列就是一个消息的链表。可以把消息看作一个记录，具有特定的格式以及特定的优先级。对消息队列有写权限的进程可以向其中按照一定的规则添加新消息；对消息队列有读权限的进程则可以从消息队列中读走消息。详细内容：**[Linux进程通信之消息队列](消息队列)**

消息队列跟命名管道有不少的相同之处，通过与命名管道一样，消息队列进行通信的进程可以是不相关的进程，同时它们都是通过发送和接收的方式来传递数据的。在命名管道中，发送数据用write，接收数据用read，则在消息队列中，发送数据用msgsnd，接收数据用msgrcv。而且它们对每个数据都有一个最大长度的限制。

与命名管道相比，消息队列的优势在于:

- 消息队列也可以独立于发送和接收进程而存在，从而消除了在同步命名管道的打开和关闭时可能产生的困难。
- 同时通过发送消息还可以避免命名管道的同步和阻塞问题，不需要由进程自己来提供同步方法。
- 接收程序可以通过消息类型有选择地接收数据，而不是像命名管道中那样，只能默认地接收。

代码示例：

~~~c
/**msgreceive.c*/
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <sys/msg.h>

struct msg_st
{
	long int msg_type;
	char text[BUFSIZ];
};

int main()
{
	int running = 1;
	int msgid = -1;
	struct msg_st data;
	long int msgtype = 0; //注意1

	//建立消息队列
	msgid = msgget((key_t)1234, 0666 | IPC_CREAT);
	if(msgid == -1)
	{
		fprintf(stderr, "msgget failed with error: %d\n", errno);
		exit(EXIT_FAILURE);
	}
	//从队列中获取消息，直到遇到end消息为止
	while(running)
	{
		if(msgrcv(msgid, (void*)&data, BUFSIZ, msgtype, 0) == -1)
		{
			fprintf(stderr, "msgrcv failed with errno: %d\n", errno);
			exit(EXIT_FAILURE);
		}
		printf("You wrote: %s\n",data.text);
		//遇到end结束
		if(strncmp(data.text, "end", 3) == 0)
			running = 0;
	}
	//删除消息队列
	if(msgctl(msgid, IPC_RMID, 0) == -1)
	{
		fprintf(stderr, "msgctl(IPC_RMID) failed\n");
		exit(EXIT_FAILURE);
	}
	exit(EXIT_SUCCESS);
}
~~~

~~~c
/*msgsend.c*/
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/msg.h>
#include <errno.h>

#define MAX_TEXT 512
struct msg_st
{
	long int msg_type;
	char text[MAX_TEXT];
};

int main()
{
	int running = 1;
	struct msg_st data;
	char buffer[BUFSIZ];
	int msgid = -1;

	//建立消息队列
	msgid = msgget((key_t)1234, 0666 | IPC_CREAT);
	if(msgid == -1)
	{
		fprintf(stderr, "msgget failed with error: %d\n", errno);
		exit(EXIT_FAILURE);
	}

	//向消息队列中写消息，直到写入end
	while(running)
	{
		//输入数据
		printf("Enter some text: ");
		fgets(buffer, BUFSIZ, stdin);
		data.msg_type = 1;    //注意2
		strcpy(data.text, buffer);
		//向队列发送数据
		if(msgsnd(msgid, (void*)&data, MAX_TEXT, 0) == -1)
		{
			fprintf(stderr, "msgsnd failed\n");
			exit(EXIT_FAILURE);
		}
		//输入end结束输入
		if(strncmp(buffer, "end", 3) == 0)
			running = 0;
		sleep(1);
	}
	exit(EXIT_SUCCESS);
}
~~~

5、共享内存

详细介绍 **[Linux进程间通信——使用共享内存](https://blog.csdn.net/ljianhui/article/details/10253345)**

demo示例：

~~~c
/*shmdata.h*/
#ifndef _SHMDATA_H_HEADER
#define _SHMDATA_H_HEADER

#define TEXT_SZ 2048

struct shared_use_st{
	int written;
	char text[TEXT_SZ];
};

#endif
~~~

~~~c
/*shmread.h 共享内存定义*/
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <sys/shm.h>
#include "shmdata.h"

int main()
{
	int running = 1;//程序是否继续运行的标志
	void *shm = NULL;//分配的共享内存的原始首地址
	struct shared_use_st *shared;//指向shm
	int shmid;//共享内存标识符
	//创建共享内存
	shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666|IPC_CREAT);
	if(shmid == -1)
	{
		fprintf(stderr, "shmget failed\n");
		exit(EXIT_FAILURE);
	}
	//将共享内存连接到当前进程的地址空间
	shm = shmat(shmid, 0, 0);
	if(shm == (void*)-1)
	{
		fprintf(stderr, "shmat failed\n");
		exit(EXIT_FAILURE);
	}
	printf("\nMemory attached at %X\n", (int)shm);
	//设置共享内存
	shared = (struct shared_use_st*)shm;
	shared->written = 0;
	while(running)//读取共享内存中的数据
	{
		//没有进程向共享内存定数据有数据可读取
		if(shared->written != 0)
		{
			printf("You wrote: %s", shared->text);
			sleep(rand() % 3);
			//读取完数据，设置written使共享内存段可写
			shared->written = 0;
			//输入了end，退出循环（程序）
			if(strncmp(shared->text, "end", 3) == 0)
				running = 0;
		}
		else//有其他进程在写数据，不能读取数据
			sleep(1);
	}
	//把共享内存从当前进程中分离
	if(shmdt(shm) == -1)
	{
		fprintf(stderr, "shmdt failed\n");
		exit(EXIT_FAILURE);
	}
	//删除共享内存
	if(shmctl(shmid, IPC_RMID, 0) == -1)
	{
		fprintf(stderr, "shmctl(IPC_RMID) failed\n");
		exit(EXIT_FAILURE);
	}
	exit(EXIT_SUCCESS);
}
~~~

~~~c
/*shmwrite.c*/

#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <sys/shm.h>
#include "shmdata.h"

int main()
{
	int running = 1;
	void *shm = NULL;
	struct shared_use_st *shared = NULL;
	char buffer[BUFSIZ + 1];//用于保存输入的文本
	int shmid;
	//创建共享内存
	shmid = shmget((key_t)1234, sizeof(struct shared_use_st), 0666|IPC_CREAT);
	if(shmid == -1)
	{
		fprintf(stderr, "shmget failed\n");
		exit(EXIT_FAILURE);
	}
	//将共享内存连接到当前进程的地址空间
	shm = shmat(shmid, (void*)0, 0);
	if(shm == (void*)-1)
	{
		fprintf(stderr, "shmat failed\n");
		exit(EXIT_FAILURE);
	}
	printf("Memory attached at %X\n", (int)shm);
	//设置共享内存
	shared = (struct shared_use_st*)shm;
	while(running)//向共享内存中写数据
	{
		//数据还没有被读取，则等待数据被读取,不能向共享内存中写入文本
		while(shared->written == 1)
		{
			sleep(1);
			printf("Waiting...\n");
		}
		//向共享内存中写入数据
		printf("Enter some text: ");
		fgets(buffer, BUFSIZ, stdin);
		strncpy(shared->text, buffer, TEXT_SZ);
		//写完数据，设置written使共享内存段可读
		shared->written = 1;
		//输入了end，退出循环（程序）
		if(strncmp(buffer, "end", 3) == 0)
			running = 0;
	}
	//把共享内存从当前进程中分离
	if(shmdt(shm) == -1)
	{
		fprintf(stderr, "shmdt failed\n");
		exit(EXIT_FAILURE);
	}
	sleep(2);
	exit(EXIT_SUCCESS);
}
~~~

6、信号量

IPC中信号量和信号的关系就像雷锋和雷锋塔的关系一样，那就是基本没关系。详细介绍：**[Linux进程间通信——使用信号量](https://blog.csdn.net/ljianhui/article/details/10243617)**

7、套接字

套接字包括流套接字和数据报套接字 详细介绍：

- **[Linux进程间通信——使用流套接字](https://blog.csdn.net/ljianhui/article/details/10477427)**
- **[Linux进程间通信--数据报套接字](Linux进程间通信——使用数据报套接字)**

各种通信方式的比较和优缺点：

- 管道：速度慢，容量有限，只有父子进程能通讯
- FIFO：任何进程间都能通讯，但速度慢
- 消息队列：容量受到系统限制，且要注意第一次读的时候，要考虑上一次没有读完数据的问题
- 信号量：不能传递复杂消息，只能用来同步
- 享内存区：能够很容易控制容量，速度快，但要保持同步，比如一个进程在写的时候，另一个进程要注意读写的问题，相当于线程中的线程安全，当然，共享内存区同样可以用作线程间通讯，不过没这个必要，线程间本来就已经共享了同一进程内的一块内存

### Linux系统中进程状态

定义：**由操作系统定义，并由操作系统所操控的一个特殊的数据结构实例叫做进程。它连接了用户代码，拥有代码运行所需的独立内存空间，在调度器的调度下使用分配给它的处理器时间片来运行。**

Linux有两类进程：**一类是普通用户进程，它既可以在用户空间运行，又可通过系统调用进入内核空间，并在内核空间运行；另一类叫做内核进程，这种进程只能在内核空间运行。**

~~~c
/*
* The task state array is a strange "bitmap" of
* reasons to sleep. Thus "running" is zero, and
* you can test for combinations of others with
* simple bit tests.
*/
static const char * const task_state_array[] = {
"R (running)",      /* 0 */正在运行 或 可运行(在运行队列排队中)
"S (sleeping)",     /* 1 */可中断睡眠(休眠中，受阻，在等待某个条件的形成或接收到信号)
"D (disk sleep)",   /* 2 */不可中断睡眠(通常是在IO操作)，收到信号不唤醒和不可运行，进程必须等待直到有中断发生
"T (stopped)",      /* 4 */暂停状态，进程收到 SIGSTOP, SIGSTP, SIGTIN, SIGTOU 信号后停止运行
"t (tracing stop)", /* 8 */跟踪状态，指的是进程暂停下来，等待跟踪它的进程对它进行操作
"X (dead)",         /* 16 */退出状态，进程即将被销毁
"Z (zombie)",       /* 32 */退出状态，进程成为僵尸进程
};
~~~

各种状态的含义：

- **运行状态（running）**，并不意味着进程一定在运行中，它表明进程要么是在运行中要么在运行队列里。
- **睡眠状态（sleeping）**，意味着进程在等待事件完成（这里的睡眠有时候也叫做可中断睡眠（interruptible sleep））。
- **磁盘休眠状态（disk sleep）**，有时候也叫不可中断睡眠状态（uninterruptible sleep），在这个状态的进程通常会等待IO的结束。
- **暂停状态（stopped）**可以通过发送 SIGSTOP 信号给进程来停止进程。这个被暂停的进程可以通过发送 SIGCONT 信号让进程继续运行。
- **跟踪状态（tracing stop）**，当进程正在被跟踪时，它处于 TASK_TRACED 这个特殊的状态。“正在被跟踪”指的是进程暂停下来，等待跟踪它的进程对它进行操作。比如在gdb中对被跟踪的进程下一个断点，进程在断点处停下来的时候就处于 TASK_TRACED 状态。而在其他时候，被跟踪的进程还是处于前面提到的那些状态。
- **死亡状态（dead）**，是内核运行 kernel/exit.c 里的 do_exit() 函数返回的状态。这个状态只是一个返回状态，你不会在任务列表里看到这个状态。
- **僵尸状态（zombie）**是一个比较特殊的状态。有些人认为这个状态是在父进程死亡而子进程存活时产生的。实际上不是这样的。父进程可能已经死了但子进程依然存活着，那个子进程的父进程将会成为init进程。当进程退出并且父进程（使用wait()系统调用）没有读取到子进程退出的返回代码时就会产生僵尸进程。僵尸进程会以终止状态保持在进程表中，并且会一直在等待父进程读取退出状态代码。

各种状态之间的转换：![linux-process-state](../images/linux-process-state.png)

### **僵尸进程**

退出一个进程用系统调用exit，但是这并不意味着该进程马上就消失了，事实上它还留下了一个被称为僵尸进程（Zombie）的数据结构。在Linux进程的5种状态中（R、S、D、T、Z），僵尸进程（Z）是非常特殊的一种，它已经放弃了几乎所有内存空间，没有任何可执行代码，也不能被调度，仅仅在进程列表中保留一个位置，记载该进程的退出状态等信息供其他进程收集，除此之外，僵尸进程不再占有任何内存空间。

出现的原因：

- 僵尸进程就是子进程已经结束，但是父进程没有处理的进程！
- 父进程可以使用 waitpid，wait4 等来处理僵尸进程！
- 如果父进程不幸在子进程之前“死了”，那么子进程就交由init(pid == 1)进程去管理

父进程还没有来得及处理子进程的退出状态等信息，所以才会有僵尸进程的出现。

#### 查看进程状态

~~~shell
ps -aux  //查看进程状态
ps -axjf //列出类似程序树的程序显示(显示进程下有哪些子进程)：
ps aux | egrep '(cron|syslog)'  //找出与 cron 与 syslog 这两个服务有关的 PID 号码
~~~

也可以这样使用ps格式输出来查看进程状态:

~~~shell
ps -eo user,stat..,cmd 
~~~

其中，stat包含：

~~~shell
user          用户名 
uid           用户号 
pid           进程号 
ppid          父进程号 
size          内存大小, Kbytes字节. 
vsize         总虚拟内存大小, bytes字节(包含code+data+stack) 
share         总共享页数 
nice          进程优先级(缺省为0, 最大为-20) 
priority(pri) 内核调度优先级 
pmem          进程分享的物理内存数的百分比 
trs           程序执行代码驻留大小 
rss           进程使用的总物理内存数, Kbytes字节 
time          进程执行起到现在总的CPU暂用时间 
stat          进程状态 
cmd(args)     执行命令的简单格式 
~~~

举个🌰：

~~~shell
ps -eo pid,stat,pri,uid --sort uid //查看当前系统进程的uid,pid,stat,pri, 以uid号排序.
ps -eo user,pid,stat,rss,args --sort rss //查看当前系统进程的user,pid,stat,rss,args, 以rss排序.
~~~

在Linux下，还有一种方法检查某个进程是否存在：利用/proc文件系统.。/proc/pid/stat里面有进程的状态,进程可执行文件名等，如果该文件不存在了，那进程肯定退出了。如果存在，可以检查状态和文件名是否正确，效率可能比PS还是高一些，因为/proc是虚拟文件系统,存在与内存中。`cat /proc/pid/status`  这里pid是你的进程ID，看看输出结果，有一栏是State。

### Linux体系结构

其体系结构如图：![linux-arctecture](../images/linux-arctecture.png)

从宏观上来看，Linux操作系统的体系架构分为用户态和内核态（或者用户空间和内核）。内核从本质上看是一种软件——控制计算机的硬件资源，并提供上层应用程序运行的环境。用户态即上层应用程序的活动空间，应用程序的执行必须依托于内核提供的资源，包括CPU资源、存储资源、I/O资源等。为了使上层应用能够访问到这些资源，内核必须为上层应用提供访问的接口：即系统调用。

系统调用是操作系统的最小功能单位，这些系统调用根据不同的应用场景可以进行扩展和裁剪。总结一下，用户态的应用程序可以通过三种方式来访问内核态的资源：

- 系统调用
- 库函数
- Shell脚本

下图是对上图的一个细分结构，从这个图上可以更进一步对内核所做的事有一个“全景式”的印象。主要表现为：向下控制硬件资源，向内管理操作系统资源：包括进程的调度和管理、内存的管理、文件系统的管理、设备驱动程序的管理以及网络资源的管理，向上则向应用程序提供系统调用的接口。从整体上来看，整个操作系统分为两层：用户态和内核态，这种分层的架构极大地提高了资源管理的可扩展性和灵活性，而且方便用户对资源的调用和集中式的管理，带来一定的安全性。![linux-space](../images/linux-space.png)

#### 用户态和内核态的切换

运行于用户态的进程可以执行的操作和访问的资源都会受到极大的限制，而运行在内核态的进程则可以执行任何操作并且在资源的使用上没有限制。很多程序开始时运行于用户态，但在执行的过程中，一些操作需要在内核权限下才能执行，这就涉及到一个从用户态切换到内核态的过程。比如C函数库中的内存分配函数malloc()，它具体是使用sbrk()系统调用来分配内存，当malloc调用sbrk()的时候就涉及一次从用户态到内核态的切换，类似的函数还有printf()，调用的是wirte()系统调用来输出字符串，等等。![linux-space-switch](../images/linux-space-switch.gif)

以下三种情况下，会从用户态切换到内核态：

- 系统调用：原因如上的分析
- 异常事件： 当CPU正在执行运行在用户态的程序时，突然发生某些预先不可知的异常事件，这个时候就会触发从当前用户态执行的进程转向内核态执行相关的异常事件，典型的如缺页异常。
- 外围设备的中断：当外围设备完成用户的请求操作后，会像CPU发出中断信号，此时，CPU就会暂停执行下一条即将要执行的指令，转而去执行中断信号对应的处理程序，如果先前执行的指令是在用户态下，则自然就发生从用户态到内核态的转换。

注意：系统调用的本质其实也是中断，相对于外围设备的硬中断，这种中断称为软中断，这是操作系统为用户特别开放的一种中断，如Linux int 80h中断。所以，从触发方式和效果上来看，这三种切换方式是完全一样的，都相当于是执行了一个中断响应的过程。但是从触发的对象来看，系统调用是进程主动请求切换的，而异常和硬中断则是被动的。

#### Linux 一切皆文件

linux/unix下的哲学核心思想是‘一切皆文件’。
“一切皆文件”，指的是，对所有文件（目录、字符设备、块设备、套接字、打印机、进程、线程、管道等）操作，读写都可用fopen()/fclose()/fwrite()/fread()等函数进行处理。屏蔽了硬件的区别，所有设备都抽象成文件，提供统一的接口给用户。虽然类型各不相同，但是对其提供的却是同一套操作界面。

Linux系统中，文件具体可分为以下几种类型：

1. 普通文件	类似 mp4、pdf、html 这样，可直接拿来使用的文件都属于普通文件，Linux 用户根据访问权限的不同可以对这些文件进行查看、删除以及更改操作。
2. 目录文件  Linux 系统中，目录文件包含了此目录中各个文件的文件名以及指向这些文件的指针，打开目录等同于打开目录文件，只要你有权限，可以随意访问目录中的任何文件。
3. 字符设备文件和块设备文件 些文件通常隐藏在 /dev/ 目录下，当进行设备读取或外设交互时才会被使用。例如，磁盘光驱属于块设备文件，串口设备则属于字符设备文件。Linux 系统中的所有设备，要么是块设备文件，要么是字符设备文件。
4. 套接字文件（Socket） 套接字文件一般隐藏在 /var/run/ 目录下，用于进程间的网络通信。
5. 符号l链接文件（symbolic link）类似与 Windows 中的快捷方式，是指向另一文件的简介指针（也就是软链接）。
6. 管道文件 主要用于进程间通信。例如，使用 mkfifo 命令创建一个 FIFO 文件，与此同时，启用进程 A 从 FIFO文件读数据，启用进程 B 从 FIFO文件中写数据，随写随读。

这样做最明显的好处是，开发者仅需要使用一套 API 和开发工具即可调取 Linux 系统中绝大部分的资源。举个简单的例子，Linux 中几乎所有读（读文件，读系统状态，读 socket，读 PIPE）的操作都可以用 read 函数来进行；几乎所有更改（更改文件，更改系统参数，写 socket，写 PIPE）的操作都可以用 write 函数来进行。

不利之处在于，使用任何硬件设备都必须与根目录下某一目录执行挂载操作，否则无法使用。

图解编译原理里说 **进程的本质是代码区的指令不断执行，驱使动态数据区和静态数据区产生数据变化。**

### Select，Poll And Epoll

select，poll和epoll，三个都是IO多路复用的机制，可以监视多个描述符的读/写等事件，一旦某个描述符就绪（一般是读或者写事件发生了），就能够将发生的事件通知给关心的应用程序去处理该事件。本质上，select，poll和epoll都是同步I/O，UNIX网络编程中，给出了5中IO模型：blocking IO，nonblocking IO，IO multiplexing ，signal driven IO，和asynchronous IO 。其中前面四种都是synchronous IO。

#### IO - 同步、异步、阻塞、非阻塞

下面以network IO中的read读操作为切入点，来讲述同步（synchronous） IO和异步（asynchronous） IO、阻塞（blocking） IO和非阻塞（non-blocking）IO的异同。一般情况下，一次网络IO读操作会涉及两个系统对象：(1) 用户进程(线程)Process；(2)内核对象kernel，两个处理阶段：

~~~javascript
[1] Waiting for the data to be ready - 等待数据准备好
[2] Copying the data from the kernel to the process - 将数据从内核空间的buffer拷贝到用户空间进程的buffer
~~~

IO模型的异同点就是区分在这两个系统对象、两个处理阶段的不同上。

##### 同步IO之Blocking IO

![syncblockingio](../images/syncblockingio.png)

如上图所示，用户进程process在Blocking IO读recvfrom操作的两个阶段都是等待的。在数据没准备好的时候，process原地等待kernel准备数据。kernel准备好数据后，process继续等待kernel将数据copy到自己的buffer。在kernel完成数据的copy后process才会从recvfrom系统调用中返回。

##### 同步IO 之NonBlocking IO

![nonblockingio](../images/nonblockingio.png)

从图中可以看出，process在NonBlocking IO读recvfrom操作的第一个阶段是不会block等待的，如果kernel数据还没准备好，那么recvfrom会立刻返回一个EWOULDBLOCK错误。当kernel准备好数据后，进入处理的第二阶段的时候，process会等待kernel将数据copy到自己的buffer，在kernel完成数据的copy后process才会从recvfrom系统调用中返回。

##### 同步IO 之 IO multiplexing

![multiplexing](../images/multiplexing.png)

IO多路复用，就是我们熟知的select、poll、epoll模型。从图上可见，在IO多路复用的时候，process在两个处理阶段都是block住等待的。初看好像IO多路复用没什么用，其实select、poll、epoll的优势在于可以以较少的代价来同时监听处理多个IO。这个图和blocking IO的图其实并没有太大的不同，事实上还更差一些。因为这里需要使用两个系统调用(select和recvfrom)，而blocking IO只调用了一个系统调用(recvfrom)。但是，用select的优势在于它可以同时处理多个connection。（多说一句：所以，如果处理的连接数不是很高的话，使用select/epoll的web server不一定比使用multi-threading + blocking IO的web server性能更好，可能延迟还更大。select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接。）**在多路复用模型中，对于每一个socket，一般都设置成为non-blocking，但是，如上图所示，整个用户的process其实是一直被block的。**只不过process是被select这个函数block，而不是被socket IO给block。因此select()与非阻塞IO类似。

##### 异步IO

![asynchronousio](../images/asynchronousio.png)

从上图看出，异步IO要求process在recvfrom操作的两个处理阶段上都不能等待，也就是process调用recvfrom后立刻返回，kernel自行去准备好数据并将数据从kernel的buffer中copy到process的buffer在通知process读操作完成了，然后process在去处理。遗憾的是，linux的网络IO中是不存在异步IO的，linux的网络IO处理的第二阶段总是阻塞等待数据copy完成的。真正意义上的网络异步IO是Windows下的IOCP（IO完成端口）模型。

![comparisionfiveio](../images/comparisionfiveio.png)

很多时候，我们比较容易混淆non-blocking IO和asynchronous IO，认为是一样的。但是通过上图，几种IO模型的比较，会发现non-blocking IO和asynchronous IO的区别还是很明显的，**non-blocking IO仅仅要求处理的第一阶段不block即可，而asynchronous IO要求两个阶段都不能block住**。

blocking与non-blocking。前面的介绍中其实已经很明确的说明了这两者的区别。调用blocking IO会一直block住对应的进程直到操作完成，而non-blocking IO在kernel还在准备数据的情况下会立刻返回。
在说明synchronous IO和asynchronous IO的区别之前，需要先给出两者的定义。Stevens给出的定义（其实是POSIX的定义）是这样子的：
\* A synchronous I/O operation causes the requesting process to be blocked until that I/O operation completes;
\* An asynchronous I/O operation does not cause the requesting process to be blocked;
两者的区别就在于synchronous IO做”IO operation”的时候会将process阻塞。按照这个定义，之前所述的blocking IO，non-blocking IO，IO multiplexing都属于synchronous IO。有人可能会说，non-blocking IO并没有被block啊。这里有个非常“狡猾”的地方，定义中所指的”IO operation”是指真实的IO操作，就是例子中的recvfrom这个系统调用。**non-blocking IO在执行recvfrom这个系统调用的时候，如果kernel的数据没有准备好，这时候不会block进程。但是当kernel中数据准备好的时候，recvfrom会将数据从kernel拷贝到用户内存中，这个时候进程是被block了，在这段时间内进程是被block的。**而asynchronous IO则不一样，当进程发起IO操作之后，就直接返回再也不理睬了，直到kernel发送一个信号，告诉进程说IO完成。在这整个过程中，进程完全没有被block。

## Linux的socket 事件wakeup callback机制

在介绍select、poll、epoll前，有必要说说linux(2.6+)内核的事件wakeup callback机制，这是IO多路复用机制存在的本质。Linux通过socket睡眠队列来管理所有等待socket的某个事件的process，同时通过wakeup机制来异步唤醒整个睡眠队列上等待事件的process，通知process相关事件发生。通常情况，socket的事件发生的时候，其会顺序遍历socket睡眠队列上的每个process节点，调用每个process节点挂载的callback函数。在遍历的过程中，如果遇到某个节点是排他的，那么就终止遍历，总体上会涉及两大逻辑：（1）睡眠等待逻辑；（2）唤醒逻辑。

（1）睡眠等待逻辑：涉及select、poll、epoll_wait的阻塞等待逻辑

~~~
[1]select、poll、epoll_wait陷入内核，判断监控的socket是否有关心的事件发生了，如果没，则为当前process构建一个wait_entry节点，然后插入到监控socket的sleep_list
[2]进入循环的schedule直到关心的事件发生了
[3]关心的事件发生后，将当前process的wait_entry节点从socket的sleep_list中删除。
~~~

2）唤醒逻辑：

~~~
[1]socket的事件发生了，然后socket顺序遍历其睡眠队列，依次调用每个wait_entry节点的callback函数
[2]直到完成队列的遍历或遇到某个wait_entry节点是排他的才停止。
[3]一般情况下callback包含两个逻辑：1.wait_entry自定义的私有逻辑；2.唤醒的公共逻辑，主要用于将该wait_entry的process放入CPU的就绪队列，让CPU随后可以调度其执行。
~~~

##### select模型

在一个高性能的网络服务上，大多情况下一个服务进程(线程)process需要同时处理多个socket，我们需要公平对待所有socket，对于read而言，那个socket有数据可读，process就去读取该socket的数据来处理。于是对于read，一个朴素的需求就是关心的N个socket是否有数据”可读”，也就是我们期待”可读”事件的通知，而不是盲目地对每个socket调用recv/recvfrom来尝试接收数据。我们应该block在等待事件的发生上，这个事件简单点就是”关心的N个socket中一个或多个socket有数据可读了”，当block解除的时候，就意味着，我们一定可以找到一个或多个socket上有可读的数据。另一方面，根据上面的socket wakeup callback机制，我们不知道什么时候，哪个socket会有读事件发生，于是，process需要同时插入到这N个socket的sleep_list上等待任意一个socket可读事件发生而被唤醒，当时process被唤醒的时候，其callback里面应该有个逻辑去检查具体那些socket可读了。

于是，select的多路复用逻辑就清晰了，select为每个socket引入一个poll逻辑，该poll逻辑用于收集socket发生的事件，对于可读事件来说，简单伪码如下：

~~~c
poll()
{
    //其他逻辑
    if (recieve queque is not empty)
    {
        sk_event |= POLL_IN；
    }
   //其他逻辑
}
~~~

接下来就到select的逻辑了，下面是select的函数原型：5个参数，后面4个参数都是in/out类型(值可能会被修改返回)

~~~c
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
~~~

当用户process调用select的时候，select会将需要监控的readfds集合拷贝到内核空间（假设监控的仅仅是socket可读），然后遍历自己监控的socket sk，挨个调用sk的poll逻辑以便检查该sk是否有可读事件，遍历完所有的sk后，如果没有任何一个sk可读，那么select会调用schedule_timeout进入schedule循环，使得process进入睡眠。如果在timeout时间内某个sk上有数据可读了，或者等待timeout了，则调用select的process会被唤醒，接下来select就是遍历监控的sk集合，挨个收集可读事件并返回给用户了，相应的伪码如下：

~~~c
for (sk in readfds)
{
    sk_event.evt = sk.poll();
    sk_event.sk = sk;
    ret_event_for_process;
}
~~~

通过上面的select逻辑过程分析，相信大家都意识到，select存在两个问题：

~~~
[1] 被监控的fds需要从用户空间拷贝到内核空间
    为了减少数据拷贝带来的性能损坏，内核对被监控的fds集合大小做了限制，并且这个是通过宏控制的，大小不可改变(限制为1024)。
[2] 被监控的fds集合中，只要有一个有数据可读，整个socket集合就会被遍历一次调用sk的poll函数收集可读事件
    由于当初的需求是朴素，仅仅关心是否有数据可读这样一个事件，当事件通知来的时候，由于数据的到来是异步的，我们不知道事件来的时候，有多少个被监控的socket有数据可读了，于是，只能挨个遍历每个socket来收集可读事件。
~~~

到这里，我们有三个问题需要解决：

1. 被监控的fds集合限制为1024，1024太小了，我们希望能够有个比较大的可监控fds集合 
2. fds集合需要从用户空间拷贝到内核空间的问题，我们希望不需要拷贝 
3. 当被监控的fds中某些有数据可读的时候，我们希望通知更加精细一点，就是我们希望能够从通知中得到有可读事件的fds列表，而不是需要遍历整个fds来收集。

##### poll模型

select遗留的三个问题中，问题(1)是用法限制问题，问题(2)和(3)则是性能问题。poll和select非常相似，poll并没着手解决性能问题，poll只是解决了select的问题(1)fds集合大小1024限制问题。下面是poll的函数原型，poll改变了fds集合的描述方式，使用了pollfd结构而不是select的fd_set结构，使得poll支持的fds集合限制远大于select的1024。poll虽然解决了fds集合大小1024的限制问题，但是，它并没改变大量描述符数组被整体复制于用户态和内核态的地址空间之间，以及个别描述符就绪触发整体描述符集合的遍历的低效问题。poll随着监控的socket集合的增加性能线性下降，poll不适合用于大并发场景。

~~~c
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
~~~

##### Epoll模型

select遗留的三个问题，问题(1)是比较好解决，poll简单两三下就解决掉了，但是poll的解决有点鸡肋。要解决问题(2)和(3)似乎比较棘手，要怎么解决呢？我们知道，在计算机行业中，有两种解决问题的思想：

~~~markdown
[1] 计算机科学领域的任何问题, 都可以通过添加一个中间层来解决
[2] 变集中(中央)处理为分散(分布式)处理
~~~

###### fds集合拷贝问题的解决

对于IO多路复用，有两件事是必须要做的(对于监控可读事件而言)：1. 准备好需要监控的fds集合；2. 探测并返回fds集合中哪些fd可读了。细看select或poll的函数原型，我们会发现，每次调用select或poll都在重复地准备(集中处理)整个需要监控的fds集合。然而对于频繁调用的select或poll而言，fds集合的变化频率要低得多，我们没必要每次都重新准备(集中处理)整个fds集合。

于是，epoll引入了epoll_ctl系统调用，将高频调用的epoll_wait和低频的epoll_ctl隔离开。同时，epoll_ctl通过(EPOLL_CTL_ADD、EPOLL_CTL_MOD、EPOLL_CTL_DEL)三个操作来分散对需要监控的fds集合的修改，做到了有变化才变更，将select或poll高频、大块内存拷贝(集中处理)变成epoll_ctl的低频、小块内存的拷贝(分散处理)，避免了大量的内存拷贝。同时，对于高频epoll_wait的可读就绪的fd集合返回的拷贝问题，epoll通过内核与用户空间mmap(内存映射)同一块内存来解决。mmap将用户空间的一块地址和内核空间的一块地址同时映射到相同的一块物理内存地址（不管是用户空间还是内核空间都是虚拟地址，最终要通过地址映射映射到物理地址），使得这块物理内存对内核和对用户均可见，减少用户态和内核态之间的数据交换。

另外，epoll通过epoll_ctl来对监控的fds集合来进行增、删、改，那么必须涉及到fd的快速查找问题，于是，一个低时间复杂度的增、删、改、查的数据结构来组织被监控的fds集合是必不可少的了。在linux 2.6.8之前的内核，epoll使用hash来组织fds集合，于是在创建epoll fd的时候，epoll需要初始化hash的大小。于是epoll_create(int size)有一个参数size，以便内核根据size的大小来分配hash的大小。在linux 2.6.8以后的内核中，epoll使用红黑树来组织监控的fds集合，于是epoll_create(int size)的参数size实际上已经没有意义了。

###### 按需遍历就绪的fds集合

通过上面的socket的睡眠队列唤醒逻辑我们知道，socket唤醒睡眠在其睡眠队列的wait_entry(process)的时候会调用wait_entry的回调函数callback，并且，我们可以在callback中做任何事情。为了做到只遍历就绪的fd，我们需要有个地方来组织那些已经就绪的fd。为此，epoll引入了一个中间层，一个双向链表(ready_list)，一个单独的睡眠队列(single_epoll_wait_list)，并且，与select或poll不同的是，epoll的process不需要同时插入到多路复用的socket集合的所有睡眠队列中，相反process只是插入到中间层的epoll的单独睡眠队列中，process睡眠在epoll的单独队列上，等待事件的发生。同时，引入一个中间的wait_entry_sk，它与某个socket sk密切相关，wait_entry_sk睡眠在sk的睡眠队列上，其callback函数逻辑是将当前sk排入到epoll的ready_list中，并唤醒epoll的single_epoll_wait_list。而single_epoll_wait_list上睡眠的process的回调函数就明朗了：遍历ready_list上的所有sk，挨个调用sk的poll函数收集事件，然后唤醒process从epoll_wait返回。

 于是，整个过来可以分为以下几个逻辑：

（1）epoll_ctl EPOLL_CTL_ADD逻辑

~~~markdown
[1] 构建睡眠实体wait_entry_sk，将当前socket sk关联给wait_entry_sk，并设置wait_entry_sk的回调函数为epoll_callback_sk
[2] 将wait_entry_sk排入当前socket sk的睡眠队列上
~~~

回调函数epoll_callback_sk的逻辑如下：

~~~markdown
[1] 将之前关联的sk排入epoll的ready_list
[2] 然后唤醒epoll的单独睡眠队列single_epoll_wait_list
~~~

（2）epoll_wait逻辑

~~~javascript
[1] 构建睡眠实体wait_entry_proc，将当前process关联给wait_entry_proc，并设置回调函数为epoll_callback_proc
[2] 判断epoll的ready_list是否为空，如果为空，则将wait_entry_proc排入epoll的single_epoll_wait_list中，随后进入schedule循环，这会导致调用epoll_wait的process睡眠。
[3] wait_entry_proc被事件唤醒或超时醒来，wait_entry_proc将被从single_epoll_wait_list移除掉，然后wait_entry_proc执行回调函数epoll_callback_proc
~~~

3）epoll唤醒逻辑 整个epoll的协议栈唤醒逻辑如下(对于可读事件而言)：

```javascript
[1] 协议数据包到达网卡并被排入socket sk的接收队列
[2] 睡眠在sk的睡眠队列wait_entry被唤醒，wait_entry_sk的回调函数epoll_callback_sk被执行
[3] epoll_callback_sk将当前sk插入epoll的ready_list中
[4] 唤醒睡眠在epoll的单独睡眠队列single_epoll_wait_list的wait_entry，wait_entry_proc被唤醒执行回调函数epoll_callback_proc
[5] 遍历epoll的ready_list，挨个调用每个sk的poll逻辑收集发生的事件
[6] 将每个sk收集到的事件，通过epoll_wait传入的events数组回传并唤醒相应的process。
```

epoll巧妙的引入一个中间层解决了大量监控socket的无效遍历问题。细心的同学会发现，epoll在中间层上为每个监控的socket准备了一个单独的回调函数epoll_callback_sk，而对于select/poll，所有的socket都公用一个相同的回调函数。正是这个单独的回调epoll_callback_sk使得每个socket都能单独处理自身，当自己就绪的时候将自身socket挂入epoll的ready_list。同时，epoll引入了一个睡眠队列single_epoll_wait_list，分割了两类睡眠等待。process不再睡眠在所有的socket的睡眠队列上，而是睡眠在epoll的睡眠队列上，在等待”任意一个socket可读就绪”事件。而中间wait_entry_sk则代替process睡眠在具体的socket上，当socket就绪的时候，它就可以处理自身了。

##### ET(Edge Triggered边沿触发) VS LT(Level Triggered水平触发)

epoll事件的两种模式：

ET：

- socket的接收缓冲区状态变化时触发读事件，即空的接收缓冲区刚接收到数据时触发读事件
- socket的发送缓冲区状态变化时触发写事件，即满的缓冲区刚空出空间时触发写事件

仅在缓冲区状态变化时触发事件，比如数据缓冲去从无到有的时候(不可读-可读)

LT：

- socket接收缓冲区不为空，有数据可读，则读事件一直触发
- socket发送缓冲区不满可以继续写入数据，则写事件一直触发

通常情况下，大家都认为ET模式更为高效，实际上是不是呢？下面我们来说说两种模式的本质：epoll唤醒逻辑的第五个步骤：

`[5] 遍历epoll的ready_list，挨个调用每个sk的poll逻辑收集发生的事件`

大家是不是有个疑问呢：挂在ready_list上的sk什么时候会被移除掉呢？其实，sk从ready_list移除的时机正是区分两种事件模式的本质。因为，通过上面的介绍，我们知道ready_list是否为空是epoll_wait是否返回的条件。于是，在两种事件模式下，步骤5如下：

对于Edge Triggered (ET) 边沿触发：

`[5] 遍历epoll的ready_list，将sk从ready_list中移除，然后调用该sk的poll逻辑收集发生的事件`

对于Level Triggered (LT) 水平触发：

~~~
[5.1] 遍历epoll的ready_list，将sk从ready_list中移除，然后调用该sk的poll逻辑收集发生的事件
[5.2] 如果该sk的poll函数返回了关心的事件(对于可读事件来说，就是POLL_IN事件)，那么该sk被重新加入到epoll的ready_list中。
~~~

对于可读事件而言，在ET模式下，如果某个socket有新的数据到达，那么该sk就会被排入epoll的ready_list，从而epoll_wait就一定能收到可读事件的通知(调用sk的poll逻辑一定能收集到可读事件)。于是，我们通常理解的缓冲区状态变化(从无到有)的理解是不准确的，准确的理解应该是是否有新的数据达到缓冲区。

而在LT模式下，某个sk被探测到有数据可读，那么该sk会被重新加入到read_list，那么在该sk的数据被全部取走前，下次调用epoll_wait就一定能够收到该sk的可读事件(调用sk的poll逻辑一定能收集到可读事件)，从而epoll_wait就能返回。

两种模式性能的对比：

通过上面的概念介绍，我们知道对于可读事件而言，LT比ET多了两个操作：(1)对ready_list的遍历的时候，对于收集到可读事件的sk会重新放入ready_list；(2)下次epoll_wait的时候会再次遍历上次重新放入的sk，如果sk本身没有数据可读了，那么这次遍历就变得多余了。 在服务端有海量活跃socket的时候，LT模式下，epoll_wait返回的时候，会有海量的socket sk重新放入ready_list。如果，用户在第一次epoll_wait返回的时候，将有数据的socket都处理掉了，那么下次epoll_wait的时候，上次epoll_wait重新入ready_list的sk被再次遍历就有点多余，这个时候LT确实会带来一些性能损失。然而，实际上会存在很多多余的遍历么？

先不说第一次epoll_wait返回的时候，用户进程能否都将有数据返回的socket处理掉。在用户处理的过程中，如果该socket有新的数据上来，那么协议栈发现sk已经在ready_list中了，那么就不需要再次放入ready_list，也就是在LT模式下，对该sk的再次遍历不是多余的，是有效的。同时，我们回归epoll高效的场景在于，服务器有海量socket，但是活跃socket较少的情况下才会体现出epoll的高效、高性能。因此，在实际的应用场合，绝大多数情况下，ET模式在性能上并不会比LT模式具有压倒性的优势，至少，目前还没有实际应用场合的测试表面ET比LT性能更好。

复杂度的对比：

我们知道，对于可读事件而言，在阻塞模式下，是无法识别队列空的事件的，并且，事件通知机制，仅仅是通知有数据，并不会通知有多少数据。于是，在阻塞模式下，在epoll_wait返回的时候，我们对某个socket_fd调用recv或read读取并返回了一些数据的时候，我们不能再次直接调用recv或read，因为，如果socket_fd已经无数据可读的时候，进程就会阻塞在该socket_fd的recv或read调用上，这样就影响了IO多路复用的逻辑(我们希望是阻塞在所有被监控socket的epoll_wait调用上，而不是单独某个socket_fd上)，造成其他socket饿死，即使有数据来了，也无法处理。

接下来，我们只能再次调用epoll_wait来探测一些socket_fd，看是否还有数据可读。在LT模式下，如果socket_fd还有数据可读，那么epoll_wait就一定能够返回，接着，我们就可以对该socket_fd调用recv或read读取数据。然而，在ET模式下，尽管socket_fd还是数据可读，但是如果没有新的数据上来，那么epoll_wait是不会通知可读事件的。这个时候，epoll_wait阻塞住了，这下子坑爹了，明明有数据你不处理，非要等新的数据来了在处理，那么我们就死扛咯，看谁先忍不住。

等等，在阻塞模式下，不是不能用ET的么？是的，正是因为有这样的缺点，ET强制需要在非阻塞模式下使用。在ET模式下，epoll_wait返回socket_fd有数据可读，我们必须要读完所有数据才能离开。因为，如果不读完，epoll不会在通知你了，虽然有新的数据到来的时候，会再次通知，但是我们并不知道新数据会不会来，以及什么时候会来。由于在阻塞模式下，我们是无法通过recv/read来探测空数据事件，于是，我们必须采用非阻塞模式，一直read直到EAGAIN。因此，ET要求socket_fd非阻塞也就不难理解了。

另外，epoll_wait原本的语意是：监控并探测socket是否有数据可读(对于读事件而言)。LT模式保留了其原本的语意，只要socket还有数据可读，它就能不断反馈，于是，我们想什么时候读取处理都可以，我们永远有再次poll的机会去探测是否有数据可以处理，这样带来了编程上的很大方便，不容易死锁造成某些socket饿死。相反，ET模式修改了epoll_wait原本的语意，变成了：监控并探测socket是否有新的数据可读。

于是，在epoll_wait返回socket_fd可读的时候，我们需要小心处理，要不然会造成死锁和socket饿死现象。典型如listen_fd返回可读的时候，我们需要不断的accept直到EAGAIN。假设同时有三个请求到达，epoll_wait返回listen_fd可读，这个时候，如果仅仅accept一次拿走一个请求去处理，那么就会留下两个请求，如果这个时候一直没有新的请求到达，那么再次调用epoll_wait是不会通知listen_fd可读的，于是epoll_wait只能睡眠到超时才返回，遗留下来的两个请求一直得不到处理，处于饿死状态。

##### ET And LT 总结：

最后总结一下，ET和LT模式下epoll_wait返回的条件

- ET - 对于读操作

[1] 当接收缓冲buffer内待读数据增加的时候时候(由空变为不空的时候、或者有新的数据进入缓冲buffer)

[2] 调用epoll_ctl(EPOLL_CTL_MOD)来改变socket_fd的监控事件，也就是重新mod socket_fd的EPOLLIN事件，并且接收缓冲buffer内还有数据没读取。(这里不能是EPOLL_CTL_ADD的原因是，epoll不允许重复ADD的，除非先DEL了，再ADD) 因为epoll_ctl(ADD或MOD)会调用sk的poll逻辑来检查是否有关心的事件，如果有，就会将该sk加入到epoll的ready_list中，下次调用epoll_wait的时候，就会遍历到该sk，然后会重新收集到关心的事件返回。

- ET - 对于写操作

[1] 发送缓冲buffer内待发送的数据减少的时候(由满状态变为不满状态的时候、或者有部分数据被发出去的时候) [2] 调用epoll_ctl(EPOLL_CTL_MOD)来改变socket_fd的监控事件，也就是重新mod socket_fd的EPOLLOUT事件，并且发送缓冲buffer还没满的时候。

- LT - 对于读操作 LT就简单多了，唯一的条件就是，接收缓冲buffer内有可读数据的时候
- LT - 对于写操作 LT就简单多了，唯一的条件就是，发送缓冲buffer还没满的时候

在绝大多少情况下，ET模式并不会比LT模式更为高效，同时，ET模式带来了不好理解的语意，这样容易造成编程上面的复杂逻辑和坑点。因此，建议还是采用LT模式来编程更为舒爽。

zsh快捷键

~~~
⌃ + u：清空当前行
⌃ + a：移动到行首
⌃ + e：移动到行尾
⌃ + f：向前移动
⌃ + b：向后移动
⌃ + p：上一条命令
⌃ + n：下一条命令
⌃ + r：搜索历史命令
⌃ + y：召回最近用命令删除的文字
⌃ + h：删除光标之前的字符
⌃ + d：删除光标所指的字符
⌃ + w：删除光标之前的单词
⌃ + k：删除从光标到行尾的内容
⌃ + t：交换光标和之前的字符
 

 

⌘ + Click：可以打开文件，文件夹和链接
⌘ + n：新建窗口
⌘ + t：新建标签页
⌘ + w：关闭当前页
⌘ + 数字 & ⌘ + 方向键：切换标签页
⌥⌘ + 数字：切换窗口
⌘ + enter：切换全屏
⌘ + d：左右分屏
⇧⌘ + d：上下分屏
⌘ + ;：自动补全历史记录
⇧⌘ + h：自动补全剪贴板历史
⌥⌘ + e：查找所有来定位某个标签页
⌘ + r & ⌃ + l：清屏
⌘ + /：显示光标位置
⌥⌘ + b：历史回放
⌘ + f：查找，然后用 tab 和 ⇧ + tab 可以向右和向左补全，补全之后的内容会被自动复制， 还可以用 ⌥ + enter 将查找结果输入终端
选中即复制，鼠标中键粘贴
~~~

### 死锁

所谓死锁，是指多个进程循环等待它方占有的资源而无限期地僵持下去的局面。

产生死锁的必要条件：

1. 互斥条件。即某个资源在一段时间内只能由一个进程占有，不能同时被两个或两个以上的进程占有。这种独占资源如CD-ROM驱动器，打印机等等，必须在占有该资源的进程主动释放它之后，其它进程才能占有该资源。这是由资源本身的属性所决定的。如独木桥就是一种独占资源，两方的人不能同时过桥。
2. 不可抢占条件。进程所获得的资源在未使用完毕之前，资源申请者不能强行地从资源占有者手中夺取资源，而只能由该资源的占有者进程自行释放。如过独木桥的人不能强迫对方后退，也不能非法地将对方推下桥，必须是桥上的人自己过桥后空出桥面（即主动释放占有资源），对方的人才能过桥。
3. 占有且申请条件。进程至少已经占有一个资源，但又申请新的资源；由于该资源已被另外进程占有，此时该进程阻塞；但是，它在等待新资源之时，仍继续占用已占有的资源。还以过独木桥为例，甲乙两人在桥上相遇。甲走过一段桥面（即占有了一些资源），还需要走其余的桥面（申请新的资源），但那部分桥面被乙占有（乙走过一段桥面）。甲过不去，前进不能，又不后退；乙也处于同样的状况。
4. 循环等待条件。存在一个进程等待序列{P1，P2，...，Pn}，其中P1等待P2所占有的某一资源，P2等待P3所占有的某一源，......，而Pn等待P1所占有的的某一资源，形成一个进程循环等待环。就像前面的过独木桥问题，甲等待乙占有的桥面，而乙又等待甲占有的桥面，从而彼此循环等待。

死锁的预防：

 死锁的预防是保证系统不进入死锁状态的一种策略。它的基本思想是要求进程申请资源时遵循某种协议，从而打破产生死锁的四个必要条件中的一个或几个，保证系统不会进入死锁状态。

  〈1〉打破互斥条件。即允许进程同时访问某些资源。但是，有的资源是不允许被同时访问的，像打印机等等，这是由资源本身的属性所决定的。所以，这种办法并无实用价值。

  〈2〉打破不可抢占条件。即允许进程强行从占有者那里夺取某些资源。就是说，当一个进程已占有了某些资源，它又申请新的资源，但不能立即被满足时，它必须释放所占有的全部资源，以后再重新申请。它所释放的资源可以分配给其它进程。这就相当于该进程占有的资源被隐蔽地强占了。这种预防死锁的方法实现起来困难，会降低系统性能。  

  〈3〉打破占有且申请条件。可以实行资源预先分配策略。即进程在运行前一次性地向系统申请它所需要的全部资源。如果某个进程所需的全部资源得不到满足，则不分配任何资源，此进程暂不运行。只有当系统能够满足当前进程的全部资源需求时，才一次性地将所申请的资源全部分配给该进程。由于运行的进程已占有了它所需的全部资源，所以不会发生占有资源又申请资源的现象，因此不会发生死锁。但是，这种策略也有如下缺点：

（1）在许多情况下，一个进程在执行之前不可能知道它所需要的全部资源。这是由于进程在执行时是动态的，不可预测的；

（2）资源利用率低。无论所分资源何时用到，一个进程只有在占有所需的全部资源后才能执行。即使有些资源最后才被该进程用到一次，但该进程在生存期间却一直占有它们，造成长期占着不用的状况。这显然是一种极大的资源浪费；

（3）降低了进程的并发性。因为资源有限，又加上存在浪费，能分配到所需全部资源的进程个数就必然少了。 

〈4〉打破循环等待条件，实行资源有序分配策略。采用这种策略，即把资源事先分类编号，按号分配，使进程在申请，占用资源时不会形成环路。所有进程对资源的请求必须严格按资源序号递增的顺序提出。进程占用了小号资源，才能申请大号资源，就不会产生环路，从而预防了死锁。这种策略与前面的策略相比，资源的利用率和系统吞吐量都有很大提高，但是也存在以下缺点：

（1）限制了进程对资源的请求，同时给系统中所有资源合理编号也是件困难事，并增加了系统开销；

（2）为了遵循按编号申请的次序，暂不使用的资源也需要提前申请，从而增加了进程对资源的占用时间。

死锁的避免：

上面我们讲到的死锁预防是排除死锁的静态策略，它使产生死锁的四个必要条件不能同时具备，从而对进程申请资源的活动加以限制，以保证死锁不会发生。下面我们介绍排除死锁的动态策略--死锁的避免，它不限制进程有关申请资源的命令，而是对进程所发出的每一个申请资源命令加以动态地检查，并根据检查结果决定是否进行资源分配。就是说，在资源分配过程中若预测有发生死锁的可能性，则加以避免。这种方法的关键是确定资源分配的安全性。

1.安全序列

 我们首先引入安全序列的定义：所谓系统是安全的，是指系统中的所有进程能够按照某一种次序分配资源，并且依次地运行完毕，这种进程序列{P1，P2，...，Pn}就是安全序列。如果存在这样一个安全序列，则系统是安全的；如果系统不存在这样一个安全序列，则系统是不安全的。

 安全序列{P1，P2，...，Pn}是这样组成的：若对于每一个进程Pi，它需要的附加资源可以被系统中当前可用资源加上所有进程Pj当前占有资源之和所满足，则{P1，P2，...，Pn}为一个安全序列，这时系统处于安全状态，不会进入死锁状态。 　

 虽然存在安全序列时一定不会有死锁发生，但是系统进入不安全状态（四个死锁的必要条件同时发生）也未必会产生死锁。当然，产生死锁后，系统一定处于不安全状态。 

2. 银行家算法

 这是一个著名的避免死锁的算法，是由Dijstra首先提出来并加以解决的。　

 [背景知识] 

 一个银行家如何将一定数目的资金安全地借给若干个客户，使这些客户既能借到钱完成要干的事，同时银行家又能收回全部资金而不至于破产，这就是银行家问题。这个问题同操作系统中资源分配问题十分相似：银行家就像一个操作系统，客户就像运行的进程，银行家的资金就是系统的资源。

 [问题的描述]

 一个银行家拥有一定数量的资金，有若干个客户要贷款。每个客户须在一开始就声明他所需贷款的总额。若该客户贷款总额不超过银行家的资金总数，银行家可以接收客户的要求。客户贷款是以每次一个资金单位（如1万RMB等）的方式进行的，客户在借满所需的全部单位款额之前可能会等待，但银行家须保证这种等待是有限的，可完成的。

 例如：有三个客户C1，C2，C3，向银行家借款，该银行家的资金总额为10个资金单位，其中C1客户要借9各资金单位，C2客户要借3个资金单位，C3客户要借8个资金单位，总计20个资金单位。某一时刻的状态如图所示。

![bankalgo](../images/bankalgo.png)

 对于a图的状态，按照安全序列的要求，我们选的第一个客户应满足该客户所需的贷款小于等于银行家当前所剩余的钱款，可以看出只有C2客户能被满足：C2客户需1个资金单位，小银行家手中的2个资金单位，于是银行家把1个资金单位借给C2客户，使之完成工作并归还所借的3个资金单位的钱，进入b图。同理，银行家把4个资金单位借给C3客户，使其完成工作，在c图中，只剩一个客户C1，它需7个资金单位，这时银行家有8个资金单位，所以C1也能顺利借到钱并完成工作。最后（见图d）银行家收回全部10个资金单位，保证不赔本。那麽客户序列{C1，C2，C3}就是个安全序列，按照这个序列贷款，银行家才是安全的。否则的话，若在图b状态时，银行家把手中的4个资金单位借给了C1，则出现不安全状态：这时C1，C3均不能完成工作，而银行家手中又没有钱了，系统陷入僵持局面，银行家也不能收回投资。

 综上所述，银行家算法是从当前状态出发，逐个按安全序列检查各客户谁能完成其工作，然后假定其完成工作且归还全部贷款，再进而检查下一个能完成工作的客户，......。如果所有客户都能完成工作，则找到一个安全序列，银行家才是安全的。

 从上面分析看出，银行家算法允许死锁必要条件中的互斥条件，占有且申请条件，不可抢占条件的存在，这样，它与预防死锁的几种方法相比较，限制条件少了，资源利用程度提高了。

这是该算法的优点。其缺点是：

  〈1〉这个算法要求客户数保持固定不变，这在多道程序系统中是难以做到的。  

  〈2〉这个算法保证所有客户在有限的时间内得到满足，但实时客户要求快速响应，所以要考虑这个因素。 

  〈3〉由于要寻找一个安全序列，实际上增加了系统的开销。



### 系统监控

1、部分url过滤
skywalking支持对不需要监控的url进行过滤，使用方法：

config目录下创建apm-trace-ignore-plugin.config文件，如：touch apm-trace-ignore-plugin.config
把不需要的监控的url配置在trace.ignore_path下，如：echo "trace.ignore_path=/actuator/health,/health,/v1/catalog/services" > apm-trace-ignore-plugin.config
把ignore plugin jar放到plugins下，cp optional-plugins/apm-trace-ignore-plugin-7.0.0.jar plugins/

2、字段对应

| 页面显示字段 | 对应Skywalking字段                                           |
| ------------ | ------------------------------------------------------------ |
| 服务         | 服务                                                         |
| 健康度       | 错误数*50% + 响应时间*35% + RPM * 15%， 健康度小于X图标变红，大于等于X图标为绿色 |
|              |                                                              |

​     



SLT  SLT=SUM(接口的qps*接口平均耗时）/1000 ，即所有接口的总耗时。 Service Avg Throughput（cpm） 与ServiceAvg ResponseTime
QPS

Service Avg Throughput（cpm）
响应时间 Service Avg Response Time
RPM Service Avg Throughput（cpm）
错误率 oal计算
Apdex Service Avg ApdexScore
接口 端点
Total Service Avg Throughput（cpm）
Fail 
isError?

Fail% oal计算
AVG

Service Avg ResponseTime
MAX Service Max ResponseTime
MIN Service Min ResponseTime
P90 Golbal Response Time
同比增长率 同比增长率

3、业务线机器关系对应
接口：http://cmdb.bingex.com/openapi/service-info 可以拿到业务线、服务名、以及ip对应关系，需要维护到自身的存储。

4、Trace数据结构
segmentId、traceId、serviceId、serviceInstanceId、endpointName、endpointId、startTime、endTime、latency、isError、dataBinary、version
5、健康度计算
对于一个服务，我们用健康度来衡量其一段时间之内的健康情况，评分0~100，分值越高，越健康。考量健康度的指标有很多种，这里我们采取其中三个主要的指标，分别为：RPM（Request Per Minute），RT（Response Time），EPM（Error Per Minute）。其中，每分钟错误数，也即EPM占比最大，我们设为50%，响应时间次之，占比35%，剩下为每分钟请求数，占比15%。每个服务可配置具体的分值计算规则，下边以订单服务为例，通过RPM，RT以及EPM计算相应的健康度；


周一
周二
周三
周四
周五
RPM 150 130 80 180 160
RT（ms） 100 90 50 110 104
EPM 3 1 0 4 3
健康度 100*0.15+99*0.35+100*0.5 = 99.65 
这里对比数据取的是昨天数据，实际可以根据业务，取上周、去年同一天等数据做平均

|130-150|/150介于0到30%之间所有RPM仍然为100

100*0.15+ （80+(90-50)*(99-80)/(100-50))*0.35+ 100*0.5 =98.32

... ... ...
假设订单服务，对于RPM分值，计算规则如下：

波动范围
分值范围
举例
0%-30%或者RPM<60
100 25%->100,4%->100
31%-70% 70-99 35%->70+(99-70)*(35-31)/(70-31)=72.9
71%-99% 60-69 80%->60+(69-60)*(80-71)/(99-71) = 62.8
100%-无穷大 60 60
假设订单服务RT分值，计算规则如下：

响应时间（ms）
分值范围
举例
0%-5% 100 35->100
6%-10% 80-99 70->80+(70-50)*(99-80)/(100-50)=87.6
11%-30% 61-79 同上
假设订单服务错误数分值，计算规则如下：

每分钟错误数
分值
举例
0%-3% 100 2->100
4%-6% 71-99 5->85
7%-10% 60-70 8->62.5
6、环比，同比
环比是指当前周期与上一个周期相比，所以环比增长率：（当前值-上一周期值）/上一周期值*100%

举个栗子：

昨天RPM
今天RMP
环比增长率
1500 1800 (1800-1500)/1500*100% =20%
而同比是指当前周期与去年的同一时间的周期相比，所以同比增长率：（当前值-比较时间段同一时刻的值)/比较时间段同一时刻的值*100%

一月错误数
去年一月错误数
同比增长率
100 120 （100-120)/120*100%=-16.6%
数据结构梳理

endpoint_avg-20200810

{
"_index" : "endpoint_avg-20200810",
"_type" : "_doc",
"_id" : "202008100008_52",
"_score" : 1.0,
"_source" : {
"service_id" : 18,
"count" : 4,
"time_bucket" : 202008100008,
"service_instance_id" : 24,
"entity_id" : "52",
"value" : 3,
"summation" : 12
}
}

endpoint_percentile-20200821

{
"_index" : "endpoint_percentile-20200821",
"_type" : "_doc",
"_id" : "202008210003_51",
"_score" : 1.0,
"_source" : {
"service_id" : 19,
"precision" : 10,
"time_bucket" : 202008210003,
"service_instance_id" : 41,
"entity_id" : "51",
"value" : "0,0|1,0|2,0|3,0|4,10",
"dataset" : "0,15182|1,724|2,24|3,1"
}
}

endpoint_max-20200811

{
"_index" : "endpoint_max-20200811",
"_type" : "_doc",
"_id" : "202008110002_47",
"_score" : 1.0,
"_source" : {
"service_id" : 19,
"time_bucket" : 202008110002,
"service_instance_id" : 41,
"entity_id" : "47",
"value" : 27
}
}

endpoint_min-20200809

{
"_index" : "endpoint_min-20200809",
"_type" : "_doc",
"_id" : "202008090638_52",
"_score" : 1.0,
"_source" : {
"service_id" : 18,
"time_bucket" : 202008090638,
"service_instance_id" : 24,
"entity_id" : "52",
"value" : 2
}
}

endpoint_avg-20200817

{
"_index" : "endpoint_avg-20200817",
"_type" : "_doc",
"_id" : "202008170003_34",
"_score" : 1.0,
"_source" : {
"service_id" : 17,
"count" : 16234,
"time_bucket" : 202008170003,
"service_instance_id" : 47,
"entity_id" : "34",
"value" : 4,
"summation" : 66021
}
}

service_instance_sla-20200819

{
"_index" : "service_instance_sla-20200819",
"_type" : "_doc",
"_id" : "202008190002_41",
"_score" : 1.0,
"_source" : {
"total" : 40022,
"service_id" : 19,
"percentage" : 10000,
"match" : 40022,
"time_bucket" : 202008190002,
"entity_id" : "41"
}
}

service_instance_relation_server_percentile-20200813

{
"_index" : "service_instance_relation_server_percentile-20200813",
"_type" : "_doc",
"_id" : "202008131004_46_41",
"_score" : 1.0,
"_source" : {
"dest_service_instance_id" : 41,
"source_service_id" : 33,
"dest_service_id" : 19,
"precision" : 10,
"time_bucket" : 202008131004,
"source_service_instance_id" : 46,
"entity_id" : "46_41",
"value" : "0,0|1,0|2,0|3,0|4,0",
"dataset" : "0,2113|1,1|2,1"
}
}



### xclient下载软件无法打开

sudo xattr -d com.apple.quarantine /Applications/xmind.app

Jetbrains系列卸载后无法打开：

打开终端依次执行下列命令	有的文件可能存在JetBrains文件夹里面，需要进到JetBrains才能rm掉cd ~/Library/Preferences/           rm -rf PyCharm2019.2/         cd ~/Library/Logsrm -rf PyCharm2019.2/cd ~/Library/App



