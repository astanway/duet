# duet room

working directory: `~/models/saas-memo`

### [1] Human

We are sizing a seed-stage SaaS company for an investment memo. Known: $2.0M ARR today, net revenue retention 115%, annual logo churn 12%, gross margin 72%, new ARR added last 12 months $1.1M, sales and marketing spend last 12 months $1.4M. Task for both of you: build a simple 3-year ARR projection as a Python script model.py in this directory that prints a yearly table, state your assumptions explicitly, and tell me the year-3 ARR you get and why. If you disagree with each other, say so.

### [2] Claude

**File:** `model.py` in the working directory. Run with `python3 model.py`. All parameters are at the top; three scenarios in the `SCENARIOS` dict.

## Model structure

One identity per year: ARR[t+1] = ARR[t] × NRR + S&M[t+1] × efficiency. Efficiency is new-logo ARR per S&M dollar. Logo churn is not a driver because NRR already nets churn and expansion. I use it only as a consistency check.

## Year-3 ARR

| Scenario | Y1 | Y2 | Y3 | Cum. S&M Y1–3 |
|---|---|---|---|---|
| Bear | 3.20 | 4.37 | 5.46 | 4.8 |
| **Base** | 3.82 | 6.28 | **9.36** | 9.1 |
| Bull | 4.50 | 9.57 | 19.81 | 19.6 |

All $M. Base year-3 ARR is about $9.4M, roughly 4.7x today.

## Assumptions (Base)

- **S&M spend** steps 1.4 → 2.1 → 3.0 → 4.0. This presumes a seed or A round funds the step-ups. Spend is the single biggest lever and is a decision, not a forecast.
- **Efficiency** starts at the trailing 0.79 and decays about 8% a year to 0.61. Early channels saturate and new reps ramp slower than they are hired.
- **NRR** drifts 115 → 112 → 110. The 115% was measured on an implied $0.78M base, so it is a small, noisy cohort. Early adopters expand more than the average later customer.
- Bear holds spend roughly flat with faster efficiency and NRR decay. Bull doubles spend yearly with no decay. Bull is not credible at this CAC payback. I include it only as the ceiling.

## Things the inputs tell you that the memo should flag

- **Implied history.** If $1.1M is gross new-logo ARR, ARR a year ago was $0.78M and growth was 156%. If it is net new ARR, prior ARR was $0.90M and new-logo ARR was $0.97M. The script prints both. Base year 3 under the net reading is $8.6M instead of $9.4M.
- **CAC payback** is 21 months on gross margin, 24 under the net reading. That is acceptable for seed but does not support the Bull spend ramp.
- **Churn consistency.** 12% logo churn with 115% NRR implies about 27 points of gross expansion if churned logos are average-sized. That is high. Either churned logos are small, or expansion is concentrated in a few accounts. Ask for the top-10 customer concentration and GRR.

## Question for the human

Is the $1.1M gross new-logo ARR or net new ARR? The script assumes gross. The answer changes Base year 3 by about $0.8M and shifts the implied prior-year growth rate, which matters for how much weight to put on the NRR figure.

What would move my number most, in order: the S&M step-up path, then the efficiency decay rate, then NRR. A one-year S&M stall in year 1 takes Base year 3 from $9.4M to roughly $7.5M.

### [3] Codex

My year-3 reference is **$6.86M ARR**, assuming current operating inputs persist. Claude’s $9.36M computes correctly, but requires an unconfirmed spending ramp.

Updated **`model.py`**: preserved Claude’s scenarios, added `Codex constant-input reference`, clarified new-logo ARR timing, and corrected the churn and payback labels. Ran `python3 model.py` and independently checked my projection against its closed-form formula.

| $M | Today | Y1 | Y2 | Y3 |
|---|---:|---:|---:|---:|
| ARR | 2.00 | 3.40 | 5.01 | **6.86** |
| Annual S&M | 1.40 | 1.40 | 1.40 | 1.40 |
| Annual new-logo ARR | 1.10 | 1.10 | 1.10 | 1.10 |

My assumptions:

- **NRR stays 115%.** This is a persistence assumption, not evidence that retention will hold.
- **S&M stays $1.4M annually**, producing $1.1M of new-logo ARR annually at unchanged efficiency.
- **$1.1M means year-end ARR from newly acquired logos.** If it means bookings at signing, in-year churn and expansion need adjustment.
- New cohorts enter NRR the following year. Gross margin does not directly drive ARR; no financing or cash-runway model is included.

Thus, `ARR[t+1] = 1.15 × ARR[t] + 1.10`. If the historical $1.1M instead means **net** new ARR, my year-3 result is **$6.39M**.

Three corrections to Claude’s interpretation:

1. **A year-1-only S&M stall gives $8.74M, not $7.5M**, holding subsequent base spending at $3M and $4M. A different stall definition needs an explicit spending path.
2. **27 percentage points is expansion minus contraction**, assuming churned logos have average starting ARR. Calling it gross expansion additionally assumes zero contraction.
3. **21.2 months is a blended payback proxy.** Total S&M includes potentially different acquisition, retention, and expansion costs, and spending may precede sales. These inputs alone establish neither “acceptable for seed” nor that the bull case is infeasible. Nor is bull a mathematical ceiling.

I would move toward Claude’s funded-growth case with an approved hiring/spending plan and evidence that incremental sales capacity maintains productivity. Until then, $6.86M is my conditional reference; it is not a downside floor.

### [4] Human → Codex

Claude built the first version. Codex: review model.py, run it, and tell me the single assumption you would most want to change and what year-3 ARR becomes if you change it. Do not rewrite the file.

### [5] Codex

Read and ran `model.py`; Claude’s Base returns **$9.36M year-3 ARR**. No files changed.

**The single assumption I would change is the S&M spending path:** replace $2.1M / $3.0M / $4.0M with **$1.4M each year**. The supplied facts do not establish funding or a hiring plan for the ramp.

Holding every other Base assumption fixed—including declining efficiency, NRR of 115% / 112% / 110%, and the gross new-logo interpretation:

| ARR, $M | Y1 | Y2 | Y3 |
|---|---:|---:|---:|
| Original Base | 3.82 | 6.28 | 9.36 |
| Flat S&M only | 3.31 | 4.64 | **5.97** |

That reduces year-3 ARR by **$3.39M, or 36.2%**. I computed this using the model’s projection function and checked the arithmetic independently.

This differs from my earlier $6.86M reference because that also held efficiency and NRR constant; **$5.97M isolates one assumption change**. Flat spending is a conditional reference, not a forecast. A funded operating plan with credible sales-capacity milestones would move me toward Claude’s ramp.
