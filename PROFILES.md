# How the Cancer Profiles Were Determined

*A companion note to the Cancer Metabolism Explorer. This document exists to answer one question honestly: where did the numbers come from?*

---

## The short answer

**They are not measurements.** There is no experiment anywhere in the literature whose output is "PDAC glycolysis = 85." No such number exists to be looked up, and if this tool implied otherwise it would be lying to you.

What the numbers *are*: an ordinal encoding of qualitative claims made in the published literature, calibrated so that the **relationships between cancers** and the **relationships between pathways within a cancer** land in the right places. They are a considered reading of what the research describes, converted into something a simulation engine can act on.

The honest way to read any single value is as a **rank plus a rough magnitude**, not a quantity.

---

## What a bar does *not* mean

This is the most common misreading, and it's worth stating flatly before anything else.

**The bars are not percentages of total ATP, and they do not sum to 100.**

Pancreatic sits at glycolysis 85, mitochondrial 30, glutamine 55, fat 10 — that adds to 180. It is not a pie chart. Each bar is an **independent 0–100 scale of how active that route is**, calibrated against the same scale for every other cancer in the tool.

So `glycolysis: 85` does not mean "85% of this tumor's ATP comes from glycolysis." It means: *on a scale where a strongly Warburg-committed tumor sits near 90 and a genuinely non-glycolytic one sits near 30, this tumor sits high.* The comparison that carries meaning is **PDAC's 85 against prostate's 30**, or **PDAC's glycolysis 85 against PDAC's own fat 10**. The absolute digit, taken alone, carries far less information than it appears to.

---

## What each parameter actually claims

A profile is six numbers and a channel. Each one encodes a specific, checkable claim.

| Field | What it encodes | What would make it wrong |
|---|---|---|
| `baseline.glycolysis` | How Warburg-committed the tumor is at rest | Evidence of low FDG-avidity / minimal aerobic glycolysis |
| `baseline.oxphos` | How much ATP the mitochondrion contributes | Evidence of respiration-impaired *or* OXPHOS-anchored biology |
| `baseline.glutamine` | How glutamine-reliant it is | Glutamine-withdrawal experiments showing tolerance |
| `baseline.fatox` | How much it runs on fat/ketones at rest | Evidence of primary lipid reliance (or none at all) |
| `fatoxCap` | **Ceiling** on fat/ketone oxidation — the ketolytic-enzyme story | OXCT1/BDH1/ACAT1 expression data pointing the other way |
| `plasticity` | How hard it compensates when a route is blocked | Studies where blocking one fuel *did* corner it |
| `compensationChannel` | *Which* fuel it reaches for first under glucose restriction | Tracing data showing it leans elsewhere |
| `redoxSensitivity` | Relative fragility under oxidative stress | ROS-tolerance data |

`fatoxCap` deserves special mention because it's doing more work than it looks like. Most solid tumors **downregulate the ketolytic enzymes** (OXCT1, BDH1, ACAT1), which is the actual scientific rationale behind ketogenic approaches in oncology: starve the tumor while normal tissue adapts. That's why nearly every profile has a low ceiling (16–30) — and why **prostate is set to 68**, which is not a rounding difference but a categorical claim that this tumor is *different in kind* on this axis.

---

## Worked example: how Pancreatic (PDAC) was built

Take the question at face value — *"how'd you get the baseline numbers for pancreatic?"* Here is the actual reasoning behind each value.

```
baseline:  glycolysis 85 · oxphos 30 · glutamine 55 · fatox 10
fatoxCap 22 · plasticity 1.35 · compensationChannel 'oxphos' · redoxSensitivity 1.4
```

**glycolysis: 85 — high, but deliberately not the highest.**
KRAS-driven aerobic glycolysis is among the best-established facts about PDAC. It sits below esophageal (90) and TNBC (90) because those are described in even more strongly Warburg-committed terms, and above NSCLC (78), which the tracing literature shows running a genuinely mixed economy. The *ordering* is the claim. The exact digit is a placement within it.

**oxphos: 30 — modest at rest, but see `plasticity`.**
PDAC is not described as an OXPHOS-anchored tumor at baseline. But this number alone would badly mislead you, which is the point of the next two fields.

**glutamine: 55 — high, and the load-bearing number in this profile.**
PDAC runs a *non-canonical* glutamine pathway (glutamine → GOT1/GOT2 → NADPH) that appears to serve **redox management** more than TCA fueling. That makes glutamine unusually load-bearing here — which is why it's set well above esophageal (34) or melanoma (36), and why `redoxSensitivity` is raised to **1.4** (the highest in the tool). Those two numbers are making a single joint claim: *this tumor's glutamine habit is buying it antioxidant protection, so pressuring glutamine should leave it unusually exposed to ROS.* That's the mechanistic bet, and it's what makes the press-pulse sequence bite in this profile.

**fatox: 10 / fatoxCap: 22 — low, and specifically evidenced.**
This isn't just the generic default. PANC-1 cells show low BDH1/OXCT1 expression and were **suppressed, not fed**, by a ketogenic diet in a controlled xenograft comparison against a ketolysis-competent line. That's about as direct as the evidence gets for this axis.

**plasticity: 1.35 — high compensation.**
Inhibiting glycolysis in PDAC has been observed to **upregulate OXPHOS and TCA/ETC enzymes**. This is the single most important fact about PDAC for a tool like this one, and it's *why* the modest `oxphos: 30` baseline is not the whole story: press glycolysis and that number climbs. It's also the reason single-pathway targeting so often fails here — a lesson the tool exists to make visible rather than assert.

**compensationChannel: 'oxphos' — where it runs when you squeeze glucose.**
Follows directly from the observation above.

Notice what happened: **no single number in that profile is independently meaningful.** The `oxphos: 30` is only honest when read with `plasticity: 1.35`. The `glutamine: 55` is only honest when read with `redoxSensitivity: 1.4`. The profile is a *system of claims*, not a list of readings.

---

## The general procedure

For each cancer:

1. **Read what the literature says qualitatively** — is it Warburg-committed? glutamine-addicted? oxidative? Does blocking one fuel corner it, or does it re-route?
2. **Place it relative to the others** on each axis. This is the step that carries the real content: is PDAC more or less glutamine-reliant than SCLC? (Less — SCLC is 60, because glutamine withdrawal was *lethal* there while glucose withdrawal barely registered.)
3. **Set the ceiling and the compensation behavior** — `fatoxCap` from ketolytic-enzyme evidence where it exists, `plasticity` and `compensationChannel` from what actually happened when researchers blocked a pathway.
4. **Check the profile behaves correctly under pressure**, not just at rest. A profile is only right if applying agents to it produces the responses the literature describes.

Step 4 is the one that catches errors. A baseline can look perfectly reasonable sitting still and still be wrong, because the interesting claims are all about what happens under stress.

---

## Where the two models differ

Almost every profile has **one** baseline, shared by both mitochondrial models. This is deliberate and it's the tool's central point: the two models mostly **do not disagree about which fuels a tumor uses**. They disagree about the *mechanism* of the mitochondrial bar — genuine respiration, or glutamine-fed substrate-level phosphorylation (mSLP).

**Glioblastoma is the one exception.** It carries a separate `baselineFermentation` (mitochondrial 30 → **58**, glutamine 55 → 62), because the fermentation model makes a genuinely *different quantitative claim* about GBM: that more of its ATP comes from that route than the respiratory reading allows. Where a model asserts different numbers, the tool shows different numbers.

---

## The three profiles that argue against the tool

These are included on purpose, and they are the ones to look at hardest:

- **Classical Hodgkin Lymphoma** (`oxphos: 78`, `plasticity: 0.4`) — the deliberate counter-example. OXPHOS-anchored, lactate-*importing*, barely glycolytic. Its `plasticity` of 0.4 is by far the lowest in the tool, encoding a single-anchor tumor with almost no fuel-switching. It is the most direct published challenge to the fermentation model's premise.
- **Prostate** (`glycolysis: 30`, `fatox: 55`, `fatoxCap: 68`) — the fat-fueled outlier. Its numbers are inverted relative to every other profile, and its fat ceiling is triple the next-highest.
- **NSCLC** (`oxphos: 50`) — the highest mitochondrial baseline outside the two outliers, because *in vivo* ¹³C tracing shows glucose carbon being oxidized through the TCA cycle in living tumors.

A tool built to flatter one model would have quietly left these out. They're in because a profile set that only contains agreeable cases isn't evidence of anything.

---

## Honest limitations of the profiles specifically

- **They are type-level, not tumor-level.** The same cancer behaves differently between patients, between primary and metastatic sites, and between regions of one tumor. A profile is a central tendency drawn from a literature that is itself mostly describing cell lines and models.
- **Precision is an illusion of the format.** 85 vs. 87 means nothing. 85 vs. 30 means a great deal. The two-digit display is a rendering convenience, not a claim to that resolution.
- **Cell-culture bias.** Much of the underlying evidence comes from cultured cells, which are well known not to reproduce *in vivo* tumor metabolism faithfully. Where *in vivo* tracing exists (NSCLC), it was weighted more heavily — and it pushed that profile away from where a culture-only reading would have put it.
- **The four bars are not four peers.** Glycolysis makes ATP on its own. Glutamine and fat are *feeders* whose ATP is collected downstream in the mitochondrion — which is why the engine gates their effective contribution on mitochondrial capacity. The bar chart's visual grammar implies four parallel routes; the biology is not that shape. This is the deepest unresolved tension in the tool's design, and it is a cost of the metaphor, not a bug in the numbers.
- **Metabolic phenotype shifts under treatment.** Every profile is a snapshot of a naïve tumor. The literature on TNBC residual disease after chemotherapy — reported as becoming OXPHOS-*dependent* — is a direct reminder that the starting numbers may not survive first contact with therapy.

---

## If you think a number is wrong

**You are probably sometimes right, and the tool is built on that assumption.**

The Metabolic Reasoning Sandbox exists precisely so that a disagreement about a baseline doesn't have to become a disagreement about the whole tool. Move the sliders. The moment you do, the profile relabels itself as a **Custom Metabolic Profile** — the tool stops claiming the configuration is literature-derived, and authority visibly transfers to you.

That is the intended response to "I don't think PDAC's glutamine is really that high." Don't argue with the number; move it, and see whether the *behavior* the engine produces still matches what you know. If it does, the disagreement was cosmetic. If it doesn't, you've found something worth reporting.

Corrections are welcome. If a profile, mechanism, or citation is wrong, that is worth knowing — and the numbers above are stated plainly enough to be argued with, which is the entire point of writing them down.
