# 办公网无线 AP 部署改造

## 现有环境弊端

```
1.无线设备太多信道会干扰
2.不能统一管理访问控制
3.性能不稳定，时常卡
4.跨办公室不能漫游
5.SSID信道混乱
```

**之前也部署过华为、华三无线项目，此次考虑实施简易性和性价比，本方案使用 TP-link 设备重组企业内部无线网**

## 规划

### 网段地址

```
企业网带宽 100M：
服务器有线网段： 10.0.0.0/22
研发无线线网段： 172.16.0.0/22

办公普通带宽 300M：
办公用户网段：192.168.15.0/23  wifi: XX-Office 密码：office123456
测试终端网段：192.168.10.0/23  wifi: XX-Test  密码:   123456
访客来宾：192.168.9.0/23    wifi: XX-Guest 密码：guest
```

### 无线 ap 架构

![](https://oscimg.oschina.net/oscnet/3ce4392d248d3b55ca940d76b3ea2d20893.jpg)

## 实施部署

### 三层交换机配置

```sh
interface GigabitEthernet0/0/13
 description to POE-switch
 port link-type trunk
 port trunk pvid vlan 8
 port trunk allow-pass vlan 2 to 4094
 stp edged-port enable
```

### TL—AC300 配置

![](https://oscimg.oschina.net/oscnet/a4272882ebd8ced58bf81801aeb0bbf6b3f.jpg)

6/6 》 数量/在线状态

![](https://oscimg.oschina.net/oscnet/1a9def4776d81b7a97907aff861968fc436.jpg)
![](https://oscimg.oschina.net/oscnet/cef34e90cb6a88e74806262139caa47ca97.jpg)
![](https://oscimg.oschina.net/oscnet/ae6691c22535edbf1eafb82933684f1a21b.jpg)

### TL-SG3218P POE 交换机

![](https://oscimg.oschina.net/oscnet/f7ba5be572ceb2923657f1a0392e314976e.jpg)
![](https://oscimg.oschina.net/oscnet/58b986c3b8a5115c97cb070057b88247a2f.jpg)
![](https://oscimg.oschina.net/oscnet/135852a1754072e2cdecdbbaf07013d21a8.jpg)
![](https://oscimg.oschina.net/oscnet/2a9e176cdb037000da7e593986ce081d315.jpg)

## 报价

![](https://oscimg.oschina.net/oscnet/496ffdee7f7f3b808062f99fd9b52105279.jpg)

企业无线覆盖解决方案：
https://service.tp-link.com.cn/detail_article_2302.html
