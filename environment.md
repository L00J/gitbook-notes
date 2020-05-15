[toc]

### 主机环境


|**主机名**|操作系统|配置|
|:---|:--:|:--:|
|Test-1|Centos7|1核2G、硬盘100G|
|Test-2|Centos7|1核2G、硬盘100G|
|Test-3|Centos7|1核2G、硬盘100G|



### 环境准备

- 安装操作系统CentOS-7 x86_64。
- 基本系统：1核2G、硬盘100G。
- 网络选择：使用网络地址转换（桥接）。
- 软件包选择：Minimal Install。
- 关闭iptables和SELinux。
- 设置所有节点的主机名和IP地址，同时使用内部DNS或者/etc/hosts做好主机名解析。

  

### 初始化脚本

```shell
wget -c http://download.opsbase.cn:8000/init/exec.sh # 远程脚本
bash exec.sh xxx-1
```


```shell
#!/bin/bash
# --------------------------------------------------
#Author:  LJ
#Email:   admin@attacker.club

# bash templateos_init.sh  TemplateOS

function color_message() {
  case "$1" in
  "warn")
    echo -e "\e[1;31m$2\e[0m"
    ;;
  "info")
    echo -e "\e[1;33m$2\e[0m"
    ;;
  esac
}

function confirm() {
  read -p 'Are you sure to Continue?[Y/n]:' answer
  case $answer in
  Y | y)
    echo -e "\n\t\t\e[44;37m Running the script \e[0m\n"
    ;;
  N | n)
    echo -e "\n\t\t\033[41;36mExit the script \e[0m\n" && exit 0
    ;;
  *)
    echo -e "\n\t\t\033[41;36mError choice \e[0m\n" && exit 1
    ;;
  esac
}

confirm

if [ $(id -u) -ne 0 ]; then
  color_message "warn" "Use root to execute the program !"
  exit 0
fi
# 判断root权限

Init_Install() {
  Yum_aliyun_repo
  # Yum_lan_repo
  Yum_update_pkg
  Selinux_off
  Set_hostname $1
  Set_iptables
  #Centos_init
  #Set_IP
  Set_dns
  Set_ntp
  Set_ssh
  Set_limits
  Set_profile
  Set_timezone
  Optimize_Performance
  #Set_passwd
}

Yum_aliyun_repo() {
  color_message "info" "---- yum install ----"
  # find  /etc/yum.repos.d/ -type f   ! -name "*Base.repo" -exec rm -f {} \;
  mkdir /etc/yum.repos.d/tmp
  mv /etc/yum.repos.d/*repo /etc/yum.repos.d/tmp

  if [ $rhel_version = 6 ]; then
    wget -O /etc/yum.repos.d/alyun-Centos-7.repo http://mirrors.aliyun.com/repo/Centos-6.repo
    wget -O /etc/yum.repos.d/aliyun-epel6.repo http://mirrors.aliyun.com/repo/epel-6.repo

  elif [ $rhel_version = 7 ]; then
    curl -s http://mirrors.aliyun.com/repo/epel-7.repo >/etc/yum.repos.d/epel-7.repo
    curl -s http://mirrors.aliyun.com/repo/Centos-7.repo >/etc/yum.repos.d/aliyun-Centos7.repo
  else
    echo "Unknown version"
  fi
}
Yum_lan_repo() {
  color_message "info" "---- yum install ----"
  # find  /etc/yum.repos.d/ -type f   ! -name "*Base.repo" -exec rm -f {} \;
  mkdir /etc/yum.repos.d/tmp && mv /etc/yum.repos.d/*repo /etc/yum.repos.d/tmp

  if [ $rhel_version = 6 ]; then
    wget -O /etc/yum.repos.d/alyun-Centos-7.repo http://yum.ops.net/repo/Centos-6.repo
    wget -O /etc/yum.repos.d/aliyun-epel6.repo http://yum.ops.net/repo/epel-6.repo

  elif [ $rhel_version = 7 ]; then
    curl -s http://yum.ops.net/repo/epel-7.repo >/etc/yum.repos.d/epel-7.repo
    curl -s http://yum.ops.net/repo/Centos-7.repo >/etc/yum.repos.d/aliyun-Centos7.repo
  else
    echo "Unknown version"
  fi

}

Yum_update_pkg() {
  #yum update -y
  #Update all packages
  
  
  yum install chronyd ntpdate -y
  # time

  yum install gcc gcc-c++ openssl-devel ntpdate nfs-utils libtool \
    openssl-perl ncurses-devel pcre-devel zlib zlib-devel unzip -y
  #base

  yum install nmap iotop sysstat dstat iftop nload iperf iproute net-tools \
    lrzsz wget vim-enhanced mlocate lsof telnet yum-utils dmidecode -y
  #tools

  #yum install OpenIPMI OpenIPMI-devel OpenIPMI-tools OpenIPMI-libs -y
  #物理机ipmi
}

Set_hostname() {
  #bash host_init.sh hostname 主机名传参
  if [ $# -lt 1 ]; then
    #传参少于1个
    color_message "warn" "---- no set hostname ----"
    HOSTNAME="TemplateOS"
    #默认主机名TemplateOS
  else
    HOSTNAME=$1
  fi

  if [ -f /etc/hostname ]; then
    echo "$HOSTNAME" >/etc/hostname
  fi
  sed -i "/HOSTNAME/c HOSTNAME=$HOSTNAME" /etc/sysconfig/network || echo "HOSTNAME=$HOSTNAME" >>/etc/sysconfig/network
  hostname $HOSTNAME
  grep $HOSTNAME /etc/hosts || echo "127.0.0.1 $HOSTNAME" >>/etc/hosts
}

Selinux_off() {
  color_message "info" "---- close  selinux ----"
  if [ -s /etc/selinux/config ]; then
    setenforce 0
    sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
  fi
}

Set_iptables() {
  color_message "info" "---- setup iptables ----"
  if [ ! -f /etc/sysconfig/iptables ]; then
    yum install iptables-services -y
    chkconfig iptables on
    systemctl enable iptables
    systemctl disable firewalld
    systemctl stop firewalld
    service iptables restart
  fi

  iptables -F
  iptables -X
  iptables -t nat -F
  iptables -t nat -X
  iptables-save
  service iptables save
}

Centos_init() {

  if [ $rhel_version = 6 ]; then
    echo >/etc/udev/rules.d/70-persistent-net.rules &>/dev/null
  fi

  if [ $rhel_version = 7 ]; then
    cp /etc/sysconfig/grub /etc/sysconfig/grub.bak
    grub2-mkconfig -o /boot/grub2/grub.cfg
    #  net.ifnames=0 biosdevname=0
    systemctl disable NetworkManager
    systemctl stop NetworkManager
    echo >/etc/udev/rules.d/90-eno-fix.rules &>/dev/null
  fi
}

Set_IP() {
  color_message "info" "static IPAddress"
  WAN=$(route | grep default | head -1 | awk '{print $NF}')
  #外网出接口
  IPADDR=$(ifconfig $WAN | grep inet | awk '{print $2}')
  NETMASK=$(ifconfig $WAN | grep inet | awk '{print $4}')
  GATEWAY=$(route -n | grep ^0.0.0.0 | awk '{print $2}')
  HWADDR=$(ifconfig $WAN | awk '/ether/ {print "HWADDR="$2}')
  interface=$(ifconfig -a | grep mtu | grep -v $WAN | grep -v lo | awk -F: '{print $1}')
  ifcfg="/etc/sysconfig/network-scripts/ifcfg-eth"

  mv ${ifcfg}-$WAN ${ifcfg}0
  cat >${ifcfg}0 <<EOF
#version1
DEVICE=eth0
BOOTPROTO=none
ONBOOT=yes
USERCTL=no
NM_CONTROLLED=no
MTU=1454

IPADDR=$IPADDR
NETMASK=$NETMASK
GATEWAY=$GATEWAY
$HWADDR
EOF
}

Set_dns() {
  color_message "info" "dns"

  echo "nameserver 223.5.5.5" >/etc/resolv.conf
  echo "nameserver 223.6.6.6" >>/etc/resolv.conf
  # 阿里云dns
}

Set_ntp() {
  color_message "info" "dns"
  grep ntpdate /var/spool/cron/root &>/dev/null || echo '*/3 * * * *  ntpdate ntp.aliyun.com' >>/var/spool/cron/root
  # 默认使用阿里云时间服务


 
grep ntp.aliyun.com /etc/chrony.conf|| cat >/etc/chrony.conf <<EOF
server ntp.aliyun.com iburst
server time1.cloud.tencent.com iburst
# 阿里、腾讯ntp授时服务器

driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
local stratum 10

logchange 0.5
logdir /var/log/chrony  
EOF
}

Set_timezone() {
  \cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
  dmidecode -s system-product-name | awk '{if($1!~"VMware")exit 1}' || hwclock -w
  sed  -i '/bell-style/c set bell-style none' /etc/inputrc # 替换禁止滴滴声
  timedatectl set-timezone "Asia/Shanghai"

}

Set_ssh() {
  color_message "info" "SSH优化"

  #sed  -i '/^#Port/c Port 6022'  /etc/ssh/sshd_config  &>/dev/null  # 默认端口修改
  grep '#UseDNS yes' /etc/ssh/sshd_config && sed -i "s/#UseDNS yes/UseDNS no/g" /etc/ssh/sshd_config
  grep '#AuthorizedKeysFile' /etc/ssh/sshd_config && sed -i "s/#AuthorizedKeysFile/AuthorizedKeysFile/" /etc/ssh/sshd_config
  grep 'GSSAPIAuthentication yes' /etc/ssh/sshd_config && sed -i "s/GSSAPIAuthentication yes/GSSAPIAuthentication no/g" /etc/ssh/sshd_config
  service sshd restart # sshd服务重启
}

Set_limits() {
  color_message "info" "limits"
  chmod +x /etc/rc.local
  grep ulimit /etc/rc.local || echo ulimit -HSn 1048576 >>/etc/rc.local

  grep 1048576 /etc/security/limits.conf || cat >>/etc/security/limits.conf <<EOF
* soft nproc 1048576
* hard nproc 1048576
* soft nofile 1048576
* hard nofile 1048576
* soft stack 1048575
EOF
}

Set_profile() {
  color_message "info" "profile"
  grep vi ~/.bashrc || sed -i "/mv/a\alias vi='vim'" ~/.bashrc
  grep PS /etc/profile || echo '''PS1="\[\e[37;1m\][\[\e[32;1m\]\u\[\e[37;40m\]@\[\e[34;1m\]\h \[\e[0m\]\t \[\e[35;1m\]\W\[\e[37;1m\]]\[\e[m\]/\\$" ''' >>/etc/profile
  grep HISTTIMEFORMAT /etc/profile || echo '''export HISTTIMEFORMAT="%F %T `whoami` " ''' >>/etc/profile
}

Optimize_Performance() {
  color_message "info" "kernel optimize "

  grep 65535 /etc/sysctl.conf || cat >/etc/sysctl.conf <<EOF
fs.file-max = 9999999
# 所有进程最大的文件数
fs.nr_open = 9999999
# 单个进程可分配的最大文件数
fs.aio-max-nr = 1048576
# 1024K；同时可以拥有的的异步IO请求数目

fs.inotify.max_queued_events = 327679
fs.inotify.max_user_instances = 65535
fs.inotify.max_user_watches = 50000000
# 

net.ipv4.ip_local_port_range = 9000 65000
# 被动端口

net.ipv4.tcp_keepalive_time = 180
# 客户端每次发送心跳的周期，默认值为7200s（2小时）；检测服务端是否活着
net.ipv4.tcp_keepalive_intvl = 15
# 探测包的发送间隔 默认75秒
net.ipv4.tcp_keepalive_probes = 5
# 没有接收到对方确认，继续发送保活探测包次数 默认9次

net.ipv4.tcp_tw_reuse = 1
#启用tcp重用
net.ipv4.tcp_fin_timeout = 3
# 决定FIN-WAIT-2状态的时间
net.ipv4.tcp_tw_recycle = 0
# TIME-WAIT的tcp快速回收；入口网关禁用此项


net.core.somaxconn = 4000
#监听队列的长度
net.netfilter.nf_conntrack_max = 262144
# 网络并发连接数等限制
net.nf_conntrack_max = 262144
# 网络并发连接数等限制

# vm.nr_hugepages=512 # 内核大页内存
# net.core.somaxconn = 65535 # 端口最大监听队列长度
# net.ipv4.tcp_max_syn_backlog  # SYN同步包的最大客户端数量

#net.ipv4.conf.all.send_redirects = 0
#net.ipv4.conf.default.send_redirects = 0
#禁止数据包重定向发送 (安全)
EOF

  sysctl -p
}

Set_passwd() {
  echo "tmp123456" | passwd --stdin "root" #修改密码
}

rhel_version=$(grep -oP '\d' /etc/redhat-release | head -1)
#系统版本
Init_Install $1
#调用执行

echo "++++++++++++++++ END To Initialize Server  +++++++++++++++++"
#reboot
rm $0 -f # 回收此脚本文件

```
