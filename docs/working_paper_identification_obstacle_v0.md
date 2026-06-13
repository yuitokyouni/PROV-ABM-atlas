# 観測等価と介入弁別性 —— 同じ点・異なる接ベクトルとしての定式化

### Observationally Equivalent, Interventionally Distinct: A Sensitivity Formulation

**ワーキングペーパー草稿 v2（2026-06-13）**。狙い: 主論文（JEDC）。P1 メイン（「SF 等価な異機構を
介入で識別」）の、より深く正しい再定式化として統合する候補。本稿は §2 の定式化と §5 の予測まで。
§6 の検証（感度の測定法 + order book での構成）は次段。

> **改訂史**: v0 は3試行の失敗を「構造的に不可能」と書いた（過剰一般化、撤回）。v1 は「観測量クラスは
> 政策の作用点から演繹される」とした。v2 は v1 の「作用点演繹」基準の3つの穴（後述 §2.2）を受けて、
> 識別性を**観測量の θ 感度の機構間差**で定義し直す。作用点演繹は捨てず感度基準の下位に置く。

---

## Abstract

Whether two agent-based models (ABMs) are "observationally equivalent" is relative to an observable
class. We give a sensitivity criterion for which observables make a policy intervention identify
the mechanism, computable a priori from the constructed mechanisms and not from whether any quantity
turns out to respond. For mechanisms M1, M2 with the distribution of an observable O written F_k(θ)
under policy parameter θ, O is identifying for (M1, M2) at θ0 iff (i) F_1(θ0) ≈ F_2(θ0) — the two
models coincide in O at the baseline policy — and (ii) ∂F_1/∂θ ≠ ∂F_2/∂θ at θ0 — the policy moves O
differently for the two models. Condition (i) is "same point," condition (ii) is "different tangent
vector"; together they are the literal content of observational equivalence with interventional
distinctness. This criterion separates observables by the magnitude and the between-model difference
of their θ-sensitivity, not by whether their definition references θ (every price-derived quantity
does, to some order); it is computable before any data (the sensitivities are properties of the
constructed models); and it requires no single, injective "point of action," so it applies to
batch-auction and speed-bump policies whose action is diffuse in time. The apparent tension between
(i) and (ii) is not structural: two functions generically cross transversally — equal value, unequal
slope — and the three prior attempts failed because the observable they used collapsed the crossing
(raw trajectories force equal value to mean identical function; return moments give equal slope
across models). Two conditions bound the criterion: it requires θ-smoothness of F (a discrete,
non-monotone response, as in our N=20 artifact, is where it fails), and O must be a pre-specified
small set of decision-relevant observables (searching arbitrary functionals re-introduces the
circularity at the level of observable choice).

---

## 1. 問題

ABM を市場設計政策の評価に用いる前提は「歴史データでは区別できない機構を、政策介入への応答で区別
できる」である。これは2要件を要する: (a) 2モデルが観測データで区別不能（でなければ介入は冗長）、
(b) 介入下で応答が異なる。本稿は、(a)(b) を満たす観測量を**機構の構成から事前計算できる**基準で特徴
づける。

## 2. 識別性の感度基準

### 2.1 観測等価は観測量クラス O に相対的

「観測等価」は絶対概念でなく観測量クラス O に相対的で、「O に属する量の下で2モデルが区別できない」を
意味する。経済学の観測等価は「意思決定者が利用可能な統計量の下で区別できない」であり、「生の有限
trajectory を任意の検出器で見て少しでも違えば区別可能」ではない。後者を基準にすると相異なる確率過程は
ほぼ全て区別可能になり equifinality が空になる。因果推論も、観測変数の分布（特定の O）の一致を介入で
区別する。観測等価は O に相対的である。

### 2.2 識別的観測量 = θ 感度が機構間で異なる量

機構 M1, M2（市場結果を生成する確率過程）、政策パラメータ θ（tick size、batch interval）、観測量 O
（市場結果の汎関数）を考える。Mk の下での O の分布を F_k(θ) と書く。

> **定義（識別的観測量）**: O が政策 θ と機構ペア (M1, M2) に対して baseline θ0 で**識別的**であるとは、
> (i) **F_1(θ0) ≈ F_2(θ0)**（O の分布で観測等価）かつ (ii) **∂F_1/∂θ|_{θ0} ≠ ∂F_2/∂θ|_{θ0}**（θ に
> 対する O の動き方が機構間で異なる）こと。

(i) は「同じ点」、(ii) は「異なる接ベクトル」。両者は "Observationally Equivalent, Interventionally
Distinct" の文字通りの数学的意味である。

この定義は、前版 v1 の「O が θ を定義上参照するか」基準の3つの穴を塞ぐ:

- **穴1（二値では連続な依存度を割れない）**: return も価格から計算され、価格はグリッド θ 上にある
  （bid-ask bounce・price discreteness が return の自己相関・分散に θ 依存を与える）。よって return
  moment も θ を定義上参照し、「参照の有無」では spread と分けられない。感度基準は参照の有無でなく
  ∂F/∂θ の大小で測るので、return も spread も同一尺度で比較でき、機構間差を連続量で言える。
- **穴2（後知恵）**: ∂F_k/∂θ は機構を構成すれば計算できる量で、実データの検出結果を参照しない。PRISM の
  失敗は「return moment の ∂F/∂θ が4機構間でほぼ同一（~10⁻⁴ に縮退）」と**事前計算可能**な形で言える
  （これは既に測られた縮退そのもの）。「作用点に近い」は結果非依存に言えても「検出される」を導けず、
  v1 は実質 Aquilina 等の実測に依拠していた。感度基準は ∂F/∂θ の機構間差を直接計算するので、この橋を
  実データに頼らず架ける。
- **穴3（作用点が非単射）**: batch auction は単一の作用点対象を持たず、清算タイミングに作用して価格・
  注文流・約定タイミングの複数量に同時に効く。感度基準は単一作用点を要求せず、「どの O の感度が機構間で
  分かれるか」を直接計算するので、作用点が時間構造に分散する政策にも適用できる。

作用点演繹は捨てず感度基準の下位に置く: **作用点に近い O は経験的に θ 感度が高い傾向があるが、決定的
なのは近さでなく機構間の感度差であり、それは構成すれば計算できる。** 作用点はどこを**探すか**を導き、
感度基準が**決める**。

### 2.3 「2要件の緊張」は、誤った O が横断交差を潰した症状

(i) と (ii) は一般には緊張しない。2つの関数 F_1, F_2 が θ0 で一致し、そこで異なる微分を持つ（横断的に
交わる）のは generic で、特別な事象ではない。前版の「構造的障害」という見え方は、**誤った O が交差を
潰した症状**である:

- 強すぎる O（生 trajectory の任意差）: F_1(θ0)=F_2(θ0) が関数の恒等を強制 → ∂ も同一 → (ii) 不成立
  （§3.1 channel-band）。
- 非識別的 O（PRISM の日次 return moment）: ∂F が機構間で縮退 → (ii) 不成立（§3.3 PRISM）。
- 識別的 O（機構間で ∂F が異なる microstructure 量）: θ0 で値を合わせ微分を分ける横断交差が generic に
  存在しうる。その O が、政策関連かつ事前指定の小集合の中に在るかが §5 の問い。

### 2.4 2つの適用条件

- **θ 滑らかさ**: 感度基準は ∂F/∂θ を要する。θ が離散（batch interval N）だと微分は差分になり、F(θ) が
  θ で非滑らか/非単調な領域では感度が well-defined でない。我々の batch dose-response の非単調（N=20 で
  生 CNN=1.00・要約統計 LR=0.55）は、この条件that破れた領域で CNN が感度でなく不連続（バッチのゼロ配置）を
  読んだ例と再解釈できる。感度基準は θ 応答が滑らかな領域でのみ適用する。
- **O の事前指定（過適合の防止）**: 感度基準は「∂F が機構間で異なる O を探す」を許すので、O を任意の
  汎関数まで広げると識別的 O を自明に見つけられ、「識別的 O を後から選んだ」という循環（介入選択の循環の
  O 版）that再発する。これを塞ぐため、**O は意思決定者が実際に用いる経済的に意味のある観測量の、事前
  指定された小集合に限る**（§2.1 の経済学的定義と作用点ヒューリスティックが選択を導き、事前登録する）。
  任意汎関数の探索は禁じる。

## 3. 3つの試行 —— 感度基準による再解釈

### 3.1 channel-band（強すぎる O が横断交差を潰した）

単一資産市場 `p_{t+1}=p_t·exp(λ·ED_t/N)` では return = λ·ED/N で価格と注文流が同一信号。価格を読む A と
注文流を読む B は連続市場で bit 同一の出力を生む。観測等価を生系列（最強の O）で定義したため、
F_A(θ0)=F_B(θ0) が関数の恒等を強制し、∂F も同一になった（再パラメータ化）。識別的 O の (ii) が原理的に
立たない。order book では price は net flow の決定論的関数でなく、価格と注文流は別の O になりうる。

### 3.2 T/H（強すぎる O）

異機構ペア。要約統計クラスでは F を一致させられた（LR=0.58）が、生系列 CNN（最強の O）が 0.85–0.91 で
分離した。要約統計も tick/batch の感度が機構間で分かれる保証がなく、識別的 O として選ばれていない。

### 3.3 PRISM（非識別的 O）

tick/取引税の効果を日次 return moment で測り、4機構の ∂F/∂θ が ~10⁻⁴ に縮退（モデルが同一方程式の
パラメータ変種で独立でない）。(ii) が立たない。これは §2.2 穴2 の通り、機構を構成すれば事前計算できる
縮退であって、実データの非検出を待つ必要がない。

## 4. 主張しないこと

- **一般的不可能性を主張しない**。識別的 O が存在するかは機構ペアと政策に依存する経験的問いで、§2.3 の
  通り横断交差は generic である。
- **ABM が道具として終わっているとは主張しない**。ABM は P2 で政策含意のある結果を出している。終わって
  いるのは「SF 再現が機構を識別する」パラダイムである。

## 5. 反証可能な予測と事前登録分岐

> **予測**: 事前指定した政策関連の観測量クラス O（order book の spread・depth・price impact、intraday）に、
> genuinely 異機構ペア (M1, M2) に対し識別的な O が存在する。すなわち F_1(θ0)≈F_2(θ0) かつ ∂F_1/∂θ≠∂F_2/∂θ。

**成功判定（事前登録）**: (i) F_1(θ0)≈F_2(θ0) は O での TOST 等価（差の CI が事前帯内）。(ii) ∂F_1/∂θ≠∂F_2/∂θ
は、θ0 と θ0+Δ での O の変化量が機構間で ±2SE を超えて異なること（Δ は §2.4 の滑らかさが成り立つ範囲）。

**分岐（後知恵の逃げ場を塞ぐ）**:
- **A（支持）**: O で (i)(ii) ともに成立。
- **B（反証）**: (i) は成立するが (ii) 不成立（∂F が機構間で一致）。→ 識別性は弱い形に後退。
- **C（設計失敗）**: (i) が成立するが2機構が再パラメータ化に潰れる。→ genuinely 異機構でなく設計やり直し。
- **D（検証不能）**: O で (i) の等価化 calibration that達成できない（spread/depth/impact の同時分布を2機構で
  TOST 通過まで合わせられない）。→ 「この設計では検証できなかった」と報告し、反証とも支持とも主張しない。
- **滑らかさ違反**: Δ を取る θ 範囲で F が非滑らか（§2.4）なら、その範囲を除外し、滑らかな範囲で判定する。
  除外して滑らかな範囲が残らなければ分岐 D 扱い。

## 6. 検証設計（次段、感度の測定法 + order book）

未確定の詰めどころ:
1. **∂F/∂θ の測定法**: 候補 = (a) 各観測量モーメントの θ 感度ベクトル（有限差分）を機構ごとに出し、機構間
   差を検定、(b) 分布間距離（Wasserstein 等）の θ 微分。刃: 有限差分の Δ 選択（小→ノイズ、大→非線形/非
   滑らか §2.4）、分布微分の高次元性（事前指定 O の低次元モーメントに射影して回避）。
2. **order book での構成**: 板厚・約定過程を持つ市場（P2 のインフラ）で M1, M2 を構成し、O = spread/depth/
   impact で §5 を機械判定する。
実データ照合は同じ O を実在改革に適用する段（`docs/realdata_method_and_p3_coherence.md`）。

## 7. 含意

ABM の検証論は「観測等価」を観測量クラスを明示せず用いてきた。SBI 批判は暗にそれを最強検出器へ押し上げ
equifinality を空にした。本稿は、識別的観測量を**θ 感度の機構間差**で定義する: 観測等価（同じ点）と介入
弁別（異なる接ベクトル）は横断交差として generic に両立し、決めるのは観測量の感度であって検出器の強弱
ではない。機構識別を主張する ABM 研究は、用いる事前指定 O と、その O での感度の機構間差（構成から事前
計算可能）を示さねばならない。Fagiolo, Moneta & Windrum (2007) 系の検証方法論に接続する。

## 8. 関連
- 3例: PRISM（内部・撤退済）、T/H（`docs/program_claims_v1.md`、Issue #11）、channel-band
  （`toy/channel_band.py`、`docs/findings/0001-channel-decoupling-band.md`、機構識別の解釈は撤回済）。
- 検証設計・実データ・P3: `docs/realdata_method_and_p3_coherence.md`。
- 文献: Fagiolo, Moneta & Windrum (2007); Aquilina, Budish & O'Neill (2022);
  Comerton-Forde, Gregoire & Zhong (2019); Guerini & Moneta (2017); Cranmer, Brehmer & Louppe (2020)。
