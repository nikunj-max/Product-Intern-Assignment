# Subspace.money — Product Teardown

> **Product Intern Assignment ·**
> Submitted by **Nikunj Bhardwaj** ·

---

## Overview

A comprehensive product teardown of [Subspace.money](https://subspace.money) — India's bootstrapped subscription marketplace (₹36.5 Cr ARR, FY25). The analysis covers five product pillars with specific, observed problems, tradeoff thinking, and actionable solutions — including production-ready ML code and system architecture diagrams.

The teardown was conducted after real app usage (web + Android), cross-referenced with 2026 Play Store reviews, and grounded in a Porter's Five Forces strategic framework.

---

## Contents

```
├── subspace_teardown.html   # Full interactive teardown report (open in browser)
└── README.md
```

The HTML file is self-contained — all charts, code blocks, and architecture diagrams render directly in the browser with no build step or dependencies.

---

## The 5 Feedbacks

| # | Pillar | Feedback | Priority |
|---|--------|----------|----------|
| 01 | GTM & ICP | Local-first homepage — the "India's first local subscription marketplace" claim is invisible in the actual UI | P0 |
| 02 | Competitor Analysis | The Negotiate API (Subspace's strongest moat) is completely hidden from users; the product looks like a coupon aggregator | P0 |
| 03 | Features / Services | Gift card fulfillment is a black box with no SLA, no circuit-breaker, and no auto-refund — root cause of all 2026 1-star reviews | P0 |
| 04 | UX | Chat tab is a ghost town for new users; group subscription payment happens before any admin trust signal is established | P1 |
| 05 | Potential Collaborations | BNPL integration (Simpl/Slice) for annual subscription bundles — zero credit risk for Subspace, ~₹7.5L/month incremental revenue | P1 |

---

## Technical Implementations

### 1. Negotiate API — ML Price Optimizer (`negotiate_api.py`)

Embedded in the teardown report. Uses `scipy.optimize.minimize_scalar` to find the clearing price `P*` that maximizes expected revenue:

```
E[Revenue] = P × P(provider accepts P) × group_size
```

- Provider acceptance modeled as a logistic function of discount depth and committed volume
- Platform-specific floor prices (Netflix: 60%, JioHotstar: 50%, Spotify: 65%)
- Designed for production: accepts a `DemandSignal` dataclass, returns clearing price + acceptance probability

**Stack:** Python · XGBoost · scipy · dataclasses

---

### 2. Churn Prediction Model (`churn_predictor.py`)

XGBoost classifier trained on behavioral features derived from the Play Store review analysis:

**Features:**
- `gift_card_failure_flag` — highest-signal churn predictor (immediate trust collapse)
- `support_ticket_unresolved_48h` — binary flag
- `admin_removal_count_90d` — bilateral trust signal
- `days_since_last_active`, `active_group_count`, `avg_renewal_delay_days`, and 4 more

**Output:** Churn probability → tiered intervention dispatch

| Churn Prob | Intervention |
|------------|-------------|
| > 0.80 | Human call within 2h |
| 0.60 – 0.80 | Push notification + discount offer |
| 0.30 – 0.60 | In-app wallet nudge (₹20 cashback) |
| < 0.30 | No action |

**Stack:** Python · XGBoost · scikit-learn · pandas

---

### 3. System Architecture Diagrams (Mermaid.js)

Two architecture diagrams rendered inside the report:

**A. AI Agent Fulfillment Circuit-Breaker**
Maps the proposed gift card / subscription fulfillment failure handler with T+0 → T+30min SLA gates and an automatic refund trigger. This is the missing piece causing every 2026 Play Store complaint.

**B. Churn Prediction Pipeline**
End-to-end data flow: Kafka event stream → Flink feature computation → Redis feature store → XGBoost inference API → intervention dispatch (FCM / Freshdesk / in-app).

---

### 4. Data Visualizations (Chart.js)

Two simulated datasets rendered interactively in the report:

**A. Cohort Retention Curves**
Three cohorts: healthy users, OTT-only users, and trust-failure users (gift card / fulfillment incident). Demonstrates the retention collapse hypothesis — Day-30 retention drops from ~52% to ~18% after a single fulfillment incident.

**B. CAC vs LTV by Subscription Tier**
Compares five user segments (OTT group sharing, gift cards, annual OTT bundles with BNPL, local MSME subs, power users). Local subscription users have the lowest CAC (word-of-mouth) and highest LTV (bilateral lock-in) — the core GTM argument.

---

## Strategic Framework

**Porter's Five Forces applied to Subspace:**

- **Threat of new entrants** — Medium. The Negotiate API and AI-native ops are hard to replicate, but the UI layer is not.
- **Buyer bargaining power** — High. Leaving a group is one tap; switching cost is near zero for OTT users.
- **Threat of substitutes** — High. Informal credential sharing is the real substitute, not PhonePe.
- **Supplier power** — High. Netflix / JioHotstar T&C changes can destroy group-sharing GMV overnight (cf. Netflix 2023).
- **Competitive rivalry** — Medium. No direct competitor, but PhonePe can ship a subscription tab in one sprint.

**Strategic verdict:** Subspace's risk is not competition — it is operational trust collapse. Three P0 fixes (GTM, Negotiate API visibility, Fulfillment SLA) all share a single root cause: the gap between Subspace's technical excellence and the user-perceived reliability of that excellence.

---

## How to View

Open `subspace_teardown.html` directly in any modern browser. No server, no npm, no build step required.

```bash
# Clone and open
git clone https://github.com/<your-username>/subspace-teardown
open subspace_teardown.html   # macOS
# or
start subspace_teardown.html  # Windows
```

---

## Author

- **Nikunj Bhardwaj**
- **Stack:** XGBoost · LSTMs · PyTorch · C++ · Python
