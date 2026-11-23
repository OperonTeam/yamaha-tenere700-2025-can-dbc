# Yamaha Tenere 700 (2025+) CCU CAN — **仮説カタログ**

2025年式 ヤマハ テネレ700（Euro5+ / 6.3インチ TFT / YCC-T）の  
**CCU（コミュニケーションコントロールユニット）コネクタ**で観測した CAN の  
**仮説ベース**の整理ドキュメントおよび DBC です。

> **このリポジトリの位置づけ**  
> 確定仕様ではありません。停車ベンチ＋短時間ダンプからの **仮説（hypothesis）** です。  
> 実走行検証・長時間ログ・独立計測（メーター表示・別センサ）が揃うまで、  
> 信号名・計算式は **すべて仮** として扱ってください。

An independent, community-driven **hypothesis catalog** and DBC for the **2025+ Yamaha Tenere 700**, observed via the CCU connector. **Not a confirmed OEM specification.**

---

## ⚠️ Disclaimer / 免責

* **Passive Observation Only** — Listen-Only。送信・インジェクションは行わない前提。
* **Hypothesis, not ground truth** — ID の存在観測と、バイト変化からの推測を混ぜています。推測は誤りを含みます。
* **No Liability** — 学習・ロギング用途。バスへの送信は自己責任・非推奨。

---

## Confidence legend / 信頼度

| Tag | 意味 |
|-----|------|
| **OBS** | 当該 ID がバス上に出現することを観測済み |
| **HYP** | バイト意味・計算式は仮説（未確定） |
| **CONFLICT** | 他ソース（例: [motocan R9](https://github.com/DeziderMesko/motocan)）や別セッションと解釈が衝突。要再検証 |
| **PLACEHOLDER** | 走行未検証。枠だけ定義 |

DBC 内の信号名も **仮説ラベル** です。ツール表示用であり、確定デコードを意味しません。

---

## Overview / 通信概要

| 項目 | 内容 |
|------|------|
| Tap | CCU コネクタ Y 分岐 · Listen-Only · 終端なし想定 |
| Architecture (HYP) | 2025 世代ヤマハ新電装（R9 / 近年 MT・T7 等）と **ID 帯が近い** |
| Endianness (HYP) | Big Endian 寄り · Byte 0 始まり |
| Evidence so far | キー ON · クランキング · アイドル · 空ぶかし · スタンド/ギア · CCU/ナビ操作 |
| Not done | **実走行**（車速・ABS・燃費の確定）· 長時間 · 独立校正 |

関連: 同世代の公開ベンチマークとして Yamaha R9 2026 の [DeziderMesko/motocan](https://github.com/DeziderMesko/motocan)（診断枝の放送デコード）。T7 と完全一致は未証明。

---

## Files

* `README.md` — 本ドキュメント（仮説カタログ）
* `yamaha_tenere700_2025_ccu.dbc` — Wireshark / SavvyCAN / `cantools` 用。**仮説ラベル付き**

---

## CAN ID catalog / 仮説一覧

凡例: 右端の Tag を必ず見る。

| CAN ID | DLC | Category (label) | Notes (仮説の要約) | Tag |
| :--- | :---: | :--- | :--- | :--- |
| **`0x27C`** | 7 | ECU 基本 | RPM: `((B0<<8)\|B1)*0.4` ≒ R9 の `*100/256`。ギアは **B4 下位**が 0=N … 6、7=中間、という解釈が有力（R9 と整合）。スターター/スタンド等ビットは **HYP** | OBS · **HYP**（ギア/RPMは比較的強い） |
| **`0x2A3`** | 8 | 制御系? | 停車中ほぼ固定（例 `00 08 00…`）。走行で動くかは **PLACEHOLDER** | OBS · PLACEHOLDER |
| **`0x383`** | 8 | 走行/ブレーキ? | 停車中ほぼ固定。R9 はブレーキビット候補。車速説は **CONFLICT/弱** | OBS · HYP · CONFLICT |
| **`0x512`** | 4 | 温度 **または** 同期 | **衝突あり。** R9 / 短時間ログでは `B0/B1` が水温・吸気温（`°C = raw*0.625-30`）として暖機と一致。本リポジトリ旧「同期カウンタ」説は格下げ | OBS · **CONFLICT** |
| **`0x58A`** | 8 | CCU 時計? | Y-Connect 時刻同期仮説。未同期時ゼロ埋めを観測しうる | OBS · HYP |
| **`0x58B`** | 8 | メーター同期? | アイドル・パディング仮説 | OBS · HYP |
| **`0x591`** | 8 | 電話セッション? | 接続時に値変化、という仮説 | OBS · HYP |
| **`0x592`** | 8 | メディア状態? | 同上 | OBS · HYP |
| **`0x594` `0x595`** | 8 | 通知マスク? | スマホ通知アイコン系の仮説 | OBS · HYP |
| **`0x59A`–`0x59C`** | 8 | ナビ文字? | マルチフレーム UTF-8 転送の仮説 | OBS · HYP |
| **`0x59D` `0x59E`** | 8 | UI 同期? | 画面モード/ハートビート仮説 | OBS · HYP |
| **`0x750`** | 8 | ECU 高速 | **燃料流量説は弱い。** R9 は RPM 百の位＋スロットル粗値寄り。要再検証 | OBS · HYP · CONFLICT |
| **`0x751`** | 8 | ECU ステータス | **暖気タイマ説は弱い。** R9 は B6 が N/クラッチ/スタンド系候補 | OBS · HYP · CONFLICT |
| **`0x755`** | 8 | ECU | 暖気補正など未命名。R9 も unknown 寄り | OBS · HYP |
| **`0x756`** | 8 | ECU | **噴射レート説は弱い。** R9 は後輪速。停車ログでは速度≈0と整合 | OBS · HYP · CONFLICT |
| **`0x757`** | 8 | 灯火? | ハイビーム仮説。未確定 | OBS · HYP |
| **`0x758` `0x759` `0x75F`** | 8 | 開度/環境? | **`758`/`759` を環境センサとする旧説は弱い。** R9 および空ぶかしログでは APS/TPS（開度）仮説が有力 | OBS · HYP · CONFLICT |
| **`0x760`–`0x763`** | 8 | ABS? | 前後輪速枠。停車中ゼロ/固定が多い。走行必須 | OBS · PLACEHOLDER |
| **`0x764`** | 8 | 電源? | 発電前後でバイト変化、という仮説 | OBS · HYP |
| **`0x765`** | 8 | 車体/前輪? | R9 は前輪速候補。停車中は要再検証 | OBS · HYP |
| **`0x769`–`0x76E`** | 8 | メーター/CCU/SW | タイマ・BT・スイッチ等は仮説。R9 は一部 PWR/クルーズ等 | OBS · HYP |
| **`0x7E0` `0x7E8`** | 8 | UDS | CCU が ECU へ問う診断トラフィックを **OBS**。周期は「0.5秒」固定ではない（短時間ログでは高頻度・複数サービス混在）。燃費DIDの中身は **HYP** | OBS · HYP |

---

## Working rules / 使い方

1. Excel / SavvyCAN / Wireshark で **ID の有無と変化**を見る用途には使ってよい。  
2. 燃費・車速・ABS・開度の **製品ロジックに組み込む前に**、走行ログと独立基準で確認する。  
3. 衝突タグ（CONFLICT）は、採用前に必ず切り分ける。  
4. 送信しない（Listen-Only / Sniff only）。

### 燃費について（特に仮説）

Y-Connect に燃費表示があることから、UDS 応答や未命名放送に関連量が載る **可能性**はある。  
ただしアプリ側 GPS 計算の可能性も残る。`0x7E8` マルチフレームの特定バイト＝積算燃料、は **未確定（HYP）**。

---

## Changelog

| Date | Note |
|------|------|
| 2026-11-23 | 初版を **仮説カタログ** として再定義。信頼度タグ・R9/短時間ログとの CONFLICT を明記。DBC コメントを仮説表記に更新 |
