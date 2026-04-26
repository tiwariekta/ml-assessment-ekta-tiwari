# 📊 Part B: Business Case Analysis

## Scenario: Promotion Effectiveness at a Fashion Retail Chain

---

# 🔹 B1. Problem Formulation

---

## (a) Machine Learning Problem Definition

### 🎯 Target Variable

* **`items_sold`** — number of units sold per store per month

This directly captures how effective a promotion is in driving **customer purchase behavior**, independent of pricing distortions.

---

### 📥 Candidate Input Features

Rather than listing generic features, we group them based on how they influence buying behavior:

| Driver Type              | Features                             | Business Role                                   |
| ------------------------ | ------------------------------------ | ----------------------------------------------- |
| 🏬 Store Characteristics | `store_size`, `location_type`        | Defines capacity and customer profile           |
| 🎯 Promotion Strategy    | `promotion_type`                     | Core intervention we are optimizing             |
| 📅 Temporal Context      | `month`, `is_weekend`, `is_festival` | Captures seasonal demand spikes                 |
| 🌍 Market Pressure       | `competition_density`                | Impacts customer choice and pricing sensitivity |

👉 These features together explain **why a customer buys**, not just *what happened*

---

### 🤖 Type of ML Problem

* **Supervised Learning — Regression**

---

### 🧠 Justification (Non-generic)

We are not just predicting sales—we are estimating:

> “How many additional items will be sold given a specific promotion under specific store conditions?”

This is inherently:

* **Continuous output** → regression
* **Causal-influenced prediction** → promotion acts as a treatment variable

Unlike classification (e.g., high vs low sales), regression preserves **granularity**, which is critical for:

* Inventory planning
* Promotion ROI estimation

---

## (b) Why "Items Sold" is Better than Revenue

### ⚠️ Revenue is Confounded

Revenue is not a pure signal of promotion success because:

* A **Flat Discount** may increase volume but reduce per-unit margin
* A **Free Gift** may inflate perceived value without increasing actual units sold
* A **Category Offer** may shift sales across categories rather than increase total demand

👉 So revenue mixes:

* Pricing strategy
* Product mix
* Promotion effect

---

### ✅ Items Sold = Clean Demand Signal

`items_sold` isolates:

* **Customer response to promotion**
* **Actual movement of inventory**

This is especially important in retail where:

* Overstock risk is real
* Sell-through rate matters more than top-line revenue

---

### 🌟 Broader Principle

> **The target variable must represent the decision you want to optimize, not just a downstream financial outcome.**

In this case:

* Decision: *Which promotion increases customer buying?*
* Correct signal: **volume (items sold)**
* Not: revenue (which is post-pricing and noisy)

---

## (c) Improved Modelling Strategy

### ❌ Why a Single Global Model Fails

A global model assumes:

> “BOGO works the same in an urban premium store and a rural discount-driven store”

This is unrealistic because:

* Urban customers may respond to **convenience + brand**
* Rural customers may respond more to **price sensitivity**
* Competition density changes how attractive a promotion feels

👉 Same promotion ≠ same effect

---

### ✅ Recommended Strategy: Context-Aware Modelling

#### 🔹 Option 1: Segment-Based Models (Practical & Strong)

Train separate models for:

* Urban
* Semi-urban
* Rural

✔ Captures **macro-level behavioral differences**

---

#### 🔹 Option 2: Interaction-Driven Single Model (Most Efficient)

Instead of separate models, enrich features:

* `promotion_type × location_type`
* `promotion_type × store_size`
* `promotion_type × competition_density`

👉 This allows the model to learn:

> “BOGO works well in low-competition urban areas but not in high-competition rural zones”

---

#### 🔹 Option 3: Cluster-Based Strategy (Most Advanced)

Using insights from clustering (Q2):

* Group stores based on behavior (not just geography)
* Train one model per cluster

✔ Captures **hidden patterns beyond simple labels**

---

### 🧠 Final Recommendation

> Use a **single model with interaction features** as the primary approach, and validate performance against a **segmented model**.

This balances:

* Scalability (one model)
* Personalization (context-aware predictions)

---

### 🚀 Business Impact

This approach enables:

* Store-level promotion optimization
* Reduced blanket campaigns
* Higher ROI per promotion

👉 Moving from **“one promotion fits all” → “right promotion for the right store”**

---


# 📊 B2. Data and EDA Strategy

---

## (a) Data Integration and Dataset Design

### 🔗 How the Tables Would Be Joined

We have four tables:

* **Transactions** (store_id, transaction_date, items_sold)
* **Store Attributes** (store_id, store_size, location_type, competition_density)
* **Promotion Details** (store_id, month, promotion_type)
* **Calendar** (date, is_weekend, is_festival)

---

### 🧩 Join Strategy (Not Generic — Logical Flow)

1. Start with **transactions** as the base (most granular table)

2. Join **store attributes**

   * Key: `store_id`
   * Type: Left join (every transaction must retain store info)

3. Join **calendar table**

   * Key: `transaction_date = date`
   * Adds temporal context (weekend, festival)

4. Join **promotion table**

   * Key: (`store_id`, `month`)
   * Assumption: one promotion per store per month

👉 Important: This ensures that each transaction is enriched with:

* Store context
* Time context
* Promotion exposure

---

### 📏 Grain of Final Dataset

> **One row = one store per day**

---

### 🧠 Why this grain?

* Promotions operate at **store-month level**, but customer response happens **daily**
* Daily grain allows capturing:

  * Weekend spikes
  * Festival-driven demand
  * Short-term promotion effects

---

### 🔄 Required Aggregations

Before modelling, we ensure consistency:

| Feature          | Aggregation                                       |
| ---------------- | ------------------------------------------------- |
| items_sold       | Sum per store per day                             |
| promotion_type   | Assigned per store-month (forward-filled to days) |
| calendar flags   | Direct join (no aggregation needed)               |
| store attributes | Static (no aggregation)                           |

---

### ⚠️ Critical Consideration

If transactions are at **transaction-level**, we must:

* Aggregate to **daily level per store**
* Avoid multiple rows per day (prevents duplication bias)

---

## (b) Exploratory Data Analysis (EDA Strategy)

EDA is not just visualization—it’s about **identifying drivers of demand and model risks**.

---

### 📊 1. Promotion Effectiveness Analysis

**Chart:**

* Boxplot / bar chart of `items_sold` by `promotion_type`

**What to look for:**

* Which promotions consistently drive higher volume
* Variability across promotions

**Impact:**

* Strong separation → promotion_type is highly predictive
* Overlapping distributions → need interaction features

---

### 📊 2. Store Segmentation Patterns

**Chart:**

* `items_sold` vs `location_type` and `store_size`

**What to look for:**

* Do urban stores outperform rural?
* Does store size correlate with sales?

**Impact:**

* If strong differences exist → need:

  * Segmented models OR
  * Interaction terms (`promotion × location`)

---

### 📊 3. Temporal Patterns (Seasonality)

**Chart:**

* Time series of `items_sold` aggregated by:

  * Month
  * Weekend vs weekday
  * Festival vs non-festival

**What to look for:**

* Seasonal spikes (festivals, month-end)
* Weekend uplift

**Impact:**

* Justifies:

  * `is_weekend`, `is_festival`, `month_end` features
* May require:

  * Non-linear models (Random Forest)

---

### 📊 4. Competition Impact

**Chart:**

* Scatter plot: `competition_density` vs `items_sold`

**What to look for:**

* Negative correlation (high competition → lower sales?)
* Non-linear patterns

**Impact:**

* If strong effect → include feature directly
* If non-linear → tree-based models preferred

---

### 📊 5. Feature Correlation / Redundancy Check

**Chart:**

* Correlation heatmap (numerical features)

**What to look for:**

* Highly correlated variables
* Redundant signals

**Impact:**

* Helps:

  * Simplify model
  * Avoid instability in linear regression

---

## (c) Handling Promotion Imbalance (80% No Promotion)

### ⚠️ Problem

* Model sees mostly **“no promotion” cases**
* Learns to predict baseline behavior
* Underestimates **true impact of promotions**

👉 Result:

* Biased predictions
* Poor promotion recommendation quality

---

### 🧠 Why this is tricky

This is not a standard class imbalance problem—it’s:

> **Treatment imbalance (promotion vs no promotion)**

---

### ✅ Solutions

#### 🔹 1. Balanced Sampling Strategy

* Downsample “no promotion” rows OR
* Oversample promotion cases

👉 Ensures model learns promotion effects properly

---

#### 🔹 2. Feature Engineering (Better Approach)

Create:

* `is_promotion` (0/1)
* Interaction features:

  * `promotion × store type`

👉 Helps model isolate promotion impact

---

#### 🔹 3. Separate Baseline vs Lift Modelling (Advanced)

* Model 1 → baseline sales (no promotion)
* Model 2 → incremental lift due to promotion

👉 More aligned with business objective

---

### 🚀 Final Insight

> If not handled, the model will default to predicting “business as usual” and fail to capture the true value of promotions.

---
# 📊 B3. Model Evaluation and Deployment

---

## (a) Train-Test Strategy and Evaluation Metrics

### 📆 Train-Test Split Design

Given:

* 3 years of monthly data
* 50 stores

The split should follow a **time-based (temporal) approach**:

* **Training Set:** First ~2.5 years
* **Test Set:** Most recent ~6 months

---

### 🧠 Why this structure?

This setup simulates the real business scenario:

> Using historical data to predict **future promotion performance**

---

### ❌ Why Random Split is Inappropriate

A random split would:

* Mix past and future data
* Allow the model to learn patterns from **future months during training**

👉 This leads to:

* **Data leakage**
* Overestimated performance
* Poor real-world reliability

Example:

* If December (festival-heavy) data leaks into training, the model may unrealistically perform well on seasonal predictions

---

### 📏 Evaluation Metrics

#### 🔹 1. RMSE (Root Mean Squared Error)

* Penalizes large prediction errors more heavily

**Interpretation:**

> “How badly are we misestimating high-demand months?”

Important because:

* Underestimating demand → stockouts
* Overestimating → excess inventory

---

#### 🔹 2. MAE (Mean Absolute Error)

* Average absolute difference between predicted and actual sales

**Interpretation:**

> “On average, how many items are we off per store per month?”

Useful because:

* Easy for business teams to understand
* Directly relates to planning errors

---

#### 🔹 3. Business-Oriented Interpretation

* Low RMSE → stable performance during peak periods
* Low MAE → consistent day-to-day prediction accuracy

👉 Together, they ensure:

* Reliable inventory planning
* Better promotion decisions

---

## (b) Explaining Different Promotion Recommendations

### 🧠 Scenario Insight

The same store receives:

* Loyalty Points Bonus (December)
* Flat Discount (March)

This indicates:

> The model is responding to **contextual differences**, not just store identity

---

### 🔍 Investigation Using Feature Importance

Step 1: Extract feature importance from the model
Step 2: Focus on key drivers such as:

* `month` / seasonal indicators
* `is_festival`
* `competition_density`
* Interaction: `promotion_type × time`

---

### 🧠 Likely Explanation

#### 📅 December (Festival Period)

* Higher baseline demand
* Customers less price-sensitive
* Loyalty rewards reinforce repeat purchases

👉 Model prefers **Loyalty Points Bonus**

---

#### 📅 March (Regular Month)

* Lower baseline demand
* Customers more price-sensitive

👉 Model prefers **Flat Discount** to stimulate demand

---

### 📢 How to Communicate to Marketing

Instead of saying:
❌ “The model says so”

Explain:

> “The model identifies that during high-demand months like December, customers respond better to value-added incentives like loyalty rewards. In contrast, during slower months like March, direct price reductions are more effective in driving purchases.”

---

### 🎯 Key Takeaway

> The model is not recommending promotions randomly—it is **adapting strategy based on seasonal customer behavior**

---

## (c) Deployment Strategy and Monitoring

---

### 🚀 Step 1: Model Packaging and Storage

* Save the full pipeline (preprocessing + model) using:

  * `joblib` or `pickle`

👉 Example:

* `model_pipeline.pkl`

---

### 🔄 Step 2: Monthly Data Preparation

At the start of each month:

1. Collect latest data:

   * Store attributes
   * Upcoming promotion options
   * Calendar features

2. Apply same feature engineering:

   * Date features
   * Encoding (handled by pipeline)

3. Feed data into saved pipeline:

   * Generate predicted `items_sold` for each promotion

---

### 🧠 Step 3: Recommendation Logic

For each store:

* Simulate all promotion options
* Select the one with **highest predicted items_sold**

👉 Output:

* Monthly promotion plan for all 50 stores

---

### 📊 Step 4: Monitoring and Performance Tracking

#### 🔹 1. Prediction vs Actual Tracking

* Compare predicted vs actual items sold monthly
* Track RMSE / MAE over time

---

#### 🔹 2. Drift Detection

Monitor:

* Changes in feature distributions (e.g., competition_density shifts)
* Changes in promotion effectiveness

👉 If patterns change → model assumptions no longer valid

---

#### 🔹 3. Alert Thresholds

Set triggers:

* RMSE increases beyond threshold
* Prediction error increases consistently

---

### 🔄 Step 5: Retraining Strategy

* Retrain model:

  * Every 3–6 months OR
  * When performance drops significantly

* Use latest data to capture:

  * Changing customer behavior
  * New market conditions

---

### 🎯 Final Insight

> Deployment is not just about making predictions—it is about continuously ensuring that the model remains aligned with evolving business realities.

---
