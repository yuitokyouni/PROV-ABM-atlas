# P1 paper-grade 事前登録 — draft v0（計算なし・起草のみ）

**Status**: draft v0（2026-06-11）。**登録は toy §14 GO の後**（claims v1.3 §2.3 の
二層構造）。本書は claims v1.3 を執行可能な事前登録文書に落とす起草であり、確定時に
v1 として凍結 → frozen PDF → OSF 登録（公開タイムスタンプ）→ 実行、の chain を踏む。
P2（ABM-Microstructure）で実地確立した統治様式を §6 で移植する。

## 1. 主張と family（claims v1.3 §2 を執行形に）

- 目標文・三層構造・縮退規則は `program_claims_v1.md`（v1.3）§2 が一次ソース。
- 識別 family: 少なくとも一方が観測チャネルを持つペア × 4 scheme。Genoa 採用時
  **13 ペア × 4 = 52 検定**（Holm、family-wise α = 0.05）／不採用時 8 ペア × 4 = 32。
  (H, ALW) は識別 family 外の**順序予測 probe**（fundamental masking 軸、v1.3 fix）。
  (CB, FPZ-ZI) は等価予測側（TOST）。
- 検出力: MDE = chance+15pp、1−β ≥ 0.9 @ Holm 最悪 α'、n ≥ 440 runs/条件。

### 1.1 全ペア conjunct の k-of-n 化（P2 教訓の事前適用）

「全ペアを識別」は all-n conjunct であり、per-pair 検出力 0.9 でも全成功確率は
0.9¹³ ≈ 0.25——**降格がモーダルになる主張を最初から書かない**（P2 Tier-3 の収束
全数条項と同型の脆さ）。維持条件を確率算術から逆算して固定する:

- **主張の成立 = 識別 family 13 ペア中 k = 11 以上で個別有意（Holm 下）**。
  根拠: per-pair 検出力 0.9 の下で P(X ≥ 11 | n=13, p=0.9) ≈ 0.87 ≥ 0.8。
  k = 12 だと 0.62 で不足。全 13/13 は「強い枝」として併記する（条件ではない）。
- Genoa 不採用時（8 ペア）: k = 7（P(X≥7|8,0.9) ≈ 0.81）。
- 解釈の非対称を同時に固定: k 未達 = 「k-of-n 維持条件の非生存」であって
  「識別不能」ではない。個別ペアの結果は全て報告（落ちたペアの同定自体が
  Atlas の弁別地図の情報）。

## 2. 機構集合と導出（提示ではなく導出）

### 2.1 Model T = 正典チャーティスト需要 block の単離

- 導出: FW / LM / Chiarella-Iori に共通のチャーティスト需要 block を、**switching の
  凍結（混合比率の定数化）= removal-only** で単離。追加方程式ゼロ。
- **対応表（確定時に式番号まで埋める）**: T の各方程式 ↔ 出自（CI 2009 eq.(x) /
  FW の対応項 / LM の対応項）、凍結したパラメータ（switching 強度 → 0）の一覧。
- 共有 chassis（CI 価格更新 + fundamentalist 錨 + noise）は H と同一に保つ
  （v0.2 死因の回避: 単機構孤立にしない）。

### 2.2 Model H = Kirman 1993 遷移率 block（= ALW noise-trader block）

- 導出: ALW chassis から fundamentalist 需要を凍結し、Kirman 1993 遷移率
  a + b·n_k をそのまま herding block として残す。対応表同上。

### 2.3 ALW アダプタ仕様（composite 古典）

- 実装: toy H に fundamentalist block を**戻す**導出（H の対応表の逆操作、
  追加方程式は ALW 原典の fundamentalist 需要のみ）。
- Model Contract 適合: channels = ("aggregate_action", "price")（herding は集約
  行動を、fundamentalist は価格乖離を観測）——宣言は実装前に固定し、ctx ログ
  突合（C2）で検証。
- **(H, ALW) 順序予測 probe**: fundamental masking 軸（scheme 適用 channel =
  "price"）で、ALW の応答 > H の応答（H は fundamentalist 錨が無く price channel
  の degrade に鈍感）という**順序**を事前予測。検定は順序仮説の片側（identification
  claim ではなく、block 加算の検出可能性の demonstration）。
- SF 等価: ALW も同一 battery を TOST 通過すること（calibration は §3 の手続き）。

### 2.4 陰性対照と条件付き機構

- **ZI**: Farmer-Patelli-Zovko (2005) 原典 audit 通過版のみ正準（port 草稿は audit
  まで非正準）。SF battery 通過は期待しない（v1.2 撤回済み）。役割 = IR-flat。
- **CB**: 新規実装。{CB, ZI} 応答等価類の事前予測（TOST）。SF 通過は非自明 →
  階層縮退（v1.3 §2.2）。
- **Genoa-ZI+**: 自前実装が SF battery（SF1–4 TOST）を通過した場合のみ集合に
  加える条件付き gate。採用/不採用で family と k が変わる（§1.1 に両系を事前記載）。

## 3. 敵対的 SF マッチング手続き（calibration の事前登録）

calibration を「分類器を欺く敵対探索」として定式化し、leakage を遮断する:

1. **凍結**: SF battery（SF1–4）と SF 分類器（アーキテクチャ・学習手続き・
   ハイパーパラメータ）を探索開始前に凍結（commit pin）。
2. **探索**: 各機構ペアについて、凍結分類器の CV accuracy 最小化を直接目的とする
   敵対探索（SF 距離は探索の proxy に使ってよいが判定には使わない）。探索予算
   （trial 数 × runs/trial）を事前固定し、予算 ledger（§6）で enforce。
3. **判定**: 探索で得た較正点で**新規 seed の holdout**（n ≥ 440/条件、探索に
   使った run は判定に再利用しない）を生成し、TOST（CV accuracy の 90% CI ⊂
   [0.45, 0.55]）で等価性を判定。
4. **縮退**: TOST 不成立の機構は SF 等価集合から階層縮退（識別はできるが
   「SF では分けられない」前提を失う——その機構のペアは family から除外し、
   k を §1.1 の算術で再導出した値に置換。置換表を事前記載）。
5. SF5/6 は post-equivalence 独立検証量（calibration に使わない、v0.3 踏襲）。

## 4. 介入応答プロトコルと判定

- B2 介入 4 scheme（average / EMA / noise / delay）× graded θ、θ=0 恒等性の
  property test（Model Contract §2.3）を全アダプタの受け入れ条件にする。
- IR 分類器も §3.1 と同じ凍結規律（結果を見てからの再学習・再チューニング禁止）。
- 判定は §1 の family / k-of-n / TOST を機械適用。**全判定スクリプトは commit pin
  し、人手の再判定をしない**（P2 の certify 機械適用と同じ規律）。

## 5. 報告様式

- 全指標 seed 横断 mean ± SE + n。**null は必ず MDE 併記**（「検出されない」は
  「MDE = X の下で検出限界以下」としてのみ主張。棄却失敗 ≠ ゼロ。逆符号の点推定
  ペアを「相殺」と書かない）。
- 確率算術の事前注記様式（P2 prereg-tier3 §2.5 型）: conjunct を含む全ての維持
  条件について、成分ごとの判定構造（プール/合格率/全数）と維持確率の算術を
  登録文書に明記する。

## 6. 統治様式の移植（P2 で実地確立、2026-06-11）

1. **追記台帳**: 予算 ledger は追記専用 journal（1 行 1 イベント）を一次記録とし、
   snapshot は検証付きキャッシュ（`rebuild`/`verify`）。スナップショット上書きは
   台帳ではない。
2. **同一性キー = フル構成 hash**: 結果行は表示用 id でなく config hash でキー
   （摂動軸の追加で衝突しない）。resume は hash 照合。
3. **未読破棄の手順**: パイプライン欠陥を発見したら、部分結果を読まずに破棄・
   修正・監査 entry 付き精算・決定論再投入（0002a の手順を標準化）。
4. **汚染時の有界化**: 数値の修正と判定の無汚染証明を分離し、後者は
   「真値ピーク < cap」型の delta 非依存の論理で書く。
5. **予算**: tier 別 cap を事前固定し機械 enforce（数値は toy GO 後の確定時に、
   §3 探索予算と合わせて記入）。

## 7. 登録時点で存在するデータの開示（確定時に更新）

- toy: SF battery 単体測定、SF-等価 calibration 探索の中間結果（T* 固定、H 候補
  dist=2.74、SF2 gap 未解決）。SF/IR 分類器の判定結果・介入応答は未生成。
- paper-grade 機構（ALW/CB/ZI 正準版/Genoa）は未実装・未較正。
- 本 draft 自体の git 履歴が内部タイムスタンプとして OSF 登録に先行する。

## 8. 確定までの残作業（計算なしで進められるもの）

- [ ] T/H 対応表の式番号埋め（CI 2009 / FW / LM / Kirman 1993 / ALW 2008 の原典固定）
- [ ] ZI の FPZ 原典 audit チェックリスト（order-flow 仕様の条文化）
- [ ] Bouchaud 系レビュー・ZI-Plus・Genoa の正確な引用 pin（v1.2 の宿題）
- [ ] 縮退時の k 置換表（機構脱落の全組合せ × k 再導出）
- [ ] 探索予算・tier cap の数値（toy GO 後、wall-clock 実測から）
