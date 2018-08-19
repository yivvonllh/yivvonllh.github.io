# GIT提交规范

格式：**类别#ID 分支 操作 desc:描述**

* 类别：task,story,bug, issues。task:指禅道的任务，story:禅道的需求，bug:禅道的bug ，issues：指gitlab的Issues 。
* ID对应禅道或者Gitlab的ID
* 分支：具体的分支，eg: Feature_Flight_1.0
* 操作：[DEV] [BUG] [REF] 。DEV对应功能开发，BUG对应BUG修复，REF对应重构。
Eg:
```
task#4803 [V1] [DEV|REF] desc:repository修復 & Refactor
task#4803 [V1] [DEV|REF] desc:invitee as coupon's entity.
bug#17770 [V3.0-WingedCat] [BUG] desc：黑名單更新的問題。
bug#17379 [MKT_v3.0-WingedCat] [DEV] desc:查詢優惠券模板，不包含已刪除模板
```
* 描述：突出代码的修改

结果：
![](/assets/s2.png)