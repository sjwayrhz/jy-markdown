prometheus将采集到的样本以时间序列的方式保存在内存（TSDB 时序数据库）中，并定时保存到硬盘中。与zabbix不同，zabbix会保存所有的数据，而prometheus本地存储会保存15天，超过15天以上的数据将会被删除，若要永久存储数据，有两种方式，方式一：修改prometheus的配置参数“storage.tsdb.retention.time=10000d”；方式二：将数据引入存储到Influcdb中。为保证数据安全性，本文主要介绍的是promethues本地存储备份数据的方法。

------

## 一、存储原理

　　prometheus 提供了本地存储（TSDB）时序型数据库的存储方式，在2.0版本之后，压缩数据的能力得到了大大的提升，单节点情况下可以满足大部分用户的需求，但本地存储阻碍了prometheus集群化的实现，因此在集群中应当采用 其他时序性数据来替代，比如influxdb。
　　prometheus 分为三个部分，分别是：**抓取数据**、**存储数据**和**查询数据**。

　　prometheus按照block块的方式来存储数据，每2小时为一个时间单位，首先会存储到内存中，当到达2小时后，会自动写入磁盘中。block的目录结构如下：　　

```
chunks     　　　　　多个，是个目录、保存timeseries数据``meta.json      配置文件，包含起止时间、包含哪些block``index      　　　　 通过metric名和labels查找时序数据在chunk文件中的位置``tombstones 　　　　 删除操作会首先记录到这个文件
```

　　为防止程序异常而导致数据丢失，采用了WAL机制，即2小时内记录的数据存储在内存中的同时，还会记录一份日志，存储在block下的wal目录中。当程序再次启动时，会将wal目录中的数据写入对应的block中，从而达到恢复数据的效果。

 　当删除数据时，删除条目会记录在tombstones 中，而不是立刻删除。

   prometheus采用的存储方式称为“**时间分片**”，每个block都是一个独立的数据库。优势是可以提高查询效率，查哪个时间段的数据，只需要打开对应的block即可，无需打开多余数据。

　　**目录结构：**

**[![img](https://img2018.cnblogs.com/i-beta/1489604/202001/1489604-20200117174740635-335192823.png)](https://img2018.cnblogs.com/i-beta/1489604/202001/1489604-20200117174740635-335192823.png)**

[![img](https://img2018.cnblogs.com/i-beta/1489604/202001/1489604-20200117172829196-1635318202.png)](https://img2018.cnblogs.com/i-beta/1489604/202001/1489604-20200117172829196-1635318202.png)

　　prometheus的存储层使用了全文检索中的“**倒排索引**”概念，将每个时间序列视为一个小文档。而metric和label对应的是文档中的单词。

## 二、数据备份

**1、完全备份**

　　备份prometheus的data目录可以达到完全备份的目的，但效率较低。

**2、快照备份**

　　prometheus提供了一个功能，是通过API的方式，快速备份数据。

　　实现方式：

　　首先，修改prometheus的启动参数，新增以下两个参数：

```
--storage.tsdb.path=``/usr/local/share/prometheus/data` `\``--web.``enable``-admin-api    
```

　　重启prometheus

　　调用API

```
curl -XPOST http:``//prometheusIP``:端口``/api/v1/admin/tsdb/snapshot``返回结果： ``{``"status"``:``"success"``,``"data"``:{``"name"``:``"20191220T012427Z-21e0e532e8ca3423"``}}
```

　　此时，数据将快速的备份到 data/snapshots下。

**[![img](https://img2018.cnblogs.com/i-beta/1489604/202001/1489604-20200117170517465-265872622.png)](https://img2018.cnblogs.com/i-beta/1489604/202001/1489604-20200117170517465-265872622.png)**

 　**【注意】**上述API还有一个参数

```
skip_head=<bool>            默认是``false``作用：是否跳过存留在内存中还未写入磁盘中的数据，仍在block块中的数据
```

完整的调用方式为：

```
# 不跳过内存中的数据，即同时备份内存中的数据``curl -XPOST http:``//127``.0.0.1:9090``/api/v2/admin/tsdb/snapshot``?skip_head=``false``# 跳过内存中的数据``curl -XPOST http:``//127``.0.0.1:9090``/api/v2/admin/tsdb/snapshot``?skip_head=``true
```

[![img](https://img2018.cnblogs.com/i-beta/1489604/202001/1489604-20200117170857193-1236593801.png)](https://img2018.cnblogs.com/i-beta/1489604/202001/1489604-20200117170857193-1236593801.png)

##  三、数据还原

　　利用api方式制作成snapshot后，还原时将snapshot中的文件覆盖到data目录下，重启prometheus即可！

　　**添加定时备份任务（**每周日3点备份）

```
crontable -e               ``#注意时区，修改完时区后，需要重启 crontab  systemctl restart cron` `0 3 * * 7 ``sudo` `/usr/bin/curl` `-XPOST -I http:``//127``.0.0.1:9090``/api/v1/admin/tsdb/snapshot` `>> ``/home/bill/prometheusbackup``.log
```

Link

```
https://www.cnblogs.com/zqj-blog/p/12205063.html
```

