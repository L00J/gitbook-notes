### grep 参数
参数
-c 只输出匹配行的个数。
-i 不区分大小写（只适用于单字符）。
-h 查询多文件时不显示文件名。
-l 查询多文件时只输出包含匹配字符的文件名。
-n 显示匹配行及行号。
-s 不显示不存在或无匹配文本的错误信息。
**-v** 显示不包含匹配文本的所有行（反向匹配）。
-V 显示软件版本信息
-E  egrep 扩展的正则表达式
-P perl的正则
使用grep匹配时最好用双引号引起来，防止被系统误认为参数或者特殊命令，也可以匹配多个单词。

### grep 精确匹配
```shell
grep "\<abc\>" file
grep –w "abc" file
# 精确匹配内容
grep –wc "abc" file
# 精确匹配行数，wc -l
```

### grep 判断追加
```shell
grep PS /etc/profile || echo '''PS1="\[\e[37;1m\][\[\e[32;1m\]\u\[\e[37;40m\]@\[\e[34;1m\]\h \[\e[0m\]\t \[\e[35;1m\]\W\[\e[37;1m\]]\[\e[m\]/\\$" ''' >>/etc/profile
# 如果grep没有过滤到含'PS'的行,追加新内容到profile文件；这里使用||逻辑或判断
```

### grep 多条件匹配

1.同时满足多个条件：
```shell
fdisk -l |grep D|grep dev #套用两次grep过滤，查看物理硬盘
```


2.匹配任意条件
```shell
ethtool eno16777736 |egrep 'Speed|Duplex'
#egrep增强命令,查看eno16777736网卡(物理机) 速度和双工模式
```



### grep 搜索内容

1.字符串内容
```shell
grep -r copyright|grep  index
# r参数归档目录下所有文件，查找包含copyright并且是index文件名的文件
```
2.数字内容
```shell
cat /proc/meminfo |awk  'NR==1'|grep  -o '[0-9]\{1,\}'
# o参数显示匹配的内容，数字0-9范围，如果{1,99\} 1行99位；查看内存大小
id|grep -oP "\d"|head -1
# perl的正则
```
3.只列出文件
```shell
grep -rl  localhost
#搜索网站连接数据库的文件并只列出文件名
```

位置
```shell
seq 10 | grep 5 -A 3    #上匹配
seq 10 | grep 5 -B 3    #下匹配
seq 10 | grep 5 -C 1    #上下匹配
```