## find 常用
```shell
find /home/admin /tmp /usr -name \*.log(多个目录去找)
find . -iname \*.txt(大小写都匹配)
find . -type d(当前目录下的所有子目录)
find /usr -type l(当前目录下所有的符号链接)
find /usr -type l -name "z*" -ls(符号链接的详细信息 eg:inode,目录)
find /home/admin -size +250000k(超过250000k的文件，当然+改成-就是小于了)
find /home/admin f -perm 777 -exec ls -l {} \; (按照权限查询文件)
find /home/admin -atime -1  1天内访问过的文件
find /home/admin -ctime -1  1天内状态改变过的文件
find /home/admin -mtime -1  1天内修改过的文件
find /home/admin -amin -1  1分钟内访问过的文件
find /home/admin -cmin -1  1分钟内状态改变过的文件
find /home/admin -mmin -1  1分钟内修改过的文件
```


### find 搜索并执行
```shell
find .  -name   "*.log" -mtime +999 -type f -print -exec rm -f {} \;
#搜索大于6天，log后缀文件并执行删除动作
```

### find 安全搜索
```shell
find / -uid 0 -perm -4000 -print
find / -size +10000k -print|  xargs  du -sh|sort -nr #10M以上的文件
find / -name “…” -print
find / -name “.. ” -print
find / -name “. ” -print
find / -name ” ” -print
注意SUID文件，可疑大于10M和空格文件
find / -name core -exec ls -l {} ;（检查系统中的core文件）
```

### 查看和拷贝文件
```shell
find / -type f -name "vars.example" | xargs -i cat  {}
# 查看内容
find / -type f -name "vars.example" | xargs -i cp  {} .
# copy文件
```

