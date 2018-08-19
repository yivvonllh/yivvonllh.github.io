
* #### DOCO SSR

  對於DOCO SSR，如果簽證信息的要求因提供者而異。對於所有的提供者，附加的旅行信息證件類型（例如，簽證）和附加的旅行信息號碼（例如，簽證號碼）必須一起登記或者全部不登記。<br/>                                                                                                               證件類型“R”：旅客護照識別號碼和“K”：已知的旅行者編號可以在DOCO SSR中輸入他們各自的編號。一個護照識別碼或已知的旅行者編號類型總是需要一個補充的履行信息號碼。<br/>                                                                                                                                                                                   在一個DOCO SSR中，一個護照識別碼必須為旅行者提供一個DOCS SSR。 <br/>                                                                                                                             在所有情況下，驗證都是由提供者執行的。<br/>                                                                                                                                                                                                                下表列出了可以在Universal API作為SSR DOCO的FreeText發送的visa（簽證）信息，以及在Galileo和ACH中需要哪些字段。                   

  * 例如：                                                                                                                                                                                                                   SSR Type="DOCO" HK1//V/9891404/LONDON/14MAR07/USA-1SMITH/JOHNMR Carrier="BA"                                                                   /V/指定簽證作為備份文件，倫敦是簽發地，DDMMMYY是簽發時間。                                                                                                                                                       一些供應商對DOCO數據有不同的要求。<br/>
  
| 提供商 | 出生地 | 簽證號碼 | 簽發地點 | 發行日期 | 簽證適用國家 | 乘客姓名 | 截止日期 |
| --- | --- | --- | --- |--- |--- |--- |--- |
|Galileo|No|Yes|Yes|Yes|Yes|No|No|
|ACH|No|No|No|No|No|No|Yes|





