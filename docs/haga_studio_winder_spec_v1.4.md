# Haga Studio Pickup Winder 仕様書（ver.1.4）

## 更新概要（v1.3 → v1.4）

- RPM 計測窓を**可変式（FAST/SLOW）**に変更、EMA 平滑化を追加
- RUN↔PAUSE の割り込み切替を**安全側（明示的に無効化 → 再登録）**へ修正
- エラー復帰時に**再開／リセット再開／メニュー戻り**を選択可能化
- LED は**初期選択方向を保持**し、逆転時には切り替えない（画面アイコンで方向表示）
- センサノイズ対策（最短パルス幅フィルタ）を追加
- RUN/COUNT モード処理を共通関数`run_loop()`に整理

---

## RPM 計測（可変窓＋ EMA）

### パラメータ

| 名称     | 値                              | 説明                |
| -------- | ------------------------------- | ------------------- |
| FAST_WIN | 120 ms                          | 高速時（追従重視）  |
| SLOW_WIN | 320 ms                          | 低速時（安定重視）  |
| 切替条件 | fast≥30 → FAST / slow≤20 → SLOW | ヒステリシス切替    |
| 平滑化   | α=0.25                          | EMA（指数移動平均） |

```cpp
float rpm_disp = 0;
enum RpmWin { FAST, SLOW };
RpmWin win = SLOW;

void on_tick_100ms() {
  static uint32_t fast_p = 0, slow_p = 0;
  static uint32_t fast_ms = 0, slow_ms = 0;

  noInterrupts();
  uint32_t p = run_pulses;
  run_pulses = 0;
  interrupts();

  fast_ms = min(fast_ms + 100, 120u);
  slow_ms = min(slow_ms + 100, 320u);
  fast_p += p; slow_p += p;

  if (win == SLOW && fast_p >= 30 && fast_ms == 120) win = FAST;
  if (win == FAST && slow_p <= 20 && slow_ms == 320) win = SLOW;

  auto rpm_from = [](uint32_t pulses, uint32_t ms) {
    if (ms == 0) return 0.f;
    float rev = float(pulses) / 4.0f;
    return rev * (60000.0f / ms);
  };

  float rpm_raw = (win == FAST)
      ? rpm_from(fast_p, fast_ms)
      : rpm_from(slow_p, slow_ms);

  rpm_disp += 0.25f * (rpm_raw - rpm_disp);

  if (fast_ms == 120) { fast_ms = 0; fast_p = 0; }
  if (slow_ms == 320) { slow_ms = 0; slow_p = 0; }
}
```

---

## 割り込み切替（安全側）

```cpp
void enter_pause() {
  detachInterrupt(pinA_run);
  detachInterrupt(pinB_run);
  attachInterrupt(digitalPinToInterrupt(pinA_pause), isr_pause_A_edge, CHANGE);
}

void leave_pause() {
  detachInterrupt(pinA_pause);
  attachInterrupt(digitalPinToInterrupt(pinA_run), isr_run_A_rise, RISING);
}
```

---

## エラー復帰（選択式）

エラー発生時：`ERROR: No pulse` → 一時停止画面表示  
選択肢：

| 項目         | 内容                           |
| ------------ | ------------------------------ |
| 再開         | 巻き数・設定保持               |
| リセット再開 | 巻き数・RPM リセット、設定保持 |
| メニュー     | リザルト画面 → メニューへ戻る  |

UI 例：右端に`PUSH:決定 / 回して選択`

---

## LED 動作（方向保持）

| 状態     | 青 LED              | 赤 LED       | 備考                         |
| -------- | ------------------- | ------------ | ---------------------------- |
| RUN 中   | 選択方向 LED を点灯 | もう一方消灯 | 手巻き方向が逆でも変更しない |
| ポーズ中 | 選択方向 LED を点滅 | もう一方消灯 |                              |
| 待機中   | 選択方向 LED を点滅 | もう一方消灯 |                              |

逆転方向は画面上に矢印アイコンで表示。

---

## センサノイズ対策

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

---

## モード共通化（ポリシー設計）

```cpp
struct RunPolicy {
  bool count_target;      // 目標カウントあり
  bool allow_add_more;    // 追加巻可能
  bool adjust_duty_live;  // エンコーダで出力調整
};

void run_loop(const RunPolicy& p) {
  // 方向選択→WAIT→RUN↔PAUSE→… を共通処理
}
```

| モード   | count_target | allow_add_more | adjust_duty_live |
| -------- | ------------ | -------------- | ---------------- |
| フリー   | false        | false          | true             |
| カウント | true         | true           | true             |

---

## 参考

- RPM 可変窓で**低速安定・高速追従**を両立
- 割り込み付替えによる**安全な状態遷移**
- **LED 方向保持**で視認性向上
- **エラー復帰選択**により運用柔軟性 UP
- ノイズ対策で誤カウント防止
