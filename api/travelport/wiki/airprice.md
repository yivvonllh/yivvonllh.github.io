# Request

# **TypeInventoryRequest\(機票庫存查詢類型\)**

Seamless：将在预订尝试之前轮询参与1P无缝库存的运营商以确认库存。

DirectAccess：将在预订尝试之前轮询参与直接访问库存和Worldspan（1P）无缝库存的运营商以确认库存。

Basic：从本地可用性来源获取数据。最接近于不检查1P中的可用性。如果在Galileo（1G）或Apollo（1V）中选择了Basic，则不会检查库存。

# TypeEticketability\(電子機票類型\)

Yes：可以獲取通過電子票務發送的行程。

NO：行程不能用電子機票出票，僅返回具有Paper票證的行程。

Required：僅返回可以通過電子票務發送的行程。

Ticketless：票務不適用於此運營商/解決方案。

## FaresIndicator\(票價制定返回類型\)

PublicFaresOnly

PrivateFaresOnly

AgencyPrivateFaresOnly

AirlinePrivateFaresOnly

PublicAndPrivateFares

NetFaresOnly

AllFares

---

# Response

## AirPricingSolution

* ### AirPricingInfo
* * #### PricingMethod

  票價方式，對應傳參有以下種類：

Auto , Manual , ManualFare , Guaranteed , Invalid , Restored , Ticketed , Unticketable , Reprice , Expired ,   AutoUsingPrivateFare , GuaranteedUsingAirlinePrivateFare , Airline , AgentAssisted , VerifyPrice , AltSegmentRemovedReprice , AuxiliarySegmentRemovedReprice , DuplicateSegmentRemovedReprice , Unknown , GuaranteedUsingAgencyPrivateFare , AutoRapidReprice

* * #### PricingType

定價類型：TicketRecord，StoredFare，PricingInstruction。由request入參AirPricingModifiers.FaresIndicator指定返回對應類型。

如果PNR中只有一个自动定价票价，则自动定价票价将作为PricingType=“StoredFare”返回。由于如果将另一个存储票价添加到PNR，Worldspan和Axess将覆盖现有的存储票价，如果它们位于不同的定价组中，则必须将多个票价作为票证记录返回。

对于具有多种票价的请求：

如果每个PricingInfo实例的AirPricingInfoGroup值不同，则返回PricingType=“TicketRecord”。

如果每个PricingInfo实例的AirPricingInfoGroup值相同，则返回PricingType=“Stored Fare”。Universal API为PNR添加了一个票价，它将所有AirPricingInfo值组合在一起。

* * #### FareInfo
* * * PrivateFare

只有私有運價才會出現該參數，對應的類型：UnknownType , PrivateFare , AgencyPrivateFare , AirlinePrivateFare

```
<ns2:FareInfo Key="w5h8AU7Q2BKAvXryBAAAAA==" FareBasis="H13GBOL" PassengerTypeCode="ADT" Origin="LHR" Destination="HKG" EffectiveDate="2018-08-07T16:54:00.000+08:00" DepartureDate="2018-09-07" Amount="HKD5580" 
        PrivateFare="AirlinePrivateFare" NegotiatedFare="false" TourCode="POO" NotValidBefore="2018-09-08" NotValidAfter="2018-09-08" PseudoCityCode="7X6F">
    <ns2:FareSurcharge Key="w5h8AU7Q2BKAKYryBAAAAA==" Type="Other" Amount="NUC39.71"/>
    <ns2:BaggageAllowance>
        <ns2:MaxWeight Value="30" Unit="Kilograms"/>
    </ns2:BaggageAllowance>
    <ns2:FareRuleKey FareInfoRef="w5h8AU7Q2BKAvXryBAAAAA==" ProviderCode="1G">gws-eJxNT8sSgjAM/Bhm72lTCL0hoFRliu+xHvz/zzAFdEwnbaab7G6aprFkaqpJmr8ogKQH4TiMe0SYnMzEzgHyrHbQDzLawu02IBfairjpbw/2ZIb2NBEZ0jDeC3krOu10ZCaBHF6MDKK4nhHvXeazmpmlkpJklkwgrksEw0M7jZidesWSYvF3Z3WiTjqrD3rpqwXKgRRjihjDRTExTupVFm/dMfsj4FvoJph3WOw766zndgVVVq1+ALiWQgc=</ns2:FareRuleKey>
    <ns2:Brand Key="w5h8AU7Q2BKATeryBAAAAA==" BrandID="115871" BrandTier="0002"/>
</ns2:FareInfo>
```



