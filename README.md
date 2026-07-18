# Cancer Metabolism Explorer

**An Educational Visualization of Tumor Bioenergetics**

An interactive, single-file browser tool that illustrates **how active each of four metabolic pathways is** in a tumor — glycolysis, the mitochondrial pathway, glutaminolysis, and fatty-acid/ketone oxidation — and how those pathways renegotiate when metabolic conditions or candidate agents are applied. It lets you reason from two competing scientific frameworks for cancer mitochondrial function and watch the modeled consequences.

> ⚠️ **This is an educational resource, not medical advice.** It does not recommend treatments and must not be used for decisions about cancer diagnosis, prevention, or therapy. See the Educational Disclaimer and Known Limitations below.

> **Bar movement is not efficacy.** This tool models **ATP production only**. Cornering a tumor's fuel routes inside this model is *not* evidence that an intervention shrinks tumors or extends life. Please read [Known Limitations](#known-limitations) before drawing conclusions from anything you see here.

**▶ [Open the live tool](#usage)** · **[How the profiles were derived](PROFILES.md)** · **[How the engine works](ENGINE.md)**

---

## Reading the bars correctly

This is the most common misreading, so it comes first.

**The four bars are not percentages of total ATP, and they do not sum to 100.** Pancreatic sits at glycolysis 85, mitochondrial 30, glutamine 55, fat 10 — that adds to 180. It is not a pie chart.

Each bar is an **independent 0–100 scale of how active that route is**, calibrated against the same scale for every other cancer in the tool. So `glycolysis: 85` does not mean "85% of this tumor's ATP comes from glycolysis." It means: *on a scale where a strongly Warburg-committed tumor sits near 90 and a genuinely non-glycolytic one sits near 30, this tumor sits high.*

**Read 85-versus-30 as meaningful. Read 85-versus-87 as noise.**

The values are **conceptual illustrations of relative tendencies**, not measured quantities or validated predictions. Every number is documented in [PROFILES.md](PROFILES.md), and every number is adjustable in the Sandbox.

---

## Two Learning Modes

The tool offers two complementary ways in. They share the same simulation engine, agents, and visual language — only the starting point differs.

**Guided Cancer Explorer** — pick a named cancer and explore a representative metabolic baseline synthesized from published literature. The approachable entry point: *what metabolic behaviors are commonly described for this cancer?* A **Why these numbers?** panel under the readout shows the reasoning behind every baseline value.

**Metabolic Reasoning Sandbox** — pick a metabolic *archetype* (e.g. "Glycolytic + Glutamine-reliant") rather than a named tumor, then optionally fine-tune the pathway sliders to build a state you want to reason about. This asks: *what happens when metabolism itself changes?*

Three features of the Sandbox are worth calling out:

- **Custom relabeling.** The moment you move any slider, the profile relabels itself as a *Custom Metabolic Profile*. The tool stops claiming the configuration is literature-derived — authority transfers visibly to you.
- **Soft plausibility guidance.** Drag a pathway outside the range commonly reported for that archetype and a subtle cue appears. It never blocks exploration.
- **Fuzzy reverse-mapping.** A custom state surfaces a *set* of cancers "commonly reported with similar characteristics" — deliberately soft, overlapping, with no scores, no percentages, no nearest-match. Cancers enter and leave the set as you drag, which is the point: metabolic phenotype is contextual, not a fingerprint.

## Educational Purpose

This project visualizes concepts from cancer biology, tumor metabolism, and mitochondrial bioenergetics for **learning and discussion**. It aims to help students, researchers, and curious readers build intuition for:

- how the four major ATP-producing pathways relate to one another,
- why blocking a single fuel often triggers **compensatory upregulation** of another,
- how the same interventions play out differently under two competing models of cancer mitochondrial function,
- and how metabolic behavior differs across tumor types.

It is **not** a treatment planner, a clinical decision-support tool, or a validated simulation.

## Scientific Scope

This application is an **educational synthesis of published research**, intended to help users understand concepts in cancer metabolism. It is **not a validated computational model**, does not estimate patient-specific biology or treatment response, and should not be used to guide clinical decisions. Where the scientific literature contains differing interpretations or evolving evidence, the application aims to present representative mechanisms while acknowledging uncertainty and known limitations.

Each cancer profile is a best effort to represent, from the published literature, the fuel sources that type of tumor tends to favor. These are **general tendencies drawn from research, not measurements of any specific tumor** — the same cancer can behave very differently between people, and even between regions of the same tumor. The goal was to get each one into the right ballpark of what the literature describes, as a starting point for understanding, not a precise or individual prediction. The Sandbox exists precisely so that anyone who reads the evidence differently can adjust the model rather than dismiss it.

The full derivation of every baseline value — and what would make each one wrong — is documented in **[PROFILES.md](PROFILES.md)**. The engine's calibration constants are documented in **[ENGINE.md](ENGINE.md)**.

### The two mitochondrial models

Both are presented as competing, unresolved scientific positions; the tool takes no side. Each rests on established biochemistry while making a **contested** empirical claim about cancer:

| | Respiratory-flexible | Fermentation (mSLP) |
|---|---|---|
| **Established** | Oxidative phosphorylation and its biochemistry | Substrate-level phosphorylation at the succinate-CoA ligase step |
| **Contested** | That tumor mitochondria retain respiration as a major ATP source | That mSLP is a major ATP source in cancer, and that tumor mitochondria are respiration-impaired |
| **Status** | The majority view in oncology | A minority view — peer-reviewed, actively researched, making testable predictions with preclinical support. Its proponents argue it has been sidelined rather than answered. |

This may not be a clean either/or. The metabolic phenotypes in this tool differ substantially between tumors, and the honest reading may be that different cancers sit at different points between the two models.

**Several profiles argue against the minority model, and they are included deliberately.** Classical Hodgkin Lymphoma (OXPHOS-anchored, lactate-*importing*), NSCLC (in-vivo ¹³C tracing showing glucose oxidized through the TCA cycle in living tumors), prostate (primarily fat-fueled, which cannot be routed through mSLP), and melanoma (PGC1α-driven mitochondrial *biogenesis*) each sit awkwardly with the fermentation framework — and each says so, in the tool, on the profile itself. A profile set containing only agreeable cases would not be evidence of anything.

### On the agents, and on evidence

The tool includes a number of repurposed and off-patent agents alongside conventional metabolic interventions, and one standard-of-care chemotherapy (gemcitabine, shown only on the pancreatic profile where it is first-line). Gemcitabine is included partly to correct a bias: a tool containing only repurposed and dietary agents would implicitly frame metabolic intervention as the whole universe of options. It also happens to be the clearest demonstration of this tool's central limitation, since almost nothing that makes it a standard of care is visible on these four bars.

The position the tool takes on agents is deliberately narrow:

**Many off-patent compounds have not undergone large randomized oncology trials. This reflects multiple practical, financial, and scientific factors. As a result, clinical efficacy for these agents remains unestablished, regardless of promising mechanistic or preclinical findings.**

The tool does not argue about *why* that evidence is missing. It reports *that* it is missing, and is specific about what is known instead:

Agent mechanisms carry three **separate** ratings, because they answer three different questions and they routinely disagree:

- **Mechanistic confidence** — *is the molecular action real?*
- **Clinical evidence tier** — *how far did it get in humans?*
- **Modeled mechanism in humans** — *are the concentrations at which this tool's modeled mechanism was demonstrated reachable at a tolerable dose?*

That third axis is **a statement about the bars, not a verdict on the agent** — and the distinction is essential. An agent can fail to reach the modeled concentrations and still have real clinical effects through routes this tool does not measure. **Metformin is exactly that case.** The bioenergetic mechanism modeled here (complex-I inhibition) requires millimolar concentrations, while human plasma tops out near 30–40 µM — so the mitochondrial bar barely moves, and that is the *honest* result. Meanwhile retrospective cohorts in pancreatic cancer repeatedly report a survival association concentrated in **early-stage** disease, and the two randomized trials that were negative both enrolled **advanced/metastatic** patients — the very stage at which the observational data also shows no benefit. The randomized evidence has not tested the hypothesis the cohorts generate; the question is open.

The reconciliation the tool offers: metformin's plausible benefit operates at *achievable* micromolar concentrations through AMPK/mTOR, insulin lowering, and the immune microenvironment — all of which lie **off the four axes these bars measure**. A flat bar is the tool pointing somewhere it cannot see, not a dismissal.

Where an effect is applied **by analogy** rather than shown directly, that is flagged explicitly. Preprints are labeled as preprints. The distinction between *mechanistically plausible*, *clinically demonstrated*, and *pharmacologically reachable at the modeled concentration* is the one this tool works hardest to hold.

## Known Limitations

- **This tool models ATP production only.** Metabolism does a great deal that is not ATP accounting — it regulates gene expression, growth signalling, and immune visibility — and none of that appears on these four bars. Glucose availability controls histone acetylation via ATP-citrate lyase (Wellen et al., Science 2009); nutrient sensing runs through AMPK and mTOR; the hexosamine pathway shunts glucose into protein glycosylation; β-hydroxybutyrate is an endogenous HDAC inhibitor. **A small movement on these bars does not mean little is happening — it may mean that what is happening is not on the axes this tool measures.** Metformin is the clearest case: its bioenergetic effect is modest, while its better-supported rationale (AMPK/mTOR, immune microenvironment) lies entirely off-axis.
- **The four bars are not four peers.** Glycolysis makes ATP independently. Glutaminolysis and fatty-acid oxidation are *feeders* whose ATP is collected downstream in the mitochondrion — which is why the engine gates their effective contribution on mitochondrial capacity. The bar chart's visual grammar implies four parallel routes; the biology is not that shape.
- **The engine models fuel routes, never doses.** There is no pharmacokinetics here of any kind.
- Tumors are not metabolically uniform.
- Metabolism differs between primary tumors and metastases.
- Cell culture models often do not replicate human tumors.
- Blocking one pathway frequently causes compensatory pathway activation.
- Metabolic phenotypes change during treatment.
- No single metabolic classification applies universally.

## Educational Disclaimer

This project is an educational resource designed to explain concepts in cancer biology, tumor metabolism, mitochondrial function, and metabolic research. It is not medical advice, does not recommend treatments, and should not be used to make decisions regarding cancer diagnosis, prevention, or therapy.

Cancer metabolism is a rapidly evolving research field. Many mechanisms described here are based on preclinical studies, cell culture experiments, animal models, and early-stage clinical research. Findings from laboratory studies do not necessarily translate into effective or safe human treatments.

Treatment decisions should always be made in consultation with qualified healthcare professionals using evidence-based clinical guidelines.

This tool simplifies complex biological systems for educational visualization and does not represent a complete model of human cancer biology. The pathway values shown are conceptual illustrations of relative tendencies, not measured quantities or validated predictions.

---

## Usage

**Live version:** open `index.html` in any modern browser, or visit the hosted page.

The file is **fully self-contained** (React is inlined) and works with **no internet connection** — suitable for sharing, archiving, or restricted networks. A standalone copy is attached to each [Release](../../releases).

The educational disclaimer appears on every load and must be acknowledged; it can be re-opened at any time from the **About** dialog.

Throughout the tool:

- **Why?** buttons explain engine behaviors (why a pathway moved, why compensation happens) with an evidence level and references, and offer a **Simplified** plain-language view.
- **Why these numbers?** under the Live Readout explains how each baseline value for that cancer was derived — and switches its reasoning depending on which mitochondrial model you are reading under.
- The **ⓘ** beside each agent opens its mechanism, mechanistic confidence, clinical evidence tier, and references.

`index.html` is the single source of truth. There is no build step and no dependencies.

## Feedback and Corrections

**This tool has not yet been reviewed by a domain expert.** It was built as a personal learning project by someone outside the field, and it is being shared in the hope that people who know this literature better will say where it is wrong.

The most useful feedback is specific:

- Are the baseline profiles in the right ballpark? (Every value is documented in [PROFILES.md](PROFILES.md).)
- Does the engine's compensation behavior match what you would expect? (Constants are documented in [ENGINE.md](ENGINE.md).)
- Is any mechanism, evidence tier, or citation misapplied?

If a profile, mechanism, or citation is wrong, **that is worth knowing** — please open an issue. Disagreement about a specific number does not require rejecting the tool: move it in the Sandbox and see whether the resulting behavior still matches what you know.

## Key References

Representative, not exhaustive. Additional per-agent and per-tumor citations appear contextually inside the application.

- Warburg, O. *On the origin of cancer cells.* Science (1956). doi:10.1126/science.123.3191.309

**On the boundary of what this tool models (metabolism as signal, not just fuel):**
- Wellen, K. E., Hatzivassiliou, G., Sachdeva, U. M., Bui, T. V., Cross, J. R., Thompson, C. B. *ATP-citrate lyase links cellular metabolism to histone acetylation.* Science (2009). doi:10.1126/science.1164097 · PMID 19461003

**Supporting the respiratory-flexible model:**
- Vander Heiden, M. G., Cantley, L. C., Thompson, C. B. *Understanding the Warburg effect: the metabolic requirements of cell proliferation.* Science (2009). doi:10.1126/science.1160809 · PMID 19460998
- Weinberg, F., Chandel, N. S. *Mitochondrial metabolism and cancer.* Ann N Y Acad Sci (2009). doi:10.1111/j.1749-6632.2009.05039.x
- Weinberg, F., Hamanaka, R., Wheaton, W. W., et al. *Mitochondrial metabolism and ROS generation are essential for Kras-mediated tumorigenicity.* PNAS (2010). doi:10.1073/pnas.1003428107 · PMID 20421486
- Tan, A. S., et al. *Mitochondrial genome acquisition restores respiratory function and tumorigenic potential of cancer cells without mitochondrial DNA.* Cell Metab (2015). doi:10.1016/j.cmet.2014.12.003
- Zu, X. L., Guppy, M. *Cancer metabolism: facts, fantasy, and fiction.* Biochem Biophys Res Commun (2004).
- Birkenmeier, K., et al. Int J Cancer (2016). doi:10.1002/ijc.29934 · PMID 26595876 — Hodgkin cells as OXPHOS-dependent

**Supporting the fermentation (mSLP) model:**
- Duraj, T., Kalamian, M., Zuccoli, G., Maroon, J. C., D'Agostino, D. P., Scheck, A. C., Poff, A., et al. (incl. Mukherjee, P., Seyfried, T. N.) *Clinical research framework proposal for ketogenic metabolic therapy in glioblastoma.* BMC Medicine (2024). doi:10.1186/s12916-024-03775-4 — peer-reviewed consensus framework (~50 authors); the closest published articulation of what this tool models
- Seyfried, T. N., Arismendi-Morillo, G., Mukherjee, P., Chinopoulos, C. *On the Origin of ATP Synthesis in Cancer.* iScience (2020). doi:10.1016/j.isci.2020.101761 · PMID 33251492
- Chinopoulos, C., Seyfried, T. N. *Mitochondrial Substrate-Level Phosphorylation as Energy Source for Glioblastoma.* ASN Neuro (2018). doi:10.1177/1759091418818261 · PMID 30909720
- Mukherjee, P., et al. *Therapeutic benefit of combining calorie-restricted ketogenic diet and glutamine targeting in late-stage experimental glioblastoma.* Commun Biol (2019). doi:10.1038/s42003-019-0455-x · PMID 31149644
- Mukherjee, P., Maurer, J., Stopka, S. A., et al. (incl. Seyfried, T. N.) *Ketogenic diet as a metabolic vehicle for enhancing the therapeutic efficacy of mebendazole and devimistat in preclinical high-grade gliomas grown in juvenile mice.* bioRxiv (2023). doi:10.1101/2023.06.09.544252 — **preprint, not peer-reviewed**
- Dogra, N., et al. *Fenbendazole acts as a moderate microtubule destabilizing agent and causes cancer cell death by modulating multiple cellular pathways.* Sci Rep (2018). doi:10.1038/s41598-018-30158-6
- Hirsch, H. A., Iliopoulos, D., Struhl, K. PNAS (2013). doi:10.1073/pnas.1221055110
- Shimazu, T., et al. Science (2013). doi:10.1126/science.1227166
- Youm, Y.-H., et al. Nat Med (2015). doi:10.1038/nm.3804
- Shoba, G., et al. Planta Med (1998). doi:10.1055/s-2006-957450
- Petrangolini, G., et al. Evid Based Complement Alternat Med (2021). doi:10.1155/2021/7563889

## Origin

This began as a personal learning project — an effort to understand cancer metabolism deeply enough to visualize it — and was later shared in the hope it is useful to others exploring the same questions. Corrections are welcome: if a profile, mechanism, or citation is wrong, that is worth knowing.

## License

Code is released under the **MIT License** (see [LICENSE](LICENSE)).

Written content — the profile derivations, mechanism explanations, and documentation — is released under **[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)**.

No warranty is made as to scientific completeness or accuracy for any particular purpose; see the disclaimer and limitations above.
