# SATS Automation — Impact Analysis
**Measured Results | Method | Limitations**

A companion to the [main case study](README.md) and the [solution design](solution-design.md). The README covers *what* was built; the solution design covers *why it was built that way*; this document covers **whether it worked, and how confident we can be in that**.

**Auto-approval live:** May 19, 2026 · **Analysis period:** Jan 1 – Aug 15, 2026

---

## Table of Contents
- [Three Phases, Not Two](#-three-phases-not-two)
- [Adoption & Coverage](#-adoption--coverage)
- [Fulfillment Speed](#-fulfillment-speed)
- [Chargeback Reduction](#-chargeback-reduction)
- [What These Numbers Don't Prove](#-what-these-numbers-dont-prove)

---

## 🗓️ Three Phases, Not Two

Measuring this project correctly depends on one distinction that is easy to miss in the order data:

| Phase | Period | What was happening |
|-------|--------|--------------------|
| **Pre-tag** | through mid-March 2026 | No SATS logic. Every order manually approved. |
| **Observe-only** | mid-March – May 18, 2026 | Shopify Flow tagging orders `SATS-auto-approve`, **but every order still waited on a CSR**. |
| **Live** | May 19, 2026 onward | Tagged orders auto-approve and flow straight to fulfillment. |

**The tag is not the intervention.** Orders carried `SATS-auto-approve` for roughly two months before the tag did anything — that was the [observe-only rollout stage](solution-design.md#4-observe-only-before-activation), deliberately designed to measure the qualification rate before anything was at stake.

The consequence for analysis: **every before/after comparison below splits on May 19**, not on when the tag first appears. Splitting on tag appearance would place ~64 days of fully-manual operation inside the "after" bucket, comparing the pre-automation process largely against itself. All figures on this page use the corrected boundary.

---

## 📊 Adoption & Coverage

**62.2% of orders auto-approved** — 3,203 of 5,147 orders from May 19 through Aug 15, 2026.

The remaining ~38% route to the same CSR review path that previously handled 100% of orders.

Coverage has held steady across the observe-only period, the May 19 activation, and seven subsequent criteria changes. That stability matters more than the headline number: an allowlist that decays — as fraud patterns shift, product mix changes, and exclusions accumulate — would show a sagging qualification rate. It hasn't. The [iteration loop](README.md#-iterations) is keeping the criteria current rather than slowly strangling coverage.

> **Post-window change (Aug 25, 2026):** replacing the blanket billing/shipping match with a $400 threshold — implemented *after* this analysis period — lifted auto-approval to **~94%**. That change is not reflected in the figures on this page, which end Aug 15; a like-for-like re-measurement is pending.

---

## ⚡ Fulfillment Speed

Two independent comparisons, which agree:

**1. Before vs. after activation** — all orders:

| Period | Median hours to fulfillment |
|--------|------------------------------|
| Pre-SATS (Jan 1 – May 18) | 65 hrs |
| Post-SATS (May 19 – Aug 15) | **48 hrs** |

**→ 17 hours faster, a 26% improvement**, across every order regardless of whether it auto-approved.

**2. Auto-approved vs. manual review, same period** (Jun – Aug), which isolates the automation:

| Order type | Median hours to fulfillment |
|------------|------------------------------|
| SATS auto-approved | **43 hrs** |
| Manual CSR review | 61 hrs |

**→ 30% faster for auto-approved orders specifically.**

**The two results corroborate each other, and the gap between them is the expected one.** The blended figure (26%) sits below the isolated figure (30%) because roughly 38% of post-period orders still route through manual review and carry the old timeline. If the blended number had *exceeded* the isolated one, something would be wrong with the measurement.

The second comparison also controls for what the first cannot: both groups sit in the same months under the same staffing, seasonality, warehouse conditions, and carrier performance. Since a Jan–May vs. May–Aug comparison spans SCARPA's spring ramp, the within-period cut is what rules out "summer is just faster" as the explanation.

**Order volume was essentially flat between the windows** — 56.8 orders/day pre (7,834 / 138 days) vs. 57.8 post (5,147 / 89 days), a ~2% difference. The speed gain came at comparable load, neither helped nor hindered by a volume shift.

---

## 🛡️ Chargeback Reduction

| Period | Orders | Chargebacks | Rate |
|--------|--------|-------------|------|
| Pre-SATS (Jan 1 – May 18) | 7,834 | 17 | 0.217% |
| Post-SATS (May 19 – Aug 15) | 5,147 | 5 | **0.097%** |

**→ 55% reduction in chargeback rate.** Applying the pre-SATS rate to post-SATS volume predicts ~11 chargebacks against 5 actual — roughly **6 prevented**.

> **⚠️ Preliminary. This figure will move.** Chargebacks surface 30–90+ days after the order date. The pre-SATS window is fully dispute-matured; the post-SATS window ends today, so July and August orders have barely entered the dispute period and an unknown share of post-SATS chargebacks has not yet been filed. **The true post-SATS rate is higher than 0.097%, and the 55% reduction will shrink** — by how much is not yet knowable.
>
> A like-for-like read requires holding dispute maturity constant: count only chargebacks filed within 30 days of order date, and cap the post window at July 15 so every order in both periods has had identical exposure. That cut is runnable now. A fully-matured comparison needs roughly November 2026.

**If the direction survives maturation, it is the more interesting result of the two.** The stated goal was to remove a bottleneck *without losing the safety the manual check provided* — a defensible outcome was "chargebacks hold flat." A decline would mean the allowlist outperforms the check it replaced. Two plausible mechanisms:

- **The allowlist encodes signals the manual check never applied consistently.** The Delaware exclusion, billing-country gating, and the billing/shipping address match are explicit, uniform rules. A CSR eyeballing hundreds of orders applies them unevenly by definition.
- **Concentrated review attention.** CSRs now review ~38% of orders instead of 100%, and that subset is pre-filtered to the genuinely unusual — more scrutiny per suspicious order, less time rubber-stamping obviously clean ones.

Both are consistent with the data; neither is proven by it.

---

## ⚠️ What These Numbers Don't Prove

Stated plainly, because a results page that only argues one direction isn't worth much:

- **Chargeback counts are small.** 17 vs. 5 events. The rate difference is large and the direction consistent, but at these counts the confidence interval is wide — a handful of events either way moves the number materially. Combined with the maturity problem above, treat 55% as a provisional signal, not a measurement.
- **No control group.** Orders weren't randomized into auto-approve vs. manual — they're separated by the allowlist criteria themselves. Auto-approved orders are, by construction, the *simpler* orders: no PFAS restrictions, domestic billing, in-stock standard sizes. Some of the 43-vs-61-hour gap reflects that simplicity rather than the automation. The gap is real; its full size is not entirely attributable to SATS.
- **The same selection effect applies to chargebacks**, and cuts the opposite way. Auto-approved orders are pre-filtered toward low-risk profiles, so a lower chargeback rate among them is partly definitional. The meaningful question is whether the *overall* rate fell — which is what the table above measures, and why the maturity caveat matters so much.
- **Median hides the tail.** Order-to-fulfillment medians say nothing about worst-case orders, which is where customer complaints originate. A p90 cut would be more informative about experience, and hasn't been run.
- **CSR labor savings are unquantified.** The freed capacity is real and observed, but no time-tracking data supports a specific number, so none is claimed.
- **Warehouse intake smoothing is qualitative.** The warehouse manager confirms intake is materially steadier — one of the three original goals — but no distribution-of-arrivals metric was captured to demonstrate it.

---

## 🔑 Bottom Line

SATS is delivering across the dimensions it was designed for:

- **Speed** — 26% faster fulfillment across all orders, 30% on auto-approved orders specifically. Two independent comparisons agree, and the within-period cut rules out seasonality.
- **Scale** — 62.2% of orders auto-approve, stable across five months and seven criteria revisions, at comparable daily volume. (A post-window Aug 25 change later raised this to ~94%; not yet re-measured.)
- **Risk** — chargeback rate down 55%, **provisionally**. The post-period cohort has not matured and this figure will decline.

The speed result is established. The risk result is directionally encouraging and not yet settled, and is reported that way deliberately: the project's credibility rests on the same discipline that produced the [observe-only rollout](solution-design.md#4-observe-only-before-activation) — measuring before claiming.
