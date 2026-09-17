---
title: "CVIRL Work - Rebuilding the Annotation Tool"
date: "2026-06-10"
description: "Cutting annotation time from 15 minutes a video to 5, what actually makes a labeling tool slow, and why step order matters more than step speed"
dayNumber: 1
---

# Rebuilding the annotation tool

my lab's training labels are drawn by hand. someone opens a GUI, traces the aortic root walls on an echo video, saves, moves to the next one.

i inherited that GUI and then had to use it a few dozen times, which is the fastest possible way to find out what's wrong with a tool.

took annotation from **~15 minutes a video down to 5–7**. here's what actually did it.

### what the tool produces

the important design decision was already there and it's a good one: **you don't click every point on the curve.**

you place a handful of control points and the tool fits a smooth curve through them.

```
valsalva_control_points.npz   2 curves x 8 control points   <- what a human places
valsalva_nurbs_curves.npz     2 curves x 100 points         <- what gets fitted
annular_plane.npz             2 annulus points
idx_key_frames.npz            start + end frame indices
```

**NURBS:**  Non-Uniform Rational B-Spline. a curve defined by a few control points instead of by every point along it

so a human does 8 clicks and the model gets 100 coordinates. that ratio is the entire reason manual annotation is feasible at all.

‼️ **save both.** the 100-point curve is what the model trains on, but the 8 control points are what a person can come back and *edit*. throw those away and every correction is a redo

---

### where the 15 minutes went

none of it was the drawing. the drawing is the job. it was everything around the drawing:

- **the tool did too much.** it had a batch mode bolted onto it, plus a pile of files that weren't part of the annotation path at all. more surface, more ways to be in the wrong state
- **phase ordering was wrong.** the tool walks you through phases, and one landmark — the **sinotubular junction**, where the root transitions into the ascending aorta — was being marked *after* the phase that needed it as a reference. so you'd do a phase, realize the reference wasn't placed yet, go back
- **every redo was a full redo**, because corrections weren't cheap

the big one is the second. it wasn't slow because any step was slow, it was slow because the **order forced backtracking**.

---

### what i changed

**1. strip it to one job.**

removed batch mode, isolated only the files the GUI path actually needs, committed that as its own thing.

a tool that does one workflow well beats a tool that technically supports three. batch mode wasn't being used by anyone annotating — it was just extra state to get stuck in.

**2. reorder the phases so information flows forward.**

mark the STJ *before* the phase that references it. that's it. that's the change that did most of the work.

the general version:

> order the steps so that each one only needs things the annotator has already provided. every time a step needs something from a later step, you've built a loop.

**3. measure it on real videos.**

i timed myself annotating before and after, on actual videos, not a stopwatch guess:

```
before:  ~15 min / video
after:   5–7 min / video
```

this matters more than it sounds. "feels faster" is not a result, and if you're going to ask other people to switch tools you need an actual number. it's also how you find out that the change you were proud of did nothing.

**4. record a screen capture of the new flow.**

the fastest way to onboard the other annotators, and it doubles as documentation that doesn't go stale as fast as a written doc does.

---

### annotators are part of the dataset

a thing i didn't expect to be thinking about: **who drew which label is data.**

we each took a color, and the assignments were tracked. that sounds cosmetic, and it isn't:

- if one annotator's cases turn out to be systematically different, you can find that out
- when a case looks wrong months later, you can ask the person who drew it
- it makes inter-annotator disagreement measurable instead of invisible

related, and worth writing down because i only noticed it from staring at frames for hours: the videos carry **burned-in extras** — text overlays, an ECG trace along the bottom, a grayscale calibration gradient. a human ignores those automatically. a model has no reason to. if those regions correlate with anything in the labels, it can learn them.

annotating is the only part of the pipeline where a person looks at every single frame. that makes it the best opportunity you'll get to notice things about your data.

---

### takeaways

- **use your own tool for real work.** i found every one of these by annotating, not by reading the code
- the fix is usually **step order**, not step speed. backtracking is where the time goes
- **delete the modes nobody uses.** less surface, fewer wrong states
- save the **editable** representation (control points), not just the derived one (fitted curve)
- **time it before and after**, on real inputs
- track **who** annotated what, and pay attention to what's burned into the frames
