#### 1、清除缓存

```java
    echo 1 > /proc/sys/vm/drop_caches
    echo 2 > /proc/sys/vm/drop_caches
    echo 3 > /proc/sys/vm/drop_caches
```
#### 2、显示进程
    jps
    
#### 3、 显示内存
    free -h

#### 4、 Banner
    1 sudo apt-get update
    2 sudo apt-get install sysvbanner
    3 banner LOVE
    4 printerbanner -w 50 A
    
