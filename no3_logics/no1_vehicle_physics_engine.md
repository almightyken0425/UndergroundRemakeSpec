# 車輛物理引擎規格

---

## 模組定位

VehiclePhysicsEngine 是整個賽車遊戲的核心基礎,負責模擬真實世界的車輛動力學。所有與駕駛相關的功能都依賴此模組提供準確且有趣的物理反饋。

**核心價值:**
- 提供真實且有趣的駕駛手感
- 支援多種車輛類型的物理特性
- 為其他系統提供穩定的物理數據介面

---

## 核心職責

### 車輛動力學模擬

- 計算車輛的加速度、速度、位置
- 模擬引擎扭力輸出與變速箱傳動
- 處理空氣阻力與滾動阻力
- 計算重心轉移對車體姿態的影響

### 輪胎與路面交互

- 計算輪胎與路面的摩擦力
- 模擬輪胎打滑與抓地力恢復
- 處理不同路面材質的物理特性
- 模擬輪胎溫度對性能的影響

### 懸吊系統模擬

- 計算懸吊彈簧的壓縮與回彈
- 模擬阻尼器對車體晃動的抑制
- 處理防傾桿對側傾的影響
- 計算車輪上下行程的限制

### 碰撞偵測與響應

- 偵測車體與環境的碰撞
- 計算碰撞力度與方向
- 處理碰撞後的速度與旋轉變化
- 觸發音效與視覺效果事件

### 空氣動力學

- 計算下壓力對抓地力的影響
- 模擬風阻對最高速度的限制
- 處理側風對車體穩定性的影響

---

## 資料結構定義

### VehiclePhysicsConfig

車輛物理參數配置,定義車輛的靜態物理特性。

```json
{
  "vehicleId": "default_sedan",
  "mass": {
    "totalMass": 1500,
    "centerOfMass": {
      "x": 0,
      "y": -0.3,
      "z": 0.2
    },
    "frontWeightBias": 0.55
  },
  "engine": {
    "maxPower": 150000,
    "maxTorque": 300,
    "maxRPM": 7000,
    "idleRPM": 800,
    "torqueCurve": [
      {"rpm": 0, "torque": 0},
      {"rpm": 1000, "torque": 200},
      {"rpm": 3000, "torque": 280},
      {"rpm": 5000, "torque": 300},
      {"rpm": 7000, "torque": 250}
    ]
  },
  "transmission": {
    "gearRatios": [3.5, 2.2, 1.5, 1.0, 0.8],
    "finalDriveRatio": 3.7,
    "shiftTime": 0.2
  },
  "aerodynamics": {
    "dragCoefficient": 0.32,
    "frontalArea": 2.2,
    "downforceCoefficient": 0.1
  },
  "suspension": {
    "stiffness": 35000,
    "damping": 4500,
    "travelDistance": 0.3,
    "antiRollBar": 2000
  },
  "wheels": {
    "radius": 0.33,
    "mass": 20,
    "frictionCoefficient": 1.0,
    "slipAngleStiffness": 15.0
  }
}
```

### VehiclePhysicsState

車輛的動態物理狀態,每幀更新。

```json
{
  "position": {"x": 100.5, "y": 0.3, "z": 250.8},
  "rotation": {"pitch": 0.05, "yaw": 1.57, "roll": 0.02},
  "velocity": {"x": 5.2, "y": 0.0, "z": 25.3},
  "angularVelocity": {"pitch": 0.1, "yaw": 0.05, "roll": 0.0},
  "acceleration": {"x": 0.5, "y": 0.0, "z": 3.2},
  "engineRPM": 3500,
  "currentGear": 3,
  "wheelStates": [
    {
      "wheelId": "front_left",
      "suspensionCompression": 0.15,
      "slipRatio": 0.05,
      "slipAngle": 2.3,
      "temperature": 85.0,
      "isGrounded": true,
      "normalForce": 3500
    }
  ],
  "speedKmh": 91.08
}
```

### PhysicsInput

駕駛輸入與環境參數。

```json
{
  "throttle": 0.8,
  "brake": 0.0,
  "steering": 0.3,
  "handbrake": false,
  "clutch": 0.0,
  "shiftUp": false,
  "shiftDown": false,
  "environment": {
    "gravity": -9.81,
    "airDensity": 1.225,
    "windVelocity": {"x": 0.0, "y": 0.0, "z": 0.0}
  },
  "roadSurface": {
    "frictionCoefficient": 0.85,
    "bumpiness": 0.02,
    "slopeAngle": 0.0,
    "surfaceNormal": {"x": 0.0, "y": 1.0, "z": 0.0}
  }
}
```

### PhysicsOutput

物理引擎輸出給其他系統的數據。

```json
{
  "vehicleState": {},
  "audioTriggers": {
    "engineRPM": 3500,
    "wheelSlip": 0.15,
    "collisionForce": 0.0,
    "surfaceType": "asphalt"
  },
  "visualEffects": {
    "wheelSpinEffect": true,
    "smokeIntensity": 0.3,
    "bodyLeanAngle": 5.2
  },
  "telemetry": {
    "gForce": {"lateral": 0.5, "longitudinal": 0.3, "vertical": 1.0},
    "wheelLoad": [3500, 3200, 4000, 4300],
    "powerOutput": 85.5
  }
}
```

---

## 介面設計

### IRoadDataProvider

物理引擎與道路系統的解耦介面。

```csharp
public interface IRoadDataProvider {
    float GetFrictionAt(Vector3 position);
    float GetSlopeAt(Vector3 position);
    Vector3 GetSurfaceNormalAt(Vector3 position);
    RoadMaterial GetMaterialAt(Vector3 position);
    float GetBumpinessAt(Vector3 position);
}
```

**實作策略:**
- **MockRoadDataProvider:** 測試階段回傳硬編碼值
- **ProceduralRoadProvider:** 程序化生成測試賽道
- **OpenWorldRoadProvider:** 真實地圖系統整合

### IVehiclePhysicsEngine

物理引擎對外提供的核心介面。

```csharp
public interface IVehiclePhysicsEngine {
    void Initialize(VehiclePhysicsConfig config);
    void UpdatePhysics(PhysicsInput input, float deltaTime);
    VehiclePhysicsState GetCurrentState();
    PhysicsOutput GetPhysicsOutput();
    void ApplyForce(Vector3 force, Vector3 position);
    void Reset(Vector3 position, Quaternion rotation);
}
```

---

## 核心演算法

### 輪胎摩擦力模型

採用 Pacejka Magic Formula 簡化版本。

**滑移率計算:**
```
slipRatio = (wheelAngularVelocity * wheelRadius - vehicleVelocity) / max(vehicleVelocity, 0.1)
```

**摩擦力計算:**
```
Fx = D * sin(C * atan(B * slipRatio - E * (B * slipRatio - atan(B * slipRatio))))

其中:
- D = peakForce * normalForce
- C = shapeFactor
- B = stiffnessFactor / (C * D)
- E = curvatureFactor
```

**側滑力計算:**
```
slipAngle = atan2(lateralVelocity, longitudinalVelocity)
Fy = D * sin(C * atan(B * slipAngle - E * (B * slipAngle - atan(B * slipAngle))))
```

### 引擎扭力計算

**扭力曲線插值:**
```csharp
float GetEngineTorque(float rpm, float throttle) {
    float baseTorque = InterpolateFromCurve(rpm, torqueCurve);
    return baseTorque * throttle;
}
```

**輪上扭力:**
```
wheelTorque = engineTorque * gearRatio * finalDriveRatio * transmissionEfficiency
```

### 懸吊系統

**彈簧力:**
```
springForce = stiffness * compressionDistance
```

**阻尼力:**
```
damperForce = damping * compressionVelocity
```

**總懸吊力:**
```
totalSuspensionForce = springForce + damperForce
```

### 重心轉移

**縱向載荷轉移:**
```
loadTransferLongitudinal = (mass * acceleration * centerOfMassHeight) / wheelbase
frontLoad += loadTransferLongitudinal
rearLoad -= loadTransferLongitudinal
```

**側向載荷轉移:**
```
loadTransferLateral = (mass * lateralAcceleration * centerOfMassHeight) / trackWidth
leftLoad += loadTransferLateral
rightLoad -= loadTransferLateral
```

---

## 測試環境策略

### 程序化測試賽道

**目標:** 零依賴驗證物理核心邏輯

**包含場景:**
- 平面加速測試場
- 標準圓形測試場
- 坡道測試: 5度、10度、15度
- 標準彎道組合: 半徑 20m、50m、100m

**Mock 路面參數:**
- 乾燥柏油: friction = 0.85
- 濕滑路面: friction = 0.6
- 碎石路面: friction = 0.4

### 簡化剛體車輛

**階段一測試實體:**
- 單一剛體車身
- 四個輪胎碰撞點
- 硬編碼物理參數
- 基礎輸入映射

**可驗證項目:**
- 加速與煞車響應
- 轉向與打滑
- 碰撞反彈
- 重心影響

### 基礎物理車輛

**階段二測試實體:**
- 獨立懸吊系統
- 輪胎摩擦力模型
- 引擎扭力曲線
- 變速箱模擬

**可驗證項目:**
- 懸吊壓縮與回彈
- 輪胎側滑與抓地力
- 檔位切換響應
- 進階車體動態

---

## 調校參數指南

### 關鍵調校參數

**操控性調整:**
- **轉向不足:** 增加前輪抓地力、減少前懸吊防傾桿
- **轉向過度:** 增加後輪抓地力、減少後懸吊防傾桿
- **反應遲鈍:** 減少懸吊阻尼、增加轉向靈敏度
- **過於敏感:** 增加懸吊阻尼、減少輪胎側滑剛性

**速度調整:**
- **加速不足:** 增加引擎扭力、調整齒輪比
- **最高速限制:** 減少空氣阻力係數、調整終傳比
- **煞車距離:** 調整煞車力度、輪胎摩擦係數

**穩定性調整:**
- **高速不穩:** 增加下壓力、降低重心
- **過彎側傾:** 增加防傾桿剛性、降低重心
- **顛簸路面:** 減少懸吊剛性、增加阻尼

---

## 效能優化策略

### 計算優化

- 物理更新頻率: 50Hz 固定時間步長
- 碰撞檢測: 使用簡化碰撞盒
- 輪胎計算: LOD 系統,遠距離車輛簡化計算
- 快取重複計算結果

### 記憶體優化

- 物理配置資料共享: 相同車型共用配置
- 狀態池化: 預分配物理狀態物件
- 減少動態記憶體分配

---

## 整合點設計

### 與 InputControlSystem 整合

**資料流:**
```
InputControlSystem → PhysicsInput → VehiclePhysicsEngine
```

**介面:**
- 接收標準化的駕駛輸入訊號
- 支援多種輸入裝置

### 與 AdvancedRenderingEngine 整合

**資料流:**
```
VehiclePhysicsState → 車輛位置與姿態 → 渲染引擎
PhysicsOutput.visualEffects → 視覺效果觸發
```

### 與 DynamicAudioEngine 整合

**資料流:**
```
PhysicsOutput.audioTriggers → 音效引擎
```

**提供資料:**
- 引擎轉速
- 輪胎打滑程度
- 碰撞力度
- 路面材質

### 與 VehicleCustomizationSystem 整合

**資料流:**
```
客製化參數 → VehiclePhysicsConfig → 物理引擎重新初始化
```

**影響參數:**
- 引擎改裝影響扭力曲線
- 懸吊改裝影響剛性與阻尼
- 輪胎改裝影響摩擦係數
- 空力套件影響下壓力

---

## 開發里程碑

### Milestone 1: 基礎物理原型

**目標:** 單一車輛可在平面賽道駕駛

**交付物:**
- 簡化剛體車輛實作
- 程序化平面測試場
- 基礎加速、煞車、轉向
- MockRoadDataProvider 實作

**驗證標準:**
- 車輛可響應輸入移動
- 速度與加速度計算正確
- 基礎碰撞偵測運作

### Milestone 2: 進階物理模擬

**目標:** 實作懸吊與輪胎物理

**交付物:**
- 獨立懸吊系統
- Pacejka 輪胎模型
- 重心轉移計算
- 坡道與彎道測試場

**驗證標準:**
- 懸吊壓縮視覺化正確
- 輪胎打滑響應真實
- 過彎時載荷轉移明顯

### Milestone 3: 完整動力系統

**目標:** 引擎與變速箱模擬

**交付物:**
- 引擎扭力曲線系統
- 變速箱檔位切換
- 空氣動力學模擬
- 完整 PhysicsOutput 介面

**驗證標準:**
- 引擎聲浪與轉速同步
- 換檔響應符合預期
- 高速下壓力影響可感知

### Milestone 4: 調校與優化

**目標:** 多種車輛類型支援

**交付物:**
- 多組車輛物理預設
- 參數調校工具
- 效能優化
- 完整測試場景

**驗證標準:**
- 不同車型駕駛手感差異明顯
- 物理更新維持 50Hz 穩定
- 調校參數影響可預測

---

## 風險與挑戰

### 高風險項目

**物理調校複雜度:**
- **風險:** 參數調校需大量測試與迭代
- **緩解:** 建立參數調校工具與自動化測試
- **備案:** 參考成熟物理引擎的參數預設

**真實感與可玩性平衡:**
- **風險:** 過於真實可能降低遊戲性
- **緩解:** 提供多種協助模式與難度設定
- **備案:** 設計模擬與街機兩種物理模式

### 中風險項目

**效能瓶頸:**
- **風險:** 複雜物理計算影響幀率
- **緩解:** 固定時間步長與 LOD 系統
- **備案:** 簡化遠距離車輛的物理計算

**第三方整合:**
- **風險:** Unity WheelCollider 可能不符需求
- **緩解:** 盡早驗證可行性
- **備案:** 自行實作完整物理系統

---

## 參考資源

### 物理引擎研究

- Unity WheelCollider 文件與限制
- PhysX Vehicle SDK 能力評估
- Pacejka Magic Formula 論文
- 真實車輛物理參數資料庫

### 競品分析

- Forza Horizon 物理手感
- Gran Turismo 模擬深度
- Need for Speed 街機平衡
- Assetto Corsa 調校系統

### 技術文獻

- 車輛動力學教材
- 輪胎摩擦力模型論文
- 遊戲物理引擎設計模式
- 即時物理模擬優化技術
