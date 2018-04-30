---
title: 常用的shell指令
date: 2018-04-29 22:21:41
tags: linux
---

用grep过滤特定字符，也可以用`grep -v`指定筛选出不包含某些字符的行
```
cat /tmp/aa | grep "request timeout" > /tmp/bb
```

用shell来做取某一行并且排列、去重
```
cat /tmp/dd | awk '{print $2}' | sort | uniq -c > /tmp/ee
```

用shell做split，很方便
```
cat /tmp/cc | awk '{split($0,a,":"); print a[3],a[2],a[1]}' > /tmp/dd
```

通过获取name service挂载的机器列表，对每个机器进行ssh登录，并且执行一系列shell指令获取输出到本地文件
```
#!/bin/sh
rm -rf LOG
rm -rf tmp_*
get_hosts_by_path XXX_name_service | while read x;
do
{
  echo -e "=====$x" > "tmp_"$x
  ssh -o StrictHostKeyChecking=no -t $x -n "nice -19 cd /xxx/job_xxx_*/xxxx/ && *do something* " >> "tmp_"$x
  #ssh -o StrictHostKeyChecking=no -t $x -n "nice -19 awk '\$1>\"2017/12/08-16:15\" && /No such file or directory/ {print \$0}' /xxx/job_xxx_*/xxxx/yyy.log" >> "tmp_"$x
  echo -e $x >> LOG
} &
done
while [[ TRUE ]]; do
    cat LOG | wc -l
    sleep 10
done
```