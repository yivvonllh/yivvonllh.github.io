#### 1、Linux安裝maven

1、如果電腦沒有wget命令的，先使用yum安裝wget命令。
eg:
```java
    yum install wget
```
2、安裝好后就可以直接使用wget命令去下載maven。

附：打开maven官网下载linux使用的版本。
http://mirror.bit.edu.cn/apache/maven/maven-3/
![](/assets/maven.jpg)
选择想要安装的版本。
下面用maven-3.5.4举例
可以再usr下建一个maven 的文件
```java
    cd usr/
    mkdir maven
    wget http://mirror.bit.edu.cn/apache/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
```
3、然后将下载好的tar包解压
```java
    tar -zxvf file.tar.gz
```
4、解压后就可以去配置maven
```java
    cd etc/profile
    vim profile
```
5、打开编辑之后，i编辑，再最后加上这些配置
```java
    export MAVEN_HOME=/usr/maven/apache-maven-3.5.4
    export MAVEN_HOME
    export PATH=$PATH:$MAVEN_HOME/bin
```
:wq记得保存后退出

6、配置好之后，让配置立即生效：
```java
    source /etc/profile
```
8、查看是否安装成功：
```java
    mvn -v
```
