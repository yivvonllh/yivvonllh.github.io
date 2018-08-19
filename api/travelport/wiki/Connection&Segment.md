# Connection Segment Logic

---

如果在Low Fare Shopping/Air Availability的響應中返回了Connection/SegmentIndex，它表示兩個或多個航段之間的連接（也就是中轉）。在Pricing和Booking時，必須使用連接指示符（中轉標誌）以確保正確的銷售航班，並且不會銷售失敗。

連接邏輯適用於Air Pricing和Air Booking。連接段索引信息返回到Low Fare Shopping/Air Availability響應作為一組數據。

```
  <air:Connection SegmentIndex="0">
  <air:Connection SegmentIndex="1">
  <air:Connection SegmentIndex="3">
  <air:Connection SegmentIndex="4">
```

SegmentIndex顯示了連接指示符必須存在的航段。SegmentIndex的值等於航段列表中的AirSegment的位置，这是一个航空定價解決方案的一部分。例如，使用Low Fare Shopping返回上述航段列表：

```
  SegmentIndex="0" 將連接到列表中的第1個位置的航段。
  SegmentIndex="1" 將連接到列表中的第2個位置的航段。
  SegmentIndex="3" 將連接到列表中的第4個位置的航段。
  SegmentIndex="4" 將連接到列表中的第5個位置的航段。
```

注意：只有顯示&lt;air:Connection SegmentIndex="\#"&gt;的元素才是連接。如果作為連接元素的一部分存在中途停留規範，它不認為是一種連接，而是旅程的終點。這意味著類似於&lt;air:Connection StopOver="true" SegmentIndex="3"/&gt;這樣的元素不是連接。

#### 示例

* ##### 一個**Air Availability request**可能返回一個帶有連接指示符（中轉標誌）的航班。

下面這個例子中，AirItinerarySolution需要4個連接指示器，也就是4個中轉。

請注意AirItinerarySolution中的Connection / SegmentIndex指示符。AirSegmentRef的key可用於查找確切的航班。

![](/assets/1.png)

* ##### 一個**Low Fare Shopping request可能會返回需要連接指示器的航班。**

下面這個例子中，AirPricingSolution的4個航班需要連接指示符。

請注意AirPricingSolution中的Connection / SegmentIndex指示符。

![](/assets/2.png)、

* ##### Low Fare Shopping或Air Availability響應中的Connection / SegmentIndex必須在Air Pricing請求中的響應Air Segments中作為Connection發送。即，元素&lt;Connection /&gt;必須制定為它所屬的AirSegment的一部分。

下面這個例子中，請注意要根據Low Fare Shopping響應中的SegmentIndex加1來防止連接指示符。

請記住，在Low Fare Shopping的響應示例中，SegmentIndex的指標為0,1,3,4，這意味著在Air Pricing的請求中，AirSegment的1,2,4,5需要連接指示符。

![](/assets/3.png)

* ##### Low Fare Shopping或Air Availability響應中的Connection / SegmentIndex必須在Air Booking請求中的響應Air Segments中發送。

下面這個例子中，請注意要根據Low Fare Shopping響應中的SegmentIndex加1來防止連接指示符。

請記住，在Low Fare Shopping的響應示例中，SegmentIndex的指標為0,1,3,4，這意味著在Air Pricing的請求中，AirSegment的1,2,4,5需要連接指示符。

![](/assets/4.png)

### NODE：Air Availability響應中的連接指示符用於Air Pricing和Air Booking，因為它來自Low Fare Shopping的響應。



