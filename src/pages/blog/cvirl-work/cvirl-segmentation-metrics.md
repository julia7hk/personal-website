---
title: "CVIRL Work - Dice vs IoU vs Hausdorff"
date: "2026-07-16"
description: "What each segmentation metric actually measures and penalizes"
dayNumber: 2
---

my model segments the aortic root in echo videos

started out scoring it with Dice only, then added IoU and Hausdorff distance because Dice by itself wasn't telling me what i needed to know.

writing down what each one actually measures, because "higher is better" is not enough when you're trying to pick between training runs.

### the setup

the model doesn't output a mask. it outputs a **probability per pixel**.

so first you threshold it:

```python
pred_mask = (prob_map > 0.5)
```

threshold:  the confidence cutoff for calling a pixel part of the object

- `0.5` → pixel must be >50% confident
- `0.2` → pixel must be >20% confident

lower threshold = more pixels included = bigger predicted blob

now you have two binary masks and you can compare them:

- `A` = prediction
- `B` = ground truth (my annotations)

---

### Dice

**Dice coefficient**, a.k.a. F1 for pixels

```
Dice = 2|A ∩ B| / (|A| + |B|)
```

```python
inter = (pred & gt).sum()
dice = 2 * inter / (pred.sum() + gt.sum())
```

range 0 → 1, higher is better

**what it measures:** how much the two regions overlap, normalized by their combined size

**what it penalizes:** pixels you missed and pixels you invented, equally

**what it ignores:** ‼️ *where* the errors are

this is the important part. Dice counts pixels, it doesn't care about geometry. a region is mostly interior, so if you nail the bulk of the blob and are sloppy around the edge, Dice barely moves. the interior pixels drown out the boundary pixels.

also it gets unstable on small objects — when `|A| + |B|` is tiny, a handful of pixels swings the score a lot.

---

### IoU

**Intersection over Union**, a.k.a. the Jaccard index

```
IoU = |A ∩ B| / |A ∪ B|
```

```python
inter = (pred & gt).sum()
union = (pred | gt).sum()
iou = inter / union
```

measures the same *thing* as Dice, but scores it harder.

the two are locked together:

```
IoU  = Dice / (2 - Dice)
Dice = 2 * IoU / (1 + IoU)
```

so **IoU ≤ Dice, always** (they're only equal at exactly 0 or 1)

| Dice | IoU |
| --- | --- |
| 0.95 | 0.905 |
| 0.90 | 0.818 |
| 0.85 | 0.739 |
| 0.80 | 0.667 |

the difference is the denominator. Dice counts the intersection twice, which softens the penalty for errors. IoU counts the union once, so every wrong pixel shows up at full weight.

‼️ **they can never disagree on ranking.** the formula is monotonic — if run A beats run B on Dice, it beats it on IoU too. so if you're comparing two models, IoU tells you nothing new.

so why report both? mostly convention (papers report both so you can compare to them), and IoU spreads out the scores more, which makes a table of near-identical runs easier to read.

---

### Hausdorff distance

this one is completely different. it's the one that told me something new.

```
h(A,B) = max over a in A of ( min over b in B of ||a - b|| )
H(A,B) = max( h(A,B), h(B,A) )
```

in words: for every point on my predicted boundary, find the closest point on the true boundary. take the worst one. then do it the other direction too and take the worse of the two.

**what it measures:** the largest distance between the two boundaries

**what it penalizes:** ‼️ the single worst point. one stray pixel ruins it

**what it ignores:** overlap entirely. it never looks at the interior

notes:

- units are **pixels** (or mm if you scale it), not a 0–1 score
- **lower is better**, which is a nice way to misread a results table at 1am
- bidirectional matters — you need both directions or you miss errors in one mask

---

### why i actually needed it

this is the case that made Dice-only useless for me:

| | Dice | IoU | Hausdorff |
| --- | --- | --- | --- |
| mask is right, edge slightly fuzzy | high | high | low ✅ |
| mask is right + one small spike off to the side | **still high** | still high | **terrible** 🚩 |
| mask is shifted a few px but same shape | slightly lower | slightly lower | moderate |
| model predicted nonsense across the frame | ~0 | ~0 | huge |

row 2 is the killer. a prediction that's 95% correct with one spur sticking out scores almost identically to a clean one on overlap metrics, and Hausdorff immediately flags it.

and for my use case that spur is the whole problem. the segmentation isn't the end product — i measure the **diameter** of the aortic root from the boundary. an error that barely dents an area-overlap score can move a diameter measurement a lot.

the rule i settled on:

> overlap metrics tell you if you found the right thing. boundary metrics tell you if you traced it correctly.

**HD95** is worth knowing too — the 95th percentile of those distances instead of the strict max. same idea, but one bad annotation pixel doesn't blow up the whole number. plain Hausdorff is genuinely worst-case, which sometimes says more about my labels than the model.

---

### threshold sweep

since the threshold changes the mask, it changes every metric. so evaluate across a few instead of picking one:

```python
for t in (0.2, 0.3, 0.4, 0.5):
    pred = prob_map > t
    ...
```

what i look for: **are the scores stable across thresholds?** if 0.2 and 0.5 give roughly the same answer, the model is confident and the decision boundary is sharp. if the score swings a lot, it's sitting on the fence and the number you report is basically a function of which threshold you picked.

lower threshold → bigger mask → usually higher Dice (catches more true pixels), at the cost of false positives.

---

### per-frame vs per-video

first version of my eval averaged over every frame in the test set. that's wrong for my data.

videos have different frame counts, so a long video quietly gets more say in the average than a short one. and i care about per-*case* performance, not per-frame.

```python
# score each frame, group by video, average within video first
# then average (or plot) across videos
```

two things that bit me:

- **deduplicate.** my inference runs in overlapping sequences of frames, so the same frame can get scored more than once. dedupe before aggregating or the overlapping regions get double-counted
- **save the per-frame values, not just the mean**

that second one cost me a rerun. i had scripts that wrote out only a `test_metrics.csv` of final averages. later i wanted box-and-whisker plots and needed the raw distribution — which i'd thrown away. had to write a separate script to regenerate the per-frame and per-video numbers from the saved checkpoints.

save all three from the start:

1. per-frame csv — every frame's Dice / IoU / Hausdorff
2. per-video csv — aggregated, mean **and** std
3. the predicted masks themselves — so you can make overlays later without rerunning inference

---

### averages hide everything

a mean Dice hides the shape of the distribution completely. two models with the same average can be:

- consistently okay on every case
- great on most and catastrophically bad on a couple

those are very different models and the second one is the one that embarrasses you. box plots made my outlier cases obvious in a way the summary table never did, and then i could go pull up those specific videos and look at what the model was doing.

---

### takeaways

- **Dice** — overlap, forgiving, dominated by the interior of the region
- **IoU** — same information, harsher scale, can't disagree with Dice on ranking
- **Hausdorff** — worst-case boundary distance, in pixels, catches what overlap metrics structurally cannot
- report a **threshold sweep**, and check stability across it
- aggregate **per case**, not per frame
- keep the **raw distributions**. you will want them later
