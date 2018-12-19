---
title: HBase集群常用告警脚本
cdn: header-off
date: 2018-12-18 18:07:51
header-img: "/img/monitoring.jpg"
tags:
    - HBase
    - Hadoop
    - Python
---
## 进程监控
进程监控是最直接能够发现系统是否正常运行的方式。集群中，由于分工的不同，每台机器上运行的进程不同，分别监控各个必须的进程，在发现挂了之后立即执行命令重启，避免系统服务down机后不可用。
### 服务进程
HBase集群中的其中一个节点，同时运行着Master服务和Region服务：
``` shell
[yellowriver@localhost:~]$ sudo jps
6840 HMaster
30317 HRegionServer
```
### Python脚本
hbase-monitor.py
```python
#!/usr/bin/python2

# -*- coding: UTF-8 -*-

'''
HBase Monitor
'''

import sys, os, commands, json, requests, time

RESTART_LOG = '/data/logs/hbase/restart.log'

HBASE_HOME = '/opt/local/hbase-2.0.1'

HADOOP_HOME = '/opt/local/hadoop-2.7.6'


def alarm(content):
    ## alarm


def logger(message):
    output = open(RESTART_LOG, 'a+')

    output.write(message + '\n')
    
    output.close()


if __name__ == '__main__':
    status, hostname = commands.getstatusoutput('hostname')

    if 'HRegionServer' == sys.argv[1]:
        status, region_status = commands.getstatusoutput('/usr/bin/jps | grep -i "HRegionServer"| wc -l')
        
        if cmp(region_status, "0") == 0:
            content = '%s %s[%s] is down, try to restart...' % (time.asctime(time.localtime(time.time())), sys.argv[1], hostname)
            
            os.system("su cluster -c '%s/bin/hbase-daemon.sh start regionserver'" % HBASE_HOME)
            
            alarm(content)
            
            logger(content)
        else:
            logger('%s %s is running: %s' % (time.asctime(time.localtime(time.time())), sys.argv[1], region_status))
            
    elif 'HMaster' == sys.argv[1]:
        status, master_status = commands.getstatusoutput('/usr/bin/jps | grep -i "HMaster"| wc -l')

        if cmp(master_status, "0") == 0:
            content = '%s %s[%s] is down, try to restart...' % (time.asctime(time.localtime(time.time())), sys.argv[1], hostname)
            
            os.system("su cluster -c '%s/bin/hbase-daemon.sh start master'" % HBASE_HOME)
            
            alarm(content)
            
            logger(content)
        else:
            logger('%s %s is running: %s' % (time.asctime(time.localtime(time.time())), sys.argv[1], master_status))

    elif 'JournalNode' == sys.argv[1]:
        status, master_status = commands.getstatusoutput('/usr/bin/jps | grep -i "JournalNode"| wc -l')

        if cmp(master_status, "0") == 0:
            content = '%s %s[%s] is down, try to restart...' % (time.asctime(time.localtime(time.time())), sys.argv[1], hostname)
            
            os.system("su cluster -c '%s/sbin/hadoop-daemon.sh start journalnode'" % HADOOP_HOME)
            
            alarm(content)
            
            logger(content)
        else:
            logger('%s %s is running: %s' % (time.asctime(time.localtime(time.time())), sys.argv[1], master_status))

    elif 'DataNode' == sys.argv[1]:

        status, master_status = commands.getstatusoutput('/usr/bin/jps | grep -i "DataNode"| wc -l')

        if cmp(master_status, "0") == 0:
            content = '%s %s[%s] is down, try to restart...' % (time.asctime(time.localtime(time.time())), sys.argv[1], hostname)
            
            os.system("su cluster -c '%s/sbin/hadoop-daemon.sh start datanode'" % HADOOP_HOME)
            
            alarm(content)
            
            logger(content)
        else:
            logger('%s %s is running: %s' % (time.asctime(time.localtime(time.time())), sys.argv[1], master_status))

    elif 'NameNode' == sys.argv[1]:

        status, master_status = commands.getstatusoutput('/usr/bin/jps | grep -i "NameNode"| wc -l')

        if cmp(master_status, "0") == 0:
            print(sys.argv[1])
            content = '%s %s[%s] is down, try to restart...' % (time.asctime(time.localtime(time.time())), sys.argv[1], hostname)
            
            print(content)

            os.system("su cluster -c '%s/sbin/hadoop-daemon.sh start namenode'" % HADOOP_HOME)
            
            alarm(content)
            
            logger(content)
        else:
            logger('%s %s is running: %s' % (time.asctime(time.localtime(time.time())), sys.argv[1], master_status))

    elif 'DFSZKFailoverController' == sys.argv[1]:

        status, master_status = commands.getstatusoutput('/usr/bin/jps | grep -i "DFSZKFailoverController"| wc -l')

        if cmp(master_status, "0") == 0:
            content = '%s %s[%s] is down, try to restart...' % (time.asctime(time.localtime(time.time())), sys.argv[1], hostname)
            
            os.system("su cluster -c '%s/sbin/hadoop-daemon.sh start zkfc'" % HADOOP_HOME)
            
            alarm(content)
            
            logger(content)
        else:
            logger('%s %s is running: %s' % (time.asctime(time.localtime(time.time())), sys.argv[1], master_status))
            
    else:
        print('No process:%s' % sys.argv[1])

```

## hdfs坏块监控
Hadoop提供fsck工具来检查HDFS中文件的健康状况。该工具会在查找那些在所有datanode中均缺失的块以及过少或过多副本的块。
```shell
[yellowriver@localhost:hadoop-2.7.6]$ sudo -u cluster  bin/hdfs -h    
Usage: hdfs [--config confdir] [--loglevel loglevel] COMMAND
       where COMMAND is one of:
  dfs                  run a filesystem command on the file systems supported in Hadoop.
  classpath            prints the classpath
  namenode -format     format the DFS filesystem
  secondarynamenode    run the DFS secondary namenode
  namenode             run the DFS namenode
  journalnode          run the DFS journalnode
  zkfc                 run the ZK Failover Controller daemon
  datanode             run a DFS datanode
  dfsadmin             run a DFS admin client
  haadmin              run a DFS HA admin client
  fsck                 run a DFS filesystem checking utility
  balancer             run a cluster balancing utility
  jmxget               get JMX exported values from NameNode or DataNode.
  mover                run a utility to move block replicas across
                       storage types
  oiv                  apply the offline fsimage viewer to an fsimage
  oiv_legacy           apply the offline fsimage viewer to an legacy fsimage
  oev                  apply the offline edits viewer to an edits file
  fetchdt              fetch a delegation token from the NameNode
  getconf              get config values from configuration
  groups               get the groups which users belong to
  snapshotDiff         diff two snapshots of a directory or diff the
                       current directory contents with a snapshot
  lsSnapshottableDir   list all snapshottable dirs owned by the current user
                                                Use -help to see options
  portmap              run a portmap service
  nfs3                 run an NFS version 3 gateway
  cacheadmin           configure the HDFS cache
  crypto               configure HDFS encryption zones
  storagepolicies      list/get/set block storage policies
  version              print the version

Most commands print help when invoked w/o parameters.
```

``` shell
[yellowriver@localhost:hadoop-2.7.6]$ sudo bin/hdfs fsck /
Connecting to namenode via http://localhost:50070/fsck?ugi=cluster&path=%2F
FSCK started by cluster (auth:SIMPLE) from /127.0.0.1 for path / at Wed Dec 19 12:02:32 CST 2018
....................................................................................................
....................................................................................................
....................................................................................................
.................................................................................Status: HEALTHY
 Total size:    44626523629 B
 Total dirs:    273
 Total files:   381
 Total symlinks:                0 (Files currently being written: 5)
 Total blocks (validated):      652 (avg. block size 68445588 B) (Total open file blocks (not validated): 4)
 Minimally replicated blocks:   652 (100.0 %)
 Over-replicated blocks:        0 (0.0 %)
 Under-replicated blocks:       0 (0.0 %)
 Mis-replicated blocks:         0 (0.0 %)
 Default replication factor:    3
 Average block replication:     3.0
 Corrupt blocks:                0
 Missing replicas:              0 (0.0 %)
 Number of data-nodes:          3
 Number of racks:               1
FSCK ended at Wed Dec 19 12:02:32 CST 2018 in 12 milliseconds


The filesystem under path '/' is HEALTHY
```
上述命令的响应的点，为每检查一个文件时打下的。

### 坏块、丢失块处理办法
在健康检查中，Corrupt blocks和Missing replicas是最需要考虑的，因为这意味着数据已经丢失了。默认情况下，fsck不会对这类块进行任何操作，但可以手动让fsck执行如下操作：
+ 移动 使用 -move 选项将受影响的文件移到 HDFS 的 /lost+found 目录。这些受影响的文件会分裂成连续的块链表，可以帮助用户挽回损失。
+ 删除 使用 -delete 选项删除受影响的文件。（删除之后文件不可恢复）

### Python脚本
``` python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

# monitor hdfs corrupt

import os, re, sys, json, requests

HADOOP_HOME = '/opt/local/hadoop-2.7.6'

reload(sys)
sys.setdefaultencoding('utf-8')


def alarm(content):
    ## alarm

if __name__ == "__main__":
    corruptlist = []
    cmd = "su cluster -c 'hadoop fsck -list-corruptfileblocks'"
    re = os.popen(cmd)
    result = re.readlines()
    print(result)
    for line in result:
        if "blk_" in line and ".Trash" not in line:
        #if "blk_" in line:
            corruptlist.append(line)
    if len(corruptlist) != 0:
        content = '坏块数量 %s，具体信息如下：\n' % str(len(corruptlist))
        id = 1
        for clist in corruptlist:
            print "blkid is " + clist.split()[0] + " file is " + clist.split()[1]
            content += "%s. 序号[%s] 块号[%s] 文件信息[%s]. \n" % (id, clist.split()[0], clist.split()[1])
            id = id + 1
        alarm(content)

```


## 添加系统定时任务
在 /etc/cron.d 下创建定时任务：
``` shell
# 每秒执行一次
*/1 * * * * root /usr/bin/python2 /data/script/hbase-monitor.py HMaster
```

## Tips
若/usr/bin下没有响应的命令，如Python2、jps等，可以通过手动增加软链接的方式把命令加到/usr/bin下，例如：
``` shell
[yellowriver@localhost:~]$ /usr/bin/jps
-bash: /usr/bin/jps: No such file or directory
[yellowriver@localhost:~]$ which jps
/opt/local/java/jdk1.8.0_181/bin/jps
[yellowriver@localhost:~]$ sudo ln -s /opt/local/java/jdk1.8.0_181/bin/jps /usr/bin/jps
[yellowriver@localhost:~]$ /usr/bin/jps
2139 Jps
```