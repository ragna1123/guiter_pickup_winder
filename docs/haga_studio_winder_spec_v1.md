# Haga Studio Pickup Winder 仕様書 v1 統合版

この文書は `haga_studio_winder_spec_v1.3.md`、`haga_studio_winder_spec_v1.4.md`、`lcd_menu_ui.md` の内容を統合した v1 系仕様のアーカイブです。

v1 系は試作実装の判断材料として残し、今後の実装基準は `haga_studio_winder_spec_v2.0.md` を参照します。

---

## 1. 起動シーケンス

1. 電源 ON
2. `Ver.x.x` を約 3 秒表示
3. `haga studio` を約 1 秒表示
4. メニュー画面へ遷移

---

## 2. メニュー構成

- フリーカウント
- カウント
- 設定

操作はロータリースイッチで行います。

| 操作 | 内容 |
| ---- | ---- |
| 回す | 選択、数値変更、速度調整 |
| 押す | 決定、一時停止 |

---

## 3. 共通仕様

### 3.1 回転カウント

RUN 中は片側センサ A の片エッジのみを検出して加算します。

- スリット数: 4
- RUN 中: 1 回転 = 4 パルス
- RPM は RUN 中のパルスだけから算出

### 3.2 ポーズ中の手巻き補正

ポーズ中または一時終了中は、A/B 2 センサの位相を使って疑似クアドラチャとして方向を判定します。

- 正方向: 巻き数を加算
- 逆方向: 巻き数を減算
- 巻き数は 0 未満にしない
- ポーズ中のパルスは RPM 算出には使わない
- 既定方式は X2

X2 では A の両エッジを検出し、B の状態で方向を判定します。

```cpp
delta_rev = pulses_pause_signed / (SLOTS * 2.0f);
winding = max(0.0f, winding_saved + delta_rev);
```

### 3.3 表示項目

- 巻き数
- PWM Duty
- 実測 RPM
- 選択方向
- 状態

---

## 4. フリーカウントモード

### 4.1 流れ

1. 方向選択
2. スタート待ち
3. RUN
4. PAUSE
5. 終了時にリザルト画面を表示
6. メニューへ戻る

### 4.2 RUN 中操作

- ロータリースイッチ回転で PWM Duty を変更
- 変更ステップは 1% を既定とする
- 範囲は 0-100%
- 実出力はソフトチェンジで滑らかに追従
- プッシュで PAUSE

### 4.3 表示例

```text
[Free Count]
巻き数: NNNN
出力: DD%   速度: RRR rpm
PUSH: PAUSE / 回す: 速度
```

---

## 5. カウントモード

### 5.1 流れ

1. 方向選択
2. 目標巻数設定
3. スタート待ち
4. RUN
5. PAUSE
6. 目標到達時に一時停止
7. 追加巻または終了を選択
8. 終了時にリザルト画面を表示
9. メニューへ戻る

### 5.2 目標巻数設定

ロータリースイッチで数値を変更します。

- 低速回転: 1 刻み
- 中速回転: 10 刻み
- 高速回転: 100 刻み

### 5.3 RUN 中操作

- ロータリースイッチ回転で PWM Duty を変更
- 変更ステップは 1% を既定とする
- 範囲は 0-100%
- 実出力はソフトチェンジで滑らかに追従
- プッシュで PAUSE

### 5.4 表示例

```text
[Count Mode]
巻き数: NNNN / 目標: TTTT
出力: DD%   速度: RRR rpm
PUSH: PAUSE / 回す: 速度
```

---

## 6. 設定モード

v1 系で想定していた設定項目です。

- カウントモード初期目標巻数
- ソフトスタート時間
- ソフトストップ時間
- ブザー ON/OFF
- ブザー音量
- ディスプレイ明るさ
- ディスプレイ省電力設定

---

## 7. 方向選択

- `-> 正転` と `<- 逆転` をロータリーで選択
- 選択方向の LED のみ点滅
- プッシュで確定
- 確定後、選択方向の LED を点灯

v1.4 では LED の意味を「現在の物理回転方向」ではなく「初期選択方向」に寄せています。ポーズ中の手巻き補正で逆方向を検出しても LED は切り替えず、画面上の矢印で方向を示します。

---

## 8. LED 動作

| 状態 | 選択方向 LED | 反対方向 LED | 備考 |
| ---- | ------------ | ------------ | ---- |
| 待機中 | 点滅 | 消灯 | 方向選択後 |
| RUN 中 | 点灯 | 消灯 | 手巻き方向では切り替えない |
| PAUSE 中 | 点滅 | 消灯 | 手巻き補正中も同じ |

---

## 9. ソフトスタート / ソフトストップ

設定秒数に応じて PWM Duty を線形変化させます。

- スタート: 0% から目標 Duty へ
- ストップ: 現在 Duty から 0% へ

---

## 10. RPM 計測

### 10.1 v1.3 仕様

```text
rpm = (pulses_run / SLOTS) * (60 / dt)
```

- `SLOTS = 4`
- `dt` は計測窓
- `pulses_run` は RUN 中の A 片エッジのみ

### 10.2 v1.4 追加方針

v1.4 では、RPM 表示を安定させるために可変計測窓と EMA 平滑化を追加しました。

| 項目 | 値 |
| ---- | ---- |
| 高速用窓 | 120 ms |
| 低速用窓 | 320 ms |
| 切替条件 | 高速側 30 パルス以上、低速側 20 パルス以下 |
| 平滑化 | EMA alpha = 0.25 |

ただし、v1.4 のサンプルコードは `100ms` tick と `120ms/320ms` 窓の相性が悪く、実装時には実経過時間ベースで再設計する必要があります。この点は v2.0 で整理します。

---

## 11. センサノイズ対策

RUN 中のカウント ISR では最短パルス幅フィルタを入れます。

```cpp
volatile uint32_t last_us = 0;
const uint32_t MIN_PULSE_US = 120;

void IRAM_ATTR isr_run_A_rise() {
  uint32_t now = micros();
  if (now - last_us < MIN_PULSE_US) return;
  last_us = now;
  run_pulses++;
}
```

`MIN_PULSE_US` は実機の最大 RPM とセンサノイズ実測に応じて調整します。

---

## 12. 割り込み切替

RUN と PAUSE で使う割り込みを明示的に切り替えます。

```cpp
void enter_pause() {
  detachInterrupt(digitalPinToInterrupt(pinA_run));
  detachInterrupt(digitalPinToInterrupt(pinB_run));
  attachInterrupt(digitalPinToInterrupt(pinA_pause), isr_pause_A_edge, CHANGE);
}

void leave_pause() {
  detachInterrupt(digitalPinToInterrupt(pinA_pause));
  attachInterrupt(digitalPinToInterrupt(pinA_run), isr_run_A_rise, RISING);
}
```

---

## 13. 例外処理

### 13.1 無検出タイムアウト

RUN 中に 2 秒以上パルスを検出できない場合はモーターを停止します。

- 画面表示: `ERROR: No pulse`
- 状態遷移: ERROR から PAUSE 相当へ移行

### 13.2 エラー復帰

| 項目 | 内容 |
| ---- | ---- |
| 再開 | 巻き数と設定を保持して再開 |
| リセット再開 | 巻き数と RPM をリセットし、設定は保持 |
| メニュー | リザルト画面を表示してメニューへ戻る |

---

## 14. 実装メモ

RUN/COUNT の処理は共通関数化し、モード差分をポリシーで渡す方針です。

```cpp
struct RunPolicy {
  bool count_target;
  bool allow_add_more;
  bool adjust_duty_live;
};
```

| モード | count_target | allow_add_more | adjust_duty_live |
| ---- | ---- | ---- | ---- |
| フリー | false | false | true |
| カウント | true | true | true |

---

## 15. v1 系の残課題

- RPM 可変窓の実装例を実経過時間ベースへ修正する
- LCD 表示の「残り」表示と「現在 / 目標」表示を統一する
- エラー復帰後の状態遷移を状態表で明確化する
- センサノイズフィルタ値の根拠を決める
- ピン割り当て、PWM 周波数、モータードライバ仕様を別表で固定する
