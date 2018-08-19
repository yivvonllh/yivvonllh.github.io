# flightSerach

搜索机票RQ API。

### 1.1 入参

| 名称 | 类型 | 是否必填 | 描述 |
| ---: | :--- | :---: | :--- |
| originCode | String | Y | 出发地 AirCode Or CityCode |
| destinationCode | String | Y | 目的地 AirCode Or CityCOde |
| outboundDate | String | Y | 去程时间 |
| inboundDate | String | N | 回程时间，如果此字段为空，则表示单程。 |
| cabinType | Enum | Y | 仓等类型 |
| permittedCarriers | List&lt;String&gt; | N | 指定搜索的航司 |
| prohibitedCarriers | List&lt;String&gt; | N | 指定排除的航司 |
| directOnly | Boolean | N | 是否直飞 |
| passengers | List&lt;Passenger&gt; | Y | 乘客列表 |
|  |  |  |  |
| Passenger对象 |  |  |  |
| passengerType | String | Y | 乘客类型 |
| age | Int | Y | 年龄 |

### 1.2 出参

| 名称 | 类型 | 是否必填 | 描述 |
| ---: | :--- | :---: | :--- |
| requestKey | String | Y | 如果轮询时，可以这么使用。 |
| airPricePoints | List&lt;AirPricePoint&gt; | N | 价格响应 |
|  |  |  |  |
| AirPricePoint对象 |  |  |  |
| outBoundOptions | List&lt;Option&gt; |  |  |
| inBoundOptions | List&lt;Option&gt; |  |  |
|  |  |  |  |
| Option对象 |  |  |  |
|  |  |  |  |

### 1.3 API逻辑

* 根据RQ生成一个KEY，根据此KEY从Redis中查找对应的数据；
* 如果有，则直接返回0；
* 如果没有，则返回1；
  * 发起一个线程，call travelport-ms；
  * 将查询结果写入REDIS；
* 如果Error，则返回其他状态；



