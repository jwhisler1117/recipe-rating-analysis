# Simple Recipes, Superior Ratings? 🍽️
**An Analysis of Ingredient Complexity vs. User Satisfaction on Food.com**
By Joe Whisler

---

## Introduction
Does a longer grocery list lead to a better meal? This project investigates the relationship between recipe complexity (measured by the number of ingredients) and user satisfaction (star ratings) using a dataset of over 200,000 recipes from Food.com. 

Understanding this relationship is vital for home cooks who often equate "more ingredients" with "better flavor." Our central question is: **Does the number of ingredients in a recipe significantly correlate with its average rating?**

**Dataset Overview:**
- **Number of Rows:** 231,637 recipes
- **Relevant Columns:** `n_ingredients`, `minutes`, `n_steps`, `calories`, and `avg_rating`.

---

## Data Cleaning and Exploratory Data Analysis
### Data Cleaning
To ensure the data was ready for analysis, we performed the following:
1. **Handling Missing Ratings:** We replaced ratings of `0` with `NaN`, as a zero typically represents a comment without a star rating.
2. **Merging:** We merged the raw recipes with average interaction ratings to create a single source of truth.
3. **Feature Engineering:** We extracted `calories` from the nutrition string and created an `ingred_per_step` column to measure "preparation density."

### Exploratory Data Analysis
Below is the distribution of average ratings. Note the heavy right-skew; most recipes on Food.com are highly rated.

<iframe src="assets/rating_dist.html" width="100%" height="500" frameborder="0"></iframe>

Next, we look at the relationship between the number of ingredients and preparation time. As expected, more ingredients generally lead to longer cooking times.

<iframe src="assets/ingredients_vs_time.html" width="100%" height="500" frameborder="0"></iframe>

Finally, we examine the distribution of ratings across different levels of ingredient complexity. This box plot shows that the median rating remains consistent regardless of the number of ingredients.

<iframe src="assets/complexity_vs_rating.html" width="100%" height="500" frameborder="0"></iframe>
---

## Assessment of Missingness
We believe the `rating` column may be **NMAR (Not Missing At Random)**. Users may be less likely to leave a star rating if they found the recipe so confusing they couldn't finish it, or if they only intended to leave a specific comment about a substitution.

To check for **MAR (Missing At Random)**, we ran a permutation test to see if missingness in ratings depended on the `minutes` (cooking time). 
- **Result:** [Insert your P-Value here from Step 3 code]
- **Conclusion:** If $p < 0.05$, the missingness depends on cooking time.

---

## Hypothesis Testing
**Null Hypothesis:** Simple recipes (≤10 ingredients) and complex recipes (>10 ingredients) have the same mean rating.
**Alternative Hypothesis:** Simple recipes have a different mean rating than complex recipes.

- **Test Statistic:** Difference in Means (Simple - Complex).
- **Observed Difference:** -0.0021
- **P-Value:** 0.6440
- **Conclusion:** We fail to reject the null hypothesis. There is no significant evidence that ingredient count affects the average rating.

---

## Framing a Prediction Problem
**Prediction Problem:** Predict the star rating (1-5) for a recipe.
**Type:** Multiclass Classification.
**Response Variable:** `avg_rating` (rounded to the nearest integer).
**Evaluation Metric:** **Weighted F1-Score**. This metric was chosen because of the heavy class imbalance (mostly 5-star ratings).

---

## Baseline Model
Our baseline model used a **Random Forest Classifier** with two features: `n_ingredients` and `minutes`.
- **Quantitative Features:** 2
- **Performance:** Weighted F1-Score of [Insert Baseline F1].
- **Assessment:** The model struggled due to class imbalance, often defaulting to 5-star predictions.

---

## Final Model
We improved the model by adding `calories` and `ingred_per_step` (Complexity Density). We also used `class_weight='balanced'` to force the model to pay attention to lower-rated recipes.
- **Added Features:** `calories`, `n_steps`, `ingred_per_step`.
- **Hyperparameters:** Tuned via `GridSearchCV` (Optimal depth: [Insert Max Depth]).
- **Improvement:** The model showed a [X]% increase in F1-score and improved precision for 4-star ratings.

---

## Fairness Analysis
We tested our model's fairness between **Simple Recipes** (≤10 ingredients) and **Complex Recipes** (>10 ingredients).
- **Metric:** Weighted Precision.
- **Null Hypothesis:** The model is fair; precision is equal for both groups.
- **P-Value:** 0.2640
- **Conclusion:** Since $p > 0.05$, we fail to reject the null hypothesis, suggesting the model is fair across complexity levels.

<iframe src="assets/fairness_test.html" width="100%" height="500" frameborder="0"></iframe>