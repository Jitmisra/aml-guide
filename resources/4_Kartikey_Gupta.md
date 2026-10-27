# Kartikey Gupta (2401010214): Slides 8, 9

**Your part:** the four models, how we tuned them, how we chose the final model, and the test results.
**Time:** about 2.5 minutes.

---

## First, understand the idea (diagrams)

### Every method = 3 separate models

```mermaid
flowchart LR
    F["475 feature columns"] --> B["Building model<br/>classifier → 0, 1 or 2"]
    F --> FL["Floor model<br/>classifier → 0 to 4"]
    F --> P["Position model<br/>regressor → x and y"]
```

### How the 3 model types think

```mermaid
flowchart TB
    subgraph KNN ["kNN: copy your neighbours"]
        k1["New scan"] --> k2["Find the k most similar training scans"] --> k3["Vote (floor) or average (position)"]
    end
    subgraph RF ["Random Forest: many trees vote together"]
        r1["Tree 1"] --> r4["Average / majority vote<br/>of 300 trees"]
        r2["Tree 2"] --> r4
        r3["... Tree 300"] --> r4
    end
    subgraph XGB ["XGBoost: trees fix each other's mistakes"]
        x1["Tree 1: rough guess"] --> x2["Tree 2: fixes the error of tree 1"] --> x3["Tree 3: fixes what is left"] --> x4["... add them all up"]
    end
```

### One decision tree (a toy example)

```mermaid
flowchart TB
    Q1{"WAP 12 louder than −70?"} -->|yes| Q2{"WAP 40 heard?"}
    Q1 -->|no| A3["Building 2"]
    Q2 -->|yes| A1["Building 0"]
    Q2 -->|no| A2["Building 1"]
```

A tree is just a list of yes/no questions. A **Random Forest** grows 300 trees, each on a random part of the data, and lets them vote.

### kNN toy example (k = 3)

| The 3 nearest training scans | Floor | Position |
|---|---|---|
| Scan A | 2 | (10, 20) |
| Scan B | 2 | (12, 21) |
| Scan C | 3 | (11, 25) |
| **kNN answer** | **2** (majority vote) | **(11, 22)** (average) |

### How we tuned and chose

```mermaid
flowchart TB
    G["Small grid of settings<br/>kNN 28, RF 6, XGBoost 6"] --> CV["Score every setting with<br/>5-fold user-group CV"]
    CV --> BEST["Each task keeps its best setting"]
    BEST --> RULE["Rule fixed BEFORE testing:<br/>lowest CV position error wins"]
    RULE --> W["Random Forest 11.09 m<br/>XGBoost 11.25 m<br/>kNN 12.75 m"]
    W --> FINAL["Final model: Random Forest"]
    FINAL --> TEST["Train on all 18,719 scans<br/>test ONCE on 1,111 scans"]
```

---

## Your script

### Slide 8: Models and tuning (about 1.5 min)

![Slide 8](slides/slide-08.png)

> Thank you, Agnik. Now the models. We compared four. Each one is really **three separate models**: a classifier for the building, a classifier for the floor, and a regressor for the x-y position.
> **kNN**, or k-nearest neighbours, is the simplest. It finds the k training scans that look most similar to the new scan and copies their answer: a vote for the floor, an average for the position. It only uses the raw RSSI.
> **Random Forest** builds **300 decision trees**. Each tree is a chain of yes/no questions, like "is router 12 louder than −70?". Each tree sees a random part of the data, and then they all vote or average. Many trees together make fewer mistakes than one.
> **XGBoost** is boosting. It adds trees **one after another**, and each new tree tries to fix the mistakes of the earlier trees.
> **Baseline RF** is just a plain Random Forest with default settings and no tuning. It is the bar to beat.
> Random Forest and XGBoost get all the features, including Agnik's AP-affinity.
> Then **tuning**: we tried small grids of settings, **28 for kNN and 6 each** for Random Forest and XGBoost. Every setting was scored with the same user-group cross-validation, and each task keeps its own best setting.
> **Before** looking at the test set, we fixed one rule: **the model with the lowest cross-validation position error wins**. Random Forest had 11.09 m, XGBoost 11.25 m and kNN 12.75 m. So our final model is **Random Forest**.

### Slide 9: Test results (about 1 min)

![Slide 9](slides/slide-09.png)

> Now the real exam. Each model is trained on all 18,719 training scans, and then tested **only once** on the 1,111 test scans.
> **Random Forest**, our final model, has the lowest error: on average **7.42 m** from the true spot, compared with 12.52 m for the baseline, so about 40% better. The median is 5.34 m, which means half of the scans are within 5.34 m. It got the **building right 100%** of the time and the **floor 92.2%**.
> **XGBoost** got the most floors right, **95.2%**, but its position error was a little worse, 7.86 m.
> So **no single model is best at everything**, and that leads to our Phase 2 idea. Lokendra will now show where the errors come from.

---

## Words you should know

| Word | Simple meaning | Example / our number |
|---|---|---|
| **Model** | A program that learns from examples and then predicts | Random Forest |
| **Classifier** | A model that predicts a category | Building 0/1/2 |
| **Regressor** | A model that predicts a number | Position x, y |
| **Baseline** | A simple model to compare against; the bar to beat | RF, 300 trees, default settings: 12.52 m |
| **kNN (k-Nearest Neighbours)** | Find the k most similar scans and copy their answer | k = 3, majority vote |
| **Distance (euclidean / manhattan)** | How kNN measures "similar". Euclidean = straight line; manhattan = the sum of the differences | Both were tried |
| **weights (uniform / distance)** | In kNN, whether all neighbours count equally, or closer ones count more | Position used "distance" |
| **Decision tree** | A chain of yes/no questions that ends in an answer | "WAP 12 > −70? → yes → ..." |
| **Random Forest** | Many trees, each trained on random rows and random columns, that vote together (Breiman, 2001) | 300 trees |
| **Bagging** | Training many models on random parts of the data and averaging them; the idea behind Random Forest | — |
| **Boosting** | Adding models one by one, each fixing the earlier errors | XGBoost |
| **XGBoost** | A fast, popular boosting library (Chen & Guestrin, 2016) | 300–600 trees |
| **Hyperparameter / setting** | A choice made *before* training; not learned from the data | k in kNN, tree depth |
| **max_features** | In a forest, how many columns each split may look at | sqrt, 0.2, 0.5 |
| **min_samples_leaf** | The smallest number of scans allowed at the end of a branch | 1 or 4 |
| **max_depth** | How many questions deep an XGBoost tree may go | 4, 6, 10 |
| **n_estimators** | The number of trees | 300 or 600 |
| **learning rate** | How big each XGBoost correction step is (fixed by us) | 0.1 |
| **Grid search / tuning** | Trying every combination from a small list and keeping the best | 28 + 6 + 6 settings |
| **Accuracy** | The % of scans with the correct answer | Building 100%, floor 92.2% |
| **2D error** | The straight-line distance between the predicted and the true spot | 7.42 m on average |
| **Mean vs median** | Mean = the average, pulled up by big misses. Median = the middle value | 7.42 m vs 5.34 m |
| **MAE x / y** | The average error in the x direction and in the y direction separately | 4.77 m and 4.64 m |
| **Final-model rule** | How we pick the winner, decided *before* seeing the test | Lowest CV mean 2D error |

### What we tuned

| Model | Features | Settings tried | Count |
|---|---|---|---|
| Baseline RF | rssi + stats | none (300 trees, defaults) | 1 |
| kNN | rssi | k = 1, 3, 5, 7, 9, 11, 15 × weights × distance | 28 |
| Random Forest | rssi + stats + affinity | max_features (sqrt, 0.2, 0.5) × min_samples_leaf (1, 4) | 6 |
| XGBoost | rssi + stats + affinity | max_depth (4, 6, 10) × n_estimators (300, 600) | 6 |

### The settings that won

| Model | Building | Floor | Position |
|---|---|---|---|
| kNN | k = 1 | k = 5, manhattan | k = 3, distance weights |
| Random Forest | sqrt, leaf 1 | 0.5, leaf 4 | 0.5, leaf 1 |
| XGBoost | depth 4, 300 trees | depth 4, 300 trees | depth 10, 600 trees |

## What you made / worked on

- **The model code** (`src/indoorloc/models.py`): one wrapper that builds the 3 task models for any model type.
- **Cross-validation and tuning** (`src/indoorloc/pipeline.py`): makes the folds, runs CV and does the grid search.
- **The main script** (`scripts/run_experiments.py`): runs every experiment in order, in about 26 minutes.
- **The results:** `cv_results.csv`, `test_results.csv`, `tuning/best_params.json` and one predictions file per model.

## Your notebook: 03 Models & evaluation (sections 1–4)

The notebook does **not** train the models, because that takes 26 minutes. It only reads the saved results from the `results/` folder.

| Section | What it shows | Main result |
|---|---|---|
| 1. Four models | Each model's features and the best settings found | The table above |
| 2. One tuning table | The 6 Random Forest settings for position, with their CV error | max_features 0.5, leaf 1 wins |
| 3. CV results + final model | The CV scores of all models and the rule | Random Forest 11.09 m wins |
| 4. Test results | The test table + a chart of CV vs test for each model | RF 7.42 m, XGBoost floors 95.2% |

(Sections 5–7 are Lokendra's part.)

## Questions they may ask you

**Q: Why these models?**
A: kNN is the classic fingerprinting method, so it is the natural one to try. Random Forest and XGBoost are strong, well-known models for table data. All three are easy to explain.

**Q: Why three separate models instead of one?**
A: It is the simplest approach: classification for building and floor, regression for position, each with its own best settings. The downside is that they share nothing, which is the Phase 2 gap.

**Q: XGBoost gets more floors right. Why choose Random Forest?**
A: We fixed the rule *before* testing: the lowest CV position error. Random Forest won, with 11.09 against 11.25 m. Changing the choice after seeing the test would mean choosing on the test set, which is cheating. We still report XGBoost's 95.2% openly.

**Q: Why such small grids?**
A: Every setting costs a full 5-fold cross-validation run, so small grids keep the work fast and easy to explain.

**Q: Why are the 300 trees not tuned?**
A: More trees make a forest slower, but not worse. 300 is enough to be stable.

**Q: Why is kNN worse than the trees?**
A: kNN used only the raw RSSI. The trees also had AP-affinity, so part of the gap may come from the features, not the model.

**Q: Why is the mean error bigger than the median?**
A: A few big misses, mostly scans with the wrong floor, pull the mean up. The median is the middle scan: 5.34 m, against a mean of 7.42 m.

**Q: What is the difference between Random Forest and XGBoost?**
A: Random Forest grows trees **independently**, in parallel, and averages them. XGBoost grows trees **one after another**, each fixing the earlier errors.

**Q: What does 7.42 m mean in real life?**
A: The dataset uses map meters; one map meter is about 0.77 real meters, so 7.42 is about 5.7 m on the ground. That is roughly the size of a room.
