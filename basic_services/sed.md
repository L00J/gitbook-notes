## 选项与参数：

    -n ：使用安静(silent)模式。在一般 sed 的用法中，所有来自 STDIN 的数据一般都会被列出到终端上。但如果加上 -n 参数后，则只有经过sed 特殊处理的那一行(或者动作)才会被列出来。
    -e ：直接在命令列模式上进行 sed 的动作编辑；
    -f ：直接将 sed 的动作写在一个文件内， -f filename 则可以运行 filename 内的 sed 动作；
    -r ：sed 的动作支持的是延伸型正规表示法的语法。(默认是基础正规表示法语法)
    -i ：直接修改读取的文件内容，而不是输出到终端。
    
    动作说明： [n1[,n2]]function
    n1, n2 ：不见得会存在，一般代表『选择进行动作的行数』，举例来说，如果我的动作是需要在 10 到 20 行之间进行的，则『 10,20[动作行为] 』
    
    function：
    a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
    c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
    d ：删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
    i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
    p ：列印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
    s ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是啦！





## 行内操作
### 删除

```
fdisk -l |cut -d " " -f 2-4 |grep dev|sed s/,//
#查看磁盘大小;将','逗号删除
```

### 替换
```
sed 's/root/admin/g' file
#将root替换为admin;类似vim参数s替换g全局匹配
```

### 前后插入字符

```shell
sed -ne 's/aaa/HELLO&/p' test 
#在aaa字符前面插入内容;输出结果：HELLOaaa
sed -ne 's/aaa/&HELLO/p' test
#在aaa字符后面插入内容;输出结果：aaaHELLO
sed 's/^/HEAD&/g' file
#在每行的头添加字符，比如"HEAD"
sed 's/$/&TAIL/g' file
#在每行的行尾添加字符，比如“TAIL”
```

### 换行、空格

```
nl /etc/passwd |sed '10a 1第十行后面开始插入三行\n2\n3 斜杠n是换行\tt大空格'
#换行\n 空格\t,空格键小空格
```



## 整行操作
### 搜索显示

```shell
nl /etc/passwd | sed -n '2p'
#打印第二行,类似于awk NR==2
nl /etc/passwd | sed -n '5,7p'
#打印5-7行
sed -n  '/root/p' /etc/passwd
#只显示包含root的行;参数-n只打印处理的行 
sed   '/nologin/d' /etc/passwd
#删除包含nologin的行，其他输出;d 参数删除
```

### 删除行
```shell
sed -i '8d' file
#删除第8行
nl /etc/passwd | sed '2d' 
#只要删除第2行
nl /etc/passwd | sed '2,5d'
#删除2-5行
nl /etc/passwd | sed '3,$d' 
#要删除第3行到最后一行
sed /PATTERN/d file
sed -i  '/ulimit/d' /etc/rc.local
#删除包含关键字的行
```




### 插入行：通过行号插入
```
nl /etc/passwd | sed '2,5c 2-5行被吃了'
#2-5行替换成指定的内容
nl /etc/passwd | sed "2i it's second line"
#第二行前面插入内容；参数i
nl /etc/passwd | sed '2a  The third line'
#第二行下面插入内容；参数a
```

### 插入行：匹配关键字前后插入
```
sed  -i  "/rm/i\alias vi='vim'"  ~/.bashrc
#在匹配的rm内容上面插入一条vim配置别名的行
grep vi ~/.bashrc || sed  -i  "/mv/a\alias vi='vim'"  ~/.bashrc
#先判断vi内容是否存在，如果不存在则匹配到mv内容在下面插入一行；
```
### 插入行： 行首、行某插入
```shell
sed '1istart'   /root/.bashrc
#首行添加start字符串
sed '$a end.'   /root/.bashrc
#结尾添加end.内容
```

## 其他高级用法
```shell
sed 's/#.*//;/^$/ d' /etc/ssh/ssh_config
#去掉空行和注释；替换#.*用空并将^$空格打头的内容删除;类似用法：egrep -v '^#|^$' /etc/ssh/ssh_config
sed -i '/^#/d;/^$/d' /etc/openvpn/easy-rsa/2.0/vars #删除废话
sed -i "/HOSTNAME/c HOSTNAME=OS" /etc/sysconfig/network
#搜索关键字，取代该行
sed -i  '/HOSTNAME/d;a HOSTNAME=TemplateOS' /etc/sysconfig/network #删除包含hostname的行并重新新建 (替换推荐/c)
    sed  "/accesscore/c Hostname=`hostname`"  /etc/zabbix/zabbix_proxy.conf
````
### 变量使用双引号
单引号有转义功能
```shell
ifcfg="/etc/sysconfig/network-scripts/ifcfg-"
interface=`ifconfig -a |grep mtu|grep -v lo|awk -F: '{print $1}'`
HWADDR=`ifconfig  -a  | awk '/ether/ {print "HWADDR="$2}'`
grep $HWADDR ${ifcfg}eth0
if [ $? = 1 ];then
	sed -i '/HWADDR/d' ${ifcfg}eth0
    sed -i  "\$a $HWADDR"  ${ifcfg}eth0
	rm -f ${ifcfg}$(interface) && reboot
fi
#centos7 替换mac地址
```