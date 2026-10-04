# Customer-Churn-Analysis

I started this as a standard churn prediction project: logistic regression,
feature engineering, a risk score for every customer. It worked fine, 80%
accuracy and decent separation between churners and stayers.

One thing kept bothering me, though. The model kept ranking "has Tech
Support" as one of the biggest churn predictors, with customers who lacked
it churning at roughly triple the rate. That's the kind of number that
turns into "offer everyone Tech Support, cut churn by 25+ points" in a
slide deck. Before trusting it, I checked whether Tech Support was
actually causing that drop, or whether it was just a marker for customers
who were already loyal for other reasons, like longer contracts and
auto-pay, who happen to have Tech Support too.

The honest answer: mostly the second one. Details below.

## Objective

Identify customers with high churn risk using historical telecom customer
data, and figure out which of the model's top "risk factors" are actually
worth acting on versus which ones just look important on the surface.

## Dataset

IBM Telco Customer Churn dataset, sourced from Kaggle.

Dataset size: 7,043 customers
Target variable: Churn (Yes/No)

Data includes:
- Customer demographics
- Service subscriptions
- Contract information
- Billing and payment details
- Customer tenure
- Churn status

## Workflow

### Data Preparation
- Cleaned and validated billing-related fields
- Converted categorical variables into machine-readable format
- Removed irrelevant identifiers

### Feature Engineering
Created business-oriented features such as:
- New customer indicator
- High monthly charge indicator
- Month-to-month contract flag
- Spending and tenure-based ratios

### Modeling
- Logistic Regression classifier
- Stratified train-test split
- 80/20 training and testing setup

### Evaluation
Evaluated model performance using:
- Accuracy
- Precision
- Recall
- F1 Score
- ROC-AUC

### Reporting
Generated:
- Churn probability predictions
- Customer risk segmentation
- Feature importance analysis
- Power BI-ready datasets

## Model Performance

| Metric | Score |
|---|---|
| Accuracy | 79% |
| Precision | 65% |
| Recall | 55% |
| F1 Score | 59% |
| ROC-AUC | 0.83 |

## Interpretation
- 65% of customers flagged as high-risk actually churned
- The model identified roughly half of actual churners
- ROC-AUC of 0.83 indicates strong class separation capability

## Key Insights (Predictive Model)
- Month-to-month contracts showed significantly higher churn risk
- Short-tenure customers were more likely to churn
- Higher monthly charges correlated with increased churn probability
- Customers without tech support or security services showed higher churn tendency
- Contract structure and tenure emerged as strong predictive signals

## Does Tech Support Actually Reduce Churn?

The predictive model flags Tech Support as one of its stronger churn
correlates. But a predictor isn't automatically a lever. Customers who
subscribe to Tech Support are also disproportionately on longer
contracts, pay by auto-debit, and have been with the company longer, and
all of those things reduce churn on their own.

To separate the real effect from the confounding, I used propensity score
matching: for each Tech Support subscriber, I found a statistically
similar non-subscriber (matched on contract type, tenure, monthly
charges, internet service type, payment method, and household
composition) and compared outcomes only within those matched pairs.

| Estimate | Churn rate difference |
|---|---|
| Naive comparison | -26.5 percentage points |
| Matched causal estimate | -3.6 percentage points (95% CI: -5.7 to -1.5) |

Roughly 85 to 90 percent of the naive effect turned out to be confounding,
not Tech Support itself. The real, defensible effect is smaller than it
looks at first glance, but it's genuine. The confidence interval doesn't
cross zero.

Full methodology, including the confounder reasoning, the propensity
model, and the covariate balance check, is in
`Causal_Analysis_TechSupport.ipynb`.

## What This Means for the Business

If a retention team is deciding whether to proactively offer Tech Support
to high-risk customers, this changes the math. The naive number (about 26
points) would justify almost any retention budget. The real number (3 to
4 points) still justifies offering it, since it's a genuine, measurable
effect, but it means Tech Support alone won't rescue a struggling
retention program. Spending decisions should be sized against a 3 to 4
point improvement, not a 26 point one. Pairing it with something that
addresses the bigger driver in this dataset, contract length, is likely
to matter more than either lever on its own.

## Business Use Cases
- Customer retention prioritization
- Risk-based customer segmentation, sized against realistic effect estimates
- Retention campaign targeting
- Churn monitoring dashboards
- Predictive support workflows

The model also categorizes customers into:
- Low Risk
- Medium Risk
- High Risk

based on predicted churn probability.

## Project Files

| File | Description |
|---|---|
| `Customer_Churn_Analysis.ipynb` | Complete analysis and modeling notebook |
| `Causal_Analysis_TechSupport.ipynb` | Propensity score matching analysis of Tech Support's causal effect on churn |
| `feature_importance.csv` | Logistic Regression coefficients |
| `churn_predictions.csv` | Prediction results on test data |
| `churn_predictions_for_powerbi.csv` | Processed dataset for Power BI |
| `churn_dashboard.pbix` | Power BI dashboard |

## Tech Stack
Python, Pandas, NumPy, Scikit-learn, Matplotlib, Seaborn, Power BI, Google Colab

## Limitations
- Dataset represents a static snapshot of customer behavior, with no time dimension
- Additional behavioral or temporal features may improve prediction quality
- The causal estimate assumes the confounders included in the matching model capture the main reasons a customer chooses Tech Support. An unmeasured factor, such as a prior support interaction not captured in this dataset, could still bias the result
- Some customer profiles have a very high or very low likelihood of subscribing to Tech Support given their characteristics, which means the matched estimate for those segments leans more on extrapolation than on a genuinely comparable match
- Production deployment would require continuous retraining and monitoring

## Future Improvements
- Experiment with ensemble models such as Random Forest or XGBoost for the prediction task
- Incorporate time-series or behavioral interaction features
- Run a randomized pilot, offering Tech Support free to a random sample of at-risk customers, to validate the causal estimate experimentally rather than relying on matching alone
- Extend the causal approach to other retention levers, such as Online Security or contract upgrade incentives
- Deploy as an interactive dashboard or API service

End-to-end churn analytics project combining data preprocessing, feature
engineering, predictive modeling, causal validation of a specific
retention lever, and business interpretation.
