# SATS Automation — Solution Design
**Design Rationale | Alternatives Considered | Risk Trade-offs**

A companion to the [main case study](README.md). The README covers *what* was built and why it mattered; this document covers the *decisions behind it* — the options weighed, the trade-offs accepted, and the alternatives deliberately not taken.

---

## Table of Contents
- [Design Constraints](#-design-constraints)
- [Key Decisions](#-key-decisions)
- [Alternatives Considered](#-alternatives-considered)

---

## 🎯 Design Constraints

The solution had to satisfy five constraints, in roughly this priority:

- **Preserve the safety the manual check provided** — remove the bottleneck, not the control it stood in for.
- **Stay tunable** — the qualifying criteria would change over time (and have: PFAS, dev orders, a hat edge case).
- **Be reversible** — any change touching production fulfillment needed a clean path back.
- **Minimize custom code and technical debt** — favor native, no-code tooling where it could carry the load.
- **Reuse what already works** — don't rebuild operations that were already trusted and functioning.

---

## 🧠 Key Decisions

### 1. Remove the gate — but audit it before touching anything

**Options:** Keep the gate (status quo) · remove it outright · confirm it was removable, then remove it.

**Choice:** Audit first. The gate traced back to early Magento days when inaccurate inventory made auto-allocation unsafe. By 2025 the inventory problem was gone, but the step remained. Before designing anything, I confirmed it was truly obsolete: the **warehouse manager** on downstream impact, then the **CFO** and **Direct Sales Manager** to trace the gate's history and verify it was no longer a real control.

**Why:** Removing a control you haven't traced is how you discover — in production — that it was load-bearing. The audit cost a few conversations and removed that risk.

### 2. Allowlist, not blocklist

**Options:** Auto-approve everything and hunt for problem orders after the fact (blocklist) · auto-approve only clearly-safe orders and route the rest to humans (allowlist).

**Choice:** Allowlist.

**Why:** A blocklist fails *open* — a problem order ships before anyone notices, and you're chasing exceptions after they've already left the building. An allowlist fails *safe*: anything that doesn't clearly qualify falls back to the existing human path. Risk is contained to a small, reviewable set instead of riding on catching every exception in time.

### 3. Put the decision logic in Shopify Flow

**Options:** Shopify Flow (no-code rules) · a NetSuite SuiteScript rules engine · business logic inside Celigo.

**Choice:** Shopify Flow evaluates orders and tags them; Celigo's role stays minimal (detect the tag, carry it through); NetSuite only had to accept pre-approved orders.

**Why:** The criteria were always going to need tuning, so the logic belongs where tuning is cheapest. Shopify Flow is no-code and editable in minutes. A SuiteScript rules engine would have been dev-dependent, slower to change, and a new source of technical debt. Celigo is an integration layer, not a business-rules engine — pushing decision logic there would have overloaded the wrong tool.

### 4. Observe-only before activation

**Options:** Activate auto-approval directly in production · run the tagging logic in observe-only mode first.

**Choice:** Ran the Shopify Flow in tag-only mode for several weeks before any order auto-shipped — measuring the qualification rate and spot-checking for orders that *should* have tagged but didn't.

**Why:** Direct cutover offers no way to know what share of orders will qualify, or whether the criteria are mis-tagging, until real orders are already moving. Observe-only turned those unknowns into measured facts before anything was at stake.

### 5. Reuse the manual path for edge cases

**Options:** Build new automated exception handling for non-qualifying orders · route them to the existing CSR manual path.

**Choice:** Edge cases (the ~39% that don't auto-approve) flow to the same manual review CSRs already ran.

**Why:** That path was already trusted and working. Building new exception automation would have added scope and risk for the minority of orders — to replace a process that wasn't broken.

### 6. Address match: risk-proportionate, not blanket

**Options:** Require billing/shipping match on every order · drop the check entirely · require it only above a value threshold.

**Choice:** Removed the blanket requirement, then reinstated it at $400 and above.

**Why:** The original check was strict normalized equality across the full address. It's a legitimate fraud signal, but it fired on benign mismatches — gift shipments, apartment fields entered inconsistently, work vs. home addresses — and blocked far more good orders than fraudulent ones. Dropping it entirely lifted the auto-approval rate but discarded a real signal on exactly the orders where it matters most. The threshold restores it where the downside justifies the friction: a mismatch on a $60 order is noise; on a $400+ order it's worth a human look. The allowlist principle holds — the check fails to the existing CSR path, not to auto-approval.

**Trade-off accepted:** Shopify Flow conditions can only compare a field to a literal, not to another field, so the comparison required a Run code action — the one place this design breaks the "minimize custom code" constraint. Ten lines of JavaScript, no network calls, and the threshold itself stays in the Flow UI where ops can tune it without touching the script.

---

## 📋 Alternatives Considered

| Considered | Why not |
|------------|---------|
| Auto-approve everything, hunt exceptions (blocklist) | Fails open — a problem order ships before detection. The allowlist fails safe to the existing human path. |
| NetSuite SuiteScript rules engine | Dev-dependent and slower to tune; adds technical debt. Shopify Flow keeps rule changes no-code. |
| Decision logic inside Celigo | Celigo is an integration layer, not a rules engine; overloading it strains the wrong tool. |
| Delete the gate outright | Removes the safety net entirely. The allowlist preserves human review for the unusual minority. |
| Direct production cutover | No way to measure qualification rate or catch mis-tagging before live impact. Observe-only de-risked it. |
| New automated exception handling | Adds scope and risk for the ~39% edge cases when the existing CSR path already handles them reliably. |
| Blanket billing/shipping address match | Strict equality fires on benign mismatches (gifts, apartment fields, work addresses) — blocked far more good orders than bad. Reinstated at $400+ where the fraud downside justifies the friction. |
| Drop address match entirely | Discards a real fraud signal on high-value orders for a marginal lift in auto-approval rate. |
