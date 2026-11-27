# Checkout A/B test analysis

End-to-end analysis of an A/B test on an e-commerce checkout: does a one-tap payment flow with a saved card increase payment conversion without hurting the post-purchase experience?

**Result in one line:** conversion rose by about 1 pp, but support contacts rose too, so the recommendation is to fix the cause and retest rather than launch.

The full analysis is in [`notebooks/analysis.ipynb`](notebooks/analysis.ipynb).

## What the analysis covers

1. **Test design:** hypothesis, primary metric, secondary metric and guardrails, with the launch rule set before looking at results
2. **Sanity checks:** duplicates, missing data, sample ratio mismatch (SRM)
3. **Power:** sample size needed for a 1 pp minimum detectable effect, compared with the actual sample
4. **Primary metric:** z-test for payment conversion with a confidence interval
5. **Revenue per user:** bootstrap CI, because revenue is heavily skewed
6. **Guardrails:** refund rate and support contact rate
7. **Segments:** effect by platform, with Holm correction for multiple testing
8. **Business impact:** extra orders, GMV and support tickets per year
9. **Recommendation:** launch / don't launch, with next steps

## Data

The data is **simulated** with `src/generate_data.py` (60,000 users, fixed seed). It imitates a real experiment, including a heavy-tailed order value and a side effect on support contacts, so that the analysis has realistic trade-offs to work through. The numbers are illustrative; the method is what matters.

| column | meaning |
|---|---|
| `user_id` | user who reached checkout |
| `group` | `control` (current flow) or `treatment` (one-tap saved card) |
| `platform`, `country` | segments |
| `paid` | whether the user completed payment |
| `order_value_eur` | order value, 0 if not paid |
| `refunded` | order was refunded |
| `contacted_support` | user contacted support after purchase |

## Project structure

```
├── data/checkout_experiment.csv   # generated dataset
├── notebooks/analysis.ipynb       # the analysis, with outputs
├── src/
│   ├── generate_data.py           # data simulation
│   └── ab_utils.py                # reusable functions: SRM, power, z-test, bootstrap, Holm
└── requirements.txt
```

`ab_utils.py` holds the statistical checks as reusable functions, so the same analysis can be rerun on a new experiment by loading different data.

## How to run

```bash
pip install -r requirements.txt
python src/generate_data.py
jupyter notebook notebooks/analysis.ipynb
```
