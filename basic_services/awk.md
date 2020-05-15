## 显示指定行

```shell
cat /proc/meminfo |awk  'NR==1'
#显示第一行
awk '/^[0-9]/ && NR==1  {print $1}' /data/hostlist
# 过来数字开头而且是第一行，打印第一列；&&和||或
ifconfig |awk '/netmask/ && !/127/  {print $2}'
# 匹配netmask 并匹配非127
```
## 分隔显示

```shell
cat /proc/meminfo |awk  'NR==1'|awk '{print $2}'
#显示第二列
curl -s --head "ops.attacker.club"|awk  '/HTTP/ {print $2}'
#过滤关键字‘HTTP’的行并将第二列内容打印出来
```

## 正则
```shell
awk  -F= '/^DEV/ {print "网卡"$2}' /etc/sysconfig/network-scripts/ifcfg-eth0 
#正则搜索DEV大头的行，打印第二列网卡名

route  -n | awk '$3~/252.0$/{print $1}'|uniq
#正则匹配第三列掩码是252.0则打印第一列网络地址

docker images | awk  '/rancher/||/busybox/  {print $3}'| xargs  docker rmi
#删除包含rancher或者busybox的容器id

ip add |grep -vw lo |awk -F '[ /]+'  '/inet/ {print $3}'
#[空格:]多分隔符写法，以空格或冒号做分隔;"+"号是正则表达式，意思是匹配前面空格或冒号，两者之一的1个或1个以上。

awk '/ldb/ {print}' f.txt   #匹配ldb
awk '!/ldb/ {print}' f.txt  #不匹配ldb
awk '/ldb/ && /LISTEN/ {print}' f.txt   #匹配ldb和LISTEN
awk '$5 ~ /ldb/ {print}' f.txt #第五列匹配ldb
```

## 判断
```
 repo=`df |grep dev |sort -nrk 2|head -1|awk '{if(length($NF)==1) print $NF"repo";else print $NF"/repo"}'`
 #最大分区做ftp家目录；是根目录不加 "/"

 grep Failed /var/log/secure |egrep -o '[0-9]{1,3}(\.[0-9]{1,3}){3}' |sort |uniq -c|sort -nr | awk '{if($1 > 8) print $2}'
# 过滤登录失败的ip地址，awk if如果第一列数字有8次以上则打印第二列ip信息

awk '{if(NR%5==0){print}}' your_file   # 取出可以被5整除的数
awk '{if(NR<=300){print}}' your_file   # 取出行数小于300的数据
```
## BEGIN和END流程控制
https://linux.cn/article-7654-1.html （有时间再测。）


## 其他高级玩法

```shell
awk -F: '$3>=1000 {print $1}' /etc/passwd
#第三列值大于等于1000则打印passwd第一列的用户名
awk -F:  'length($3)==2 {print $1}' /etc/passwd
#第三列字符串是2位长度的，打印第一列用户名信息
#;查看是否存在空口令帐户
awk -F\: '{system("passwd -S "$1)}' /etc/passwd|awk '{print $1,$3}'
#用system和bash执行命令;查看账户创建日期
awk -F\: '{system("passwd -S "$1)}' /etc/passwd|awk '{print $1,$3}'
#同上
#过滤登录失败的ip地址，awk if如果第一列数字有8次以上则打印第二列ip信息
awk '$1> 8 {print $2}'
#同上，效果
grep Failed /var/log/secure |egrep -o '[0-9]{1,3}(\.[0-9]{1,3}){3}' |sort |uniq -c|sort -nr | awk '{if($1 > 8) print $2}'


```

```reStructuredText
[0-9]{1,3}(\.[0-9]{1,3}){3}

[0-9]{1,3}：1-3位数字
\.[0-9]{1,3}：小数点.后跟1-3位数字
(...){3}：前面括号中的组合重复3次
正则表达式中：
.表示“单个任意字符”
\.表示“小数点”
关于IP地址,再提供一种更精确的写法：
\d表示“单个任意数字”
((\d{1,3})\.){3}(\d{1,3})：与你的式子基本等价
![\.\d])：后面不能有.或数字
```