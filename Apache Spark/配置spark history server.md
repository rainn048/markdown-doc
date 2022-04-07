# 配置spark history server

配置spark history server（不是cdh的history server，那个一直没有配置成功，可能是收费的），我们在调试spark程序的时候可以很方便的看到程序运行的全部过程，大家都配置一下。

配置spark history server的步骤如下：

1.修改conf/spark-env.sh文件，配置日志的存储目录，目录必须存在。
export SPARK_HISTORY_OPTS="-Dspark.history.ui.port=18080 -Dspark.history.retainedApplications=30 -Dspark.history.fs.logDirectory=hdfs://host01.byzoro.com:8020/user/root/history/"

2.修改conf/spark-defaults.conf文件，spark.eventLog.dir和1一样
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://host01.byzoro.com:8020/user/root/history/
spark.eventLog.compress          true

3.执行sbin/start-history-server.sh，开启服务

4.使用bin/spark-submit重新提交任务

4.使用浏览器访问开启服务那台主机的18080端口号，可以查看运行中和已经结束的application日志

如果看不到execctor的stdout和stderr，则需要增加以下配置：（日志有聚合）
1.yarn-site.xml中增加

```xml
<property> 
<name>yarn.log.server.url</name> 
<value>http://hostname:19888/jobhistory/logs/</value> 
</property>
2.mapred-site.xml中增加
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>hostname:18016</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>hostname:19888</value>
    </property> 
```

3.然后启动
mr-jobhistory-daemon.sh start historyserver

注意，spark log的存放路径如果是本地磁盘，history可能看不全lob信息，放到hdfs就可以
