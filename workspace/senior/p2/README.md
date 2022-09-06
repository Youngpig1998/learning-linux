# Linux—grep|awk|sort|uniq



## 管道符 |

​	前面的输出作为后面的输入

## grep 

​	查看文件中是否有我想要的内容。可以理解为正则表达式

```shell
grep [参数] 文件名 

 -c 打印符合要求的行数 
 -v 打印不符合要求的行 
 -n 在输出符合要求的行的同时连同行号一起输出 
 -i 忽略大小写 
 [0-9]
 ^


 grep -c 'root' /etc/passwd
 grep -nv 'root' /etc/passwd
 grep '[0-9]' 文件名 
 grep '^#' -v test.py 
 grep h* xxx.txt #查看h出现过0~n次的内容
 grep ^H xxx.txt #查看所有以H开头的内容
 grep ^Ho xxx.txt #查看所有以Ho开头的内容

$ grep 'r..o' /etc/passwd   #其中 . 可以是任何字符
operator:x:11:0:operator:/root:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin


 grep 'o{2}' /etc/passwd #匹配出现2次 o 的 



 
```

## wc 

​	文件内容的统计

```bash
-l 统计你有多少行 
-w 统计有多少个单词 
-c 字节数




$ cat /etc/passwd | wc -l #统计有多少个用户 
46

$ grep [A-Za-z]ill xxx.txt | wc #统计一共有多少行、多少个单词及字节大小
```

## awk

​	流式编辑器 针对文档的行去操作 一行一行的去执行

```shell
$ head -n 2 /etc/passwd | awk -F ':' '{print $1 $7}'
root/bin/bash
daemon/usr/sbin/nologin



 



-F 指定分隔符 
$0 代表所有的列 
$1 代表 第一列


$ head -n 2 /etc/passwd | awk -F ':' '{print $1"~"$7}' 指定字符串连接符 一定要双引号 
root~/bin/bash
daemon~/usr/sbin/nologi



awk /root/ /etc/passwd #匹配root 
awk -F ':' '$1 ~/root/' /etc/passwd #匹配第一列式root 的 那一行 
awk -F ':' '$3==0' /etc/passwd #匹配第三列等于0的哪一行 
awk -F ':' '$7!="/usr/sbin/nologin"' /etc/passwd
awk -F ':' '$3 < $4' /etc/passwd
awk -F ':' '$3 > 100 || $7 == "/usr/sbin/nologin"' /etc/passwd 
awk -F ':' '$3 > 5 && $3 < 7' /etc/passwd
awk -F ":" '{(total=total+$3)};END{print total}' /etc/passwd #对所有行第三列求和 


head -n 3 /etc/passwd | awk -F ':' '$1 == "root"'
```

## uniq unique 

​	删除重复的行 跟`sort`命令 组合使用

```shell
$ sort -n -t ":" -k 1 | uniq -c

-c 在每行前面加上出现的次数 
-d 只输出重复的行 多行只输出一行
-D 只输出重复的行 多行有几行输出几行 
-i 忽略大小写 



 $ sort test.txt | uniq -c 
 1 apple
 2 banana
 1 caomei
 1 huaguang
 1 juhua
 1 orange


$ sort test.txt | uniq -d 
banana


$ sort test.txt | uniq -D
banana
banana
```

## sort 

​	排序 默认按照首字母排序

```shell
-n 按照数值排序 
-t 指定分割符 
-k 指定第几列 
-r 逆向排序



$ cat /etc/passwd | sort -n -t ":" -k 3 -r 按照 ：分割符 指定第三列 纯数值排序 逆向排序 
```

面试题 : 查找你最常使用的10条命令

```shell
$ history |awk '{print $2}'| sort |uniq -c|sort -n -k 1 -r|head -n 10
```