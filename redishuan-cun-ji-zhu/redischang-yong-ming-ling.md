1、ping

2、redis-cli -h 172.18.21.217 -p 7304（連接遠程-a可以加密碼）

3、del a\(刪除鍵，如果成功返回1失敗返回0\)

4、dump a\(序列化給定key\)

5、exists a\(檢查key是否存在，如果存在返回1，不存在返回0\)

6、expire a 2000\(設置過期時間\)persist a\(移除過期時間。key將持久保持\)

7、pttl a\(以毫秒爲單位返回過期時間\)ttl\(以秒爲單位返回過期時間\)

8、randomkey（當前數據庫隨機返回一個key）

9、rename a aa\(重命名key\)

10、type a\(返回這個key的儲存值得類型\)

11、info（获取redis服务器的统计信息）

12、cluster slots\(获取集群节点的映射数组\)示例文件：redis-cloud-slots.txt

13、config get requirepass\(可以修改密码\)

14、http://www.runoob.com/redis/redis-benchmarks.html（性能测试）

15、config get \* 修改配置



