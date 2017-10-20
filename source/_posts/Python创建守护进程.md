---
title: Python创建守护进程
date: 2017-10-20 19:25:39
tags:
- 守护进程
categories:
- Python
---

### 守护进程

守护进程(daemon)生存期长的一种进程。它们没有控制终端，所以说它们是在后台运行的。使用Python做一个守护进程简单方法是`fork`, 但是有`fork`和`double fork magic`

##### fork：

```
import os
import sys

pid = os.fork()  # 只fork一次，然后主进程退出，子进程称为孤儿进程被init进程接管，在后台运行。
if pid > 0:
	sys.exit()
...
do_something_in_child_process()
```

##### double-fork:

```
import os
import sys

os.umask(0)      # ①

pid = os.fork()  # ②
if pid > 0:      # 父进程退出
	sys.exit()

os.setsid()      # ③
pid = os.fork()  
if pid > 0:
	sys.exit()  # 第一次fork的子进程退出

os.chdir('/')  # ④
si = open('/dev/null', 'r')  
so = open('/dev/null', 'a+')
se = open('/dev/null', 'a+')

os.dup2(si.fileno(), sys.stdin.fileno())
os.dup2(so.fileno(), sys.stdout.fileno())
os.dup2(se.fileno(), sys.stderr.fileno())

daemon()   # 孙子进程在后台开始运行。

```

① 修改文件权限，具体看场景，为什么要修改权限，因为继承来的权限可能会被设置为拒绝某些权限(不过大家都是用root起进程的，你懂得😉)。

② 调用fork使父进程exit。为什么？第一，让shell认为该命令已经执行完毕。第二，保证子进程不是一个进程组的组长进程。这是setsid调用的先决条件。

③ 调用setsid创建一个新会话(会话是干嘛的？留个坑以后填)。调用setsid的效果是，第一，成为新会话的首进程，第二，成为一个新进程的组长进程，第三，没有控制终端。

> 此时调用了第二次fork，这是为了防止守护进程获得控制终端。

④ 修改当前工作目录为根目录，修改stdin, stdout, stderr等。（why? 留坑）

> 为什么要double-fork，在基于System V的系统中，有些人建议再次fork，终止父进程，继续使用子进程中的守护进程。这就保证了该守护进程不是会话首进程，可以防止它获取控制终端。 
>
> 参见《UNIX环境高级编程》- - - 13.3编程规则。