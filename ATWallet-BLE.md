[How To Use Android BLE to Communicate with Bluetooth Devices - An Overview & Code examples](https://medium.com/@avigezerit/bluetooth-low-energy-on-android-22bc7310387a)
## 藍牙 4.0 
```
藍牙 4.0 是集合傳統藍牙、高速藍牙、低速藍牙（BLE）的技術
```
## 藍牙 4.0 BLE 協議簡介
```
藍牙 4.0 BLE 規範中定義了GAP(Generic Access Profile)和GATT(Generic Attribute Profile)兩個基本配置文件
- 協議棧中的GAP層負責設備訪問模式和進程，包括設備發現、建立連接、終止連接、初始化安全特性和設備配置。
- 協議棧中的GATT層用於已連接的藍牙設備之間的數據通訊
```
## 藍牙 4.0 BLE 技術入門
```
基本思路： 從發送端發送一個數據，接受端接收到數據後，校驗收到的數據是否正確，並給出相對應的顯示。
- 數據在協議棧內如何流動
- 如何調用藍牙 4.0 BLE 協議棧提供的發送函數
- 如何使用藍牙 4.0 BLE 協議棧進行數據的接收
- 如何理解藍牙 4.0 BLE 協議棧
- 藍牙 4.0 BLE 協議棧是採用分層的思想，各層具有哪些功能
- 系統硬件對藍牙 4.0 BLE 協議都提供哪些支撐
```
## GAP
```
 - GAP層定義了四種BLE角色：
      - 廣播者 - 不可連接的廣告設備
      - 觀察者 - 掃描廣播，但不發起建立連接
      - 外部設備(Peripheral) - 可連接的廣告設備
      - 集中器(Central) - 掃描廣播並發起建立連接
  (目前，BLE協議棧支持一個集中器連接三個外部設備)
```
```
設備發現的過程: 在典型的BLE中，外部設備會廣告特定的數據，來使集中器知道他是一個可以連接的設備，廣告內容包括設備地址以及一些額外的數據，如設備名等。集中器收到廣告數據後，向外部設備發送掃描請求，然後外部設備將特定的數據回應給集中器(掃描回應)，集中器收到掃描回應後，變可以知道這是一個可以建立連接的外部設備。

設備發現後，集中器便可以向外部設備發起建立連接的請求:
連接請求包括以下連接參數
- 連接間隔： 在兩個BLE設備的連接中使用調頻機制，兩個設備使用特定的信道收發數據，然後過一段時間後再使用新的信道(鏈路層處理信道切換)，兩個設備在信道切換後收發數據稱之為連接事件。即使沒有應用數據的收發，兩個設備仍然會通過交換鏈路層數據來維持連接，連接間隔以1.25ms為單位，連接間隔的值為6(7.5ms)-3200(4s)。
- 從機延遲: 這個參數的設置可以使外部設備跳過若干的連接事件，可以用來節省功耗。
- 管理超時: 這是兩個成功連接事件之間允許的最大間隔，管理超時以10ms為單位，管理超時的範圍是10(100ms)-3200(32s)(超時值必須大於有效連接間隔[= 連接間隔*(1+從機延遲)])。
```

BLE連接中安全特徵的初始化: 特定的數據只有在已認證的連接中才能被讀寫。典型應用為，外部設備請求集中器提供密鑰來完成配對工作，當集中器發送正確的密鑰後，兩個設備交換安全密鑰並加密認證連接。
> 在許多情況下，同一對外部設備和集中器會不時地連接和斷開，BLE的安全機制中有一個```綁定```的特性，允許兩個設備之間建立長期的安全密鑰訊息，它允許兩個設備重新連接是快速完成加密認證，而不需要沒錯連接時執行配對的完整過程。

## GATT
GATT在通訊過程中是以Server和Client角色存在的: Server端用於提供數據，Client端用於使用Server端提供的數據並完成處理。
> 在GAP中的Central(集中器)和Peripheral(外部設備)角色到GATT階段均可擔當Server或是Client，並不固定。

通用服务规范（Generic Attribute Profile，GATT）：该协议是构建在屬性協議(Attribute protocol,ATT)之上的，用于定义服务的流程、格式和所包含的特征。GATT和ATT对于BLE来说是强制配置的。GATT根據不同的規範Profile、服務Service、特徵值Characteristic對不同的屬性進行劃分，ATT定義了客戶端與服務端如何發送符合標準的訊息，而GATT則是定義了如何發現與使用服務、特徵與描述符的標準方法。
GATT可以可以分成三種基本的類型：發現、客戶端發起、服務端發起。

Characteristic是服務用到的值，以及其內容和配置訊息

## Advertised Services

#### Unknown Service
##### UUID ```4145```

## Attribute Table

#### Primary Service: ```Generic Access```
- ##### UUID ```1800```

#### Primary Service: ```Generic Attribute ```
- ##### UUID ```1801```

#### Primary Service: ```Device Information``` 
- ##### UUID ```180A```
-  ##### Characteristic: ```Model Number String```
   >  - ###### UUID ```2A24```
   >  - Properties: **READ**
   >  - Value: AT.Wallet
   >  - Value Sent: N/A
   
-  ##### Characteristic: ```Firmware Revision String```
   >  - ###### UUID ```2A26```
   >  - Properties: **READ**
   >  - Value: 2.00.B4.19
   >  - Value Sent: N/A

-  ##### Characteristic: ```Name String```
   >  - ###### UUID ```2A29```
   >  - Properties: **READ**
   >  - Value: AuthenTrend
   >  - Value Sent: N/A
   
#### Primary Service: ```Battery Service``` 
- ##### UUID ```180F```
-  ##### Characteristic: ```Battery Level```
   >  - ###### UUID ```2A19```
   >  - Properties: **READ**
   >  - Value: 0x49
   >  - Value Sent: N/A
   
#### Primary Service: ```Uknown Service``` 
- ##### UUID ```4145```
-  ##### Characteristic: ```Uknown Characteristic``` || ```controlPointLength```
   >  - ###### UUID ```F1D0FFF3-DEAA-ECEE-B42F-C9BA7ED623BB```
   >  - Properties: **READ**
   >  - Value: 0x01-FD
   >  - Value Sent: N/A
   
-  ##### Characteristic: ```Software Revision String```
   >  - ###### UUID ```2A28```
   >  - Properties: **READ**
   >  - Value: 1.0
   >  - Value Sent: N/A

-  ##### Characteristic: ```Uknown Characteristic``` || ```revisionBitfield```
   >  - ###### UUID ```F1D0FFF4-DEAA-ECEE-B42F-C9BA7ED623BB```
   >  - Properties: **READ**
   >  - Value: 0x80
   >  - Value Sent: N/A
   
-  ##### Characteristic: ```Uknown Characteristic```  || ```controlPoint```
   >  - ###### UUID ```F1D0FFF1-DEAA-ECEE-B42F-C9BA7ED623BB```
   >  - Properties: **Write and Write Without Response**
   >  - Value: N/A
   >  - Value Sent: N/A

-  ##### Characteristic: ```Uknown Characteristic``` || ```status```
   >  - ###### UUID ```F1D0FFF2-DEAA-ECEE-B42F-C9BA7ED623BB```
   >  - Properties: **Notify**
   >  - Value: N/A
   >  - Value Sent: N/A
   
      - ##### Descriptor: ```Client Characteristic Configuration```
       >  - ###### UUID ```2902```
       >  - Value: N/A
       >  - Value Sent: Notifications and Indications are Enabled/Disabled

