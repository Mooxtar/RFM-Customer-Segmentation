```markdown
# RFM Customer Segmentation (Hackathon) — 2nd Place Solution 🥈

This project contains our hackathon solution for **RFM analysis and customer segmentation**, where we achieved **2nd place**. The objective was to create an interpretable segmentation of clients using Recency, Frequency, and Monetary value, and then explain the segments in a way that is directly useful for business decisions.

---

## Objective

Build a practical segmentation of customers using **RFM** so the business can:
- identify **top value** clients,
- detect **high-value clients at churn risk**,
- prioritize retention and marketing efforts,
- and avoid spending resources on segments with low return.

---

## Why RFM matters (business logic)

RFM works well because it compresses customer behavior into three signals that map directly to business strategy:

- **Recency (R):** how recently a client was active  
  This helps detect who is currently engaged vs who is going cold (churn risk).

- **Frequency (F):** how often the client is active  
  This separates loyal, habitual users from rare/occasional users.

- **Monetary (M):** how valuable the client is  
  This shows who drives revenue and where retention efforts have the biggest financial impact.

Instead of treating everyone the same, RFM segmentation creates **actionable groups** that can be managed with different strategies.

---

## End-to-end approach (how we built it)

### 1) Build client-level features

We aggregate raw activity into one row per client, using fields such as:
- last activity date (for recency)
- total activity counts (for frequency)
- number of trades (activity intensity)
- total commission (direct business KPI)
- inflows/deposits and related value signals

Demographic and acquisition fields were used to **profile/explain** clusters (for interpretation), not to build them.

---

### 2) Score engineering for R, F, M

#### Recency score (exponential decay)
We compute recency in days and convert it into a smooth score using exponential decay with **half-life = 30 days**:
- very recent clients get scores close to 1
- clients inactive for longer periods get scores that drop quickly

This is more stable than raw “days since last activity” because recency distributions are typically skewed.

#### Frequency score (log + normalization)
Frequency can have extreme outliers, so we apply:
- `log1p` to compress the long tail
- min-max normalization to scale values to `[0, 1]`

#### Monetary score (value design + log + normalization)
We construct a monetary indicator that reflects:
- platform revenue (**total commission**)
- customer size/potential (**inflows/deposits**)
- activity intensity (more trades increases weight slightly)

Then we apply:
- `log1p` for stability
- min-max normalization to `[0, 1]`

This monetary definition matches real intuition: a valuable client is not only someone who deposits, but someone who also actively trades and produces commission.

---

### 3) RFM score (for ranking / interpretability)

In addition to clustering in the 3D space (R, F, M), we also compute a single **RFM score** for ranking and reporting.

We used **data-driven weights** by measuring correlations between each score (R/F/M) and the business KPI (**total commission**), then normalizing them into weights that sum to 1. This makes the score directly connected to revenue.

---

### 4) Clustering for segmentation

We apply **KMeans** clustering on:
- `recency_score`
- `frequency_score`
- `monetary_score`

The number of clusters is chosen as **k = 5** based on an elbow/WCSS trade-off:
- enough segments to be actionable
- not too many segments to confuse stakeholders

---

## Final segments (business labels)

KMeans produces numeric ids (0–4), but numeric ids are not meaningful to business users, so we mapped clusters into human-readable segments:

- **Top Value Customers**
- **Potentially Lost High-Value**
- **Active Mid-Tier Clients**
- **Emerging Growth Segment**
- **Low Activity, Low Value**

---

## Results (what we proved)

### 1) Segment sizes (client distribution)

Final client distribution by segment:

- **Low Activity, Low Value:** **45.9%**
- **Potentially Lost High-Value:** **23.5%**
- **Active Mid-Tier Clients:** **12.3%**
- **Top Value Customers:** **10.5%**
- **Emerging Growth Segment:** **7.9%**

Client counts (from the clustering output):
- Cluster 0: **56,423**
- Cluster 1: **110,263**
- Cluster 2: **25,174**
- Cluster 3: **29,459**
- Cluster 4: **18,903**

Interpretation: the base is dominated by low-activity users, while “core” segments are smaller but more important.

---

### 2) Revenue concentration (commission distribution)

Total commission distribution by segment:

- **Top Value Customers:** **53.3%** of total commissions
- **Potentially Lost High-Value:** **26.2%**
- **Active Mid-Tier Clients:** **13.6%**
- **Emerging Growth Segment:** **6.8%**
- **Low Activity, Low Value:** **~0.0%**

This is the key business insight:
- a small segment generates most of the revenue,
- and a large segment contributes almost nothing in commission.

---

## Action plan (how the business can use the segments)

**Top Value Customers**
- VIP retention and premium support
- loyalty benefits, proactive outreach
- protect from churn aggressively (highest commission concentration)

**Potentially Lost High-Value**
- win-back campaigns and personal outreach
- investigate friction (platform issues, product fit, onboarding gaps)
- best ROI segment for retention spending

**Active Mid-Tier Clients**
- upsell and engagement nudges to move them toward Top Value
- education and tools that increase frequency and monetization

**Emerging Growth Segment**
- nurture with onboarding, habit-building, targeted prompts
- offers that increase activity without expensive incentives

**Low Activity, Low Value**
- only low-cost automation (email/push)
- avoid expensive retention resources here
- useful for baseline churn and lookalike analysis, but not a priority for manual work

---

## Why this solution achieved 2nd place

Our approach worked well because it was:
- **Business-aligned**: feature design connected to commission (real KPI)
- **Robust**: decay + log scaling handled skew and outliers reliably
- **Interpretable**: clusters were converted into clear, meaningful segment names
- **Actionable**: each segment comes with a concrete strategy for retention and growth
```
