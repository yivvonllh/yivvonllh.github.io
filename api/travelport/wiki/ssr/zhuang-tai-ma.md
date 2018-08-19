# shopping


* ### SSR Status Codes
SSR有狀態碼來标志它們在執行過程中的狀態。例如：<br/>
SSR Type="DOCS" Status="HK" Carrier="YY" FreeText="P/US/1234567/US/25Jan85/M/23OCT17/Lastname/Firstname"<br/>

|Status Code|Description|
|---|----|
|NN|SSR已經被請求|
|PN|SSR還待確認|
|UN|供應商（承運人）不能滿足要求。在這種情況下，SSR將保留在PNR中作為該讀物被請求和拒絕的證據|
|HK|SSR已經被確認|
|other|承運人可以發送不同的狀態，或者選擇不以HK消息答復SSR，這意味著SSR的狀態將始終保持為“NN”|
* ## BookingTravelerName

