# AI Self-Leveling Rover

## 專題名稱

**AMB82-MINI 四輪 AI 自動水平載物平台車**

**English:**  
AI Self-Leveling Rover: AMB82-MINI Four-Wheel Mobile Platform with Two-Axis Auto-Leveling

---

## 專題簡介

本專題以 **AMB82-MINI** 為主要控制核心，製作一台四輪移動平台車。

當車體在上坡、下坡，或左右傾斜時，系統會透過外接 IMU 即時偵測車體的 **Pitch** 與 **Roll**，並利用兩顆 Servo Motor 控制上方雙軸平台，以反方向補償車體傾角，使載物平台盡可能維持水平。

除了基本的自動水平控制外，本專題也規劃加入 **Wi-Fi IoT 監控介面**與**地形狀態辨識功能**，讓使用者可以查看車體傾角、平台補償角度、移動狀態與地形狀態。

---

## 主要功能

- 四輪前進、後退、左轉、右轉與停止
- IMU 即時量測 Pitch / Roll
- 雙軸 Servo 自動水平補償
- 上坡與下坡時維持平台水平
- 左右傾斜時維持平台水平
- PID 或比例控制進行姿態補償
- Wi-Fi IoT 狀態監控
- 地形狀態辨識
- 傾角過大、感測器異常或通訊中斷時自動停止

---

## 系統架構

```text
                        使用者
                          │
                    Wi-Fi / Web UI
                          │
                          ▼
                 ┌─────────────────┐
                 │    AMB82-MINI   │
                 │                 │
                 │ IoT Interface   │
                 │ Terrain Logic   │
                 │ PID Controller  │
                 │ Safety Control  │
                 └────────┬────────┘
                          │
             ┌────────────┼────────────┐
             │            │            │
             ▼            ▼            ▼
            IMU       Motor Driver   2 Servo
             │            │            │
       Pitch / Roll    4 DC Motors   水平平台
```

---

## 自動水平原理

平台目標角度設定為：

```text
Target Pitch = 0°
Target Roll  = 0°
```

控制誤差為：

\[
e_{pitch}=0^\circ-	heta_{pitch}
\]

\[
e_{roll}=0^\circ-	heta_{roll}
\]

控制器依照誤差控制兩顆 Servo，使平台朝車體傾斜的反方向補償。

進階版本可使用 PID：

\[
u(t)=K_Pe(t)+K_I\int e(t)\,dt+K_D\frac{de(t)}{dt}
\]

---

## 硬體規劃

| 元件 | 數量 | 備註 |
|---|---:|---|
| AMB82-MINI | 1 | 老師提供 |
| 外接 6 軸 IMU | 1 | 自備 |
| 高扭力 Servo Motor | 2 | 自備 |
| 四輪車底盤 | 1 | 自備 |
| DC 減速馬達 | 4 | 自備 |
| 馬達驅動模組 | 1 或以上 | 自備 |
| 輪子 | 4 | 自備 |
| 電池 / 供電模組 | 1 組 | 自備 |
| Servo 獨立供電 / 降壓模組 | 1 | 自備 |
| 雙軸平台機構 | 1 | 自製或購買 |
| 杜邦線、螺絲、支架等 | 若干 | 自備 |

---

## AI / IoT 功能

系統可依 IMU 資料判斷目前狀態，例如：

```text
FLAT
UPHILL
DOWNHILL
LEFT_TILT
RIGHT_TILT
UNSTABLE
```

IoT 介面預計顯示：

```text
Robot Status
Pitch
Roll
Pitch Servo Angle
Roll Servo Angle
Terrain State
Auto-Level Status
Motor Speed
```

---

## 安全機制

當系統偵測到以下狀況時，可由 AMB82-MINI 直接停止移動：

```text
傾角超過安全限制
IMU 資料異常
Wi-Fi / 控制通訊逾時
Servo 超過機構安全角度
供電電壓異常
```

AI 或遠端指令不能覆蓋本地安全邏輯。

---

## 預計開發流程

```text
1. AMB82-MINI + IMU 測試
        ↓
2. 單軸 Servo 水平補償
        ↓
3. Pitch + Roll 雙軸補償
        ↓
4. 四輪底盤與馬達控制
        ↓
5. 水平平台安裝至車體
        ↓
6. 斜坡實際測試
        ↓
7. IoT Dashboard
        ↓
8. 地形狀態辨識
```

---

## 預計測試

預計使用不同坡度進行測試：

```text
0°
10°
20°
30°
```

主要評估：

- 平台水平誤差
- Servo 反應速度
- 穩定時間
- 是否產生明顯震盪
- 四輪車爬坡能力
- 車體移動時平台是否仍能保持水平

初期目標為：

```text
靜態平台水平誤差 ≤ ±3°
```

---

## 專題文件

完整專題規劃請參考：

**[PROPOSAL.md](./PROPOSAL.md)**

---

## 專題狀態

```text
目前階段：Proposal / Hardware Planning
主控板：AMB82-MINI
平台設計：雙軸自動水平
移動方式：四輪驅動
```

---

## 課程

**人工智慧與物聯網裝置設計 期末專題**
