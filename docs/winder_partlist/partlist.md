# パーツリスト

## ⭐️ 買い物リスト（BOM 簡略）

### LED・表示系

| 部品       | 仕様 / 備考            | 数量 |
| ---------- | ---------------------- | ---- |
| LED        | φ3.0mm Flat Top（THT） | 2    |
| LED 用抵抗 | R_Small / 2010         | 2    |

---

### 抵抗（Resistor）

| 抵抗値 | パッケージ | 数量 |
| ------ | ---------- | ---- |
| 100Ω   | 0805       | 3    |
| 10kΩ   | 0805       | 8    |
| 100kΩ  | 0805       | 2    |
| 100kΩ  | 1206       | 2    |

---

### コンデンサ（Capacitor）

#### 電解・ラジアル（THT）

| 容量   | サイズ     | 数量 |
| ------ | ---------- | ---- |
| 1000µF | φ13 / P5.0 | 1    |
| 470µF  | φ10 / P5.0 | 2    |
| 47µF   | φ5 / P2.0  | 1    |
| 10µF   | φ5 / P2.0  | 1    |

#### セラミック / タンタル（SMD）

| 容量  | パッケージ | 数量 |
| ----- | ---------- | ---- |
| 0.1µF | 1206       | 3    |
| 0.1µF | 2012       | 3    |

---

### ダイオード・保護部品

| 部品                   | 型番 / 用途 | 数量 |
| ---------------------- | ----------- | ---- |
| ツェナーダイオード     | TSOT-23     | 1    |
| TVS ダイオード         | SMAJ15A     | 1    |
| TVS ダイオード         | SMAJ13A     | 1    |
| ショットキーダイオード | SS14（SMA） | 1    |

---

### MOSFET・半導体

| 部品       | 型番              | 数量 |
| ---------- | ----------------- | ---- |
| Pch MOSFET | IRF4905（TO-220） | 1    |
| Nch MOSFET | AO3400A（SOT-23） | 2    |

---

### ヒューズ

| 種類                     | 定格                 | 数量 |
| ------------------------ | -------------------- | ---- |
| ブレードヒューズ         | 5A Slow              | 1    |
| ブレードヒューズ         | 3A Slow              | 1    |
| PTC リセッタブルヒューズ | 1A Fast（MF-RHT070） | 1    |

---

### アクチュエータ・音

| 部品        | 備考            | 数量 |
| ----------- | --------------- | ---- |
| ブザー      | TDK PS1240P02BT | 1    |
| DC モーター | 型番未定        | 1    |
| 12V ファン  | 2pin            | 1    |

---

### 電源・モジュール

| 部品             | 備考                | 数量 |
| ---------------- | ------------------- | ---- |
| DC-DC コンバータ | MP1584EN モジュール | 1    |
| ESP32 モジュール | Freenove Socket     | 1    |

---

### コネクタ（JST）

#### JST-XH

| 種類   | ピン数 | 数量 |
| ------ | ------ | ---- |
| JST-XH | 2pin   | 3    |

#### JST-VH

| 種類   | ピン数 | 数量 |
| ------ | ------ | ---- |
| JST-VH | 2pin   | 2    |

#### JST-PH

| 種類   | ピン数 | 数量 |
| ------ | ------ | ---- |
| JST-PH | 3pin   | 3    |
| JST-PH | 4pin   | 1    |
| JST-PH | 5pin   | 1    |
| JST-PH | 6pin   | 1    |

---

### 注意事項

- 抵抗値未確定の部品は回路図と要確認
- JST コネクタは「基板側・ハウジング・コンタクトピン」を別途用意
- ブレードヒューズはヒューズ本体も別途必要

---

## ---各基盤部材詳細一覧---

### LED 系統

| Reference | Qty | Value   | DNP | Exclude from BOM | Exclude from Board | Footprint                                                | Datasheet |
| :-------- | :-- | :------ | :-- | :--------------- | :----------------- | :------------------------------------------------------- | :-------- |
| D1,D2     | 2   | LED     |     |                  |                    | LED_THT:LED_D3.0mm_FlatTop                               | ~         |
| R1,R2     | 2   | R_Small |     |                  |                    | Resistor_SMD:R_2010_5025Metric_Pad1.40x2.65mm_HandSolder | ~         |

### power 系統

| Reference           | Qty | Value       | DNP | Exclude from BOM | Exclude from Board | Footprint                                                 | Datasheet                                                                                                                    |
| :------------------ | :-- | :---------- | :-- | :--------------- | :----------------- | :-------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------- |
| 100kR1,100kR2,100R1 | 3   | R           |     |                  |                    | Resistor_SMD:R_1206_3216Metric_Pad1.30x1.75mm_HandSolder  | ~                                                                                                                            |
| C1                  | 1   | 470uF       |     |                  |                    | Capacitor_THT:CP_Radial_D10.0mm_P5.00mm                   | ~                                                                                                                            |
| C2,C4,C5            | 3   | 0.1uF       |     |                  |                    | Capacitor_SMD:C_1206_3216Metric_Pad1.33x1.80mm_HandSolder | ~                                                                                                                            |
| C3                  | 1   | 1000uF      |     |                  |                    | Capacitor_THT:CP_Radial_D13.0mm_P5.00mm                   | ~                                                                                                                            |
| CN1                 | 1   | ~           |     |                  |                    | Connector_JST:JST_XH_B2B-XH-AM_1x02_P2.50mm_Vertical      |                                                                                                                              |
| CN2                 | 1   | ~           |     |                  |                    | Connector_JST:JST_VH_B2P-VH-B_1x02_P3.96mm_Vertical       |                                                                                                                              |
| D1                  | 1   | D_Zener     |     |                  |                    | Package_TO_SOT_SMD:TSOT-23_HandSoldering                  | ~                                                                                                                            |
| D2                  | 1   | SMAJ15A     |     |                  |                    | Diode_SMD:D_SMA                                           | https://www.littelfuse.com/media?resourcetype=datasheets&itemid=75e32973-b177-4ee3-a0ff-cedaf1abdb93&filename=smaj-datasheet |
| D3                  | 1   | SMAJ13A     |     |                  |                    | Diode_SMD:D_SMA                                           | https://www.littelfuse.com/media?resourcetype=datasheets&itemid=75e32973-b177-4ee3-a0ff-cedaf1abdb93&filename=smaj-datasheet |
| F1                  | 1   | 5A Slow     |     |                  |                    | Fuse:FuseHolder_Blade_ATO_Littelfuse_FLR_178.6165         | ~                                                                                                                            |
| F2                  | 1   | 1A Fast PTC |     |                  |                    | Fuse:Fuse_Bourns_MF-RHT070                                | ~                                                                                                                            |
| F3                  | 1   | 3A Slow     |     |                  |                    | Fuse:FuseHolder_Blade_ATO_Littelfuse_FLR_178.6165         | ~                                                                                                                            |
| Q1                  | 1   | IRF4905     |     |                  |                    | Package_TO_SOT_THT:TO-220-3_Vertical                      | http://www.infineon.com/dgdl/irf4905.pdf?fileId=5546d462533600a4015355e32165197c                                             |

### logic 系統

| Reference                                                    | Qty | Value                | DNP | Exclude from BOM | Exclude from Board | Footprint                                                | Datasheet                                       |
| :----------------------------------------------------------- | :-- | :------------------- | :-- | :--------------- | :----------------- | :------------------------------------------------------- | :---------------------------------------------- |
| 2LED1                                                        | 1   | LED                  |     |                  |                    | Connector_JST:JST_PH_B3B-PH-K_1x03_P2.00mm_Vertical      |                                                 |
| BZ1                                                          | 1   | Buzzer               |     |                  |                    | Buzzer_Beeper:Buzzer_TDK_PS1240P02BT_D12.2mm_H6.5mm      | ~                                               |
| BZ_FET1,FAN_FET1                                             | 2   | AO3400A              |     |                  |                    | Package_TO_SOT_SMD:SOT-23                                | http://www.aosmd.com/pdfs/datasheet/AO3400A.pdf |
| BZ_R1,FAN_R1                                                 | 2   | 100_R                |     |                  |                    | Resistor_SMD:R_0805_2012Metric_Pad1.20x1.40mm_HandSolder | ~                                               |
| BZ_R2,FAN_R2                                                 | 2   | 100K_R               |     |                  |                    | Resistor_SMD:R_0805_2012Metric_Pad1.20x1.40mm_HandSolder | ~                                               |
| C1                                                           | 1   | 470uF                |     |                  |                    | Capacitor_THT:CP_Radial_D10.0mm_P5.00mm                  | ~                                               |
| C2,C4,C6                                                     | 3   | 0.1uF                |     |                  |                    | Capacitor_Tantalum_SMD:CP_EIA-2012-12_Kemet-R_HandSolder | ~                                               |
| C3                                                           | 1   | 47uF                 |     |                  |                    | Capacitor_THT:C_Radial_D5.0mm_H11.0mm_P2.00mm            | ~                                               |
| C5                                                           | 1   | 10uF                 |     |                  |                    | Capacitor_THT:C_Radial_D5.0mm_H11.0mm_P2.00mm            | ~                                               |
| DCC1                                                         | 1   | DC_DW                |     |                  |                    | My_Module:MP1584EN_DCDC_Converter                        |                                                 |
| ESP1                                                         | 1   | ~                    |     |                  |                    | My_Module:ESP32_Freenove_Socket                          |                                                 |
| FAN1                                                         | 1   | 12V_Fan              |     |                  |                    | Connector_JST:JST_XH_B2B-XH-AM_1x02_P2.50mm_Vertical     | ~                                               |
| FAN_D1                                                       | 1   | SS14                 |     |                  |                    | Diode_SMD:D_SMA                                          | https://www.vishay.com/docs/88746/ss12.pdf      |
| IO_0_R1,IO_2_R1,IO_12_R1,IO_15_R1,IO_EN_R1,RE_R1,RE_R2,RE_R3 | 8   | 10K_R                |     |                  |                    | Resistor_SMD:R_0805_2012Metric_Pad1.20x1.40mm_HandSolder | ~                                               |
| J1                                                           | 1   | 12V_IN               |     |                  |                    | Connector_JST:JST_XH_B2B-XH-AM_1x02_P2.50mm_Vertical     | ~                                               |
| M1                                                           | 1   | Motor_DC             |     |                  |                    |                                                          | ~                                               |
| MD1                                                          | 1   | ~                    |     |                  |                    | Connector_JST:JST_PH_B6B-PH-K_1x06_P2.00mm_Vertical      |                                                 |
| OLED1                                                        | 1   | ~                    |     |                  |                    | Connector_JST:JST_PH_B4B-PH-K_1x04_P2.00mm_Vertical      |                                                 |
| RE1                                                          | 1   | RotaryEncoder_Switch |     |                  |                    | Connector_JST:JST_PH_B5B-PH-K_1x05_P2.00mm_Vertical      | ~                                               |
| RS_A1,RS_B1                                                  | 2   | ~                    |     |                  |                    | Connector_JST:JST_PH_B3B-PH-K_1x03_P2.00mm_Vertical      |                                                 |
