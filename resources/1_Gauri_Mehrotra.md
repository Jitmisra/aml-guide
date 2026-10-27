# Gauri Mehrotra (2401010170): Slides 1, 2, 3

**Your part:** the opening, the problem, what others did (related work), and the gap.
**Time:** about 2.5 minutes.

---

## Your script

### Slide 1: Title (about 30 sec)

![Slide 1](slides/slide-01.png)

> Good morning everyone. We are Team T084 from Section D. Our project is **Indoor Location Prediction using Wi-Fi Signals**.
> I am Gauri Mehrotra, and with me are Ansh, Agnik Misra, Kartikey Gupta and Lokendra Singh.
> This is Phase 1, where we use classical machine learning.

### Slide 2: The problem (about 1 min)

![Slide 2](slides/slide-02.png)

> We all use GPS outside, but inside a building GPS works badly, because walls and roofs block the satellite signal.
> But almost every building has Wi-Fi. A phone can hear many Wi-Fi access points at the same time, some loud and some weak.
> One **scan** is simply a list of how strongly the phone hears each access point. This strength is called **RSSI**, and it is measured in dBm. A value closer to zero means a stronger signal.
> From one scan we predict three things: the **building** and the **floor**, which are classification, and the exact **(x, y) position**, which is regression.
> We use the public UJIIndoorLoc dataset: 3 buildings, about 20 thousand training scans and 1,111 test scans.

### Slide 3: Related work and the gap (about 1 min)

![Slide 3](slides/slide-03.png)

> The classic idea comes from **RADAR**, in 2000: store scans at known places, and when a new scan comes, answer with the closest stored scans. That is basically **kNN**.
> The **UJIIndoorLoc** team made the dataset public and tested kNN on it. A later comparison found kNN best, with tree models close behind. Recent work uses neural networks.
> We found two gaps.
> **First**, many papers use a random split, which looks too good. One study got 98.2% on a random split but only 91.1% on the real later test file.
> **Second**, in our project the three tasks are learned by separate models that share nothing. But one shared network has been shown to work on this data, by Kim et al. in 2018. That is what Phase 2 is about.
> Now Ansh will explain the data.

---

## Words you should know

| Word | Simple meaning |
|---|---|
| **GPS** | Satellite positioning. Good outside, bad indoors |
| **WAP / access point** | A Wi-Fi router. The dataset has 520 of them |
| **RSSI** | Received Signal Strength: how loud a WAP is heard |
| **dBm** | The unit of RSSI. Always negative here; −40 is strong and −100 is very weak |
| **Scan** | One reading of all 520 WAPs at one place and time |
| **Fingerprinting** | Using the signal pattern of a place as its "fingerprint" |
| **Classification** | Predicting a category (building 0/1/2, floor 0–4) |
| **Regression** | Predicting a number (the x and y position) |
| **kNN** | k-Nearest Neighbours: find the most similar old scans and copy their answer |
| **Random split** | Mixing all the data randomly into train and test. Too easy here |
| **Multi-task network** | One neural network that learns several outputs together (our Phase 2) |
| **UJIIndoorLoc** | Our dataset, from Universitat Jaume I in Spain, available on UCI |

## What you made / worked on

- The **problem statement** and the **literature review** (`docs/literature_review.md`, 10 works compared).
- The **Introduction** and **Related Work** sections of the report.
- The **reference list** (17 papers, in `report/references.bib`).

## Questions they may ask you

**Q: Why not just use GPS?**
A: Indoors, walls and roofs weaken the GPS signal a lot, so it becomes unreliable. Wi-Fi is already inside most buildings, so we get it for free.

**Q: What is RADAR?**
A: The first well-known Wi-Fi fingerprinting system, from Microsoft in 2000. It matches a new scan to the nearest stored scans, which is the same idea as kNN.

**Q: Why is building and floor classification, but position regression?**
A: Building and floor are fixed categories, like 0, 1 or 2. Position is a continuous number, in meters.

**Q: What is the gap you found?**
A: First, random splits give scores that are too optimistic. Second, our three task models share nothing, while one shared network could learn all three together.

**Q: Why this dataset?**
A: It is public and widely used, so results can be compared. It also has a separate test file recorded months later with new phones, which is a realistic test.

**Q: Which papers did you use most?**
A: Bahl & Padmanabhan 2000 (RADAR), Torres-Sospedra 2014 and 2015 (the dataset), Nowicki 2017 (random split vs real test), and Kim 2018 (one shared network).
