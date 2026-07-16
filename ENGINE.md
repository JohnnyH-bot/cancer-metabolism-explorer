# How the Engine Works

*A companion note to the Cancer Metabolism Explorer. Where `PROFILES.md` explains the tumor baselines, this document explains the machinery that acts on them — and, more importantly, where its numbers came from.*

---

## The honest framing, first

The engine contains roughly a dozen coefficients: a `0.15` multiplier here, a `0.25` floor there, compensation strengths of `26` and `12`. **None of these are measured parameters.** They were not fitted to data, and no experiment produces them.

They are **calibration constants chosen so that the engine reproduces behaviors the literature describes qualitatively.** The literature says *"blocking glycolysis in PDAC upregulates OXPHOS and TCA/ETC enzymes."* It does not say by how much. The engine needs a number anyway. That number is a judgment, and it is documented here so it can be argued with rather than trusted.

The test of these constants is not *"is 0.15 the right value?"* — that question has no answer. It is **"does the engine, run across many profiles and agent combinations, produce the behavioral pattern the research describes?"** Where it does not, the constant is wrong and should be changed.

---

## The four bars, and what they actually are

Each pathway is an independent **0–100 activity scale**, clamped to `[5, 95]`. They do not sum to 100 and are not shares of total ATP. See `PROFILES.md`.

The floor of 5 is deliberate: a pathway is never fully abolished, only suppressed. The ceiling of 95 prevents any single route from being modeled as an unlimited sink.

**Critically, the four bars are not four peers.** This is the single most important structural fact about the engine:

| Route | Independent ATP? |
|---|---|
| **Glycolysis** | **Yes.** Cytosolic, substrate-level phosphorylation. Runs without mitochondria. |
| **Mitochondrial pathway** | **Yes.** This is where mitochondrial ATP is collected. |
| **Glutaminolysis** | **No — it is a FEEDER.** |
| **Fatty-acid / ketone oxidation** | **No — it is a FEEDER.** |

Glutamine and fat do not make ATP on their own. They deliver substrate to the mitochondrion, and the ATP is collected *there*. The bar chart's visual grammar implies four parallel routes; the biology is not that shape. This is the deepest unresolved tension in the tool's design, and it is a cost of the metaphor.

The engine compensates for the metaphor with the **mitochondrial gate** (below), which is why a high fat bar over a dead mitochondrion is reported as fuel the cell *cannot cash*.

---

## The two models

The engine runs one simulation and interprets the mitochondrial bar two ways.

- **Respiratory-flexible** — the mitochondrial bar is genuine oxidative phosphorylation, ETC-dependent.
- **Fermentation (mSLP)** — the mitochondrial bar is glutamine-fed substrate-level phosphorylation at the succinyl-CoA ligase (SUCL) step, which needs **no electron transport chain**.

Almost every profile has **one** baseline shared by both models, because the two frameworks largely **do not disagree about which fuels a tumor uses** — they disagree about the *mechanism* of the mitochondrial bar. Glioblastoma is the sole exception, carrying a separate `baselineFermentation` (mitochondrial 30 → 58) because that is the one place the models make a genuinely different *quantitative* claim.

### The `0.15` ETC-sparing multiplier

**The most consequential constant in the engine.**

```js
if (fermentation) bo = Math.round(bo * 0.15);
const oxEffect = fermentation && etcInhibitor
  ? Math.round(a.effects.oxphos * 0.15)
  : a.effects.oxphos;
```

Under the fermentation model, ETC inhibitors (metformin, berberine, ivermectin) retain only **15%** of their effect on the mitochondrial bar.

**Why:** if mitochondrial ATP is coming from mSLP at the SUCL step, it does not require the electron transport chain — so inhibiting complex I should *not* abolish it. Glutamine antagonists, by contrast, *do* collapse it, because they cut its fuel. **That asymmetry is the entire argument for targeting glutamine rather than the ETC**, and this coefficient is where the argument lives.

**Why 0.15 and not 0 or 0.3:** it is not zero because complex-I inhibitors have real off-target and signalling effects (AMPK/mTOR) that no model of pure bioenergetics captures, and because even mSLP proponents do not claim tumor respiration is *entirely* absent. It is small because the model's whole claim is that this route survives ETC blockade. Any value in roughly 0.1–0.2 would produce the same lesson. **The value is doing rhetorical work, and it should be inspected as such.**

---

## The mitochondrial gate

Because glutamine and fat are feeders, their **effective** ATP contribution is gated on mitochondrial capacity:

```js
const frac = clamp(state.oxphos / baselineOxphos, 0, 1);
factor    = 0.25 + 0.75 * frac;   // glutamine
fatFactor = frac;                 // fat — NO floor
```

These two gates are **deliberately different**, and the difference is a biochemical claim, not a tuning choice:

**Glutamine keeps a floor of 0.25.** It can reach the SUCL step *non-cyclically* — glutamine → glutamate → α-ketoglutarate → succinyl-CoA → succinate, with the succinate exported. It never has to complete the TCA cycle, which is precisely why it can still make ATP with no ETC. That floor **is** the mSLP claim, expressed numerically.

**Fat gets no floor at all.** β-oxidation produces **zero ATP by itself** — it produces acetyl-CoA, NADH and FADH₂, and every one of those only becomes ATP at the ETC. Acetyl-CoA can only enter at citrate synthase and must traverse the *entire* cycle to regenerate oxaloacetate, and every turn needs NAD⁺ that the ETC is supposed to regenerate. **There is no shortcut from fat to the SUCL step.** Kill the ETC and β-oxidation stalls too — after the cell has already *spent* 2 ATP equivalents activating each fatty acid to acyl-CoA. Ketones are worse still: ketolysis via OXCT1 *consumes* succinyl-CoA, actively stealing substrate from the mSLP step.

So **fat is not an escape route from a suppressed mitochondrion; it is the route that dies with it.** When the gate is active, the tool says so on the bar and explains it under **Why?**.

This produces the correct discrimination on prostate cancer, the tool's fat-fueled outlier:

| Prostate (respiratory) | Mito | Fat bar | **Effective fat** |
|---|---|---|---|
| baseline | 60 | 55 | **55** |
| ketosis + DON (press-pulse) | 67 | 68 | **68** ← fat still rescues it |
| + ivermectin (ETC hit) | 24 | 68 | **27** ← fat can no longer rescue it |

Fat escapes glucose+glutamine restriction. It does **not** escape ETC inhibition. That distinction is the point.

---

## Compensation

The engine's central teaching claim: **block one fuel and the tumor reaches for another.**

```js
const strong = Math.round(26 * profile.plasticity);
const minor  = Math.round(12 * profile.plasticity);
```

`plasticity` is a per-tumor **metabolic adaptability** coefficient (0.4 for Hodgkin lymphoma, 1.0 for the teaching baseline, up to 1.5 for melanoma). It scales how hard a tumor compensates. `compensationChannel` names the fuel it reaches for *first* under glucose restriction.

**These are ordinal, not measured.** Melanoma's 1.5 encodes *"exceptionally well-documented fuel-switching under therapeutic pressure."* Hodgkin's 0.4 encodes *"single-anchor tumor, almost no glycolytic fallback of its own."* The **ordering** carries the claim; the digits are placements within it. The engine surfaces this to the user as behavior (*"Flexible fuel-switching"*) rather than as a `LOW/MODERATE/HIGH` badge, because a badge would overstate the precision.

Compensation is **reciprocal**: a glutamine antagonist alone *raises* glycolysis, and that rise is suppressed when glucose is co-restricted. This is why press-pulse strategies target both simultaneously, and it is the behavior the engine exists to make visible.

**Known limitation:** compensation here is instantaneous and frictionless. Real fuel-switching has kinetics, costs, and intratumoral heterogeneity. The engine models the endpoint, not the transient.

---

## Redox

```js
let redox = 20;                                        // baseline oxidative stress
const gshBuffer   = clamp(q, 5, 95);                   // GSH capacity ≈ glutamine supply
const rosOverflow = Math.max(0, redox - gshBuffer - 12);
const hit         = Math.round(rosOverflow * 0.35 * profile.redoxSensitivity);
let netGlutamine  = q - redox * 0.4 * profile.redoxSensitivity;
```

Two coupled ideas, both mechanistically grounded:

1. **Glutamine buys antioxidant protection.** It is the substrate for glutathione, so the GSH buffer tracks glutamine supply. Some glutamine is therefore *diverted* to redox defense rather than ATP — which is why the glutamine bar can read as suppressed while glutamine is still being consumed.
2. **ROS only bite once the buffer is gone.** `rosOverflow` is zero until oxidative stress exceeds the GSH buffer. This is why pro-oxidant agents (HBOT, high-dose IV vitamin C, artemisinin) do little on their own but become effective **after** glutamine is pressured — the sequencing logic behind press-pulse.

`redoxSensitivity` (1.0 default; **1.4 for PDAC**, the highest) scales a tumor's fragility under oxidative stress. PDAC's raised value is not decoration: it makes a **joint claim** with its high glutamine baseline — *if this tumor's glutamine habit is buying it antioxidant protection, pressuring glutamine should leave it unusually exposed.* That is a testable mechanistic bet, and it is what makes the press-pulse sequence bite in that profile.

---

## What the engine does not model

**This tool models ATP production only.** That is a severe limitation and it is stated on the front of the app, not buried here.

Metabolism does a great deal that is not ATP accounting — it regulates gene expression, growth signalling, and immune visibility, and **none of that appears on these four bars**:

- glucose availability controls histone acetylation via ATP-citrate lyase (Wellen et al., *Science* 2009)
- nutrient sensing runs through AMPK and mTOR
- the hexosamine pathway shunts glucose into protein glycosylation
- β-hydroxybutyrate is an endogenous HDAC inhibitor

**A small movement on these bars does not mean little is happening — it may mean that what is happening is not on the axes this tool measures.** Metformin is the clearest case: its bioenergetic effect is modest, while its better-supported rationale (AMPK/mTOR, immune microenvironment) lies **entirely off-axis**.

Also absent: nutrient scavenging via autophagy and macropinocytosis (documented in KRAS-mutant NSCLC), stromal metabolic symbiosis beyond the lactate import modeled in Hodgkin lymphoma, and intratumoral heterogeneity.

### Pharmacokinetics: the engine models fuel routes, never doses

**The engine has no pharmacokinetics whatsoever.** An agent's effect is applied at full strength regardless of whether a human could ever reach the concentration at which that effect was demonstrated.

This is a serious limitation, and for several agents it may be *the* deciding fact. Many of the mechanisms modeled here were shown in cell culture at concentrations far above documented human plasma levels:

- **Metformin** — plasma does not exceed ~30–40 µM at maximum clinical doses, while significant complex-I inhibition requires **millimolar** (1–5 mM) concentrations. A 2023 study measuring metformin in lung-cancer patients' plasma, tumor and normal tissue found achievable concentrations **100–1000× lower** than the tissue-culture work the mechanism rests on.
- **Ivermectin** — plasma peaks around 50 nM at approved doses (~300 nM even at escalated doses), versus low-µM in the in vitro anticancer work, with 93% plasma protein binding. Its aqueous solubility limit is ~5 µM, and a 2022 paper argued some in vitro cell death was an artefact of drug **precipitation** in culture medium.
- **Curcumin, berberine** — bioavailability so poor that free plasma concentrations sit orders of magnitude below in vitro levels.

Because the engine cannot model this, the tool carries it as a **third rating axis on every agent — "modeled mechanism in humans"** — displayed alongside mechanistic confidence and clinical evidence tier.

**This axis rates the bars, not the agent**, and the distinction is the whole point. An agent can fail to reach the modeled concentrations and still have genuine clinical effects through routes this tool does not measure. Metformin is precisely that case: its bioenergetic mechanism is likely unreachable, *and* retrospective cohorts in pancreatic cancer repeatedly report a survival association (concentrated in early-stage disease, while the two negative randomized trials both enrolled advanced/metastatic patients — the stage where the cohorts also show no benefit). Both things are true. The reconciliation is that metformin's plausible benefit runs through AMPK/mTOR, insulin lowering, and the immune microenvironment at *achievable* micromolar concentrations — every one of which lies **off-axis** for this engine.

So a low rating on this axis means **look off-axis**, not *"this does not work."* A flat bar is the tool pointing at its own blind spot.

**Bar movement is not efficacy.** Cornering a tumor's fuel routes in this model is not evidence that an intervention shrinks tumors or extends life.

---

## If you think a constant is wrong

You may well be right. The constants above are stated plainly *so that they can be argued with* — that is the entire reason this document exists.

The most productive form of that argument is not *"0.15 should be 0.22."* It is: **"run the engine with your value and see whether the resulting behavior still matches what the literature describes."** If it does, the disagreement was cosmetic. If it doesn't, you have found something worth reporting.

The **Metabolic Reasoning Sandbox** exists for the same reason at the profile level: a disagreement about a baseline should not have to become a disagreement about the whole tool.

Corrections are welcome. If a coefficient, mechanism, or citation is wrong, that is worth knowing.
