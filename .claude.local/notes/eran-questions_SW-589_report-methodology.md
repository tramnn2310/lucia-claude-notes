# Questions for Eran — model-report methodology (SW-589)

These are the open methodology decisions blocking the remaining report fixes. Each
has my (the algo team's) recommended default so Eran can just confirm or redirect.
Framed in domain terms — no internal finding IDs. For Tram to forward.

---

**1. Counting a patient who has several visits/sites.**
The patient-level statistics currently group by visit/site as well as patient, so a
patient with two visits is counted as two "patients" in the TP/FN tallies, the ROC,
and the Mann-Whitney n (e.g. 30 rows for 26 patients). We want one row per patient.
When a patient has multiple visits, how should we collapse them to a single score?
- *Recommended:* mean predicted probability across the patient's visits, one row per patient.
- Alternatives: take the latest visit only; max probability; majority vote on the
  per-visit class.

**2. Multiple-testing correction for the confounder tests.**
The confounder section runs many factor-level comparisons with no correction, so some
"significant" confounders are expected by chance. Which correction do you want, and
over which family of tests (per section, or all confounder tests together)?
- *Recommended:* Benjamini-Hochberg (FDR) applied per report section.
- Alternative: Bonferroni (more conservative).

**3. AUC / threshold with the greyzone removed.**
AUC, the operating threshold, and the Mann-Whitney test are currently computed after
dropping greyzone patients. That makes the AUC optimistic and not comparable with a
full-cohort AUC. How should the report present this?
- *Recommended:* report full-cohort metrics as primary, and keep the greyzone-omitted
  numbers clearly labelled as secondary.
- Alternatives: full-cohort only; greyzone-omitted only (status quo).

**4. Which set the operating threshold is fit on (and reported with).**
The report builder's default fits the threshold on the test set and then reports the
test set with that same threshold — i.e. it reports on the set it tuned on. The
pipeline overrides this to validation, but anyone calling the builder directly (e.g.
from a notebook) inherits the test-set default. Confirm we should change the default
so the threshold is fit on validation, and the test set is only ever reported with a
validation-fit threshold.
- *Recommended:* yes — default to fitting on validation.

**5. Smaller consistency points (quick confirmations).**
- Greyzone boundary: the model report treats the boundary as inclusive, the feature
  report treats it differently. Confirm the boundary should be **inclusive** everywhere.
- Patient age is computed against today's date rather than the sample/collection date.
  Confirm we should compute age at sample date.
- Confidence intervals: none are reported anywhere (AUC, sensitivity/specificity, PPV/
  NPV). Do you want CIs added in this pass, or is that a later item?
