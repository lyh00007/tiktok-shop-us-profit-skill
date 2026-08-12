---
name: tiktok-shop-us-profit
description: Analyze TikTok Shop US settlement, ERP, warehouse, advertising, procurement, and after-sales exports into an auditable shop P&L, SKU profitability report, sample-cost report, and Order ID reconciliation. Use when users upload relevant reports or ask about TikTok Shop US profit, margins, SKU profitability, warehouse costs, samples, or cross-system order differences.
---

# TikTok Shop US Profit Analysis

Use this skill to calculate **auditable operating profit**, not only GMV. Respond in Chinese unless the user asks otherwise.

## Workflow

1. Identify every uploaded file as TikTok settlement, ERP, warehouse, advertising, procurement, after-sales, or unknown. Do not silently use an unknown file.
2. Record the analysis period, settlement-date basis, currency, exchange rate, and missing inputs before calculating.
3. Normalize data: trim SKU names, preserve original SKU names, normalize dates/currencies/numbers, and use TikTok Order ID as the primary reconciliation key.
4. Separate regular sales from samples. Treat `Order Income = 0` only as a sample candidate; confirm it with Order ID and ERP records where possible.
5. Reconcile at order level first, then inspect SKU lines. Never infer an order match from date + SKU.
6. Calculate profit by SKU line, then aggregate to order, SKU, and shop level. Keep known costs, estimates, and omitted costs visibly separate.
7. Run data-quality checks and present the required report tables. Explain the highest-impact actions, not only the totals.

## Calculation Guardrails

- Use `Order settled date` for the primary monthly P&L; payment-date totals may be shown only as a clearly labelled comparison.
- Count orders by distinct Order ID and sales units by actual TikTok SKU Quantity.
- Seller shipping-fee income is revenue, not automatically a cost. Net it against TikTok Shop shipping charges and show adjustments separately.
- Do not treat settlement-table `Ad commission` or `Smart Marketing Fee` as the advertiser-account spend. Deduct the separately supplied actual ad spend once, and disclose whether platform marketing fees are already in the P&L.
- Never convert an absent input into zero. Label it as `未纳入` (not included); label inferred values as `估算`.
- Allocate shared costs using a stated basis (prefer SKU revenue or units, depending on the cost). Show the allocation basis.
- Exclude warehouse channels `USPS-AW-YZG01(USPS-AW-YZG01)` and `USPS(GA)` as Amazon. Mark unconfirmed channels `待确认` rather than including them.
- A stopped SKU (currently `Smart Wallet` / `2-Pack noshipping`) remains historical only and must be excluded from current operating decisions.

## Profit Bridge

Calculate and display this bridge in USD and RMB:

`Settlement revenue − platform fees − affiliate fees − net TikTok logistics − actual ad spend − SKU procurement − first-mile cost − TikTok warehouse outbound/storage/other fees − withdrawal fee = known operating profit`

Use the rules in [Business rules](TikTok_Shop_US_Full_Profit_Analysis_Skill(2).md) for defaults and confirmed product costs. Newer user-provided data overrides those defaults.

## Required Deliverables

Use [the report template](templates/report-template.md). At minimum provide:

- Shop summary: settlement revenue, GMV where available, orders, units, AOV, all major costs, known operating profit, and net margin.
- P&L bridge: each cost in USD, RMB, and as a percentage of settlement revenue.
- SKU ranking: units, revenue, allocated costs, profit, margin, unit profit, and a stopped-SKU flag.
- Warehouse summary: TikTok-only costs by warehouse and fee type; excluded and unconfirmed channel totals.
- Order ID reconciliation: unique orders on each side, matches, ERP-only, TikTok-only, and within-order line differences.
- Sample report: matched/unmatched sample orders and total/average sample cost, kept out of normal sales profit.
- Data-quality exceptions, assumptions, and 3–5 concrete operating recommendations.

## Before Finalizing

Validate that the accounting identity reconciles within rounding tolerance, no Amazon warehouse channel was included, ad spend was not double counted, and every missing/estimated item is visible. If data cannot support a conclusion, say what file or field is needed rather than guessing.
