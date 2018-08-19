# flightVerify

## 1.1 入参

| 名称 | 类型 | 是否必填 | 描述 |
| ---: | :--- | :---: | :--- |
| searchTraceId | String | N | LFS搜索的标志。 |
| includeSeat | Boolean | Y | 是否验仓 |
| passengers | List&lt;Passenger&gt; | Y | 乘客 Ref |
| optionKeys | List&lt;String&gt; | Y | 单程：一个；双程：二个；MultiStop：按照数量传。 |

## 1.2 出参

| 名称 | 类型 | 是否必填 | 描述 |
| ---: | :--- | :---: | :--- |
| verifyTraceId | String | Y | 验证追踪Id |

## 1.3 API逻辑

* 调用AirPricing的接口；
* 0：表示成功



