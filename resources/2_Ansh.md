# Ansh (2401010077): Slides 4, 5

**Your part:** the data, cleaning, and the fair split (cross-validation).
**Time:** about 2.5 minutes.

---

## Your script

### Slide 4: The data (about 1 min)

![Slide 4](slides/slide-04.png)

> Thank you, Gauri. Our data is **UJIIndoorLoc**.
> On this map, the coloured dots are the training positions and the crosses are the test scans. There are 3 buildings, and 13 building-floor combinations in total.
> There are 520 access points, but one scan hears only about **18** of them on average. Almost 97% of the readings are "not heard".
> The file writes "not heard" as **100**. That is a problem, because 100 looks stronger than any real signal. So we replaced it with **−110 dBm**, which is weaker than any real reading. Now "not heard" means "very weak", which is what it really is.
> The test set was recorded about **3 months later**, and 9 of its 11 phones are new. So it is a realistic exam.

### Slide 5: Cleaning and fair split (about 1.5 min)

![Slide 5](slides/slide-05.png)

> We cleaned **only the training file**, with three simple rules: remove scans where nothing was heard (76), exact duplicates (634), and scans with a reading of −30 dBm or stronger (508). That is unrealistically strong, and most of these came from one faulty phone.
> 19,937 scans became **18,719**. The test set is never touched.
> Next, how do we test fairly? We use **cross-validation**: split the training data into 5 parts, train on 4, test on the 5th, and repeat five times.
> But we split by **user**, not randomly. Each fold hides whole users with their phones, because the real test set has new phones. Users who shared a phone are kept together, so 18 users became 16 groups.
> This table shows why it matters. A random split promised **3.67 m** error, but the real test gave **12.52 m**. Our user-group split said 15.87 m, which errs on the safe side.
> Now Agnik will explain the features.

---

## Words you should know

| Word | Simple meaning |
|---|---|
| **Training set** | The data the model learns from (18,719 scans after cleaning) |
| **Test set** | The data used once at the end to score the model (1,111 scans, the file `validationData.csv`) |
| **−110 dBm** | Our value for "not heard"; weaker than the weakest real reading (−104) |
| **Duplicate** | Two scans that are exactly the same in every column |
| **Outlier** | A value that is not realistic, like a reading of −30 dBm or stronger |
| **Cross-validation (CV)** | Testing on parts of the training data, 5 times, to choose settings safely |
| **Fold** | One of the 5 parts in cross-validation |
| **User-group CV** | CV where each fold holds out whole users and their phones (GroupKFold) |
| **Group** | One user with their phone. Users who shared a phone form one group: 16 groups |
| **Data leakage** | When test information sneaks into training, so the score looks better than it is |
| **Web-Mercator meters** | The map units of the dataset. 1 data meter is about 0.77 m on the ground |

## What you made / worked on

- **Loading and cleaning** the data (`src/indoorloc/data.py`): the −110 fill, the 3 cleaning rules, and finding the user groups.
- **Downloading and checking** the data files (`scripts/download_data.py`, with a SHA-256 check).
- **Notebook 01**: EDA and data quality, with the campus map and the scans per floor.
- The **cross-validation check**: random CV against user-group CV.

## Questions they may ask you

**Q: Why −110 and not 0, or the original 100?**
A: 100 and 0 would both look like strong signals. −110 is below every real reading, so the model sees "not heard" as "very weak", which is true.

**Q: Why remove readings of −30 dBm or stronger?**
A: That is unrealistically strong for this kind of survey. Of these 508 scans, 430 came from a single phone (phone 19), so it looks like a faulty device.

**Q: Why not clean the test set too?**
A: In real life we cannot pick and choose the scans users send. The test must look like real use.

**Q: Why not a normal random split?**
A: One person scanned the same spot many times, so almost-identical scans land in both train and test. That makes the test too easy: 3.67 m, against the real 12.52 m.

**Q: Why 16 groups and not 18?**
A: Users 1, 9 and 16 used the same phone (phone 14). If we split them apart, the same phone would be in train and test, so they are kept as one group.

**Q: Why 5 folds?**
A: It is a common choice. Each fold still has enough data, and it is not too slow.

**Q: Why is the test score better than the CV score?**
A: In CV, every held-out scan is from a new user and phone. In the test set, about a third of the scans come from 2 phones that were also in training, which is easier. The final model also learns from all the training data.
