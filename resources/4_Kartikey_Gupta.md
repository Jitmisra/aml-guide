# Kartikey Gupta (2401010214): Slides 8, 9

**Your part:** the models, tuning, the final-model rule, and the test results.
**Time:** about 2.5 minutes.

---

## Your script

### Slide 8: Models and tuning (about 1.5 min)

![Slide 8](slides/slide-08.png)

> Thank you, Agnik. We compared four models. Each model is really **three separate models**: one for the building, one for the floor and one for the position.
> **kNN** finds the most similar training scans and copies their answer.
> **Random Forest** builds many decision trees on random parts of the data and averages them.
> **XGBoost** adds trees one by one, and each new tree fixes the mistakes of the earlier ones.
> **Baseline RF** is a plain Random Forest with default settings and no tuning. It is our starting point.
> We tuned each model with **small grids**: 28 settings for kNN and 6 each for Random Forest and XGBoost, all scored with the same user-group cross-validation. Each task keeps its own best setting.
> Before testing, we fixed one rule: **the lowest cross-validation error wins**. Random Forest had 11.09 m, XGBoost 11.25 m and kNN 12.75 m, so our final model is **Random Forest**.

### Slide 9: Test results (about 1 min)

![Slide 9](slides/slide-09.png)

> We used the test set only **once**, on 1,111 scans.
> Random Forest has the lowest mean error, **7.42 m**, against 12.52 m for the baseline. It got the building right **100%** of the time and the floor 92.2%.
> XGBoost got the most floors right, 95.2%. So **no single model is best at everything**.
> Now Lokendra will show where the errors are.

---

## Words you should know

| Word | Simple meaning |
|---|---|
| **Baseline** | A simple starting model to compare against (RF, 300 trees, default settings) |
| **kNN** | k-Nearest Neighbours: copy the answer of the k most similar scans |
| **Decision tree** | A chain of yes/no questions, like "is WAP 12 stronger than −70?" |
| **Random Forest** | Many trees, each on random data and random columns; it averages their votes |
| **XGBoost** | Boosting: trees are added one after another, each fixing the earlier errors |
| **Hyperparameter / setting** | A choice made before training, like k in kNN or the depth of a tree |
| **Grid search / tuning** | Trying every combination in a small list of settings and keeping the best |
| **Accuracy** | The % of scans with the correct building (or floor) |
| **2D error** | The straight-line distance between the predicted and the true position, in meters |
| **Mean vs median error** | Mean = the average (pulled up by big misses). Median = the middle scan |
| **MAE x / y** | The average error in the x direction and in the y direction separately |
| **Final-model rule** | The pre-decided way to choose the final model: lowest CV mean 2D error |

### What we tuned

| Model | Features | Settings tuned | Count |
|---|---|---|---|
| Baseline RF | rssi + stats | none (300 trees, defaults) | 1 |
| kNN | rssi | k (1–15), weights (uniform/distance), distance (euclidean/manhattan) | 28 |
| Random Forest | rssi + stats + affinity | features per split (sqrt, 0.2, 0.5), leaf size (1, 4) | 6 |
| XGBoost | rssi + stats + affinity | tree depth (4, 6, 10), number of trees (300, 600) | 6 |

Fixed for XGBoost (not tuned): learning rate 0.1, row sample 0.85, column sample 0.5.

## What you made / worked on

- **The model code** (`src/indoorloc/models.py`): one wrapper that builds the 3 task models for any family.
- **Cross-validation and grid search** (`src/indoorloc/pipeline.py`).
- **The main script** that runs every experiment (`scripts/run_experiments.py`, about 26 minutes for the full run).
- **The results tables**: `cv_results.csv`, `test_results.csv` and `tuning/best_params.json`.

## Questions they may ask you

**Q: Why these models?**
A: kNN is the classic fingerprinting method. Random Forest and XGBoost are strong, well-known tree models for table data. All three are easy to explain.

**Q: Why three separate models instead of one?**
A: It is the simplest approach: classification for building and floor, regression for position, each with its own best settings. The downside is that they share nothing, and that is our Phase 2 gap.

**Q: XGBoost has better floors. Why choose Random Forest?**
A: We fixed the rule before testing: the lowest CV position error. Random Forest won (11.09 against 11.25 m). Changing the pick after seeing the test would be choosing on the test set, which is cheating. We still report XGBoost's 95.2% openly.

**Q: Why such small grids?**
A: Every setting costs one full 5-fold run, so small grids keep it fast and simple.

**Q: Why are 300 trees not tuned?**
A: More trees make a forest slower, but not worse. 300 is enough.

**Q: Why is kNN worse?**
A: kNN only used the raw RSSI. The trees also had AP-affinity, so part of the gap may come from the features.

**Q: Why is the mean bigger than the median?**
A: A few big misses, mostly wrong floors, pull the mean up. The median is the middle scan: 5.34 m, against a mean of 7.42 m.

**Q: What do the best Random Forest settings look like?**
A: Building: sqrt features, leaf 1. Floor: 0.5 of the features, leaf 4. Position: 0.5 of the features, leaf 1.
