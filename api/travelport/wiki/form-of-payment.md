# Form of Payment\(付款方式\)

---

### 創建預定時添加的付款方式

```
使用_AirCreateReservationReq_，可以在創建預定時將一種付款方式添加到存儲的票價中。在創建時，只有以
下付款方式被添加到PNR和存儲的票價中：  
    · Credit/Debit Card(信用卡/借记卡)
    · Cash(現金)
    · Check(支票)
只有在请求中包含付款且AirPricingInfo具有将付款与预订FormOfPayment相关联的PaymentRef时，才会添加
这些付款方式。在这种情况下，PNR和存储的票价都将包含用于创建预订的FormOfPayment。只有上述付款方式可
以在预订时使用（AirCreateReservation）。
```

### 修改預定并添加的付款方式

```
必須使用UniversalRecordModifyReq/UniversalModifyCmd/AirUpdate/AirPricingPayment添加所有其他付
款方式，否則將返回錯誤“FOPTypeis an unsupported FormOfPayment for host reservation.”其中的
“FOPType”就是FormOfPaymentType。
UniversalRecordModifyReq/UniversalModifyCmd/AirUpdate/AirPricingPayment/FormOfPayment將新的
付款方式添加到存儲的票價中，但不會取代PNR中的付款方式。新的付款方式將添加到AirReservation，并更新相
應的票價。
使用UR修改請求，只能將以下的付款方式添加到存儲的票價中：
    · Credit/Debit Card(信用卡/借记卡)
    · Cash(現金)
    · Check(支票)
    · Requisition(申請單)
    · MiscFormOfPayment(其他付款方式)
    · Agent Non-Refundable(代理商不可退款)
在付款方式和付款方式之間驗證參考密鑰以確保他們有效。如果不是，則返回錯誤“Payment and FormOfPayment
reference key do not match.”
```



