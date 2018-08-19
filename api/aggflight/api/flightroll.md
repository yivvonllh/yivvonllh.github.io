# flightPolling

## 1.1 入参

| 名称 | 类型 | 是否必填 | 描述 |
| ---: | :--- | :---: | :--- |
| requestKey | String | Y | 轮询Redis的Key。 |

## 1.2 出参

| 名称 | 类型 | 是否必填 | 描述 |
| ---: | :--- | :---: | :--- |
| requestKey | String | Y | 轮询Redis的Key。 |
| airPricePoints | List&lt;AirPricePoint&gt; |  |  |

## 1.3 API逻辑

* 根据requestKey从Redis中查询
* 如果有，则直接返回0；
* 如果没有，则返回1;
* 如果Error，返回其他Code。



