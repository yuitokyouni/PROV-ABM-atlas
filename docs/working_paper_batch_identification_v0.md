# 市場設計改革によるトレーダー行動機構の識別 —— バッチオークションをモデル弁別実験として

### Identifying Trader Behavior through Market-Design Reform: The Batch Auction as a Model-Discrimination Experiment

**ワーキングペーパー草稿 v0（2026-06-13）**。投稿候補: 日本銀行 IMES / JPX ワーキングペーパー / JEDC。
本草稿は3案（P1 バッチ脱共役・P2 学習 vs 固定・P3 microstructure facts）の統合分析。実装と
再現コードは `toy/channel_band.py`・`experiments/runners/band_demo.py`、finding は
`docs/findings/0001-channel-decoupling-band.md`。

---

## Abstract

Agent-based models (ABMs) are increasingly used to evaluate market-design policies such as
tick-size changes and frequent batch auctions. A foundational obstacle is **identification**:
competing ABMs calibrated to the same historical data can be observationally indistinguishable,
yet imply different policy counterfactuals. We show that this obstacle has a structural form —
models that respond differently to interventions tend to be distinguishable *without* the
intervention, while models that are observationally equivalent tend to respond *identically*
(the "two-horns" dilemma, demonstrated empirically from both sides). We then exhibit a
constructive escape: in a continuous market the price channel and the order-flow channel carry
**identical information** (return = λ × order-flow), so a price-driven model and an
order-flow-driven model are observationally indistinguishable. A **batch auction decouples the
two channels** (within a batch the price is fixed while order flow accumulates), making the
models sharply distinguishable. Thus a batch-auction reform functions as an *identification
experiment*: it reveals which behavioral mechanism describes the market — information that no
amount of continuous-market history can provide. We demonstrate this for (i) price- vs
order-flow-reading traders, (ii) learning vs fixed-rule traders, and (iii) the choice of
measured facts (microstructure vs return-distribution), and discuss the policy implications for
JPX/BoJ market-design decisions.

---

## 1. はじめに —— 政策分析における識別問題

エージェント型モデル（ABM）は、tick size 変更・frequent batch auction・speed bump といった
市場設計政策の評価に、JPX・日銀を含む各国当局で実際に用いられてきた。だが ABM を政策に使う際の
**根源的な障害は識別（identification）**である:

> 同じ歴史データに較正された複数の ABM が、歴史データ上は区別不能でありながら、政策介入下では
> **異なる反実仮想（counterfactual）を予測する**。どのモデルを信じるかで政策結論が変わるのに、
> 歴史データはどれが正しいかを決められない。

本稿の貢献は、この識別問題が**構造的な形**を持つことを示し、その上で**政策改革そのものを
識別実験として用いる**構成的な解を提示することである。中心となる政策レバーは **batch auction
（一括清算オークション）**であり、これは Budish, Cramton & Shim (2015) 以来の市場設計論争の核で
あると同時に、JPX の立会・tick 政策に直結する。

## 2. 識別問題の構造 —— 二本の角

ABM 弁別の難しさは、二つの失敗モードの間に挟まれている（両側を本研究プログラムが実証した）:

- **角 A（応答が同じ）**: 競合モデルが同一コア方程式のパラメータ変種だと、介入応答が同一になり
  弁別できない。先行研究 PRISM（撤退済）の自然実験4件で、4モデルの介入 delta が ~10⁻⁴ に縮退し、
  弁別力ゼロになったのがこれ（同モデルの `excess_demand = w_f·d_fund + w_c·d_chart + w_n·d_noise`
  変種）。
- **角 B（観測で既に分かれる）**: モデルを genuinely 別機構（トレンド追随 vs 群衆模倣）にすると、
  介入を待つまでもなく**学習判別器（1D-CNN）が生の価格系列から ~0.9 の精度で区別**する。介入が
  識別に寄与する余地がない。

> **介入による識別の価値は、「観測等価 ∧ 介入で分離」という細い帯にしか棲まない。**

本稿はこの帯の**構成的な実例**を、実在の政策（batch auction）で作って見せる。

## 3. 中核機構 —— チャネル脱共役

### 3.1 連続市場では価格と注文流は同一信号

単一資産の超過需要市場 `p_{t+1} = p_t · exp(λ · ED_t / N)` では、対数 return と集約注文流が

> **return_t = λ · ED_t / N = λ · (order-flow)_t**

と**厳密に比例**する（数値実測でも corr = 1.000）。したがって:

- **価格チャネルを読むトレーダー（A）** と **注文流チャネルを読むトレーダー（B）** は、
  連続市場では**同一の正規化シグナル**を見る（モメンタムは尺度不変）。
- 同一 seed で **bit 同一の市場軌道**を生む（`test_channel_band.py` で固定）。歴史データ・分類器で
  いくら調べても区別できない —— **角 B を回避**。

### 3.2 batch auction が両チャネルを脱共役

batch auction（interval N 期ごとに注文を集約し一括清算）では、**バッチ内は価格が動かない
（return = 0）が、注文流は毎期蓄積する**。よって:

- **A（価格 reader）**: バッチ内の flat な価格を見て沈黙し、境界の跳びにのみ反応。
- **B（注文流 reader）**: 注文流を毎期見て活動を続ける。

両機構が**別挙動**になる —— **角 A を回避**。**連続では同一、batch では分岐** という非対称が、
識別の帯そのものである。

## 4. 三つの分析

すべて `toy/channel_band.py`（self-contained・vectorized・決定論）上で実施。識別精度は SF1-4
要約統計のロジスティック回帰（LR）と生系列の 1D-CNN の 5-fold CV accuracy（0.5 = 区別不能、
1.0 = 完全識別）。M = 60 runs/条件。

### 4.1 案1 —— バッチ強度の識別 dose-response（中核）

価格 reader A vs 注文流 reader B を、batch interval N を変えて識別:

| batch interval N | LR (SF1-4) | CNN (生系列) | 解釈 |
|---|---|---|---|
| 1（連続=政策なし） | 0.38 | **0.46** | 区別不能 |
| 2 | 0.94 | **0.94** | わずかな batch で既に識別 |
| 5 | 0.93 | 0.95 | |
| 10 | 0.87 | 0.94 | |
| 20 | 0.55 | **1.00** | ほぼ完全識別 |
| 50 | 0.98 | **1.00** | 完全識別 |

**結果**: 識別力は政策強度の階段関数。連続（N=1）では原理的に区別不能なモデルが、**2期集約という
最小のバッチでも CNN=0.94 で識別**でき、強いバッチ（N≥20）で完全識別に達する。

> **政策含意**: バッチオークション改革は、市場が「価格に反応する」機構で動くか「注文流／板に
> 反応する」機構で動くかを、歴史データでは決まらないにもかかわらず、**改革への応答が一意に決める**。
> 改革は市場設計の選択であると同時に、行動機構の**識別実験**である。

### 4.2 案2 —— 学習トレーダー vs 固定ルールトレーダー

固定ルール（価格チャネル固定）vs 適応型（直近で信号の強いチャネルを各自選ぶ最小学習則）を識別:

| batch interval N | LR | CNN | 解釈 |
|---|---|---|---|
| 1（連続） | 0.40 | **0.47** | 固定 ≈ 適応、区別不能 |
| 10 | 0.83 | 0.94 | 分岐 |
| 20 | 0.56 | **1.00** | 完全識別 |

**結果**: 連続市場では両チャネルが同一なので、適応型は学習する対象がなく固定型と**観測等価**。
batch 下では適応型が**情報のある注文流チャネルを自分で発見して移行**し、固定型と分岐する。

> **政策含意**: 市場が**学習する**トレーダーで動くか**固定ルール**で動くかは、平時データからは
> 識別不能だが、バッチ改革が暴く。これは政策的に load-bearing —— アルゴ取引の高度化（学習化）が
> 市場設計改革の効果を変えるか、という日銀・JPX の関心そのものに、識別の手段を与える。

### 4.3 案3 —— microstructure facts vs return-distribution facts（PRISM の修復）

先行 PRISM（撤退済）は、tick/取引税という **microstructure 介入**を、**return-distribution
facts**（GARCH persistence・kurtosis・歪度・ACF）で測ったため category mismatch を起こした。
有効な自然実験（JPX 2014 tick、40 treatment + 20 control 銘柄、12ヶ月）でも全6 facts の CI95 が
ゼロを跨ぎ、**0/6 inconclusive** で死んだ（`PRISM/docs/FINAL_REPORT.md`）。介入の信号は spread・
depth・price impact という microstructure 量にあり、日次 return 分布にはほぼ出ないためである。

本案は、注文流の microstructure facts（流れのボラ・自己相関・尖度）が return facts より敏感かを
toy で測った:

| 介入強度 N | return facts (PRISM式) | microstructure facts |
|---|---|---|
| 1（連続） | 0.34 | 0.40 |
| 2 | 0.95 | 0.95 |
| 5 | 0.96 | **0.97** |
| 10 | 0.82 | **0.88** |

**結果と正直な限界**: microstructure facts は return facts に対し**一貫して同等以上、弱い介入下
（N=10）で僅かに優位**（0.88 vs 0.82）。だが PRISM 実データで見られた「return facts が完全に
失敗」という劇的な乖離は、この toy では**再現しない** —— 本 toy のバッチ介入は return 分布も
大きく動かすため（案1の通り A/B が return で分岐する）、PRISM の category mismatch の clean な
アナログになっていない。**案3 の強い証拠は toy ではなく (i) PRISM の実データ（return facts で
0/6）と (ii) microstructure 文献**（tick 変更が spread/depth を動かすことを Aquilina et al. 2022・
Comerton-Forde et al. 2019 が実測）である。toy での clean な実証には order book（spread・depth）を
持つ microstructure シミュレータ（P2 のインフラ）が要る。**3案中、toy 上での実証力は案3 が最も弱い。**

## 5. 政策的含意（BoJ / JPX）

1. **batch auction を診断装置として**: 市場設計改革は副作用を持つ介入であると同時に、市場の
   行動構造を**観測可能にする**装置である。連続市場では潜在的（latent）な「トレーダーが何を見て
   いるか」「学習しているか」が、バッチ下で**顕在化**する。改革の事前評価そのものに、改革後の
   観測でモデルを検証する identification が内蔵される。
2. **JPX への直接の関連**: JPX は tick size・立会方式で人工市場 ABM を政策に用いてきた。本枠組みは
   「どの ABM を信じるか」を、改革の前後データで決める手続きを与える。
3. **頑健性の警告**: 本結果は、ABM ベースの政策助言が、歴史データでは決まらない latent な行動仮定
   （価格 vs 注文流、学習 vs 固定）に**敏感**でありうることを示す。改革効果の符号がこの仮定で
   反転しうる領域を事前に地図化することが、政策の頑健性に資する。

## 6. 本研究プログラムとの接続

- **P2（共謀×市場設計、`ABM-Microstructure`）との連結**: P2 は batch vs continuous × 学習 MM ×
  実 venue（BCS ES–SPY）較正で、algo 共謀の移植可能性を監査する。本稿（P1）はその
  **identification 面**を担う —— 「P2 の介入（batch）が機構を識別する」ことの構成的基盤。両者は
  同じ batch という政策レバーを、片や共謀耐性、片やモデル識別の観点から扱う。
- **理念の保持**: 「従来モデルが出来なかった切り分けを介入で行う」という当初の理念を保ったまま、
  弁別対象を「異機構の SF-等価ペア（構造的に詰む）」から「**観測等価だが reach の異なる機構 ×
  脱共役政策**」へ sharpening した。

## 7. 限界と実証への道

1. **styl化**: 単一資産・離散行動・閾値モメンタム。order book（価格・注文流・板厚の3チャネル以上）
   への richening で、より現実的な reach-twin が組める（P2 のインフラ）。
2. **batch 下で固定型が沈黙気味**: 両機構が active なまま分岐する設計（境界跳びに反応する momentum、
   観測遅延等）への精緻化。
3. **実データ照合（essence の本番）**: 「どちらが現実か」は、実在の batch/tick 改革（JPX 2014 等）
   の前後で、どちらのモデルの予測が当たるかで決める。PRISM の category mismatch を避けるため、
   microstructure 量を intraday（J-Quants 等）で測る（案3 の方向）。
4. **事前登録**: 本草稿は探索的。GO 後、検出力設計・null・縮退規則を paper-grade で OSF 事前登録
   （プログラム共通要件）。

## 8. 結論

ABM を市場設計政策に用いる際の識別問題は、二本の角という構造的形を持つ。だが連続市場における
価格チャネルと注文流チャネルの共線性ゆえに、**観測等価だが何を見ているかが異なる**機構ペアが存在し、
**batch auction がその両チャネルを脱共役して識別する**。よってバッチ改革は、市場の行動機構を
歴史データでは決まらない形で顕在化させる**識別実験**である。これは「政策分析としての ABM 弁別」の
最も clean な vehicle であり、JPX/日銀の市場設計政策に直接の含意を持つ。

---

*再現*: `uv run python -m experiments.runners.band_demo`（案1）。案2/3 のスクリプトは本草稿確定後に
runner 化。数値は M=60、seed 固定で決定論的に再現可能。
