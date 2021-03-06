# HDFS：Edits和Fsimage详解与合并流程

## NameNode如何管理和存储元数据

计算机中存储数据有两种：内存或磁盘。
元数据如果存储在磁盘: 磁盘无法面对客户端对元数据信息的任意的快速低延迟的响应，但是安全性高；元数据如果存储在内存：可以高效的查询以及快速响应客户端的查询请求，但是如果断电，内存中的数据全部丢失。

因此，考虑上述两种存储方式的优缺点，HDFS采用了内存+磁盘的形式来管理元数据。即: NameNode(内存)+FsImage文件。其中，NameNode维护了文件与数据块的映射表以及数据块与数据节点的映射表，而Fsimage保存在磁盘上，为某一时刻下内存中元数据在本地磁盘的映射。同时，为了保证内存和磁盘中元数据的一致，hdfs采用了一个edits的日志文件，该文件记录了客户端对元数据的操作。

## FsImage

FsImage: 是namenode中关于元数据的镜像，一般称为检查点(checkpoing)，这里包含了HDFS文件系统所有目录以及文件相关信息（Block数量，副本数量，权限等信息）。

这样，当HDFS异常中断或者程序启动时，就可以利用该检查点文件，来对元数据进行初始化，以此还原到异常中断或程序停止前的最新状态。

## Edits文件

存储了客户端对HDFS文件系统所有的更新操作记录，Client对HDFS文件系统所有的更新操作都会被记录到Edits文件中(不包括查询操作)。

edits文件记录的就是当原FsImage被载入内存后，Client又对元数据进行了哪些操作。这样，只要在原FsImage中执行这些操作，对保存的元数据信息进行更新，就可以使得内存和磁盘汇总的元数据信息一致。通过此方法，即解决了为了保证一致性，要对FsImage直接进行写入的过程。这也是引入edits文件的关键原因。

## Fsimage与Edits的合并

根据上面对Edits中的介绍可知，引入载入的FsImage的元数据内容不是最新状态，所以只有在FsImage的内容上，执行edits文件中的更新操作，才能将FsImage的元数据更新为最新状态。此时，假设edits不断的进行追加写，当某一时刻需要NameNode重启时，此时NameNode会先将FsImage里面的内容映射到内存中，即相当于对元数据进行初始化，同时为了恢复到最新的状态，还需要在一条一条的执行edits中的记录。当edits文件非常大时，会导致NameNode启动过程非常慢，而在这段时间HDFS系统会处于安全模式，即保证了要将元数据恢复到最新后才能接收客户端请求。这显然是不符合用户要求的。因此，就需要设计能不能在NameNode运行的时候使得edits文件小一些，这样就能使启动过程加快。所以就要引入FsImage和Edits的合并过程。

于是，除了NameNode节点外，为了进行合并，还引入了另一个SecondaryNameNode节点。SecondaryNanoe是HDFS架构中的一个组成部分，它是用来保存namenode中对HDFS metadata的信息的备份而设定的。一般都是将SecondaryNamenode单独运行在一台机器上。
