# P1 paper draft v0 — ナラティブ骨格 + 全文 draft（結果 slot 形式）

**Status**: draft v0（2026-06-12）。Yuito のナラティブ把握と Week 7-8 writeup の土台。
**規律**: 本書は事前登録文書では**ない**。判定基準の一次ソースは `experimental_design_v0.3.md` §2/§14（toy）と `program_claims_v1.md` §2.1–2.3（paper-grade、v1.3 = PR #7）。**結果の数値はすべて `[SLOT]`**——存在しない結果を本文化しない。結果節に書かれている文章は、事前登録済みの分岐文（GO/PARTIAL/PIVOT、階層縮退、両側予測）の英語化のみ。
**venue 想定**: 主 = JEDC / Computational Economics（経済学 framing）。副 = NeurIPS Datasets & Benchmarks（artifact framing——§6.3 の組み替えで対応、二重投稿はしない）。

---

## 0. Keshav 5C（読む人のための要約）

- **Category**: 方法論 + 構成的反例（constructive counterexample）論文。「分野の標準検証手続き（SF 再現）は機構主張を支えない」を、批判 essay ではなく、検出力つきの構成実験＋置き換えプロトコル＋再利用可能 artifact（battery / Model Contract）で示す。
- **Context**: 3 系譜の交点。(1) 正準機構モデル系譜（Kirman 1993 → Lux-Marchesi 1999 → Alfarano-Lux-Wagner 2008、並走で Cont-Bouchaud / Chiarella-Iori / Franke-Westerhoff / ZI 系 / Genoa）——本論文は新機構を発明せず、この正典から block を removal-only で単離する。(2) ABM 検証・同定系譜（Barde MIC / Lamperti GSL-div / Grazzini-Richiardi / Guerini-Moneta / SBI）——全部「観測出力からの同定」であり、本論文の問いはその一段上（観測層が原理的に沈黙する regime を構成し、そこで do() 層が効くことを示す）。(3) 契約・ベンチ系譜（Gym `env.step()`、OSF 事前登録、D&B 規範）。
- **Correctness**: load-bearing 前提は 5 つ——(a) SF-等価 calibration の到達可能性（toy §14 gate で検証中・唯一の未確定）、(b) 介入面 = 観測チャネル masking の形式化（B2 ≠ A、29 モデル survey + 契約型）、(c) 識別の操作化 = classifier accuracy + null 2 層 + Nadeau-Bengio 補正、(d) 導出の正典性（sign-following は Kirman 遷移率の b 優勢極限、原型は ALW が補完）、(e) 負の統制と順序予測が同一理論から出る falsifiable 構造。
- **Contributions**: §1 の C1–C4。
- **Clarity**: ナラティブアークは「SF 再現で機構を主張してきた → SF の非識別性は folklore だが誰も測っていない → SF が統計的に沈黙する集合を構成（行動的機構ですらないものまで入りうる）→ 同じ集合を事前登録介入プロトコルが検出力つきで分ける → 分けるべきでないものは分けない → ∴ 機構主張には介入応答監査。プロトコルと契約はここにある」。看板図は F2（SF 分布の重なり）と F4（識別 heatmap）、締めが F6（機構距離 vs 識別精度）。

---

# Observationally Equivalent, Interventionally Distinct: A Pre-Registered Protocol for Mechanism Identification in Agent-Based Market Models

*Working title. Alternatives:* "Stylized Facts Do Not Identify Mechanisms: A Constructive, Powered Demonstration" / "Beyond Stylized Facts: Certified Observational Equivalence and Intervention-Response Identification in Canonical Market ABMs".

## Abstract

Agent-based market models are routinely validated by their ability to reproduce stylized facts (SF) of financial returns, and mechanism-level conclusions are routinely drawn from that fit. We test whether this inference is sound — constructively. We assemble a model set derived from the field's canon by *block isolation*: a trend-following model T (the chartist demand block common to Chiarella-Iori, Franke-Westerhoff and Lux-Marchesi, with strategy switching frozen) and a herding model H (the Kirman-1993 recruitment block underlying Alfarano-Lux-Wagner), both embedded in an identical market chassis so that they differ in exactly one mechanism block; the Alfarano-Lux-Wagner composite itself; [CONDITIONAL: a Genoa-type budget-constrained random-order model whose only state feedback is volatility-dependent limit pricing]; and two channel-free negative controls (Farmer-Patelli-Zovko zero-intelligence order flow and classical Cont-Bouchaud). After calibration, the behavioral models pass a pre-fixed SF battery as statistically equivalent (TOST: both a summary-statistic classifier and a raw-return 1D-CNN discriminator at chance, `[SLOT: certified equivalence set]`). A pre-registered intervention protocol — graded degradation of declared *observation channels*, never of mechanism coefficients — then separates every mechanism-distinct pair at family-wise α = 0.05 with power ≥ 0.9 at a minimum detectable effect of chance + 15 pp `[SLOT: identification matrix]`, while the channel-free pair {CB, ZI} remains response-equivalent as predicted `[SLOT: TOST result]` and the mechanism-proximal pair (H, ALW) is discriminated less accurately than the mechanism-distal pair (T, H), confirming that response distance tracks mechanism distance `[SLOT: ordering result]`. All criteria, families, and degradation branches were frozen on OSF before any result existed. Claims are limited to synthetic data with constructively known ground truth; we make no claim about which mechanism generates real markets.

## 1. Introduction

**The practice.** Since Cont's (2001) codification of the stylized facts — absence of linear autocorrelation, heavy tails, volatility clustering, slow decay of absolute-return autocorrelation — the agent-based literature has treated SF reproduction as the primary empirical credential of a market model. The canon was largely built on this credential: Lux and Marchesi (1999) presented herding-plus-switching as an explanation of fat tails and clustering *because* the model reproduces them; the same inferential template runs through Cont-Bouchaud, Alfarano-Lux-Wagner, Franke-Westerhoff, and the order-book literature following Chiarella and Iori. The template is: *model M reproduces the SF; therefore M's mechanism is a candidate explanation of the market's dynamics.*

**The folklore objection.** It has long been suspected that this template is too weak: many structurally different mechanisms reproduce the same SF (equifinality). The objection appears in reviews and discussion sections — but, to our knowledge, it has never been given a *constructive, statistically certified* form: no one has exhibited a controlled model set that provably saturates an SF battery while remaining mechanically distinct, together with a procedure that *does* identify the mechanism, with stated error rates. Without the constructive form, the objection has no teeth: any particular pair of models can be dismissed as badly calibrated, and any proposed remedy cannot be evaluated for power.

**This paper.** We supply the constructive form, in four steps. (i) We build a model set from canonical *blocks*, not new inventions (§3): isolating the chartist demand block and the Kirman herding block on a shared chassis yields a *component-controlled pair* (T, H) that differs in exactly one mechanism block; the ALW composite anchors the set in the canon's own executable form; channel-free classics (CB, FPZ-ZI) enter as negative controls; [CONDITIONAL] a Genoa-type strategy-free feedback rule enters if and only if it passes the SF battery in our implementation — its presence makes the equivalence set's weakness maximal, since it is not a behavioral mechanism at all. (ii) We calibrate the behavioral models to *certified observational equivalence* on a pre-fixed SF battery, where equivalence is operationalized adversarially: a summary-statistic classifier and a raw-return 1D-CNN must both sit at chance (TOST band) — so the set is constructed to defeat not just moment comparison but learned observational discrimination (§4.1–4.2). (iii) We define interventions exclusively on *declared observation channels* — what agents are allowed to see — via four graded attenuation schemes, never touching mechanism coefficients (§4.3); this is the operational content of "mechanism identification": mechanisms are individuated by the information they consume. (iv) We pre-registered, before any result existed, the identification family, test-level α corrections, power targets, negative-control predictions, an ordering prediction, and the degradation branches under which every possible outcome has a fixed interpretation (§4.5–4.7).

**Claims.** `[SLOT — the realized branch of the following pre-registered claim structure]` The headline claim has four registered components: (1) *observational equivalence*: the calibrated set passes the SF battery with both discriminators at chance; (2) *interventional identification*: the intervention-response classifier separates every mechanism-distinct pair in the family at family-wise α = 0.05 with power ≥ 0.9 at MDE = chance + 15 pp; (3) *negative control*: the channel-free pair {CB, ZI} is response-equivalent (TOST) — a falsifiable prediction, since a responding CB would refute our mechanism understanding, not the protocol; (4) *ordering*: discrimination accuracy for the same-block pair (H, ALW) is lower than for the different-block pair (T, H) — response distance tracks mechanism distance.

**Contributions.**
- **C1 (constructive counterexample).** The first statistically certified construction of an SF-equivalent, mechanism-distinct model set built from canonical blocks [CONDITIONAL: including a strategy-free member], with the certification adversarial (learned discriminators, not moment tables).
- **C2 (identification protocol).** A pre-registered observation-channel intervention protocol with explicit power analysis (per-pair binomial tests, Holm-corrected family of 52 [32 without Genoa] tests, n = 440 runs/condition after Nadeau-Bengio correction), portable to any simulator implementing a six-method contract.
- **C3 (two-sided + ordering prediction structure).** Identification claims, response-equivalence predictions, and a mechanism-distance monotonicity probe derived from the same theory and registered together — the protocol is falsifiable in both directions.
- **C4 (artifact).** The model contract (channel declaration, do()-style graded masking, θ=0 identity, bit determinism, L2 provenance), reference adapters for the canon, and the full pre-registration trail (OSF + pinned commits), enabling third-party mechanism audits.

**Scope.** Everything here concerns synthetic data with constructively known ground truth. We do not claim to identify which mechanism generates real markets; we claim that the field's standard observational credential cannot do so even in the laboratory where the answer is known, and that an interventional credential can.

## 2. Related Work

### 2.1 Canonical mechanisms and stylized facts

| Model in our set | Canon source | Role here |
|---|---|---|
| T (trend block) | chartist demand common to Chiarella-Iori-Perelló (2009), Franke-Westerhoff (2012), Lux-Marchesi (1999); switching frozen = constant mixture | block-isolated member of the controlled pair |
| H (herd block) | Kirman (1993) recruitment (a + b·n_k), the noise-trader block of Alfarano-Lux-Wagner (2008); majority-following as the b-dominant limit | block-isolated member of the controlled pair |
| ALW | Alfarano, Lux & Wagner (2008) | composite canon anchor; literal transition-rate form |
| Genoa-ZI+ [conditional] | Raberto, Cincotti, Focardi & Marchesi (2001) | strategy-free SF-passing feedback rule (sole non-behavioral member of the equivalence set, if admitted) |
| FPZ-ZI | Gode & Sunder (1993); Farmer, Patelli & Zovko (2005) | channel-free negative control (SF passage *not* expected: FPZ's claims concern microstructure SF, not return-series batteries) |
| CB | Cont & Bouchaud (2000) | channel-free classic; registered response-equivalence prediction with ZI |

Lineage note for readers locating us on the map: the herding line runs Kirman → Lux-Marchesi → ALW (ALW being the analytically tractable distillation); the chartist-fundamentalist line runs Beja-Goldman/Day-Huang → Chiarella-Iori → Franke-Westerhoff (the estimable form). We sit *underneath* both lines: we do not propose a new point on them, we isolate the blocks they share and ask what evidence can tell the blocks apart. Every equation in T and H has a citable source; the derivation is removal-only (no added equations), documented in correspondence tables (Appendix A).

### 2.2 Validation, estimation, and identification of ABMs

A mature literature measures ABM fit and selects among models from observational output: moment-based estimation (Franke & Westerhoff 2012; Grazzini & Richiardi 2015), information-criterion comparison (Barde 2016), simulated-distance methods (Lamperti 2018), Bayesian estimation (Grazzini, Richiardi & Tsionas 2017), and causal-graph identification from simulated and real series (Guerini & Moneta 2017). Simulation-based inference (Cranmer, Brehmer & Louppe 2020) is the modern statistical umbrella. All of these operate on the *observational* layer. Our question is one level up: does the observational layer contain mechanism information *at all* once models are calibrated into SF equivalence? Our construction answers by exhibiting regimes where a learned raw-series discriminator — a stand-in for the best observational identifier — is at chance, yet intervention responses are not. This does not contradict the estimation literature (which typically separates models *not* calibrated to mutual equivalence); it bounds what that literature can certify. The CNN-at-chance requirement is the SBI pre-emption: whatever signal SBI would exploit is absent by construction in the certified set.

### 2.3 Interventions in ABMs, and contracts

Interventions in the ABM literature overwhelmingly mean *mechanism ablation* — switching components on and off or moving structural coefficients (our survey of 29 models, `docs/research/abm_b2_intervention_survey.md`). Ablation presupposes source access and is undefined for black-box submissions. Our protocol intervenes only on *declared observation channels* (what agents see), which (i) is implementable against an interface, (ii) corresponds to the do-operator on the information structure rather than on the mechanism, and (iii) is the natural analogue of disclosure/display regulation rather than of model surgery. The interface follows the contract lesson of Gym's `env.step()` (and, on the agent side, Sakana's Shachi): adoption follows the cut of the contract, not the sophistication of the protocol behind it.

### 2.4 Pre-registration

Judgment criteria (§4.5–4.7) were frozen and publicly timestamped (OSF) before result generation, with degradation branches for every outcome; analysis is the mechanical application of pinned code. This imports the registered-report discipline into a literature where post-hoc metric selection is endemic — and it is load-bearing for the program this paper opens (auditing *other* models' claims requires having submitted to the same discipline first).

## 3. The Model Set: Canonical Block Isolation

### 3.1 Shared chassis

All behavioral models run on an identical single-asset chassis (Chiarella-Iori style): N = 500 agents emit discrete actions a ∈ {−1, 0, +1}; log-price updates by aggregate excess demand, p_{t+1} = p_t · exp(λ · ED_t / N); a fixed fundamental p* is disclosed to agents only as a noisy per-agent observation ŷ_{i,t} = p*·exp(N(0, σ_f²)). The chassis contributes three shared blocks: a *fundamentalist anchor* (mean-reversion on perceived mispricing; stationarity), a *noise block* (self-excitation), and the price-formation rule. Burn-in 1,000 steps, measurement 10,000 steps.

### 3.2 The component-controlled pair (T, H)

T and H differ in **exactly one block** — the speculative core:

- **T (trend block).** A chartist component reading windowed past returns: trend_i = mean(r over h_i lags), demand g_c·trend. This is the chartist demand common to CIP/FW/LM, with *strategy switching frozen* (population mixture constant — the freeze maps to "switching intensity → 0" in the source models).
- **H (herd block).** A social component reading lagged aggregate action: majority-following sign(social). This is the Kirman recruitment block in its b-dominant limit (recruitment dominates idiosyncratic switching; the idiosyncratic a-term is carried by the shared noise block). The literal transition-rate form a + b·n_k is supplied by the ALW adapter, so the pair {H (limit form), ALW (literal form)} brackets the canonical block.

Isolation is *block isolation on a shared chassis*, not single-mechanism isolation: removing the fundamentalist anchor or running a pure deterministic chartist breaks stationarity or self-excitation and, with them, the SF battery itself (the failure mode is documented in our spec lineage). The derivation is removal-only with parameter correspondence tables (Appendix A); the defense against "you invented mechanisms convenient to separate" is that every equation is canon, the difference is one block, and the table is auditable.

### 3.3 ALW composite; (H, ALW) as ordering probe

ALW enters in its published composite form (fundamentalists + Kirman-switching noise traders), with channel declaration normalized to the same types as H (social + fundamental) so that discrimination cannot degenerate to reading channel-name metadata. Because H and ALW share the same canonical block in different embodiments, the pair is **excluded from the identification family** and registered instead as an *ordering prediction*: acc(H, ALW) < acc(T, H). If the protocol measures mechanism content, the mechanism-proximal pair must be harder to tell apart than the mechanism-distal pair.

### 3.4 Genoa-ZI+ [conditional admission]

The Genoa artificial market generates fat tails and clustering from budget-constrained random orders whose *limit prices alone* depend on past volatility — zero strategy, zero heterogeneity, one state-feedback channel. Its admission to the equivalence set is gated: it enters **iff our implementation passes the SF battery under TOST**. If admitted, the headline strengthens to: SF cannot separate behavioral mechanisms *even from a strategy-free stochastic feedback rule*. If not admitted, the claim structure degrades along the registered hierarchy (§4.7) without collapsing.

### 3.5 Channel-free negative controls (FPZ-ZI, CB)

FPZ-ZI (Farmer-original order flow; audited against the source spec) and classical CB (percolation clusters with exogenous activation) consume no observation channels: channels = (). The registered prediction is *response equivalence* of {CB, ZI} — the intervention protocol must NOT separate them, and each must be flat under intervention (TOST against "no response"). SF passage is *not* expected for FPZ-ZI (its literature claims concern microstructure facts, not return-series batteries) and is empirically non-trivial for CB; both fall where they fall under the registered hierarchy.

### 3.6 The simulator contract

Every model implements: `reset(seed)` (bit-deterministic), `step()`, `observe(channel)` (post-intervention view), `intervene(channel, scheme, θ)` (graded degrade of declared channels only; θ = 0 is bit-identical to no-op, property-tested), `emit()` (output series), `provenance()` (seed/config/output digests, observation logs). Channel declarations are checked against observation logs (reach audit), closing the "undeclared channel" loophole.

## 4. Protocol

### 4.1 SF battery and the operationalization of equivalence

Battery (pre-fixed): SF1 absence of return autocorrelation (ACF sum-of-squares, lags 1–10); SF2 slow decay of |r| autocorrelation (power-law fit, lags 1–50); SF3 heavy tails (excess kurtosis + Hill top 5%); SF4 volatility clustering (GARCH(1,1) α+β). SF1–4 are the calibration targets; SF5 (leverage) and SF6 (gain-loss asymmetry) are measured post-equivalence as independent checks, never calibrated. "Observationally equivalent" is defined adversarially: (i) an L2-logistic classifier on the 4-D SF feature vector and (ii) a 3-layer 1D-CNN on raw 1,000-step return windows must both produce 5-fold CV accuracy whose 90% CI lies in [0.45, 0.55] (TOST band; failure to reject ≠ equivalence).

### 4.2 SF-equivalence calibration

Anchor-based: T is fixed at a reasonable point T*; each other behavioral model is calibrated to T* by Bayesian optimization (500 trials, Wasserstein distance on the SF1–4 feature distribution, M = 200 runs per trial), then certified at M = 1,000 runs per model via §4.1. Models that cannot reach the band enter the *SF-distinguishable stratum* of the registered hierarchy rather than being silently dropped.

### 4.3 Interventions (observation-channel masking)

Channels: trend window (price history), social signal (lagged aggregate action), fundamental observation (σ_f). Schemes (4): windowed averaging, EMA smoothing, additive noise, delay — each graded by θ with θ = 0 the identity. Interventions apply to the *observation layer only*; mechanism coefficients are untouched (B2 ≠ ablation). Runs are paired by common random numbers across θ within each (model, seed), so response curves are within-pair differences, not between-run noise.

### 4.4 Response curves and the IR classifier

For each (model, channel, scheme), sweep θ over a fixed grid; extract susceptibility features from the response curves of the SF features and of distributional summaries (slope at θ→0, saturation level, dose-response shape). The intervention-response (IR) classifier (gradient-boosted trees) is trained per pair × scheme on susceptibility features; discrimination accuracy is evaluated by 5-fold CV at n = 440 runs/condition.

### 4.5 Statistical design (pre-registered)

| Element | Value |
|---|---|
| Identification test | per mechanism pair (i, j) × scheme s: one-sided binomial H0: accuracy = 0.5 |
| Family | all pairs with ≥ 1 channel-holding member × 4 schemes, *excluding* (CB, ZI) (equivalence side) and (H, ALW) (ordering side): 13 pairs × 4 = **52 tests** with Genoa, 8 × 4 = **32** without; Holm correction, family-wise α = 0.05 |
| MDE | accuracy 0.65 (chance + 15 pp), set *below* the GO threshold 0.75 so near-threshold effects are inside the powered region |
| Power | ≥ 0.9 at MDE under worst Holm α′ ≈ 0.05/52 → n ≈ 220 by normal approximation, doubled to **n = 440 runs/condition** for CV-induced variance (Nadeau-Bengio) |
| Equivalence claims | TOST: CV accuracy 90% CI ⊂ [0.45, 0.55] (SF equivalence; {CB, ZI} response equivalence; ZI flatness) |
| Ordering claim | one-sided test of acc(H, ALW) < acc(T, H), bootstrap CI on the accuracy difference |
| Reporting | seed-wise mean ± SE with n, CV fold variance reported separately; no single-run numbers |

### 4.6 Null architecture

Null-1 (seed): T vs T, different seeds — both classifiers must sit at 50 ± 3%; failure invalidates the harness. Null-2 (parameter): T1 vs T2, same mechanism, different parameters within the calibrated region — IR accuracy must remain within SF baseline ± 10 pp; IR accuracy > 65% triggers the registered "parameter-sensitivity" caveat branch, which demotes the headline from mechanism identification to mechanism-or-parameter sensitivity.

### 4.7 Pre-registration and degradation branches

All thresholds above were frozen before result generation (OSF + pinned commits). Registered branches: (a) *SF hierarchy*: models failing the equivalence band form an SF-distinguishable stratum; the claim becomes "SF separates the set only into strata; within the equivalence stratum, interventions identify"; (b) *Genoa gate*: family size and headline rhetoric are pre-specified for both admission outcomes; (c) *main outcome*: GO (≥ 3 of 4 schemes ≥ 0.75 for the controlled pair) / PARTIAL (1–2; claim becomes scheme-conditional) / PIVOT (0; the negative result is reported as the refutation of the program's premise — the paper in this form does not exist, and the program's registered consequence is to halt its successors); (d) *negative-control failure*: a responding CB refutes our mechanism reading of CB and is reported as such, not absorbed.

## 5. Results `[ALL SLOTS — nothing below is claimable until the corresponding runs exist]`

- **5.1 Equivalence certification** `[SLOT: which models entered the equivalence stratum; TOST CIs; SF5/6 independent-check table]`
- **5.2 Nulls** `[SLOT: Null-1 (must be 50±3); Null-2 (param-sensitivity branch or clean)]`
- **5.3 Identification matrix** `[SLOT: pairs × schemes accuracy heatmap, Holm-significant cells; per-scheme power posteriors; GO/PARTIAL/PIVOT branch text]`
- **5.4 Negative controls** `[SLOT: {CB, ZI} response-equivalence TOST; ZI/CB flatness TOST; falsification branch if violated]`
- **5.5 Ordering probe** `[SLOT: acc(H, ALW) vs acc(T, H), CI on difference]`
- **5.6 Dose-response** `[SLOT: accuracy as a function of θ — identification should emerge gradually with intervention strength; θ=0 must reproduce SF-equivalence chance level (internal consistency check)]`

## 6. Discussion

**6.1 What the result licenses.** If the registered claims land on their positive branches, the inferential template "reproduces SF ⟹ mechanism is a candidate explanation" is constructively refuted *with stated power*: there exist canon-derived sets where the template's premise holds and its conclusion is undecidable from observation, yet decidable from declared-channel interventions. Mechanism-level claims about simulators then require an interventional credential, and this paper supplies a portable one: contract + battery + power analysis. The protocol is GT-free (no real-data reference enters any criterion) — it audits *internal* mechanism identifiability, which is exactly the part the SF template pretends to deliver.

**6.2 What it does not license.** Nothing here identifies the mechanism of real markets; equivalence is battery-relative (SF1–4) and chassis-conditional; identification is relative to the declared channel set and the four schemes; power statements are for the registered MDE, not for arbitrarily small effects.

**6.3 The artifact.** For the benchmark audience: the contract's six methods are the entire integration surface; reference adapters cover the canon; the negative controls are typed (channels = ()); reach audit closes declaration gaming. A leaderboard is explicitly out of scope here — the deliverable is the audit instrument, not a ranking.

**6.4 The program.** This paper is the first of a sequence: the same contract's first external adapter is a verified market-microstructure harness used for a portability audit of algorithmic-collusion claims (P2, companion work); the audit protocol (reproduction → variance → intervention control) is then pointed at published LLM-ABM claims (P3). The discipline demonstrated here — registration before results, degradation branches, mechanical verdicts — is the qualification for that program, which is why it is not optional ornament.

## 7. Limitations

(1) Equivalence is defined on SF1–4; richer batteries (Epps, intraday seasonality, impact curves) may separate the set observationally — the claim is battery-relative by construction and the battery is pre-fixed. (2) Single asset, discrete actions, one chassis; cross-chassis generality is probed only by the (H, ALW) embodiment pair. (3) Classifier-based identification lower-bounds the information in response curves but ties power statements to specific detectors. (4) The calibration may fail to reach the equivalence band for some models — handled by the registered hierarchy, but a thin equivalence stratum weakens C1's strongest form. (5) LM and FW run as extension adapters only (LM's N-dependent SF fragility and partial-observation surface are documented risks); their lineage enters through T's correspondence table. (6) Pure-block isolates without the shared anchor are not testable (non-stationarity); our isolates are therefore *anchored* isolates — the correspondence tables state exactly what was removed and what retained.

## Reproducibility statement

Every run emits L2 provenance (UUID v7, config hash, seeds, output digests, observation logs); same (config, seed) reproduces bit-identical trajectories; `uv.lock` pins the environment; OSF registrations pin the criteria; analysis is the pinned code's mechanical output. A single script reproduces all figures from the archived run set.

---

## Appendix A — Correspondence tables (stub; paper-grade TODO)

| toy block | canon equation | mapping | removed (removal-only audit) |
|---|---|---|---|
| H herd block: a = sign(mean ā over h^s lags) | Kirman (1993) switch prob a + b·(n_k/N); ALW noise-trader block | b-dominant limit; a-term → shared noise block; n_k/N → lagged mean action | fundamentalist demand form of ALW (replaced by shared-chassis anchor); transition stochasticity (→ ALW adapter) |
| T trend block: d = g_c · mean(r over h lags) | CIP (2009) chartist demand; FW (2012) chartist rule; LM (1999) chartist demand | direct (g_c ↔ source chartist gain; window ↔ source memory) | strategy switching (mixture frozen); wealth dynamics |
| shared fundamentalist: d = g_f · log(ŷ/p) | ALW / FW fundamentalist demand | direct | — |
| chassis: p_{t+1} = p_t exp(λ·ED/N) | CIP (2009) price impact | direct | order-book depth structure |

## Appendix B — Power derivation

One-sided binomial vs 0.5 at accuracy 0.65: z(n) = 0.15/√(0.25/n). Worst Holm α′ = 0.05/52 ⇒ z_crit ≈ 3.10. Power 0.9 ⇒ z(n) ≥ z_crit + 1.282 ⇒ n ≈ 214 → 220. CV folds share training data ⇒ accuracy variance exceeds binomial (Nadeau-Bengio); registered correction ×2 ⇒ **n = 440**. At n = 440: z = 6.29, power ≈ Φ(6.29 − 3.10) ≈ 0.999 — slack absorbs non-normality and fold dependence beyond the NB factor.

## Appendix C — Figure & table plan

- **F1** chassis/block diagram: shared blocks vs the one swapped block; channel declarations per model.
- **F2** *(flagship)* SF1–4 feature distributions overlaid across the equivalence set + CNN/LR accuracy with TOST band — "observation is silent."
- **F3** response curves per channel × scheme (one panel per model) — "interventions speak."
- **F4** *(flagship)* identification heatmap (pairs × schemes), Holm-significant cells marked; negative-control row visibly flat.
- **F5** dose-response: accuracy vs θ with θ=0 at chance.
- **F6** *(closer)* mechanism distance vs discrimination accuracy: (H,ALW) < (T,ALW) ≈ (T,H), the ordering probe in one picture.
- **T1** model set & lineage (the §2.1 table). **T2** statistical design (§4.5). **T3** registered branches and which one realized.

---

## 内部文書とのマッピング（執筆者用）

| Paper section | 一次ソース |
|---|---|
| §1 Claims / §4.5 / §4.7 | `program_claims_v1.md` §2(P1 目標文・(i)–(iv))、§2.1(検出力)、§2.2(トリオ・導出・縮退)、§2.3(GO/PARTIAL/PIVOT) — v1.3 = PR #7 |
| §3.1–3.2 / §4.1–4.4 / §4.6 | `experimental_design_v0.3.md` §3(chassis/機構)、§4(SF battery)、§5(calibration)、§6(SF classifier)、§7(B2)、§8–10(response/IR/null)、§14(decision tree) |
| §3.6 / §6.3 | `model_contract_v0.md` |
| §2.3 の ablation/masking 区別 | `docs/research/abm_b2_intervention_survey.md` |
| §6.4 | P2 = `ABM-Microstructure`(finding 0002、osf.io/63pj2)、P3 = claims §2 |

## 執筆ロードマップ（slot が埋まる順）

1. **いま書ける**: §1–4、§6–7、App A–C(本 draft)。
2. **toy §14 GO 後**: paper-grade prereg を OSF 登録(§4.5/§4.7 の数値を凍結内容として参照)→ Genoa gate 実験 → calibration → 本実験。
3. **run 完了後**: §5 の slot を機械判定の出力で埋める。Abstract の [SLOT] を実現枝で確定。F2/F4/F6 生成。
4. **PIVOT の場合**: 本 draft は廃棄し、claims §2.3 の short note(二重 obit)に切り替える——その分岐も事前に書いてあるのがこの構造の点数。
