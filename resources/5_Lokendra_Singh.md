# Lokendra Singh (2401010254): Slides 10, 11

**Your part:** where the errors are, the Phase 2 gap, and the closing.
**Time:** about 2 minutes.

---

## Your script

### Slide 10: Where the errors are (about 1 min)

![Slide 10](slides/slide-10.png)

> Thank you, Kartikey. Now let's see where our final Random Forest still makes mistakes on the test set.
> The **building is never wrong**: 0 mistakes.
> The floor is wrong for only **7.8%** of the scans. Those scans have a bigger error, 13.06 m, but there are few of them.
> The surprising part: **86% of the total error** is on scans where the building and floor are already right. So the main problem is the **fine position** inside the floor.
> Also, **81 of the 87 floor mistakes are just one floor off**, up or down.
> And **new phones are harder**: 8.30 m error, against 5.84 m on phones seen in training.

### Slide 11: Phase 2 gap and closing (about 1 min)

![Slide 11](slides/slide-11.png)

> So what did Phase 1 show?
> AP-affinity helped most. There is no single best model: Random Forest is best for position and XGBoost for floors. And most of the error is in fine position.
> **The gap:** our three task models share nothing. AP-affinity shares location knowledge between them, but only by hand.
> **Phase 2:** we will build **one multi-task neural network**, with shared layers and three outputs, for building, floor and position, so the tasks can learn from each other. We will use the same split, the same test set and the same metrics, so we can compare fairly with Phase 1.
> Thank you! We are happy to take questions.

---

## Words you should know

| Word | Simple meaning |
|---|---|
| **Error analysis** | Looking at *where* and *why* the model is wrong, not just the total score |
| **Error breakdown** | Splitting the scans into 3 cases: building wrong / floor wrong / both right |
| **Share of error** | How much of the total error comes from one case |
| **Floor confusion (matrix)** | A table of true floor against predicted floor, showing which floors get mixed up |
| **Seen vs new phones** | Phones that were in training (2) against phones never seen (9) |
| **Phase 2** | The next part of the course, using deep learning |
| **Neural network** | A model made of layers of simple units that learn patterns |
| **Multi-task learning** | One model learning several related tasks together (Caruana, 1997) |
| **Shared layers** | The part of the network used by all three tasks |
| **Output heads** | The 3 final parts: one each for building, floor and position |

### The numbers (Random Forest, test set)

| Case | Scans | Share of scans | Mean error | Share of total error |
|---|---:|---:|---:|---:|
| Building wrong | 0 | 0% | – | 0% |
| Building right, floor wrong | 87 | 7.8% | 13.06 m | 13.8% |
| Both right | 1,024 | 92.2% | 6.94 m | 86.2% |

| Phones | Scans | Floor accuracy | Mean error |
|---|---:|---:|---:|
| Seen in training (2 phones) | 397 | 93.7% | 5.84 m |
| New (9 phones) | 714 | 91.3% | 8.30 m |

## What you made / worked on

- **The metrics code** (`src/indoorloc/metrics.py`): the per-scan errors, the error breakdown, the per-phone results and the floor confusion.
- **The figures** (`src/indoorloc/plots.py`): 7 charts, including the floor confusion and the error map.
- **Notebook 03**: the ML models and the evaluation.
- **The Phase 2 plan**, and the conclusion of the report.

## Questions they may ask you

**Q: What is the main problem left?**
A: Fine position. 86% of the error is on scans where the building and floor are already right.

**Q: Why is a wrong floor costly?**
A: If the floor is wrong, the position model is often confused too: 13.06 m error, against 6.94 m when everything is right.

**Q: Why are new phones worse?**
A: Different phones measure signal strength a bit differently, so the same place can "look" different to a new phone.

**Q: Where do most floor mistakes happen?**
A: 81 of the 87 are one floor off, and 51 of the 87 are in building 1.

**Q: What will Phase 2 change?**
A: One neural network with shared layers learns all three tasks together, instead of three separate models. It will be compared with Phase 1 using the same split, test set and metrics.

**Q: Why do you think multi-task will help?**
A: The tasks are related: knowing the floor helps the position, and the reverse. In Phase 1, AP-affinity shared this knowledge by hand and gave our biggest gain. A shared network can learn that sharing by itself. Kim et al. (2018) already trained one network like this on the same dataset.

**Q: What does the error map show?**
A: Most predictions are close. A few in building 2 land between corridors, where no scan was ever taken. The model probably averages places it cannot tell apart.
