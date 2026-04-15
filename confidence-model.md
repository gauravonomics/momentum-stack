# Multi-Signal Confidence Model

## Overview

The confidence model is a gating system that decides when to surface suggestions to the operator and when to stay silent. It prevents low-quality noise from overwhelming the operator by requiring results to pass a multi-signal threshold before being shown.

The core design principle: **below threshold means silence, not noise.** A system that shows everything teaches the operator to ignore it. A system that speaks only when confident earns attention.

## The Problem It Solves

Without confidence gating, a knowledge system surfaces everything -- stale notes, low-authority session closures, goal-irrelevant tangents. The operator spends time filtering noise instead of acting on signal. Over time, they stop trusting the system's suggestions entirely.

The confidence model solves this by scoring every potential result on multiple dimensions and only surfacing results that clear a caller-specific threshold. Different rituals have different tolerances for noise, so each caller defines its own weight profile and threshold.

## Five Signals

### 1. Recency

- **What it measures:** How recently the entity was created or updated
- **How it works:** Decay curve -- newer entities score higher, older entities decay toward zero
- **Why it matters:** Recent entities are more likely to be relevant to current work
- **Score range:** 0.0 (very old) to 1.0 (created/updated today)

### 2. Entity Authority

- **What it measures:** The inherent weight of different entity types
- **How it works:** Goals and projects score higher than session closures. Type-based weighting.
- **Example hierarchy:** Goals > Projects > Atomic Notes > Literature Notes > Session Closures
- **Why it matters:** Not all entities are equally authoritative. A goal file carries more weight than a single session closure.
- **Score range:** 0.0 (lowest authority type) to 1.0 (highest authority type)

### 3. Source Count

- **What it measures:** How many independent raw sources back this entity
- **How it works:** More independent sources = higher confidence
- **Why it matters:** Multi-sourced claims are more reliable than single-source claims
- **Score range:** Normalized -- single source = 0.2, two sources = 0.5, three+ sources = 0.8-1.0

### 4. Goal Alignment

- **What it measures:** Whether this entity connects to an active goal with open to-dos
- **How it works:** Binary -- connected to active goal = 1, not connected = 0
- **Why it matters:** Goal-aligned results are more likely to move work forward
- **Score range:** 0 or 1

### 5. Access Recency

- **What it measures:** How long since the operator last accessed this entity
- **How it works:** Inverse of days since last access -- entities not seen in 14+ days score higher
- **Why it matters:** Resurfaces forgotten-but-relevant knowledge that might otherwise rot
- **Score range:** 0.0 (accessed today) to 1.0 (not accessed in 30+ days)

## Per-Caller Weight Profiles

Different rituals need different signal emphasis:

| Caller | Recency | Authority | Source Count | Goal Alignment | Access Recency | Default Threshold |
|--------|---------|-----------|-------------|----------------|----------------|-------------------|
| Daily plan (backward look) | 0.4 | 0.2 | 0.1 | 0.2 | 0.1 | 0.5 |
| Daily plan (surfacing) | 0.2 | 0.2 | 0.1 | 0.3 | 0.2 | 0.7 |
| Weekly review | 0.1 | 0.2 | 0.2 | 0.3 | 0.2 | 0.6 |
| Essay pipeline | 0.1 | 0.3 | 0.2 | 0.1 | 0.3 | 0.7 |
| User query (default) | 0.2 | 0.3 | 0.2 | 0.2 | 0.1 | 0.5 |

**Why different weights:**

- **Daily plan backward look** emphasizes recency because it is reviewing what just happened.
- **Daily plan surfacing** emphasizes goal alignment because it is recommending what to work on next.
- **Weekly review** emphasizes goal alignment and source count for structural pattern discovery.
- **Essay pipeline** emphasizes authority and access recency to find forgotten high-quality insights.
- **User query** is balanced as the general-purpose default.

## Two-Dimensional Gating

Each result receives two scores:

### Dimension 1: Relevance

How relevant is this entity to the query? Calculated as a weighted combination of the 5 signals using the caller's weight profile.

### Dimension 2: Momentum

How likely is this entity to move work forward? Three factors:

- Is the entity connected to a goal with active to-dos?
- Has the entity been accessed recently? (inversely -- forgotten entities have higher momentum potential)
- Is the entity type actionable? (to-dos > projects > essay ideas > atomic notes > session closures)

**Both dimensions must meet the caller's threshold.** If either falls below, the result is not surfaced.

## Threshold Behavior

- **Above threshold on both dimensions:** Surface the result
- **Below threshold on either dimension:** Silence -- do not show
- **At threshold:** Include but rank lower

**Default threshold:** 0.5 for user queries, higher (0.6-0.7) for proactive ritual surfacing.

Higher thresholds for proactive contexts prevent ritual noise -- the daily plan should only surface things worth mentioning, not everything tangentially related.

## Score Calculation

```
relevance = sum(weight_i * signal_i) / sum(weights)
```

Weights come from the per-caller profile. The formula is a simple weighted average normalized to 0-1. All weights in a profile should sum to 1.0.

## Calibration Guidance

### When to Adjust Weights

- If rituals surface too much irrelevant content: increase authority weight
- If rituals miss recent developments: increase recency weight
- If the same entities keep appearing: increase access recency weight
- If suggestions do not connect to current work: increase goal alignment weight

### When to Adjust Thresholds

- If rituals are too quiet: lower threshold (but never below 0.3)
- If rituals are too noisy: raise threshold (0.8 max for proactive contexts)
- Different ritual types should have different thresholds based on tolerance for noise

### Iterative Tuning

The confidence model is not set-and-forget. After running rituals for a week, review:

1. How many surfaced items were useful? (target: 70%+ for daily plans, 80%+ for proactive surfacing)
2. Were there items you wished had been surfaced but were not? (indicates threshold too high or missing signal)
3. Were there items surfaced that were clearly irrelevant? (indicates threshold too low or wrong weight emphasis)

Adjust weights and thresholds based on observed signal quality, not theoretical preferences.
