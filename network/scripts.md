[TOC]

## 介绍

使用脚本远程执行我们可以批量执行以及定时任务完成设备配置修改和备份。


### telnetlib 模块

telnetlib 一个模拟tenet登录设备的python模块

```python
#!/usr/bin/python2
# -*- coding: utf-8 -*-
# --------------------------------------
# Author:  LJ
# Email:   admin@attacker.club

# Last Modified: 2018-06-22 22:38:37
# Description: python2 网络设备批量备份和ftp上传
# --------------------------------------

import telnetlib
import time,datetime
import sys
import getpass
reload(sys)
sys.setdefaultencoding("utf-8")



def run(host):
    now = datetime.datetime.now()
    today = now.strftime('%Y%m%d')
    logdate = now.strftime("%b %d %H:%M:%S")
    print("INFO:\t%s\t[\033[1;32m%s\033[0m] Trying to connect　..." % (logdate, host))
    try:
        telnetsession = telnetlib.Telnet(host, timeout=10)  # 实例化telnet对象，建立一个主机连接
        # 开启调试，按需开启，方便判断
        # telnetsession.set_debuglevel(2)

        # read_until()来判断缓冲区中的数据是否有想要的内容，如果没有就等待
        # 当然也可以使用expect方法，与read_until差不多，但是它可以支持正则表达式，功能要强大得多


        telnetsession.write(user + "\n")
        telnetsession.read_very_lazy()
        telnetsession.write(pwd + "\n")

        # 如果登录成功，则出现类似<R1>,使用UsermodTag来进行捕获
        response = telnetsession.read_until('>')
        if response.find('>') > -1:
            print("INFO:\t%s\t[\033[1;32m%s\033[0m] Login successfully　..." % (logdate, host))

            telnetsession.write("dis ip int b\n")
            output=telnetsession.read_until('>')
            print(output)

            telnetsession.write("save\n")
            output=telnetsession.read_until('N')
            print(output)

            telnetsession.write("y\n")
            # telnetsession.read_until('key')
            output=telnetsession.read_until('>')
            print(output)
            telnetsession.write("\n")
            response = telnetsession.read_until('Y/N')
            print(response)
            telnetsession.write("y\n")
            output=telnetsession.read_until('successfully')
            print(response)
          
          
            telnetsession.close()  # 执行完毕后，关闭连接
            print("1111")
            

    except Exception as e:
        print(e, type(e))
    finally:
        print("INFO:\t%s\t[\033[1;32m%s\033[0m] Session Close　..." % (logdate, host))



if __name__ == '__main__':
    
    DEBUG = True #False  # False 读取host.txt列表主机

    # system-view模式如[R1] 来提示用户输入命令，所以取]为作为标志符
    SysrmodTag = ']'

    if not DEBUG:
        with open("host.txt") as f:
            for host in f:
                run(host)
    # 测试
    else: 
        user = 'admin'
        pwd = getpass.getpass()
        host = '172.16.8.10'
        run(host)
```

### 