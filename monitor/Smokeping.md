#  Smokeping 网络链路监控




**环境**

- centos 7
- apache
- smokeping


## 部署

### apache部署
```
yum -y install httpd httpd-devel gcc make curl wget
```

### 安装依赖库
```
yum -y install libxml2-devel libpng-devel glib pango pango-devel \
freetype freetype-devel fontconfig cairo cairo-devel \
libart_lgpl libart_lgpl-devel

yum -y install perl-Sys-Syslog podofo  mod_fcgid bind-utils
```

安装rrdrool
```
yum -y install perl perl-Net-Telnet perl-Net-DNS perl-LDAP perl-libwww-perl \
perl-RadiusPerl perl-IO-Socket-SSL perl-Socket6 perl-CGI-SpeedyCGI \
perl-FCGI perl-CGI-SpeedCGI perl-Time-HiRes perl-ExtUtils-MakeMaker \
perl-RRD-Simple rrdtool rrdtool-per
```



### 编译安装源码包
```
wget http://www.fping.org/dist/fping-3.10.tar.gz
wget https://fossies.org/linux/misc/old/echoping-6.0.2.tar.gz
wget http://oss.oetiker.ch/smokeping/pub/smokeping-2.6.9.tar.gz
```

```
tar xf fping-3.10.tar.gz
cd fping-3.10
./configure
make && make install
```

```
tar xf echoping-6.0.2.tar.gz
cd echoping-6.0.2
./configure
make && make install
```

```
tar xf smokeping-2.6.9.tar.gz
cd smokeping-2.6.9
./setup/build-perl-modules.sh /usr/local/smokeping/thirdparty
./configure --prefix=/usr/local/smokeping
/usr/bin/gmake install
```

smokeping参数
```
# 创建cache var　data 三个目录和smokeping.log日志文件
cd /usr/local/smokeping/
mkdir cache data var

# 刚刚创建的文件和目录授予apache权限，这里将整个smokeping都赋权
chown -R apache:apache /usr/local/smokeping/
# web文件
cd /usr/local/smokeping/htdocs
mv smokeping.fcgi.dist smokeping.fcgi
cd /usr/local/smokeping/etc
mv config.dist config
```

### 配置文件

vim config
```
cgiurl = http://some.url/smokeping.cgi  #把some.url 改成本地IP或者域名
step = 60  #默认300 建议改为 60 ， 一分钟采集一次数据
```
修改验证密码文件权限
```
chmod 600 /usr/local/smokeping/etc/smokeping_secrets.dist
```
修改apache配置
/etc/httpd/conf/httpd.conf
```nginx
# 在DocumentRoot “/var/www/html” 这一行下添加如下代码：
  Alias /cache "/usr/local/smokeping/cache/"
  Alias /cropper "/usr/local/smokeping/htdocs/cropper/"
  Alias /smokeping "/usr/local/smokeping/htdocs/smokeping.fcgi"
<Directory "/usr/local/smokeping">
  AllowOverride None
  Options All
  AddHandler cgi-script .fcgi .cgi
  Order allow,deny
  Allow from all
  AuthName "Smokeping"
  AuthType Basic
  AuthUserFile /usr/local/smokeping/htdocs/htpasswd
  Require valid-user
  DirectoryIndex smokeping.fcgi
</Directory>
```
```
htpasswd -cd   /usr/local/smokeping/htdocs/htpasswd admin
# 添加web认证密码
```

### 解决中文乱码
```
yum -y install wqy-zenhei-fonts.noarch
```
/usr/local/smokeping/etc/config
```
*** Presentation ***
charset = UTF-8    # 第50行左右添加字符编码
template = /usr/local/smokeping/etc/basepage.html.dist
```

/usr/local/smokeping/lib/Smokeping/Graphs.pm
```
"DEF:maxping=$cfg->{General}{datadir}${host}.rrd:median:AVERAGE",
 '--font TITLE:20:"WenQuanYi Zen Hei Mono"',  #第149左右插入内容
             'PRINT:maxping:MAX:%le' );
```

### 添加检测项
 /usr/local/smokeping/etc/config
```
###########################################################
#+Test
#menu= Targets
##parents = owner:/Test/James localtion:/
......

#menu = Multihost
#title = James and James as seen from Boomer
#host = /Test/James /Test/James~boomer
###########################################################
#以上几行可以直接删除，也可注释掉，没有用，下面添加监控项



# 监控节点样例如下，注意+是第一层，++是第二层，+++ 是第三层
+ Other
menu = 网络链路稳定性监控
title = 监控列表


++ company
menu = 公司监控
title = 网络监控列表
+++  xx-idc
menu = 西溪机房防火墙
title = 西溪机房防火墙
alerts = someloss
host = 183.136.xxx.xxx


++ public
menu = 公共服务
title = 公共服务列表
+++ 114dns
menu = 114.114.114.114
title = 114 DNS
alerts = someloss
host = 114.114.114.114

+++ aliyundns
menu = 223.5.5.5
title = Aliyun DNS
alerts = someloss
host = 223.5.5.5
```

### 重启httpd,和smokeping服务
```
systemctl enable httpd
grep smokeping  /etc/rc.local || echo "/usr/local/smokeping/bin/smokeping --restart" >>  /etc/rc.local
# 开启启动

systemctl restart httpd #启动http
/usr/local/smokeping/bin/smokeping --restart  #启动smokeping
```

### 效果

![](https://oscimg.oschina.net/oscnet/fae68cf0ea64678d61bbb19216692c1a0cb.jpg)




## 扩展功能

### 告警设置
```
*** Alerts ***
to = |/usr/local/smokeping/bin/alert.sh
from = sentinel@huored.com

+bigloss
type = loss
# in percent
pattern = ==0%,==0%,==0%,==0%,>0%,>0%,>0%
comment = suddenly there is packet loss

+someloss
type = loss
# in percent
pattern = >0%,*12*,>0%,*12*,>0%
comment = loss 3 times  in a row

+startloss
type = loss
# in percent
pattern = ==S,>0%,>0%,>0%
comment = loss at startup

+rttdetect
type = rtt
# in milli seconds
pattern = <10,<10,<10,<10,<10,<100,>100,>100,>100
comment = routing messed up again ?

+hostdown
type = loss
# in percent
pattern = ==0%,==0%,==0%, ==U
comment = no reply

+lossdetect #我一般使用lossdetect 策略
type = loss
# in percent
pattern = ==0%,==0%,==0%,==0%,>20%,>20%,>20%
comment = suddenly there is packet loss
```

### 检测类型

```
*** Probes ***

+ FPing
binary = /usr/sbin/fping
packetsize = 1048  #设置ping的包大小

+ TCPPing  # 检测端口，有些高防IP不让ping的
binary = /usr/local/smokeping/bin/tcpping-smokeping
# tcpping是一个shell脚本并调用traceroute，测试:tcpping-smokeping  -C -x 10  attacker  80
# https://github.com/tobbez/tcpping-smokeping
pings = 5
port = 80


监控列表例子
++ SD-JCZJ
probe = TCPPing  # 此处调用了tcpping来测试，不写的话默认是Fping
menu=SD-JCZJ
title=SD-JCZJ-XXXXXX
alerts=lossdetect
host=XXXXXXX
```

### 告警脚本
/usr/local/smokeping/bin/alert.sh
```
########################################################
# Script to email a mtr report on alert from Smokeping #
########################################################
alertname=$1
target=$2
losspattern=$3
rtt=$4
hostname=$5
smokename="ALIYUN-smokeping-"
if [ "$losspattern" = "loss: 0%" ];
then
subject="Clear-${smokename}-Alert: $target host: ${hostname}"
else
subject="${smokename}Alert: ${target} – ${hostname}"
fi
echo "MTR Report for hostname: ${hostname}" > /tmp/mtr.txt
echo "" >> /tmp/mtr.txt
#echo "sudo mtr -n –report ${hostname} "
#sudo /usr/sbin/mtr -n –report ${hostname} >> /tmp/mtr.txt
#echo "" >> /tmp/mtr.txt
echo "Name of Alert: " $alertname >> /tmp/mtr.txt
echo "Target: " $target >> /tmp/mtr.txt
echo "Loss Pattern: " $losspattern >> /tmp/mtr.txt
echo "RTT Pattern: " $rtt >> /tmp/mtr.txt
echo "Hostname: " $hostname >> /tmp/mtr.txt
echo "" >> /tmp/mtr.txt
#echo "Full mtr command is: sudo /usr/sbin/mtr -n –report ${hostname}" >> /tmp/mtr.txt
echo "subject: " $subject
if [ -s /tmp/mtr.txt ]; then
echo "----------------"
#cat /tmp/mtr.txt|mail -s "${subject}" $email

Email=/usr/bin/sendEmail

smtp=smtp.attacker.club
#发件人SMTP服务器
user=info@attacker.club
#发件人账号
passwd='xxxxxx'
#发件人密码
#cc=admin@attacker.club
#抄送
to="admin@attacker.club"
#收件人邮件地址
#subject=主题
body=$(cat /tmp/mtr.txt)

$Email  -f $user -s $smtp -xu $user -xp $passwd -t $to -u "$subject" -m "$body" -o  message-charset=utf-8

fi
```



## smokeping 展示图分析
![](https://oscimg.oschina.net/oscnet/5e4edb0cf16b97e7d6a236ea9bf02388f83.jpg)
X 轴表示时间轴
Y 轴表示 ping 的时间值
3.6ms 表示 Ping 质量测试的响应速度平均值
中间红线能看出网络是否有抖动
直线表示稳定, 有频繁曲线表示网络抖动；如果是阴影表示有网络小幅度抖动
ls 字段表示 Ping 质量测试的丢包率


根据网络抖动判断，抖动范围超过 10ms 的都属于网络不稳定我们要每天观察是否都有规律的网络抖动现象!


![](https://oscimg.oschina.net/oscnet/9a7b6eedda0c0134fc862fc58a17cbd3f65.jpg)
从这个报告图里可以看出:
1. 曲线都是绿色的 0 丢包或偶尔一两个丢包算合格
2. 曲线无抖动, 阴影不明显或偶尔有抖动的算合格 (包裹阴影部分)
3.Ping 值小于 30ms 如果小于 50ms 还算合格

----

## 维护管理

```
/usr/local/smokeping/bin/smokeping  --debug # debug
rm /usr/local/smokeping/data/Other/* -rf &&  /usr/local/smokeping/bin/smokeping --restart # 清理数据
```