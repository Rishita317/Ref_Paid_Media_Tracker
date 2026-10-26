# Paid Media Anomaly Monitor

A small dashboard that watches daily Google Ads and Meta Ads performance and flags days when return on ad spend (ROAS) moves sharply. For each flagged day it explains in plain language what likely went wrong and where to look.

I built it as a demo for the **AI Operations Analyst, Growth** role at Reformation.

**▶ To run it:** download `reformation_paid_media_anomaly_dashboard.html` and open it in any browser. There's no install or build step. The only outside file it loads is Chart.js, from a CDN.

---

## About the project

Marketing teams usually find out about a broken pixel or a runaway budget days later, while reviewing a report. This project shows what an automated first line of defense could look like:

1. **Data:** 60 days of mock daily data for Google Ads and Meta Ads (spend, impressions, clicks, conversions, revenue).
2. **Metrics:** ROAS (revenue ÷ spend), CAC (spend ÷ conversions) and a 7-day rolling ROAS baseline per channel.
3. **Detection:** a day is flagged when its ROAS is more than **±30%** away from the average of the previous 7 days.
4. **Explanation:** simple rules compare spend, click-through rate (CTR), conversion rate (CVR), conversions and average order value (AOV) with their own 7-day averages. The first rule that matches names the likely cause and a next step.
5. **Dashboard:** one tab per channel, each with a date filter, KPI tiles, a spend chart, a ROAS chart with anomaly markers, and an anomaly table.

### Example output

> **Meta Ads, Sep 19 (Critical).** ROAS dropped 63% vs. the 7-day average (1.00x vs. 2.70x). Conversions fell 54% even though clicks held up (+23%), so the conversion rate is down 63%.
> → Likely a tracking or checkout break. Verify the pixel / Conversions API and site checkout before cutting budget.

---

## Results

I injected five realistic incidents into the mock data so the detector could be checked against a known answer key:

| Date | Channel | Injected incident | What the detector reported |
|---|---|---|---|
| Sep 12 | Google | Budget runaway | Spend +79%, conversions flat |
| Sep 19 | Meta | Pixel / CAPI outage | CVR −63% while clicks held up |
| Oct 2 | Meta | Viral / influencer moment | Conversions +119% |
| Oct 10 | Google | Discount-code leak | AOV −44%, order count steady |
| Oct 18 | Meta | Creative fatigue | CTR −41% |

**All 5 caught, with 0 false positives.** The dashboard recomputes this score live on every load.

---

## How the mock data was made realistic

The data is simulated, but its scale comes from Reformation's public numbers:

| Input | Value | Source |
|---|---|---|
| FY2025 net revenue | $507.1M | S-1 filing |
| Marketing spend | ~9% of net revenue, steady for 4 years (≈ $45M/yr) | S-1 filing |
| Average order value | $315 | S-1 filing |
| DTC share | ~90% of revenue; e-commerce = 67% of DTC | S-1 filing |
| New customers from non-paid channels | ~75% of new DTC customers | S-1 filing |
| Paid social mix | Mostly Instagram, then Facebook (older estimate) | Pathmatics via Winmo |
| Funnel benchmarks | Meta CTR ~1.1%, ROAS ~2.5–3x; Google CTR ~1.7–1.9%, ROAS ~4x | TrendTrack, Triple Whale |

**My assumption (not public data):** about 57% of the marketing budget goes to Google and Meta, which works out to roughly **$72K/day** (Google ~$30K, Meta ~$42K). With that, the mock data comes out at:

- **Google:** ROAS 4.3x, CAC $73
- **Meta:** ROAS 2.6x, CAC $114

The data also includes realistic patterns:

- **Weekly rhythm:** Meta spends more on weekends; Google peaks early in the week.
- **Budget ramp:** a 12% increase over the 60 days, heading into the holiday season.
- **Meta CPMs:** about 10% higher by the end of the window (Q4 auction pressure).

A fixed random seed means the dashboard shows the same data every time it loads.

---

## Design decisions

- **The baseline leaves out the day being checked.** If today were part of its own average, a 45% drop would pull the average down and look like only ~39%.
- **Baselines always use the full history.** The date filter only changes what's shown, so a "Last 7d" view still has real baselines behind it.
- **Blended ROAS and CAC are total ÷ total**, not the mean of daily ratios, so high-spend days carry their proper weight.
- **Spend and ROAS are on separate charts that share the same dates**, not one chart with two y-axes. Two axes can suggest correlations that aren't there.
- **Rule-based explanations, not a model.** They're transparent, easy to audit and easy for a marketer to trust. The rules are checked in priority order, so a tracking break isn't mistaken for a budget problem.
- **The code is split into labeled sections:** configuration → data generation → metric calculation → anomaly detection → rendering.

---

## Limitations and next steps

- **Mock data only.** The next step would be pointing it at real Google Ads and Meta exports or a warehouse table, for example in Databricks.
- **The baseline can be contaminated.** A big anomaly day stays in the next 7 baselines. A more robust version would leave flagged days out of the baseline or use a median.
- **"CAC" here is really cost per conversion**, because conversions include repeat buyers. True CAC would need new-customer flags.
- **A fixed 30% threshold** could be replaced with per-channel thresholds based on each channel's own volatility (for example, a z-score).
- **It only shows alerts on screen.** In production, flags would go to Slack or email, ideally with campaign-level breakdowns.
- **An LLM layer could be added.** It could turn the rule outputs plus campaign context into richer explanations, keeping the rules as the source of truth.

---

## Repository contents

```
reformation_paid_media_anomaly_dashboard.html   # the complete dashboard (HTML + CSS + JS)
README.md                                       # this file
```

---

## Sources

- [Reformation Inc. Form S-1 (SEC)](https://www.sec.gov/Archives/edgar/data/0001787117/000110465926077832/tm2513004-7_s1.htm)
- [Sovereign Magazine: Reformation IPO, first public numbers](https://www.sovereignmagazine.com/article/reformation-ipo-first-public-numbers-customers-stores)
- [Retail TouchPoints: Six things to know about Reformation's business](https://www.retailtouchpoints.com/?p=620215)
- [Quartz: Reformation IPO](https://qz.com/reformation-ipo-nyse-womenswear-sustainable-revenue-062626)
- [Winmo: Reformation digital spend (Pathmatics estimates)](https://open.winmo.com/open/article/38434)
- [TrendTrack: Meta ad benchmarks by industry, 2025](https://www.trendtrack.io/blog-post/meta-ad-spend-by-industry)
- [Triple Whale: Google Ads benchmarks](https://www.triplewhale.com/blog/google-ads-benchmarks)
- [Triple Whale: 2025 Apparel & Accessories benchmarks](https://www.triplewhale.com/reports-guides/fashion-industry-benchmarks-2025)

---

*Built by Rishita Meharishi. All performance data in this project is simulated; only the calibration inputs above are drawn from public sources.*
