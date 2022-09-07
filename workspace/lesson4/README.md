# linux 命令：nohup 详解

​	**nohup** 英文全称 no hang up（不挂起），用于在系统后台不挂断地运行命令，退出终端不会影响程序的运行。**nohup** 命令，在默认情况下（非重定向时），会输出一个名叫 nohup.out 的文件到当前目录下，如果当前目录的 nohup.out 文件不可写，输出重定向到 **$HOME/nohup.out** 文件中。

​	即使关掉终端，该程序依然能运行，该命令是进程忽略hangup（挂起）信号。

​	用法：

```shell
$ nohup COMMAND [ARG]...
$ nohup OPTION
```

​	如果标准输入是终端，则将其从不可读的文件中重定向。 如果标准输出是终端，则尽可能将输出附加到“nohup.out”，否则附加到“$HOME/nohup.out”。 如果标准错误是终端，则将其重定向到标准输出。 要将输出保存到 FILE，请使用“nohup COMMAND > FILE”。

```shell
#选项

--help        帮助文档

--version        版本信息
```

​	在终端如果看到以下输出说明运行成功：

```
appending output to nohup.out
```

​	这时我们打开 root 目录 可以看到生成了 nohup.out 文件。

​	如果要停止运行，你需要使用以下命令查找到 nohup 运行脚本到 PID，然后使用 kill 命令来删除：

```shell
$ nohup /root/runoob.sh &
$ ps -aux | grep "runoob.sh" 
```



​	使用示例：

​	写一个实例脚本，每一秒输出一个数字，数字自动增加：

```sh
# cat nohuptest.sh
#!/bin/bash
count=0
while [[ $count -lt 10000 ]]
do
	echo $count
	sleep 1
	((count++))
done
```

​	首先在前台执行一下，看看是什么效果：

```shell
$ sh nohuptest.sh
0
1
2
3
4
5
^C
```

​	在 linux 系统中，^C（Ctrl + C）发出的是 SIGINT 信号，可以终止进程。在接收到 SIGINT 信号后，前台执行的进程立刻就终止了。

下面用 nohup 试一下：

```shell
$ nohup sh nohuptest.sh
nohup: ignoring input and appending output to ‘nohup.out’

^C

$ cat nohup.out
0
1
2
3
4
5

$ ps -ef | grep nohup
```


​	如果使用 nohup 接启动命令，虽然输出是从终端转到了 nohup.out 中，但是前台还是被占用，无法执行其他操作。但是在操作了 ^C 后，虽然可以进行其他操作，进程也停止了。

```shell
$ nohup sh nohuptest.sh &
[1] 9153
nohup: ignoring input and appending output to ‘nohup.out’

$ cat nohup.out
0

$ cat nohup.out
0
1

$ cat nohup.out
0
1
2
...

$ ps -ef | grep nohup
root      9153  6178  0 14:38 pts/0    00:00:00 sh nohuptest.sh
```


​	使用 & 后，输出一个进程ID，进程在后台执行，前台不受影响。查看 nohup.out 文件，发现数字在逐渐增加。

```shell
$ nohup sh nohuptest.sh > /dev/null &
[1] 10334
 nohup: ignoring input and redirecting stderr to stdout

$ cat nohup.out

```

​	可以把输出重定向到 /dev/null，/dev/null 是一个黑洞，重定向到它的数据都会被扔掉。查看 nohup.out，发现是空文件。



​	以下命令在后台执行 root 目录下的 runoob.sh 脚本，并重定向输入到 runoob.log 文件：

```shell
$ nohup /root/runoob.sh > runoob.log 2>&1 &
```

​	**2>&1** 解释：

​	将标准错误 2 重定向到标准输出 &1 ，标准输出 &1 再被重定向输入到 runoob.log 文件中。

- 0 – stdin (standard input，标准输入)
- 1 – stdout (standard output，标准输出)
- 2 – stderr (standard error，标准错误输出)