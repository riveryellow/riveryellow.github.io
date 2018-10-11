---
title: 服务器进程监控
cdn: header-off
date: 2018-10-11 11:12:44
header-img: "/img/monitoring.jpg"
tags:
	- Python
	- Linux
---
> As we all know，诸如HBase、Hadoop、Zookeeper之类的通过启动Java进程来启动功能的服务，在做监控的时候只需要对进程进行是否存在的定时监控即可。下面将以python脚本监控HBase进程为例，展示如何在Linux中监控进程。

## 服务进程
HBase集群中的其中一个节点，同时运行着Master服务和Region服务：
``` shell
[user@host:~]$ sudo jps
6840 HMaster
30317 HRegionServer
```

## Python脚本
那么监控脚本就要对HMaster和HRegionServer进程进行监控。

hbase-monitor.py
``` python
import sys, os, commands, json, requests

# 重启日志
RESTART_LOG = open("/data/logs/hbase/restart.log", "w")

# hbase路径
HBASE_HOME = '/opt/local/hbase-2.0.1'


def alarm(content):
    alarm_data = json.dumps({'contact':'JonhdoA,Janedo,JohndoB,JohndoC', 'values':{'title': 'Music Hbase Service - %s Alarm!!!!' % hostname, 'content': content}})
    
	# Wechat通知
    response = requests.post('http://host/uri/%s' % 'xxx', headers={'Content-Type': 'application/json'}, data=alarm_data, timeout=30)
    
    print('WeiXin Alarm End: %s' % response.content)


if __name__ == '__main__':
    status, hostname = commands.getstatusoutput('hostname')

	# HBase region进程监控
    if 'HRegionServer' == sys.argv[1]:
        status, region_status = commands.getstatusoutput('sudo jps | grep -i "HRegionServer"| wc -l')
        
        if region_status == '0':
            content = '%s is down, try to restart...' % sys.argv[1]
			# 重启进程
            os.system('sudo -u cluster %s/bin/hbase-daemon.sh start regionserver' % HBASE_HOME)
            alarm(content)
            print >> RESTART_LOG, content
        else:
            print('%s is running' % sys.argv[1])
            
    elif 'HMaster' == sys.argv[1]:
        status, master_status = commands.getstatusoutput('sudo jps | grep -i "HMaster"| wc -l')
        if master_status == '0':
            content = '%s is down, try to restart...' % sys.argv[1]
            os.system('sudo -u cluster $HBASE_HOME/bin/hbase-daemon.sh start master')
            alarm(content)
            print >> RESTART_LOG, content
        else:
            print('%s is running' % sys.argv[1])
            
    else:
        print('No process:%s' % sys.argv[1])
```

## 添加系统定时任务
在 /etc/cron.d 下创建定时任务：
``` shell
# 每秒执行一次
*/1 * * * * root /data/script/hbase-monitor.py HMaster
```
