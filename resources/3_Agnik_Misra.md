# Agnik Misra (2401010044): Slides 6, 7

**Your part:** the features, AP-affinity (our own idea), and the feature check.
**Time:** about 2.5 minutes.

---

## Your script

### Slide 6: Features (about 1.5 min)

![Slide 6](slides/slide-06.png)

> Thank you, Ansh. A model cannot use a scan directly, so we turn it into **features**, which means columns of numbers. We have three kinds.
> **First, RSSI:** one column per access point, 464 in total. Each column is just the signal strength, with −110 for "not heard".
> **Second, three signal statistics:** how many access points were heard, the strongest reading, and the average reading.
> **Third, AP-affinity, which is our own idea.** A raw column tells how *loud* a router is, but not *where* it is. So we do it in three steps.
> **Step 1:** for each access point, we look at all the training scans that hear it and learn its usual building, floor and location. We call this its profile. We only trust access points heard in at least 5 scans.
> **Step 2:** for a new scan, we take the average of the profiles of the access points it hears. Stronger signals count more, because a loud router is probably close.
> **Step 3:** we also add the location of the strongest known access point.
> That gives 8 new columns. They are learned **only from training data**, inside every fold, so a test scan never shapes its own features.

### Slide 7: Feature check (about 1 min)

![Slide 7](slides/slide-07.png)

> Did it help? We kept the same Baseline Random Forest and changed only the features, scored with user-group cross-validation.
> **Adding AP-affinity:** the error went from **15.87 m to 11.38 m**, which is 28% lower, and floor accuracy went from 78.0% to 80.6%. That was our biggest gain.
> To be honest, the **signal statistics did not help**: the error went slightly up, from 15.55 to 15.87 m.
> We still kept them, because the full set together scored best.
> Now Kartikey will show the models.

---

## Words you should know

| Word | Simple meaning |
|---|---|
| **Feature** | One input column the model uses |
| **Feature engineering** | Making new, more useful columns from the raw data |
| **RSSI features** | The 464 signal-strength columns (WAPs heard in training) |
| **n_heard** | How many access points the scan heard |
| **max_rssi** | The strongest reading in the scan |
| **mean_rssi** | The average of the heard readings |
| **AP-affinity** | Our feature: where the access points a scan hears usually are |
| **Profile** | For one WAP: its usual building (3 shares), floor, and location (x, y) |
| **Weighted average** | An average where some values count more. Here, stronger signals count more |
| **Powed weight** | The weight formula `((rssi + 105) / 105) ^ e`. Roughly 0.27 at −40 dBm, 0.05 at −70 and almost 0 at −100. Taken from the UJIIndoorLoc authors (2015) |
| **Min support = 5** | A WAP needs at least 5 training scans, or its profile is not reliable |
| **Strongest known WAP** | The loudest WAP in the scan that has a profile |
| **Leakage** | The answer sneaking into the input. We avoid it by learning profiles only from training rows |

### The 8 AP-affinity columns

| Column | Meaning |
|---|---|
| aff_b0, aff_b1, aff_b2 | How much the heard WAPs "belong" to building 0, 1 and 2 |
| aff_floor | The average floor of the heard WAPs |
| aff_lon, aff_lat | The average location of the heard WAPs |
| aff_top_lon, aff_top_lat | The location of the strongest known WAP |

## What you made / worked on

- **The feature code** (`src/indoorloc/features.py`): the signal statistics, the `APAffinity` class with fit and transform, and the `FeatureBuilder`.
- **Notebook 02**: feature engineering and the feature check.
- **The feature check experiment** (`results/metrics/feature_check.csv`) and its figure.

## Questions they may ask you

**Q: What is AP-affinity in one line?**
A: It tells the model where the routers a scan hears usually are, so the model gets a "where" clue and not just a "how loud" clue.

**Q: Is it cheating, or leakage?**
A: No. Profiles are learned only from training rows, and rebuilt in every CV fold. A held-out or test scan's true location never goes into its own features. We judge it only on held-out folds and the test set.

**Q: Why do stronger signals count more?**
A: A loud router is probably close to you, so its location says more about where you are.

**Q: Why at least 5 scans?**
A: With only 1 or 2 scans, the profile could be badly wrong. 5 is a simple safe minimum.

**Q: What if a scan hears no known WAP?**
A: It gets the training average values. This is rare.

**Q: Why the exponent e?**
A: It comes from the "powed" formula suggested by the dataset's authors (Torres-Sospedra et al., 2015). We did not tune it.

**Q: The stats did not help. Why keep them?**
A: The feature sets were tested in a nested way (rssi → +stats → +affinity), and the full set scored best. We did not test RSSI plus AP-affinity without the stats. We say this honestly.

**Q: Is the gain real, or just luck?**
A: AP-affinity lowered the error in **all 5 folds**, not just on average.
