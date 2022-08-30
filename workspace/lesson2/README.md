# 文件查看和处理

| 常用命令                      | 作用                                    |
| ----------------------------- | --------------------------------------- |
| cat file1                     | 查看⽂件内容                            |
| cat -n file1                  | 查看内容并标示⾏数                      |
| tac file1                     | 从最后⼀⾏开始反看⽂件内容              |
| more file1                    | more file1                              |
| less file1                    | 类似more命令，但允许反向操作            |
| head -2 file1                 | 查看⽂件前两⾏                          |
| tail -2 file1                 | 查看⽂件后两⾏                          |
| tail -f /log/msg              | 实时查看添加到⽂件中的内容              |
| grep codesheep hello.txt      | 在⽂件hello.txt中查找关键词codesheep    |
| grep ^sheep hello.txt         | 在⽂件hello.txt中查找以sheep开头的内容  |
| grep [0-9] hello.txt          | 选择hello.txt⽂件中所有包含数字的⾏     |
| sed 's/s1/s2/g' hello.txt     | 将hello.txt⽂件中的s1替换成s2           |
| sed '/^$/d' hello.txt         | 从hello.txt⽂件中删除所有空⽩⾏         |
| sed '/ *#/d; /^$/d' hello.txt | 从hello.txt⽂件中删除所有注释和空⽩⾏   |
| sed -e '1d' hello.txt         | 从⽂件hello.txt 中排除第⼀⾏            |
| sed -n '/s1/p' hello.txt      | 查看只包含关键词"s1"的⾏                |
| sed -e 's/ *$//' hello.txt    | 删除每⼀⾏最后的空⽩字符                |
| sed -e 's/s1//g' hello.txt    | 从⽂档中只删除词汇s1并保留剩余全部      |
| sed -n '1,5p;5q' hello.txt    | 查看从第⼀⾏到第5⾏内容                 |
| sed -n '5p;5q' hello.txt      | 查看第5⾏                               |
| paste file1 file2             | 合并两个⽂件或两栏的内容                |
| paste -d '+' file1 file2      | 合并两个⽂件或两栏的内容，中间⽤"+"区分 |
| sort file1 file2              | 排序两个⽂件的内容                      |
| comm -1 file1 file2           | ⽐较两个⽂件的内容(去除'file1'所含内容) |
| comm -2 file1 file2           | ⽐较两个⽂件的内容(去除'file2'所含内容  |
| comm -3 file1 file2           | ⽐较两个⽂件的内容(去除两⽂件共有部分)  |