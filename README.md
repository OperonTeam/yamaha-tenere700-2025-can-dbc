# Yamaha Tenere 700 (2025+) CCU CAN — **仮説カタログ**

2025年式 ヤマハ テネレ700（Euro5+ / 6.3インチ TFT / YCC-T）の  
**CCU（コミュニケーションコントロールユニット）コネクタ**で観測した CAN の  
**仮説ベース**の整理ドキュメントおよび DBC です。

> **このリポジトリの位置づけ**  
> 確定仕様ではありません。停車ベンチ＋短時間ダンプからの **仮説（hypothesis）** です。  
> Tag 列に **現在の検証状態** を付与しています。公式 OEM 仕様ではありません。

An independent, community-driven **hypothesis catalog** and DBC for the **2025+ Yamaha Tenere 700**, observed via the CCU connector. **Not a confirmed OEM specification.**

---

## ⚠️ Disclaimer / 免責

* **Passive Observation Only** — Listen-Only。送信・インジェクションは行わない前提。
* **Hypothesis, not ground truth** — ID の存在観測と、バイト変化からの推測を混ぜています。
* **No Liability** — 学習・ロギング用途。バスへの送信は自己責任・非推奨。

---

## Confidence / Tag（検証状態込み · 2026-11-23）

| Tag | 意味 |
|-----|------|
| **OBS** | 当該 ID がバス上に出現することを観測済み |
| **HYP** | バイト意味・計算式は仮説（未確定） |
| **MATCH** | 停車ダンプで操作／状態と突合済み（値または役割） |
| **ROAD** | 実走行またはメーター突合待ち |
| **LONG** | 長距離＋実測消費（L）待ち |
| **RES** | リザーブ点灯差分待ち |
| **HW** | 次回宿題（再操作突合） |
| **NEG** | 本 CCU タップでは該当量未検出 |
| **HOLD** | 走行持ち越し |
| **PH** | 枠のみ · 未検証 |

DBC 内の信号名も **仮説ラベル** です。

---

## Overview / 通信概要

| 項目 | 内容 |
|------|------|
| Tap | CCU コネクタ Y 分岐 · Listen-Only |
| Evidence | キー ON · クランキング · アイドル · 空ぶかし · スタンド/ギア（停車） · CCU/ナビ |
| Negatives | **オド**（申告 2913 km · NEG）· **タンク残量 L**（`0x764`≠残量 · NEG） |
| Pending | **ROAD** 車速/輪速/RPM式 · **LONG** 消費スケール · **RES** 残量/F-31 · **HW** ハイビーム |

---

## Files

* `README.md` — 本ドキュメント
* `yamaha_tenere700_2025_ccu.dbc` — Wireshark / SavvyCAN / `cantools` 用

---

## CAN ID catalog

| CAN ID | DLC | Category | Notes | Tag（検証状態） |
| :--- | :---: | :--- | :--- | :--- |
| **`0x27C`** | 7 | ECU 基本 | **B4 ギア** `0`=N · `1`–`6` · `7`=中間（MATCH）。RPM フィールド MATCH · 式 `×0.4` は HYP（ROAD）。スタンド B3.5 · 稼働 B6=`30`/`B0` MATCH | **OBS · MATCH · HYP(RPM式) · ROAD** |
| **`0x2A3`** | 8 | 制御系? | 停車中ほぼ固定 | **OBS · PH · ROAD** |
| **`0x383`** | 8 | 走行? | 停車固定。車速本線にしない | **OBS · PH · ROAD** |
| **`0x512`** | 4 | 水温候補 | TFT **&lt;39℃=Low.Temp**。本試験 39→**40℃**。B0=`112`⇔仮式 **40.0℃**（有力）。多点突合は ROAD | **OBS · HYP(有力) · ROAD** |
| **`0x58A`** | 8 | 時計 | Y-Connect 日時 | **OBS · MATCH** |
| **`0x58B`** | 8 | メーター同期? | 維持フレーム | **OBS** |
| **`0x591`** | 8 | 電話セッション | 接続時変化 | **OBS · MATCH** |
| **`0x592`** | 8 | メディア? | 待機 | **OBS** |
| **`0x594` `0x595`** | 8 | 通知マスク | スマホ系 · タンク残量ではない | **OBS · NEG(残量)** |
| **`0x59A`–`0x59C`** | 8 | ナビ文字 | UTF-8 マルチフレーム | **OBS · MATCH** |
| **`0x59D` `0x59E`** | 8 | UI 同期? | 画面/セッション | **OBS** |
| **`0x750`** | 8 | ECU 高速 | 負荷連動 · 噴射補助 | **OBS · HYP · ROAD** |
| **`0x751`** | 8 | ECU ステータス | B6 段階変化 | **OBS · HYP** |
| **`0x755`** | 8 | ECU | 暖気補正など | **OBS · HYP** |
| **`0x756`** | 8 | **噴射** | 停止≈0・始動立ち上がり・常時変化 → **役割 MATCH**。式は ROAD | **OBS · MATCH(役割) · HYP(式) · ROAD** |
| **`0x757`** | 8 | 灯火? | Hi 未確定（B6=`03` 長さ不一致）→ **HW**。ウィンカー点滅は本バス **NEG** | **OBS · HYP · HW · NEG(ウィンカー)** |
| **`0x758` `0x759` `0x75F`** | 8 | 開度/環境? | APS 単独放送は本タップで未検出（NEG） | **OBS · HYP · NEG(APS)** |
| **`0x760`–`0x763`** | 8 | ABS/輪速? | `761`/`762` 停車ゼロ → **ROAD**（前後別） | **OBS · PH · ROAD** |
| **`0x764`** | 8 | **電源・電圧** | 停止/発電切替 MATCH。**残量ではない（NEG）** | **OBS · MATCH · NEG(残量)** |
| **`0x765`** | 8 | 車体? | B4 微動 · 輪速本体ではなさそう | **OBS · HYP · HOLD** |
| **`0x769`–`0x76E`** | 8 | メーター/CCU/SW | 残量としては未証明（NEG）· 再探索は **RES** | **OBS · HYP · NEG(残量) · RES** |
| **`0x7E0` `0x7E8`** | 8 | UDS | CF22 積算≈0.50/s · ml/count **LONG**。オド2913 **NEG** | **OBS · HYP · LONG · NEG(オド)** |

### `0x27C` 値の要約

| Byte | 内容 | Tag |
|------|------|-----|
| 0–1 | RPM raw BE · 仮置き ×0.4 | MATCH(フィールド) · HYP(式) · ROAD |
| 3.0 | スターター | MATCH 寄り |
| 3.5 | サイドスタンド 0=出し / 1=収納 | MATCH |
| 3.6 | ギアイン（段判定に使わない） | HYP |
| **4** | **ギア** 0=N · 1–6 · 7=中間 | **MATCH** |
| 6 | `0x30`=停止 · `0xB0`=稼働 | MATCH |

---

## Working rules

1. Tag の **MATCH** 以外は製品ロジックに直結しない。  
2. **ROAD / LONG / RES / HW** は次の実車条件が必要。  
3. 送信しない（Listen-Only）。

---

## Changelog

| Date | Note |
|------|------|
| 2026-11-23 | Tag に検証状態を付与。停車再レビュー反映。他車種との差分言及を削除 |
| 2026-11-23 | 初版を仮説カタログとして再定義 |
