#Eclipse连接远程hadoop并运行WordCount

> ####由于在实操过程中需要在本地编写mapreduce程序，然后打包发送到客户端进行运行，这个过程繁琐复杂。以为了高效这一最终目的于是有了这一折腾过程。

###在本地Eclipse环境连接运行在远程的hadoop集群需要以下先决条件：

> ###[hadoop 2.7.3 for eclipse 插件](http://pan.baidu.com/s/1b2tooI)(密码：ea0v)
> ###远程hadoop成功启动并且Linux防火墙已经关闭
> ###本地需要hadoop软件

###首先下载插件jar包，并将hadoop-eclipse-plugin-2.7.3.jar拷贝到eclipse安装目录下的dropins文件下，重启Eclipse。
###找到【Window】->【Preferences】->【Hadoop Map/Reduce】,如下图![配置Hadoop](http://i.imgur.com/6UE7cen.png)

###设置本地hadoop路径（不用配置etc/hadoop里面的配置文件也可）

###显示Map/Reduce窗口 【Window】->【Show View】->【Other】找到MapReduce　Tools,然后选择Map/Reduce Locations.如下图
![](http://i.imgur.com/hU4ehmU.png)
###完成以上操作后你就能在任务视图窗口看见Map/Reduce Locations，如下图
![](http://i.imgur.com/6yR6niW.png)
###右键选择【New Hadoop location】新建连接,
![](http://i.imgur.com/Ef3b7B6.png)
###其中Host为远程或本地hadoop的namenode节点的地址，prot为你在core-site.xml中设置的prot,然后点击Advance parameters配置如下参数
<table>
    <tr>
        <td>参数名</td>
 		<td>值</td>
    </tr>
    <tr>
        <td>Hadoop.tmp.dir</td>
 		<td>core-site.xml里hadoop.tmp.dir设置一致</td>
    </tr>
    <tr>
        <td>Dfs.replication</td>
 		<td>hdfs-site.xml里面的dfs.replication一致</td>
    </tr>
    <tr>
        <td>Dfs.permissions.enabled</td>
 		<td>false</td>
    </tr>
</table>
###最后一步。。。

###点击【Window】->【Perspective】->【Open Perspective】->【Other】,找到Map/Reduce，并选择点击OK，完工，你的Project Explorer视图中出现了DFS Locations。
----
##问题汇总
###Hdfs权限问题导致的各种情况
>###1.eclipse创建目录和提交文件失败(有可能没有任何提示)
>###2.org.apache.hadoop.security.AccessControlException: Permission denied:user=xxxx, access=EXECUTE报错
###解决办法.

>###简单粗暴型:
>###在master执行如下命令
>
	hadoop fs -chmod 777 /user/hadoop 
>###语句的具体目录根据实际情况改变.

>###高枕无忧型:
>###在master的hadoop目录的etc/hadoop/hdfs-site.xml增加如下配置项
>
	<property>
		<name>dfs.permissions</name>
		<value>false</value>
	</property>
>###这样关闭了整个hdfs文件的权限验证功能.

###eclipse插件问题
>###在添加集群之后点'>'出现错误
>###
	An internal error occurred during: "Map/Reduce location status updater".
		java.lang.NullPointerException
>###解决办法
>###1.找到如下文件放到对应目录winutils.exe放到hadoop的bin目录(指开发环境)hadoop.dll放到c:\windows\system32目录下

>###2.创建HADOOP_HOME环境变量,设置为hadoop目录把hadoop/bin的目录加到PATH变量
>>####winutils.exe网上一般能下载到的是2.2.0版本
>>####此版本未必适用于高级版本的hadoop如果要使用高级版本需要编译
>>####本人功力不够ToT能尝试通过在单独写一个编译案例.

>###Tmp目录问题
>
	Exception in thread "main" java.io.IOException: (null) entry in command string: null chmod 0700 C:\tmp\hadoop-dell\mapred\staging\dell521065740\.staging
>###这是由于使用默认的Advance parameters配置，解决方法是添加如下参数
><table>
    <tr>
        <td>参数名</td>
 		<td>值</td>
    </tr>
    <tr>
        <td>Hadoop.tmp.dir</td>
 		<td>core-site.xml里hadoop.tmp.dir设置一致</td>
    </tr>
    <tr>
        <td>Dfs.replication</td>
 		<td>hdfs-site.xml里面的dfs.replication一致</td>
    </tr>
    <tr>
        <td>Dfs.permissions.enabled</td>
 		<td>false</td>
    </tr>
</table>
>###设置位置![](http://i.imgur.com/rziPvee.png)