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

## Per-order profitability model (required)

Build the calculation at **TikTok Order ID + normalized SKU line** level, then roll it up to one Order ID. The order-level output is the source of truth for the monthly P&L and must retain both the source rows and allocation details.

### Matching rules

1. Normalize Order ID as text only: trim whitespace, remove a spreadsheet-generated trailing `.0`, and never coerce it to a numeric value.
2. Aggregate TikTok settlement rows and Lingxing ERP rows separately by `Order ID + normalized SKU`. Keep source-line counts and the original SKU text for auditability.
3. Match only when the normalized TikTok Order ID exactly equals the Lingxing platform Order ID. Do not use dates, customer names, SKUs, tracking numbers, or amounts as a substitute match key.
4. A matched Order ID can legitimately have multiple SKU lines. Compare the set of SKUs and quantity per SKU after the order-level match; do not treat a multi-SKU order as duplicate data.
5. ERP outbound cost is assigned from the matched ERP order. If ERP contains multiple cost rows for an order, sum those rows but keep the component rows in the audit sheet. Never replace a missing ERP cost with a product-cost estimate.

### Required exception flags

Create one exception row per affected Order ID, with source values and a clear status:

- `TK-only`: TK settlement Order ID has no ERP Order ID.
- `ERP-only`: ERP Order ID has no TK settlement Order ID.
- `duplicate-TK`: the same Order ID + normalized SKU occurs more than once after excluding legitimate settlement adjustment lines; retain all lines and require review.
- `duplicate-ERP`: the same ERP order/SKU is duplicated beyond its documented split-shipment or fee detail.
- `multi-SKU`: either source has more than one distinct SKU within the same Order ID (information flag, not an error by itself).
- `SKU mismatch`: matched order, but normalized SKU sets differ.
- `quantity mismatch`: matched order, but total quantity differs for one or more normalized SKUs.
- `zero/blank ERP cost`: matched regular-sale order has no usable ERP outbound cost.
- `stopped SKU`: Smart Wallet / `2-Pack noshipping`; preserve as historical and exclude from current operating recommendations.
- `sample candidate`: TikTok order income equals zero. Confirm against ERP site = TikTok US and item amount = zero when fields are available.

### Cost attribution and allocation

All per-order values must show their currency and allocation basis.

- **TikTok revenue and direct fees:** aggregate the settlement rows belonging to the order. Seller-paid shipping is revenue when positive; calculate net TikTok logistics separately from TikTok shipping charges and shipping adjustments.
- **Affiliate/creator fees:** take the order-level value from the settlement report where supplied; otherwise mark as not included.
- **ERP outbound cost:** direct order cost, preferably as a native currency amount. Convert using the documented exchange rate only when it is not already RMB.
- **Procurement:** calculate from normalized SKU × TikTok actual quantity. Current confirmed RMB unit costs: Bubble Pipe 1PC 5; Bubble Pipe 2PC 10; Saturn 35; Bubble Party Bundle 10.5; bubble product one box 5.5; bubble product two boxes 11. Flag an unmapped SKU rather than pricing it at zero.
- **First mile:** RMB 3 × TikTok actual sales units. Do not use ERP or warehouse outbound quantity. For a multi-SKU order, calculate by SKU quantity then sum.
- **Actual ad spend:** when only a monthly total is provided, default to allocate it across regular, non-stopped TK orders in proportion to positive settlement revenue. Show `广告分摊口径 = 收入占比（估算）`. If all eligible revenue is zero, allocate by actual sales units and label it `件数占比（估算）`. Never allocate a monthly total both by revenue and by order count. When campaign/SKU/order attribution exists, use direct attribution first and allocate only the residual.
- **Warehouse shared fees:** first assign order-identifiable warehouse fees directly. Allocate remaining confirmed TikTok-only warehouse costs over regular, non-stopped orders by actual sales units; if units are unavailable, use positive settlement revenue. Exclude `USPS-AW-YZG01(USPS-AW-YZG01)` and `USPS(GA)` as Amazon. Keep unconfirmed channels outside profit and report them separately.
- **Withdrawal fee:** estimate at 0.2% of each order's settlement revenue (or actual withdrawal basis if supplied). Mark the estimated version clearly.

### Order-profit formula

Use RMB for the final order comparison, converting USD items at the declared exchange rate (default 6.75):

`订单利润 = 结算收入 + 买家支付运费 - 平台费用 - 达人/联盟佣金 - TikTok净物流 - 直接ERP出库成本 - 采购成本 - 头程 - 广告分摊 - 仓库分摊 - 提现手续费`

Do not subtract the same TikTok fee twice when the settlement export provides a net-order-margin field. Use that field as a reconciliation check, not as an additional expense unless its definition proves otherwise.

### Required order-level report

Produce a filterable workbook sheet named `逐订单利润` with at least:

`Order ID | 结算日期 | 订单状态 | 原始 SKU | 标准 SKU | TK 件数 | ERP 件数 | 收入 USD | 平台费用 USD | 达人佣金 USD | TK 净物流 USD | 广告分摊 USD | 仓库分摊 RMB | ERP 出库成本 RMB | 采购成本 RMB | 头程 RMB | 提现手续费 USD | 订单利润 RMB | 利润率 | 匹配状态 | 异常标记 | 广告/仓库分摊口径`

Add `订单匹配异常` and `分摊明细` sheets. The exception sheet must include the source-side SKU/quantity/cost fields needed to resolve the discrepancy. The allocation sheet must reconcile each allocated shared-cost total back to the monthly input total within rounding tolerance.

## Confirmed export schemas (Lingxing + TikTok)

Treat the following user-confirmed export formats as the default import schemas. New exports with the same headings should be recognized automatically; retain unknown/new columns in the raw-input sheet and flag a required mapping review rather than discarding them.

### Lingxing ERP `订单管理` export

The header is row 1. Use these fields:

| Purpose | ERP field |
|---|---|
| Exact matching key | `平台单号` |
| Source identifier | `系统单号` |
| Sales channel filter | `平台` = `TikTok`; `站点` = `[TikTok].美国` |
| SKU / product | `SKU`, `品名`, `ASIN/商品Id`, `订单商品ID` |
| Quantity check | one ERP row is one exported item line; aggregate by `平台单号 + SKU` and use a dedicated quantity field if later exports include one |
| Direct outbound cost (preferred) | `商品出库成本` |
| Direct order outbound cost (fallback/check) | `订单出库成本(CNY)` |
| Order amount / currency | `订单商品金额`, `客付运费`, `订单总金额`, `订单币种` |
| Operations audit | `状态`, `订购时间`, `发货时间`, `发货仓库`, `物流方式`, `运单号`, `跟踪号`, `TikTok仓库` |

Do not add `商品出库成本` and `订单出库成本(CNY)` together by default. Reconcile them at Order ID level: use the item-line cost where complete; otherwise use the order cost once as the direct ERP cost. Flag a material difference for review.

### TikTok `merchant_statement_profit_loss` export

Use the `Orders` sheet, where the real headers are on row 6. The first five rows are metadata/instructions and must not be imported as transactions. Use `Order payment info` for order pricing/discount support and `Adjustment` only as a separate settlement-level reconciliation source.

| Purpose | TikTok field |
|---|---|
| Exact matching key | `Order ID` |
| SKU / product | `SKU ID`, `SKU name`, `Product name` |
| Order economics | `Order Income`, `Order Cost`, `Net Order Margin`, `Sold Quantity` |
| Dates / status | `Order paid date`, `Order shipment date`, `Order delivery date`, `Order settled date`, `Order status`, `Order source` |
| Direct platform/creator fees | `联盟佣金`, `联盟服务商佣金`, `推荐费`, `交易手续费`, `智能营销费`, `联盟店铺广告佣金`, `合资营销（商家出资）` and other non-zero fee-detail columns |
| Shipping and logistics | `买家支付运费`, `客户运费退款`, `实际退货运费`, `TikTok Shop 运费`, `客户运费抵扣`, `FBT 配送费`, shipping subsidies/discounts/compensation columns |
| Sample marker | `Order Income = 0`, then compare `Sample order type` and ERP record |

For `Order payment info`, headers begin on row 1 and should be joined by `Order ID + SKU ID` only after the order-level ERP match. It supplies pre-/post-discount SKU subtotal, shipping fee, taxes, and order amount. Do not use it to create a new order match.

`Adjustment` has no order-level matching key in this export. Reconcile its `linked statement id` / `linked payout id` to settlement totals; do not distribute its amounts to individual orders unless a later source explicitly establishes the link.

## Required Deliverables

Use [the report template](templates/report-template.md). At minimum provide:

- Shop summary: settlement revenue, GMV where available, orders, units, AOV, all major costs, known operating profit, and net margin.
- P&L bridge: each cost in USD, RMB, and as a percentage of settlement revenue.
- SKU ranking: units, revenue, allocated costs, profit, margin, unit profit, and a stopped-SKU flag.
- Warehouse summary: TikTok-only costs by warehouse and fee type; excluded and unconfirmed channel totals.
- Order ID reconciliation: unique orders on each side, matches, ERP-only, TikTok-only, and within-order line differences.
- Per-order profitability: one filterable row per Order ID, direct costs, allocated shared costs, order profit/margin, matching status, and exception flags.
- Sample report: matched/unmatched sample orders and total/average sample cost, kept out of normal sales profit.
- Data-quality exceptions, assumptions, and 3–5 concrete operating recommendations.

## Before Finalizing

Validate that the accounting identity reconciles within rounding tolerance, no Amazon warehouse channel was included, ad spend was not double counted, and every missing/estimated item is visible. If data cannot support a conclusion, say what file or field is needed rather than guessing.
