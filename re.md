## 3. Scoring Arithmetic (Negative Weight table, row 3 — completing what's on this page)

| # | Penalty trigger | Weight | Proportionate? | Duplicated elsewhere? | Explicit requirement in prompt |
|---|---|---|---|---|---|
| 3 | Concentration dismissed without sizing | −6 | Y | N | Yes — direct match to "...equally by analysts who waved concentration off without sizing it" |

## Scoring Arithmetic

| Measure | Value |
|---|---|
| Total positive weight available | 100 |
| Total negative weight exposure | −26 (10 of which, penalty-contract-equals-cash, flagged Non-relevant — see Section 7) |
| Ideal Response — estimated score | ~90/100, executing the rubric's required methodology (even the Prescriptive-flagged criteria are still gradeable and satisfiable — the flag is about fairness of what's demanded, not about whether a compliant answer can score well) |
| Maximum realistically achievable score | ~93-95/100 — capped slightly below full marks by genuine word-budget pressure fitting 3-4 required proxy calculations, 3 limitations, and both closing items inside 550 words |

**Conclusion:** No — the negatives do not by themselves prevent a good answer from scoring well. A hedged, correctly-sourced response triggers zero penalties and clears the 70% threshold comfortably even accounting for realistic imperfect coverage. The rubric's problems are concentrated in the *positive* side (method-locking, off-scope criteria), not in punitive negative weighting.

## 6. Factual & Numerical Verification

| Rubric claim / expected figure | Source checked (filing, form, period, section) | Verified? | Correct value if different |
|---|---|---|---|
| FY2026 ending RPO ≈$638B | Oracle IR press release, "Record Q4 and FY2026 Results," June 10, 2026 — https://investor.oracle.com/investor-news/news-details/2026/Oracle-Announces-Record-Q4-and-FY-2026-Results-Driven-by-Cloud-Infrastructure--Cloud-Applications/default.aspx | Yes | — |
| Near-term RPO 12% / $76.6B | Oracle Q4 FY2026 earnings call transcript, June 10, 2026 — https://finance.yahoo.com/quote/ORCL/earnings/ORCL-Q4-2026-earnings_call-592465.html | Yes | — |
| FY2027 revenue target ≈$90B | Same IR press release as above, June 10, 2026 | Yes | — |
| Q4 OCI revenue $5.8B / +93% YoY | Oracle official Q4 FY26 press release PDF — https://sherwood.news/tech/oracle-q4-earnings-and-revenue-top-estimates/ (reproduces primary release figures), June 10, 2026 | Yes | — |
| FY2026 capex ≈$55.7B | MLQ News, sourced to Oracle earnings materials, June 11, 2026 — https://mlq.ai/news/oracle-reports-557b-fy2026-capex-guides-to-70b-net-outlay-in-fy2027/ | Yes | — |
| FY2027 gross capex $90–95B | Outlook Business, sourced to Oracle Q4 FY26 earnings call (CFO Maxson), June 11, 2026 — https://www.outlookbusiness.com/corporate/oracle-forecasts-95-bn-in-capex-for-fy27-plans-to-raise-40-bn-in-debt-and-equity | Yes | — |
| Lease commitments ≈$260B, FY27–29 commencement, 15–19yr terms | Oracle FY2026 Form 10-K (period ending 5/31/26, filed 6/22/26), as reported in TradingKey analysis, published ~July 17, 2026 — https://www.tradingkey.com/analysis/stocks/us-stocks/262034371-oracle-ai-rpo-credit-downgrade-cash-flow-analysis-tradingkey | **Unverifiable** — confirmed via secondary reporting that cites the 10-K directly; I did not independently pull the primary EDGAR document in this session, so flagging as unverified-against-primary rather than fully verified | — |
| OpenAI annualized revenue >$25B, Feb 2026, Reuters-unverified caveat | Reuters wire via Yahoo Finance, March 4, 2026, citing The Information — https://finance.yahoo.com/news/openai-tops-25-billion-annualized-033836274.html | Yes — including the exact "Reuters could not verify the report" caveat | — |
| $300B Oracle-OpenAI contract, reported not company-disclosed | CNBC, June 12, 2025 report referenced via CNBC lease-commitments piece, December 11, 2025 — https://www.cnbc.com/2025/12/11/oracle-lease-commitments-increase-almost-150percent-to-accommodate-ai-demand.html | Yes — confirmed as press-reported, not present in Oracle's own financial statements | — |
| S&P downgrade citing >50% RPO concentration in OpenAI | Motley Fool / Yahoo Finance, July 19, 2026 — https://www.fool.com/investing/2026/07/19/oracle-just-hit-a-fresh-52-week-low-and-had-its-cr/ | Yes | — |

## 7. Summary of Flags

**Punitive:** None. (Earlier working flag on penalty-proxy-as-disclosure was reversed on review — it's method-agnostic and directly grounded in the prompt's own stated concern, not stacked unfairly.)

**Overly prescriptive:** contract-rpo-proxy (7), openai-capacity-burden (6), oci-scale-comparison (5) — each locks one specific calculation methodology with no accepted alternative, where the prompt explicitly grants "whatever quantified concentration analysis the public record supports."

**Non-relevant:** fy2027-revenue-target (3), fy2026-capex (3), q4-oci-scale (4) — real, accurate, Oracle-wide figures with no independent tie to the single-counterparty concentration question asked; q4-oci-scale exists only to feed the flagged oci-scale-comparison criterion, so these two share one root cause. Also penalty-contract-equals-cash (−10) — a genuine accounting-literacy check, but not anchored to anything the prompt actually raises as a concern.

**Factually inaccurate:** None. Every numeric claim checked against primary or high-quality secondary sourcing was confirmed accurate; one item (lease commitments) is flagged unverifiable-against-primary rather than wrong.

**Sources used:** direct Oracle IR press release, Oracle Q4 FY2026 earnings call transcript (via Yahoo Finance), Reuters wire (via Yahoo Finance), CNBC, MLQ News, Outlook Business, Sherwood News, TradingKey, Motley Fool/Yahoo Finance — full links above. SEC EDGAR was not directly fetched in this session for the 10-K; recommend the assigned analyst pull the primary filing directly before finalizing this line item.
