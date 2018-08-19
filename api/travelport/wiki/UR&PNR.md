# Universal Recordsand PNRs

---

Universal API創建并從多個提供商和供應商處獲取預定數據。當一個預定被Universal API創建時，一個PNR（Passenger Name Record），也稱為預定文件，和UR（Universal Record）就被創建了。

## Passenger Name Records \(PNR\)

PNR也稱為預定文件，是來自特定供應商的唯一標識符，用作特定旅行的記錄。PNR包括一個PNR的記錄定位符，旅行供應商的信息，特定段的旅行者信息，以及選定或保留的行程信息。

在Universal API中，PNR有一些特別的請求不一定能在相同的提供商和供應商的預定文件中找到。對於Universal API,每個PNR可以包含以下類型的旅行段的任意組合：

* 一個或多個air segments.

* 一個或多個rail segments.

* 一個或多個hotel segments.

* 一個或多個vehicle segments.

* 一個或多個passive segments.

在一個Universal API請求中不能預定多個hotel segments或者vehicle segments。如果需要多個hotel segments或者vehicle segments，客戶可以選擇：

* 把這個segment加入現有的UR和PNR中。
* 創建一個新的PNR並且把它加到一個現有的UR中。

## Universal Records \(UR\)

Universal API服務提供對來自多個數據源（提供商）的數據訪問。Universal API生成一個名為Universal Record Locator（UR）的預定文件記錄\(UniversalRecordLocatorCode\)。

UR識別旅行者的整個預定文件，包括一般旅行者信息和旅遊供應商信息，以及通過Universal API提供的航空，汽車，酒店，鐵路或者其他旅行段的任何PNR數據。這個預定記錄是特定於Universal API的，並且可以包含來自一個或多個提供預定數據的供應商的PNR。

個別旅行段還可能包含來自供應商的額外記錄定位符和供應商（航空，汽車和酒店供應商）的確認碼。

## Example of aUniversal Recordwith Multiple PNRs

一個UR可以包含簡單的數據，比如一個PNR，一個旅行段到多個多個PNR，其中包含旅行段和非旅行段信息的數據。

下面的例子展示了如何在Universal API中管理UR和相關的PNR。這個例子展示了一個可能的端到端流程，用於在旅途中創建和添加數據，在大多數情況下，有多種方法可以添加或修改UR和PNR數據。

### _Transaction 1_

客戶端預定了Montreal和Vancouver的往返航班。出境航班通過Galileo GDS在A航空公司預定，入境航班通過ACH在B航空公司預定。這個預定是通過AirCreateReservationReq發起的，用於處理一個聚合的1G/ACH請求。

在響應中，返回了UR **123456**。這個UR是自動生成的，并包含來自兩個PNR的數據，這些數據是由各自的主機系統生成的：

* Galileo **ABCDEF**，存儲出境航班數據。

* ACH **UVWXYZ**，存儲入境航班數據。

![](/assets/5.png)

### _Transaction 2_

在搜索和選擇酒店之後，客戶端就通過Galileo使用UniversalRecordModifyReq / UniversalModifyCmd / HotelAdd來增加一個對酒店預定的請求。通過在請求中指定U R**123456**和Galileo PNR **ABCDEF**，酒店預定將被添加到現有的UR **123456**中的Galileo PNR **ABCDEF**中。

響應返回的是包含酒店預定的PNR **ABCDEF** 修改的UR。

![](/assets/6.png)

### _Transaction 3_

在搜索和選擇一輛汽車后，客戶端就通過Galileo使用VehicleCreateReservationReq來增加一個對汽車預定的請求。按照設定，此次交易產生了一個新的PNR，VehicleCreateReservationReq不能用來對現有的PNR進行修改。

通過在請求中指定UR **123456**，汽車預定就作為第二個Galileo PNR **GHIJKL**被添加到現有的UR **123456**中。

**注意**：車輛預定可以使用UniversalRecordModifyReq/UniversalModifyCmd/VehicleAdd來交替的添加到最初的Galileo PNR **ABCDEF**中。

這個請求返回了一個添加了新的Galileo PNR **GHIJKL**的UR。

![](/assets/7.png)

### _Transaction 4_

PNR還可用於存儲不受Universal API支持的提供商和供應商預定的被動數據。在下面的示例中，一個SDK segment用於通過航空供應商D為Vancouver本地航班預定添加數據。

由於Universal API提供商不支持此運營商。

* 不能通過Universal API來創建預定。不過，預定的數據可以通過使用通用的“SDK”提供商進行AirCreateReservationReq請求調用被分配到Air Segment然後存儲在Universal API中。
* 存儲在PNR中的數據無效，不能通過Universal API修改或取消。要修改或取消預定，必須聯繫外幣供應商。隨後對預定數據的任何修改都必須在Universal API中手動修改。

**注意：**如果一個PNR是由一個Universal API支持的供應商在Universal API之外創建的，那麼PNR可以被導入到Universal API中。

該響應返回了一個添加了SDK PNR **MNOPQR**的UR。

![](/assets/8.png)

### _Transaction 5_

Passive segment（無源段）也用來包含與旅行段沒有直接關係的手動輸入的輔助數據。比如，可以創建一個PNR來存儲旅行者後台使用所需的數據。在這個例子中，PassiveCreateReservationReq用來創建一個無源段，其中包含會計備註，發票信息或其他可以發送到旅行供應商的第三方數據庫或應用程序的數據。

該響應返回了一個添加了Passive PNR **STUVWX**的UR。

![](/assets/9.png)

