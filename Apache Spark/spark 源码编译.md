# spark源码编译

下面以编译spark-3.2为例：

## 编译整个工程

1. 官网下载源码；
2. 按官网指导，修改maven内存大小： `export MAVEN_OPTS="-Xss64m -Xmx2g -XX:ReservedCodeCacheSize=1g"`
3. 根据需要选择编译时需要的组件： `./dev/make-distribution.sh --name ldq-spark --tgz -Phive -Phive-thriftserver -Pyarn -DskipTests`
4. 编译完成后会在当前目录下生成 spark-3.2.0-bin-ldq-spark.tgz压缩文件；

## 编译某一个模块

比如，编译spark-sql： `./build/mvn -pl :spark-streaming_2.12 clean install`
