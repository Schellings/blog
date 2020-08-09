---
title: Shell脚本保证服务高可用
date: 2020-03-17 16:56:24
tags: Linux
---
<meta name="referrer" content="no-referrer" />

## 服务启动脚本
### 解决方案
`添加开机自启脚本、添加定时任务`
### 监听服务是否启动
```shell script
server_status=`netstat -ntlp|grep 8089`
if [ "$server_status" != "" ];then
    echo "`date "+%Y-%m-%d %H:%M:%S"` | server already started,exit $sso_status"
else
    echo "`date "+%Y-%m-%d %H:%M:%S"` | begin start server"
    ....
fi
```
### 监听数据库状态
```shell script
DB_VIP=192.168.43.193
DB_PORT=3306
cnt=1
while [[ $cnt -le 100 ]]
        do
                        db_status=`echo ""|telnet $DB_VIP $DB_PORT 2>/dev/null|grep "\^]"|wc -l`
                        if [[ "$db_status" -eq 1 ]]; then
                                        echo "`date "+%Y-%m-%d %H:%M:%S"` | mysql connector success,restart server"
                                        # start server
                                        break
                        else
                                        echo "`date "+%Y-%m-%d %H:%M:%S"` | mysql connector error,sleep 3s"
                                        sleep 3
                        fi
                        let cnt++
        done
```

### 定时任务
```shell script
vim /etc/crontab
# 每五分钟执行一次服务检查
 */5 *  *  *  * root nohup /opt/task/server_start.sh >> /var/run/log/server_start.log 2>&1 &
# 每周清理一次服务启动日志
  0   0  *  *  0 root echo "" > /var/run/log/server_start.log 2>&1 &
```
### 开机自启
```shell script
echo "/opt/task/server_start.sh >> /var/run/log/server_start.log 2>&1 &" >> /etc/rc.local
```