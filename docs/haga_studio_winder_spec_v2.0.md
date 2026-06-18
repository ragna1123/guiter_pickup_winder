# Haga Studio Pickup Winder 仕様書 v2.0

この文書は v1 系仕様を統合し、今後の実装基準として整理したものです。

v2.0 では「実装者が迷わないこと」を優先し、状態遷移、表示、回転計測、手巻き補正、エラー復帰を同じ文書内で定義します。

---

## 1. 目的

ギターピックアップのコイル巻き作業を安定して行うための小型ワインダーを制御します。

主な要件は次の通りです。

- フリー巻きと目標巻数指定の両方に対応する
- RUN 中は自動回転の巻き数と RPM を計測する
- PAUSE 中は手動での追い巻き、手戻しを巻き数に反映する
- PWM Duty をロータリーエンコーダで調整できる
- 無検出、ノイズ、誤操作に対して安全側に停止する

---

## 2. 操作系

操作入力はロータリーエンコーダ付きプッシュスイッチを基本とします。

| 操作 | 通常時 | 数値入力時 | RUN 中 |
| ---- | ---- | ---- | ---- |
| 回す | メニュー選択 | 値を増減 | PWM Duty 調整 |
| 押す | 決定 | 決定 | PAUSE へ移行 |

---

## 3. 起動

1. 電源 ON
2. バージョン表示
3. `haga studio` 表示
4. メインメニューへ遷移

表示時間は実装時の体感に合わせて調整可能ですが、起動完了までに不要な待ち時間を作らないことを優先します。

---

## 4. メニュー

メインメニューは次の 3 項目です。

- Free Count
- Count Mode
- Settings

---

## 5. 基本状態

| 状態 | 内容 |
| ---- | ---- |
| BOOT | 起動画面 |
| MENU | メインメニュー |
| DIR_SELECT | 回転方向選択 |
| TARGET_SET | 目標巻数設定 |
| STANDBY | 開始待ち |
| RUN | モーター駆動中 |
| PAUSE | 一時停止、手巻き補正可能 |
| TARGET_REACHED | 目標到達後の選択 |
| ERROR | 異常停止 |
| RESULT | 結果表示 |
| SETTINGS | 設定 |

---

## 6. 状態遷移

### 6.1 Free Count

```text
MENU
  -> DIR_SELECT
  -> STANDBY
  -> RUN
  -> PAUSE
  -> RUN
  -> RESULT
  -> MENU
```

PAUSE では再開または終了を選択します。

### 6.2 Count Mode

```text
MENU
  -> DIR_SELECT
  -> TARGET_SET
  -> STANDBY
  -> RUN
  -> PAUSE
  -> RUN
  -> TARGET_REACHED
  -> RESULT
  -> MENU
```

目標到達後は、追加巻または終了を選択します。

### 6.3 Error

```text
RUN
  -> ERROR
  -> STANDBY
  -> RUN
```

または

```text
ERROR
  -> RESULT
  -> MENU
```

---

## 7. モード仕様

### 7.1 Free Count

目標巻数を持たず、巻き数を累積表示します。

RUN 中に行う処理:

- A センサの片エッジで巻き数を加算
- RUN 中パルスから RPM を算出
- ロータリー操作で PWM Duty を変更
- プッシュで PAUSE

PAUSE 中に行う処理:

- モーター停止
- A/B センサで手巻き方向を判定
- 巻き数を加算または減算
- RPM は更新しない

### 7.2 Count Mode

目標巻数を設定して巻きます。

RUN 中に行う処理:

- A センサの片エッジで巻き数を加算
- 目標巻数に到達したらモーター停止
- RUN 中パルスから RPM を算出
- ロータリー操作で PWM Duty を変更
- プッシュで PAUSE

目標到達後の選択:

| 項目 | 内容 |
| ---- | ---- |
| 追加巻 | 追加する巻数を指定し、目標値へ加算 |
| 終了 | RESULT を表示して MENU へ戻る |

---

## 8. 回転方向

DIR_SELECT で選択した方向を「選択方向」と呼びます。

- 選択方向は RUN 中のモーター駆動方向として使う
- 選択方向は LED 表示の基準として使う
- PAUSE 中に手で逆方向へ回しても LED は切り替えない
- 手巻き補正の方向は画面上の矢印または符号で示す

---

## 9. LED 仕様

LED は「現在検出した回転方向」ではなく「選択方向」を示します。

| 状態 | 選択方向 LED | 反対方向 LED |
| ---- | ------------ | ------------ |
| DIR_SELECT | 点滅 | 消灯 |
| STANDBY | 点滅 | 消灯 |
| RUN | 点灯 | 消灯 |
| PAUSE | 点滅 | 消灯 |
| ERROR | 両方点滅 | 両方点滅 |

LED 点滅周期の初期値は約 400 ms とします。

---

## 10. 表示仕様

### 10.1 Free Count RUN

```text
Free Count    ->
Count: 1234.5
Duty :  75%  RPM: 850
PUSH:Pause
```

### 10.2 Count Mode RUN

```text
Count Mode    ->
Count: 1234.5 / 8000
Duty :  75%  RPM: 850
PUSH:Pause
```

Count Mode は「残り」ではなく「現在 / 目標」を基本表示とします。作業者が実際の巻き数を確認しやすいためです。

### 10.3 PAUSE

```text
Paused        ->
Count: 1234.5
Manual: +0.5
Resume / End
```

手巻き補正が入った場合は `Manual` に符号付きで変化量を表示します。

### 10.4 ERROR

```text
ERROR: No pulse
Count: 1234.5
Resume / Reset / Menu
```

---

## 11. カウント仕様

### 11.1 基本定数

| 項目 | 値 |
| ---- | ---- |
| スリット数 | 4 |
| RUN 中カウント方式 | A 片エッジ |
| PAUSE 中カウント方式 | A 両エッジ + B 方向判定 |
| PAUSE 中分解能 | X2 |

### 11.2 RUN 中カウント

```cpp
winding += float(run_pulses) / SLOTS;
```

RUN 中の `run_pulses` は A センサの片エッジだけを集計します。

### 11.3 PAUSE 中カウント

```cpp
manual_delta = float(pause_pulses_signed) / (SLOTS * 2.0f);
winding = max(0.0f, winding_at_pause + manual_delta);
```

PAUSE 中は RPM を更新しません。

---

## 12. RPM 計測仕様

RPM は RUN 中のパルスだけから算出します。

v2.0 では、計測窓を実経過時間で管理します。固定 tick 回数から窓長を推定しません。

### 12.1 パラメータ

| 項目 | 初期値 | 備考 |
| ---- | ---- | ---- |
| FAST_WIN_MS | 150 | 高速時の追従重視 |
| SLOW_WIN_MS | 400 | 低速時の安定重視 |
| FAST_ENTER_PULSES | 30 | FAST へ入る目安 |
| SLOW_EXIT_PULSES | 20 | SLOW へ戻る目安 |
| EMA_ALPHA | 0.25 | 表示平滑化 |

### 12.2 計算式

```cpp
rpm_raw = (pulses / float(SLOTS)) * (60000.0f / elapsed_ms);
rpm_disp += EMA_ALPHA * (rpm_raw - rpm_disp);
```

`elapsed_ms` は実測した窓の経過時間です。

### 12.3 実装方針

```cpp
struct RpmWindow {
  uint32_t pulses;
  uint32_t start_ms;
};

float rpm_from(uint32_t pulses, uint32_t elapsed_ms) {
  if (elapsed_ms == 0) return 0.0f;
  return (pulses / float(SLOTS)) * (60000.0f / elapsed_ms);
}

void reset_window(RpmWindow& w, uint32_t now_ms) {
  w.pulses = 0;
  w.start_ms = now_ms;
}

void update_rpm(uint32_t now_ms) {
  noInterrupts();
  uint32_t p = run_pulses;
  run_pulses = 0;
  interrupts();

  fast.pulses += p;
  slow.pulses += p;

  uint32_t fast_elapsed = now_ms - fast.start_ms;
  uint32_t slow_elapsed = now_ms - slow.start_ms;

  bool rpm_updated = false;
  float rpm_raw = 0.0f;

  if (fast_elapsed >= FAST_WIN_MS) {
    float fast_rpm = rpm_from(fast.pulses, fast_elapsed);
    if (win == SLOW && fast.pulses >= FAST_ENTER_PULSES) {
      win = FAST;
    }
    if (win == FAST) {
      rpm_raw = fast_rpm;
      rpm_updated = true;
    }
    reset_window(fast, now_ms);
  }

  if (slow_elapsed >= SLOW_WIN_MS) {
    float slow_rpm = rpm_from(slow.pulses, slow_elapsed);
    if (win == FAST && slow.pulses <= SLOW_EXIT_PULSES) {
      win = SLOW;
    }
    if (win == SLOW) {
      rpm_raw = slow_rpm;
      rpm_updated = true;
    }
    reset_window(slow, now_ms);
  }

  if (rpm_updated) {
    rpm_disp += EMA_ALPHA * (rpm_raw - rpm_disp);
  }
}
```

このコードは方針を示すための例です。FAST/SLOW の窓は独立して集計、計算、リセットし、切替直後に古いパルスが過度に残らないようにします。

---

## 13. センサノイズ対策

カウント ISR には最短パルス幅フィルタを入れます。

```cpp
const uint32_t MIN_PULSE_US = 120;
volatile uint32_t last_run_pulse_us = 0;

void IRAM_ATTR isr_run_A_rise() {
  uint32_t now = micros();
  if (now - last_run_pulse_us < MIN_PULSE_US) return;
  last_run_pulse_us = now;
  run_pulses++;
}
```

`MIN_PULSE_US` は固定値ではなく、実機評価で調整する値です。

目安:

```text
min_valid_interval_us = 60,000,000 / (max_rpm * SLOTS)
```

`MIN_PULSE_US` は、上記の有効パルス間隔より十分小さく、ノイズ幅より大きい値に設定します。

---

## 14. 割り込み仕様

RUN と PAUSE ではカウント方式が異なるため、状態遷移時に割り込みを明示的に付け替えます。

```cpp
void enter_run() {
  detachInterrupt(digitalPinToInterrupt(pinA));
  detachInterrupt(digitalPinToInterrupt(pinB));
  attachInterrupt(digitalPinToInterrupt(pinA), isr_run_A_rise, RISING);
}

void enter_pause() {
  detachInterrupt(digitalPinToInterrupt(pinA));
  detachInterrupt(digitalPinToInterrupt(pinB));
  attachInterrupt(digitalPinToInterrupt(pinA), isr_pause_A_edge, CHANGE);
}
```

状態変更時は、モーター出力を先に安全側へ移してから割り込みを切り替えます。

---

## 15. モーター制御

### 15.1 PWM Duty

- 範囲: 0-100%
- 初期変更ステップ: 1%
- RUN 中にロータリーで変更可能
- 設定値と実出力値を分ける

### 15.2 ソフトスタート

STANDBY から RUN へ入るとき、実出力 Duty を 0% から目標 Duty まで線形に上げます。

### 15.3 ソフトストップ

RUN から PAUSE、ERROR、TARGET_REACHED、RESULT へ移るとき、可能な範囲で実出力 Duty を 0% へ落とします。

緊急性の高い ERROR では即時停止を優先します。

---

## 16. エラー処理

### 16.1 No Pulse

RUN 中に一定時間パルスが検出されない場合、モーターを停止して ERROR に入ります。

初期値:

| 項目 | 値 |
| ---- | ---- |
| NO_PULSE_TIMEOUT_MS | 2000 |

復帰選択:

| 項目 | 内容 |
| ---- | ---- |
| Resume | 巻き数、目標、Duty を保持して STANDBY へ戻る |
| Reset | 巻き数と RPM をリセットし、目標と方向は保持 |
| Menu | RESULT を表示して MENU へ戻る |

### 16.2 カウント下限

手戻し補正で巻き数が 0 未満になる場合は 0 に丸めます。

### 16.3 目標超過

Count Mode で目標を超過した場合も巻き数は実測値を保持します。表示は `現在 / 目標` のままとし、終了または追加巻へ進みます。

---

## 17. 設定項目

v2.0 の初期設定項目です。

| 項目 | 初期値 | 備考 |
| ---- | ---- | ---- |
| Count Mode 初期目標 | 8000 | 変更可能 |
| PWM Duty 初期値 | 50% | 変更可能 |
| PWM 変更ステップ | 1% | 変更可能 |
| Soft Start | 1.0 s | 変更可能 |
| Soft Stop | 1.0 s | 変更可能 |
| Buzzer | ON | 変更可能 |
| Display Brightness | 中 | 変更可能 |

---

## 18. 実装構造

Free Count と Count Mode は共通の `run_loop()` で扱い、差分はポリシーで表現します。

```cpp
struct RunPolicy {
  bool has_target;
  bool allow_add_more;
  bool allow_live_duty_adjust;
};
```

| モード | has_target | allow_add_more | allow_live_duty_adjust |
| ---- | ---- | ---- | ---- |
| Free Count | false | false | true |
| Count Mode | true | true | true |

---

## 19. 未確定項目

次の項目は実機または回路図と合わせて確定します。

- ESP32 のピン割り当て
- PWM 周波数
- モータードライバの最小 Duty
- センサ種別と立ち上がり特性
- 最大想定 RPM
- `MIN_PULSE_US` の実測値
- LCD/OLED の実表示桁数
- ブザーの通知パターン

---

## 20. v2.0 での整理点

- v1.3 と v1.4 の内容を統合
- RPM 計測を実経過時間ベースに修正
- Count Mode 表示を `現在 / 目標` に統一
- LED は選択方向表示として定義
- ERROR 復帰の選択肢を明文化
- RUN と PAUSE の割り込み切替を状態仕様に組み込み
- 未確定項目を仕様から分離して一覧化
