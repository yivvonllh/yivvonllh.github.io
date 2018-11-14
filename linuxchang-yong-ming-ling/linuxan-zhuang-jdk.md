#### 1、Linux安裝jdk

如果電腦沒有wget命令的，先使用yum安裝wget命令。

eg:
```java
    yum install wget
```
安裝好后就可以直接使用wget命令去下載jdk。

附：打開官網連接：https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

![](/assets/jdk1.8.jpg)

選擇要下載的版本，然後按F12打開調試器，點擊下載獲得真正的下載連接。
（如本地不需要點擊取消下載即可）
![](/assets/jdk1.8獲取連接.jpg)

复制这条链接，打开linux，可以再usr下新建一个存放jdk的文件
```java
    cd usr/
    mkdir java
    cd java/
    wget http://download.oracle.com/otn-pub/java/jdk/8u191-b12/2787e4a523244c269598db4e85c51e0c/jdk-8u191-linux-i586.tar.gz?AuthParam=1542174960_8617e406ce32466dce98214285e04b55
```
然后就可以看到有了一个jdk1.8的tar包。这时候需要使用tar命令去解压

```java
    tar -xzvf file.tar.gz
```
