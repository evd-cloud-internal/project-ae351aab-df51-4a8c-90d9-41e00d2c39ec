---
name: Home
assetId: 54a4344a-9ad4-414c-8d19-f91763719467
type: page
---

# Resale Marketplace Performance

```sql data_freshness
SELECT formatDateTime(max(resale_requested_at), '%b %d, %Y') as last_resale_date
FROM resale_enriched
WHERE property_code NOT IN ('SAFE-21', 'SAFE-23')
```

📅 Data as of: **{% value data="data_freshness" value="last_resale_date" /%}**

{% filter_bar %}
    {% dropdown
        id="property_filter"
        data="resale_enriched"
        value_column="property_name"
        title="Property"
        where="property_code NOT IN ('SAFE-21', 'SAFE-23')"
        multiple=true
    /%}
    {% dropdown
        id="status_filter"
        data="resale_enriched"
        value_column="resale_status_label"
        title="Status"
        where="property_code NOT IN ('SAFE-21', 'SAFE-23')"
        multiple=true
    /%}
    {% range_calendar
        id="date_filter"
        title="Date Range"
        default_range="all time"
    /%}
{% /filter_bar %}

## Marketplace Overview

```sql sold_rate_q
SELECT
    countIf(resale_status = 'SOLD') * 1.0 / count(*) as sold_rate
FROM resale_enriched
WHERE property_code NOT IN ('SAFE-21', 'SAFE-23')
```

```sql safe_revenue_q
SELECT
    sum(commission_amount + buyer_transfer_fee) as safe_revenue
FROM resale_enriched
WHERE property_code NOT IN ('SAFE-21', 'SAFE-23')
    AND resale_status = 'SOLD'
```

```sql avg_time_to_sell_q
SELECT
    avg(dateDiff('day', resale_requested_at, buyer_order_created_at)) as avg_days_to_sell
FROM resale_enriched
WHERE property_code NOT IN ('SAFE-21', 'SAFE-23')
    AND resale_status = 'SOLD'
```

```sql avg_take_rate_q
SELECT
    sum(commission_amount + buyer_transfer_fee) / sum(total_resale_contract_price) as avg_take_rate
FROM resale_enriched
WHERE property_code NOT IN ('SAFE-21', 'SAFE-23')
    AND resale_status = 'SOLD'
```

```sql avg_gross_profit_rent_q
SELECT
    avg(seller_gross_profit_including_rent_per_share) as avg_gross_profit_per_share,
    avg(estimated_annual_gross_return) as avg_annual_gross_return
FROM resale_enriched
WHERE property_code NOT IN ('SAFE-21', 'SAFE-23')
    AND resale_status = 'SOLD'
```

{% big_value
    data="resale_enriched"
    value="count(*) as total_resale_requests"
    fmt="num0"
    title="Total Resale Requests"
    info="COUNT(*) of all resale requests from resale_enriched, excluding test properties (SAFE-21, SAFE-23). Includes all statuses."
    where="property_code NOT IN ('SAFE-21', 'SAFE-23')"
    filters=["property_filter", "status_filter"]
    date_range={
        date="resale_requested_at"
        range={{date_filter}}
    }
/%}

{% big_value
    data="resale_enriched"
    value="count(*) as total_sold"
    fmt="num0"
    title="Total Sold"
    info="COUNT(*) of resale requests with resale_status = 'SOLD', excluding test properties (SAFE-21, SAFE-23)."
    where="property_code NOT IN ('SAFE-21', 'SAFE-23') AND resale_status = 'SOLD'"
    filters=["property_filter", "status_filter"]
    date_range={
        date="resale_requested_at"
        range={{date_filter}}
    }
/%}

{% big_value
    data="sold_rate_q"
    value="sold_rate"
    fmt="pct1"
    title="Sold Rate"
    info="Sold / Total Requests. Calculated as countIf(resale_status='SOLD') / COUNT(*) from resale_enriched, excluding test properties."
/%}

{% big_value
    data="resale_enriched"
    value="sum(number_of_shares) as total_shares_traded"
    fmt="num0"
    title="Total Shares Traded"
    info="SUM(number_of_shares) WHERE resale_status = 'SOLD', excluding test properties."
    where="property_code NOT IN ('SAFE-21', 'SAFE-23') AND resale_status = 'SOLD'"
    filters=["property_filter", "status_filter"]
    date_range={
        date="resale_requested_at"
        range={{date_filter}}
    }
/%}

{% line_break /%}

{% big_value
    data="resale_enriched"
    value="sum(total_resale_contract_price) as total_resale_volume"
    fmt="#,##0' EGP'"
    title="Total Resale Volume"
    info="SUM(total_resale_contract_price) WHERE resale_status = 'SOLD', excluding test properties. Displayed in EGP."
    where="property_code NOT IN ('SAFE-21', 'SAFE-23') AND resale_status = 'SOLD'"
    filters=["property_filter", "status_filter"]
    date_range={
        date="resale_requested_at"
        range={{date_filter}}
    }
/%}

{% big_value
    data="safe_revenue_q"
    value="safe_revenue"
    fmt="#,##0' EGP'"
    title="SAFE Revenue"
    info="SUM(commission_amount + buyer_transfer_fee) WHERE resale_status = 'SOLD', excluding test properties. This is the resale take rate revenue."
/%}

{% big_value
    data="avg_take_rate_q"
    value="avg_take_rate"
    fmt="pct1"
    title="Avg Take Rate"
    info="SUM(commission_amount + buyer_transfer_fee) / SUM(total_resale_contract_price) WHERE resale_status = 'SOLD', excluding test properties. This is SAFE's average revenue as a percentage of total resale volume."
/%}

{% line_break /%}

{% big_value
    data="avg_time_to_sell_q"
    value="avg_days_to_sell"
    fmt="#,##0.0' days'"
    title="Avg Time to Sell"
    info="AVG(dateDiff('day', resale_requested_at, buyer_order_created_at)) WHERE resale_status = 'SOLD', excluding test properties. Average number of days from listing to buyer order."
/%}

{% big_value
    data="avg_gross_profit_rent_q"
    value="avg_gross_profit_per_share"
    fmt="#,##0' EGP'"
    title="Avg Gross Profit (Incl. Rent) / Share"
    info="AVG(seller_gross_profit_including_rent_per_share) WHERE resale_status = 'SOLD', excluding test properties. This is the average gross profit per share including accumulated rent distributions received during the seller's holding period."
/%}

{% big_value
    data="avg_gross_profit_rent_q"
    value="avg_annual_gross_return"
    fmt="#,##0.0'%'"
    title="Avg Annualized Gross Return"
    info="AVG(estimated_annual_gross_return) WHERE resale_status = 'SOLD', excluding test properties. This is the average annualized gross return including rent, calculated based on the seller's holding period and total gross profit (capital gains + rent collected)."
/%}

{% callout type="info" title="Key Takeaways — Marketplace Overview" %}
- **24.2% sold rate** — roughly 1 in 4 resale requests result in a completed sale, indicating a buyer's market with more supply than demand.
- **Fast turnaround** — sold shares take an average of only **7.8 days** from listing to buyer order, suggesting strong buyer intent once a match is found.
- **Healthy investor returns** — sellers earn an average **14.3% annualized gross return** (including rent), with ~4,246 EGP gross profit per share.
- **SAFE revenue of 1.73M EGP** at a **4.0% take rate** — consistent monetization from the resale marketplace.
- **42.9M EGP total resale volume** across 846 shares traded — a growing secondary market.
{% /callout %}

---

## Property Performance

```sql property_performance
SELECT
    property_name,
    count(*) as total_requests,
    countIf(resale_status = 'SOLD') as sold,
    countIf(resale_status = 'SOLD') * 1.0 / count(*) as sold_rate,
    sum(number_of_shares) as shares_listed,
    sumIf(number_of_shares, resale_status = 'SOLD') as shares_sold,
    avgIf(dateDiff('day', resale_requested_at, buyer_order_created_at), resale_status = 'SOLD') as avg_days_to_sell,
    sumIf(total_resale_contract_price, resale_status = 'SOLD') as total_volume,
    sumIf(commission_amount + buyer_transfer_fee, resale_status = 'SOLD') as safe_revenue,
    sumIf(commission_amount + buyer_transfer_fee, resale_status = 'SOLD') / nullIf(sumIf(total_resale_contract_price, resale_status = 'SOLD'), 0) as take_rate,
    avgIf(estimated_annual_gross_return, resale_status = 'SOLD') as avg_annual_gross_return
FROM resale_enriched
WHERE property_code NOT IN ('SAFE-21', 'SAFE-23')
GROUP BY property_name
ORDER BY total_requests DESC
```

{% table
    data="property_performance"
    title="Property Performance"
    info="One row per property. All metrics from resale_enriched, excluding test properties (SAFE-21, SAFE-23). Sold-specific metrics filtered to resale_status = 'SOLD'."
    subtotals=false
    search=true
    page_size=15
%}
    {% dimension value="property_name" title="Property" /%}
    {% measure value="sum(total_requests)" title="Total Requests" fmt="num0" sort="desc" info="COUNT(*) of all resale requests for this property." /%}
    {% measure value="sum(sold)" title="Sold" fmt="num0" info="COUNT(*) WHERE resale_status = 'SOLD'." /%}
    {% measure value="sum(sold) / sum(total_requests)" title="Sold Rate" fmt="pct1" info="Sold / Total Requests as a percentage." /%}
    {% measure value="sum(shares_listed)" title="Shares Listed" fmt="num0" info="SUM(number_of_shares) across all statuses." /%}
    {% measure value="sum(shares_sold)" title="Shares Sold" fmt="num0" info="SUM(number_of_shares) WHERE resale_status = 'SOLD'." /%}
    {% measure value="avg(avg_days_to_sell)" title="Avg Days to Sell" fmt="num0" info="AVG(dateDiff('day', resale_requested_at, buyer_order_created_at)) WHERE resale_status = 'SOLD'." /%}
    {% measure value="sum(total_volume)" title="Total Volume (EGP)" fmt="#,##0" info="SUM(total_resale_contract_price) WHERE resale_status = 'SOLD'. In EGP." /%}
    {% measure value="sum(safe_revenue)" title="SAFE Revenue (EGP)" fmt="#,##0" info="SUM(commission_amount + buyer_transfer_fee) WHERE resale_status = 'SOLD'. SAFE's revenue from this property." /%}
    {% measure value="sum(safe_revenue) / sum(total_volume)" title="Take Rate %" fmt="pct1" info="SAFE Revenue / Total Volume. SAFE's revenue as a percentage of total resale volume for this property." /%}
    {% measure value="avg(avg_annual_gross_return)" title="Avg Annualized Gross Return" fmt="#,##0.0'%'" info="AVG(estimated_annual_gross_return) WHERE resale_status = 'SOLD'. Annualized gross return including rent, based on seller holding period." /%}
{% /table %}

{% callout type="info" title="Key Takeaways — Property Performance" %}
- **Mivida The Place B5-207** has the highest sold rate (40.1%) and one of the best annualized returns (18.3%), making it the strongest resale property.
- **Mivida The Place B5-201** leads in volume (73 sold, 4.3M EGP) with an 18.3% annualized return — the most liquid and profitable property.
- **Park St. East** sells the fastest (avg 2.2 days) but has a low sold rate (14.7%) and the lowest annualized return (6.3%) — high demand but lower profitability.
- **Katameya Heights properties** (G5 & G6) have the most requests but the lowest sold rates (13–16%), suggesting oversupply or pricing misalignment.
- **Mivida Business Park 2** and **Hydepark Admin Office** sell quickly (4–5 days avg) with solid returns (13–14%), indicating healthy demand-supply balance.
{% /callout %}

---

## Status Breakdown

*"Others" includes: Cancelled, Rejected, Pending Approval, and Pending Transfer.*

{% bar_chart
    data="resale_enriched"
    x="case when resale_status_label in ('Listed', 'Sold', 'Pending Contracts Signing') then resale_status_label else 'Others' end as status_group"
    y="count(*) as request_count"
    title="Resale Requests by Status"
    info="COUNT(*) of resale requests grouped into 3 main statuses (Listed, Pending Contracts Signing, Sold) plus Others (Cancelled, Rejected, Pending Approval, Pending Transfer). Excludes test properties (SAFE-21, SAFE-23)."
    order="count(*) desc"
    fmt="num0"
    where="property_code NOT IN ('SAFE-21', 'SAFE-23')"
    filters=["property_filter", "status_filter"]
    date_range={
        date="resale_requested_at"
        range={{date_filter}}
    }
    stacked=false
    data_labels={
        position="above"
        fmt="num0"
    }
/%}

{% callout type="info" title="Key Takeaways — Status Breakdown" %}
- **58.6% of all requests are cancelled** (1,595 out of 2,722) — the dominant outcome, suggesting many sellers list but withdraw before finding a buyer.
- **24.2% result in a sale** (660 sold) — a healthy conversion given the high cancellation rate.
- **14.4% are currently listed** (391) — active supply in the marketplace awaiting buyers.
- **Only 1.3% are in Pending Contracts Signing** (36) — the pipeline between matched and closed is thin, indicating fast contract execution once a buyer is found.
{% /callout %}

---

## Timeline Analysis

```sql monthly_trend
SELECT
    toStartOfMonth(resale_requested_at) as month,
    count(*) as total_requests,
    countIf(resale_status = 'SOLD') as sold,
    countIf(resale_status = 'SOLD') * 1.0 / count(*) as conversion_rate
FROM resale_enriched
WHERE property_code NOT IN ('SAFE-21', 'SAFE-23')
GROUP BY month
ORDER BY month
```

{% line_chart
    data="monthly_trend"
    x="month"
    y=["total_requests", "sold"]
    y2="conversion_rate"
    y_fmt="num0"
    y2_fmt="pct1"
    title="Monthly Resale Trend"
    info="Monthly count of resale requests (total and sold only) with conversion rate on secondary axis. Total Requests = COUNT(*), Sold = COUNT(*) WHERE resale_status = 'SOLD', Conversion Rate = Sold / Total Requests. Excludes test properties."
    y2_axis_options={
        title="Conversion Rate"
    }
    chart_options={
        series_colors={
            "total_requests"="#3b82f6"
            "sold"="#22c55e"
            "conversion_rate"="#9ca3af"
        }
    }
/%}

{% callout type="info" title="Key Takeaways — Timeline Analysis" %}
- **October 2025 was the peak month** with 801 requests and 251 sold — likely driven by a launch or promotional push.
- **Declining trend since October** — requests dropped from 801 → 551 → 392 → 381 → 336, suggesting the initial surge is normalizing.
- **Sold count is declining faster than requests** — from 251 sold in Oct to 38 in Feb, indicating the conversion rate is also dropping month-over-month.
- **February 2026 is partial data** (up to Feb 18), so the apparent drop may recover by month-end.
{% /callout %}

---

## Pricing Analysis

{% bar_chart
    data="resale_enriched"
    x="price_comparison"
    y="count(*) as request_count"
    title="Price Comparison Distribution"
    info="COUNT(*) of resale requests grouped by price_comparison (Above/Below/At Market Price). Filtered to Listed or Sold requests only. Excludes test properties."
    order="count(*) desc"
    fmt="num0"
    where="property_code NOT IN ('SAFE-21', 'SAFE-23') AND resale_status IN ('LISTED', 'SOLD')"
    filters=["property_filter", "status_filter"]
    date_range={
        date="resale_requested_at"
        range={{date_filter}}
    }
    stacked=false
    data_labels={
        position="above"
        fmt="num0"
    }
/%}

```sql pricing_by_property
SELECT
    property_name,
    avg(demanded_price / number_of_shares) as avg_demanded_price,
    avg(market_price / number_of_shares) as avg_market_price,
    avg(demanded_price / number_of_shares) - avg(market_price / number_of_shares) as avg_price_diff,
    countIf(price_comparison = 'Above Market Price') as above_market,
    countIf(price_comparison = 'At Market Price') as at_market,
    countIf(price_comparison = 'Below Market Price') as below_market
FROM resale_enriched
WHERE property_code NOT IN ('SAFE-21', 'SAFE-23')
    AND resale_status IN ('LISTED', 'SOLD')
GROUP BY property_name
ORDER BY avg_price_diff DESC
```

{% table
    data="pricing_by_property"
    title="Pricing by Property"
    info="Average demanded_price vs market_price per property. Only Listed or Sold requests. Shows whether sellers on each property are pricing aggressively or competitively. Excludes test properties."
    subtotals=false
    page_size=15
%}
    {% dimension value="property_name" title="Property" /%}
    {% measure value="avg(avg_demanded_price)" title="Avg Demanded Price / Share" fmt="#,##0' EGP'" info="AVG(demanded_price / number_of_shares) for Listed/Sold requests. Per-share price." /%}
    {% measure value="avg(avg_market_price)" title="Avg Market Price / Share" fmt="#,##0' EGP'" info="AVG(market_price / number_of_shares) for Listed/Sold requests. Per-share price." /%}
    {% measure value="avg(avg_price_diff)" title="Avg Price Diff / Share" fmt="#,##0' EGP'" info="AVG(demanded_price / shares) - AVG(market_price / shares). Positive = above market, negative = below market. Per-share difference." /%}
    {% measure value="sum(above_market)" title="Above Market" fmt="num0" info="Count of requests priced above market price." /%}
    {% measure value="sum(at_market)" title="At Market" fmt="num0" info="Count of requests priced at market price." /%}
    {% measure value="sum(below_market)" title="Below Market" fmt="num0" info="Count of requests priced below market price." /%}
{% /table %}

{% callout type="info" title="Key Takeaways — Pricing Analysis" %}
- **60.7% of listings are priced below market** (638 out of 1,051) — sellers are predominantly competitive, willing to discount to attract buyers.
- **Only 17.8% price above market** (187) — aggressive pricing is rare, reinforcing the buyer-friendly market dynamics.
- **Katameya Heights Admin Mall G6** has the largest negative price diff (-3,256 EGP below market avg) — sellers here are most willing to undercut.
- **Park St. East is the only property priced above market** (+1,484 EGP avg) — sellers here are more confident, possibly due to the property's unique appeal or limited supply.
- **The below-market pricing trend correlates with the 24% sold rate** — even with discounts, only 1 in 4 requests convert, suggesting demand-side constraints rather than pricing issues.
{% /callout %}
