# 对接保险API踩到的坑

1、文档不可靠，许多参数不是required却写着required，可以参考示例的数据，或者通过接口返回查看哪些字段是必须的

2、对于空的字段并不允许为 {"xx":null} 否则接口会报 json format 错误，所以空字段不能参与序列化

3、purchase接口也需要channel字段但文档描述、以及示例中完全没有提到

4、getPolicy接口email是必须的，所以在purchase mainContact一定要带email，并且getPolicy接口有一定延迟，在单元测试中休眠30秒依然报NO_DATA，最后直接改为1分钟

5、项目中RestTemplate使用代理quote接口都报校验错误，但是复制请求body到postman是可以的，现在原因未知

6、改用Unirest使用代理解决问题5

7、editValidate接口purchasedOffers需要email字段，文档中没写

8、cancel接口purchasedOffers需要productType字段，文档中没写

9、修改主联系人email后，通过旧email、purchaseOfferId还是可以获取以前的记录

10、请参考跑通的单元测试，需要添加其他参数请先测试
