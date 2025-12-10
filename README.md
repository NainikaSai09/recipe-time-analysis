# 🍽️ Recipe Time Predictor
### Author: Nainika Neerukonda

---

## **Introduction**

This project uses the **Food.com Recipes + Interactions** dataset, consisting of two large tables that together describe over 80,000 recipes and more than 700,000 user interactions.

### Dataset Size 

| Dataset        | Rows    | Columns |
|----------------|---------|---------|
| **Recipes**    | 83,782  | 12      |
| **Interactions** | 731,927 | 5       |

### Central Question

> **What factors help us predict how long a recipe will take to prepare (in minutes)?**

For everyday cooking, time is one of the most important constraints. Accurately estimating preparation time helps users:

- choose recipes that fit their schedule  
- avoid overly time-consuming dishes  
- plan meals efficiently  

The Food.com dataset is uniquely valuable because it combines detailed recipe structure (ingredients, steps, nutrition) with real user behavior (ratings and reviews) across tens of thousands of recipes.

This makes `minutes` an essential and practical prediction target.

---

### Dataset Description

#### Recipes Table

| Column | Description |
|--------|-------------|
| **`name`** | Recipe name |
| **`id`** | Recipe ID |
| **`minutes`** | minutes required to prepare the recipe |
| **`contributor_id`** | ID of the user who submitted the recipe |
| **`submitted`** | Date the recipe was submitted |
| **`tags`** | Food.com tags assigned to the recipe |
| **`nutrition`** | Nutrition information in the form:<br> `[calories, total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]`, `PDV stands for "percentage of daily value` |
| **`n_steps`** | Number of steps in recipe|
| **`steps`** | List of step-by-step instructions |
| **`description`** | User-provided recipe description |
| **`n_ingredients`** | Number of ingredients (computed during cleaning) |
| **`average_rating`** | Mean rating aggregated from the interactions table |

---

#### Interactions Table

| Column | Description |
|--------|-------------|
| **`user_id`** | ID of the user leaving a rating or review |
| **`recipe_id`** | ID of the recipe being reviewed |
| **`date`** | Date of the interaction |
| **`rating`** | Rating given (1–5; 0 indicates no rating provided) |
| **`review`** | Text of the user's review |

---

### Relevant Columns for Prediction

For predicting preparation time (`minutes`), the most relevant variables include:

- **`n_steps`** : procedural complexity  
- **`n_ingredients`** : number of components required  
- **`nutritional features`** : calorie density, sugar, protein, etc.  
- **`step_bin`** : binned complexity level  
- **`engineered features`** : such as calories-per-ingredient and steps-per-ingredient  

These features describe recipe structure and complexity, which logically influence preparation time.


## **Data Cleaning and Exploratory Data Analysis**

### Data Cleaning

To make the recipes dataset usable for analysis and modeling, I performed several cleaning steps that reflect how the data were generated on Food.com:

1. **Merging recipes with user interactions**

   The raw data come in two files: `RAW_recipes.csv` (recipe metadata) and `RAW_interactions.csv` (user ratings and reviews). On the website, each recipe can have many user interactions, but some recipes may have none. To mirror this structure, I performed a **left merge** of recipes with interactions on `id` (recipe ID) and `recipe_id`. A left merge preserves all recipes (even those without ratings), which is important because users can still view and cook unrated recipes.

2. **Treating 0-star ratings as missing (NaN)**

   On Food.com, users cannot actually give a 0-star rating. In practice, a `rating` of 0 in the interactions file indicates that the user left a review without selecting a star value. Because a 0 here does **not** represent a true rating on the same scale as 1–5, I replaced all `rating = 0` values with `NaN`. This avoids artificially pulling down average ratings and better reflects the true data-generating process: “no rating given” rather than “worst possible rating.”

3. **Computing average rating per recipe**

   After cleaning the ratings, I aggregated the interactions by recipe and computed an **average rating** for each `id`. I then merged this average back into the recipes table as a new column, `average_rating`. This creates a single, recipe-level dataset that summarizes user feedback and can be used directly in plots, hypothesis tests, and prediction models.

4. **Parsing the `nutrition` column into numeric features**

   The `nutrition` column originally stores a **string representation of a list** containing seven values:
   calories, total fat (%DV), sugar (%DV), sodium (%DV), protein (%DV), saturated fat (%DV), and carbohydrates (%DV).  
   Since these are meaningful numeric attributes that users see on recipe pages, I:
   - Converted each string to a real Python list,  
   - Split the list into seven separate numeric columns (`calories`, `total_fat`, `sugar`, `sodium`, `protein`, `saturated_fat`, `carbohydrates`), and  
   - Dropped the original `nutrition` string column.  
   This makes nutritional information much easier to analyze and include as features in models.

5. **Removing extreme outliers in `minutes`**

   The `minutes` column contains a few unrealistically large values (e.g., recipes recorded as taking tens of thousands of minutes). These are likely data entry errors or artifacts of scraping, and they strongly distort visualizations and inflate error metrics like RMSE. To obtain a more realistic and stable dataset, I restricted the data to recipes with preparation times between the **5th and 95th percentiles** of `minutes`. This keeps the vast majority of plausible cooking times while trimming extreme outliers that do not reflect typical user experience.


The cleaned dataframe ended up with 83782 rows and 19 columns. Because the full dataset contains many columns, I display only the most relevant ones for readability.Below is the head of the cleaned `recipes` DataFrame used for the rest of the analysis (scroll right to view more columns):  

|   minutes |   n_steps |   n_ingredients |   average_rating | description                                                                                                                                                                                                                                                                                                                                                                       |   calories |   sugar |   protein |
|----------:|----------:|----------------:|-----------------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------:|--------:|----------:|
|        40 |        10 |               9 |                4 | these are the most; chocolatey, moist, rich, dense, fudgy, delicious brownies that you'll ever make.....sereiously! there's no doubt that these will be your fav brownies ever for you can add things to them or make them plain.....either way they're pure heaven!                                                                                                              |      138.4 |      50 |         3 |
|        45 |        12 |              11 |                5 | this is the recipe that we use at my school cafeteria for chocolate chip cookies. they must be the best chocolate chip cookies i have ever had! if you don't have margarine or don't like it, then just use butter (softened) instead.                                                                                                                                            |      595.1 |     211 |        13 |
|        40 |         6 |               9 |                5 | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008 |      194.8 |       6 |        22 |
|       120 |         7 |               7 |                5 | why a millionaire pound cake?  because it's super rich!  this scrumptious cake is the pride of an elderly belle from jackson, mississippi.  the recipe comes from "the glory of southern cooking" by james villas.                                                                                                                                                                |      878.3 |     326 |        20 |
|        90 |        17 |              13 |                5 | ready, set, cook! special edition contest entry: a mediterranean flavor inspired meatloaf dish. featuring: simply potatoes - shredded hash browns, egg, bacon, spinach, red bell pepper, and goat cheese.                                                                                                                                                                         |      267   |      12 |        29 |


### Univariate Analysis

To better understand the structure of the dataset and the behavior of key variables, I first examined the distributions of several columns that are relevant to predicting preparation time. Since my prediction problem centers around the minutes variable, understanding its distribution is essential before building any models.

The distribution of preparation times is highly right-skewed. Most recipes take under 150 minutes, while a small number take significantly longer, creating a long tail. These extreme values motivated the removal of unrealistic outliers during data cleaning.

<iframe 
    src="assets/minutes_dist.html"
    width="800"
    height="600"
    frameborder="0">
</iframe>


#### Interpretation: 
The strong right skew indicates that a large portion of recipes are quick to prepare, while only a few require exceptionally long cooking times.

### Bivariate Analysis

To explore how different recipe attributes relate to preparation time, I examined several pairwise relationships using scatter plots and trendlines. Understanding these associations helps identify meaningful predictors for modeling.

One of the relationships was the one between number of steps and preparation time. This relationship is one of the most important for the prediction problem, since the number of steps is a natural proxy for procedural complexity.

<iframe 
  src="assets/steps_minutes.html" 
  width="800" 
  height="600" 
  frameborder="0">
</iframe>

#### Interpretation: 

This plot reveals a weak but noticeable positive relationship between the number of steps and preparation time. Recipes with more steps tend to require longer cooking times, but the association is highly variable, even recipes with few steps can take anywhere from a few minutes to several hours. The log scale shows that prep times span multiple orders of magnitude, highlighting the complexity and diversity of recipe structures.

### Interesting Aggregates

To better understand patterns in recipe complexity and preparation time, I computed a grouped table summarizing average prep time and number of ingredients across different step ranges. 

The table below summarizes three aggregates for each step range:

| step_bin   |   minutes |
|:-----------|----------:|
| 0–5        |   30.574  |
| 6–10       |   43.2888 |
| 11–20      |   55.3458 |
| 21–40      |   75.968  |
| 41–80      |  102.516  |
| 80+        |  131      |

More complex recipes (those with many steps) tend to require more ingredients and significantly longer preparation times. However, the increase in prep time is not linear, some moderately-step-heavy recipes still have relatively short prep times, suggesting that factors beyond step count influence total cooking time. These aggregates helped guide feature engineering decisions in later modeling steps.

## **Assessment of Missingness**

### NMAR Analysis
I believe the **`description`** column is **NMAR (Not Missing At Random)**. Whether a contributor writes a description likely depends on unobserved, person-specific factors such as their motivation, writing habits, or how strongly they feel about the recipe. These factors are **not recorded** anywhere else in the dataset, so the probability that `description` is missing cannot be fully explained using observed variables like `n_steps`, `minutes`, or `average_rating`.

Because this missingness mechanism depends on unobserved contributor traits, the missingness in `description` is best characterized as **NMAR**. To potentially treat it as **MAR**, we would need additional data about user behavior, for example, per-user history of how often they leave descriptions, time spent on the site, or whether they usually post detailed content. With such behavioral features, we might be able to model the missingness using observed information instead of unobserved preferences.

### Missingness Dependency

To understand whether the missingness in the `description` column depends on observable recipe characteristics, I performed **two permutation tests**:

1. Does missingness depend on **`n_steps`** (recipe complexity)?
2. Does missingness depend on **`calories`** (nutritional content)?

For both tests, the test statistic was the **absolute difference in means** between:
- recipes **with** missing descriptions, and  
- recipes **without** missing descriptions.

#### 1. Does missingness depend on `n_steps`?

**Hypotheses**

- **H₀:** Missingness of `description` does **not** depend on `n_steps`.  
- **H₁:** Missingness of `description` **does** depend on `n_steps`.

**Observed statistic:** ≈ **0.995**  
**Permutation test p-value:** **0.201**

This p-value is larger than 0.05, meaning the observed difference in average step count is **not statistically significant**. Although recipes with missing descriptions appear *slightly* more complex, this difference is consistent with what could arise by random chance.

**Conclusion:**  
There is **no evidence** that `n_steps` influences whether a description is missing.
  

<iframe
  src="assets/missing.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Recipes with missing descriptions have slightly more steps on average, but the distributions overlap heavily, suggesting no meaningful difference.

#### 2. Does missingness depend on `calories`?

**Hypotheses**

- **H₀:** Missingness of `description` does **not** depend on `calories`.  
- **H₁:** Missingness of `description` **does** depend on `calories`.

**Observed statistic:** ≈ **75.993**  
**Permutation test p-value:** **0.248**

This p-value is well above 0.05, meaning the difference in mean calories between missing and non-missing descriptions is not statistically significant.
Calorie content appears unrelated to whether someone writes a description.

**Conclusion:**  
There is no evidence that recipe calories influence missingness of the description field.
 

<iframe
  src="assets/absdiff_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The observed difference in mean steps lies well within the null distribution, indicating that the small difference could easily occur by random chance.

## **Hypothesis Testing**

To investigate whether recipe complexity affects preparation time, I tested whether high-step recipes take longer, on average, than low-step recipes.

#### Null Hypothesis (H₀):

There is no difference in average preparation time between high-step and low-step recipes.
Any observed difference is due to random chance.

#### Alternative Hypothesis (H₁):

High-step recipes have a higher mean preparation time than low-step recipes.

#### Test Statistic

Using the difference in mean minutes between the two groups:

𝑇 = mean(minutes for high-step recipes) − mean(minutes for low-step recipes)

A one-sided permutation test was used because:

* It makes no assumptions about the data distribution 

* Shuffling the group labels simulates a world where step count does not matter, directly aligning with our null hypothesis.

#### Significance level:
0.05

These choices fit the question because the mean-difference statistic directly measures whether high-step recipes take longer, and a permutation test is robust to the skewed minutes distribution. A one-sided test matches the directional hypothesis, and α = 0.05 is a standard threshold for evaluating evidence.

#### Results

Observed statistic: ≈ 19.30 minutes

Permutation test p-value: 0.000 (≈ 0 when rounded)

#### Interpretation

The observed difference is far larger than nearly all differences generated under the null distribution.
Because the p-value is less than 0.05, we reject the null hypothesis.

However, this does not prove that high-step recipes always require more time.
Rather, the evidence suggests that recipes with more steps tend to take longer to prepare, beyond what could be explained by random variation in this dataset.

<iframe   
  src="assets/diff_mean_mins.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

#### Interpretation:
The observed difference in mean preparation time (red line) is far outside the range of values generated under the null distribution, indicating that the difference between high-step and low-step recipes is much larger than what would be expected by chance.


## **Framing a Prediction Problem**

### Prediction Problem

The goal of this project is to predict the total preparation time (in minutes) for a recipe before it is cooked. Because the target variable, minutes, is continuous, this is a **regression** problem.

### Response Variable
minutes: total preparation time
I chose this variable because:

* It is a practical and meaningful quantity that affects how users choose recipes.

* My earlier EDA showed connections between recipe complexity (steps, ingredients) and preparation time.

* Predicting prep time is useful for real-world decision-making (e.g., “Do I have time to cook this?”).

### Features Available at Time of Prediction

To keep the prediction process realistic, I only include information that a user would already know when viewing a recipe, such as:

#### Recipe structure

* n_steps

* n_ingredients

* steps_per_ingredient (engineered)

* step_bin

#### Ingredients & nutrition

* calories, protein, sugar, etc.

* calories_per_ingredient (engineered)

#### Excluded features

* Ratings and reviews (occur after cooking)

* Anything derived from user interaction history

This ensures temporal validity: the model does not use information that would not exist yet at prediction time.

#### Evaluation Metric
**Primary Metric**: RMSE (Root Mean Squared Error) was chosen because: It penalizes large errors more heavily, which is important since misestimating a recipe by a large amount (e.g., predicting 30 minutes when it really takes 120) is worse than being slightly off.

Preparation times exhibit skew and outliers, and RMSE helps highlight models that manage extreme cases better.

**Secondary Metric** : MAE (Mean Absolute Error) is included for interpretability, it reflects the typical deviation between predicted and actual minutes.

Accuracy, F1-score, and other classification metrics are inappropriate because this is not a classification task.


This prediction problem aligns with the theme of the project:
**understanding what features influence recipe complexity and prep time**

My EDA showed that steps, ingredients, and nutritional values relate to prep time. The modeling pipeline naturally extends those findings by quantifying their predictive power. The use of time-valid features ensures the model could be deployed realistically on a recipe website.

## **Baseline Model**

#### Model Description

For my baseline model, I built a Linear Regression model to predict recipe preparation time (minutes) using the simplest meaningful set of features.

#### Features Used

I selected three interpretable features based on earlier EDA:

* n_steps: quantitative, number of procedural steps in the recipe
* n_ingredients: quantitative, number of ingredients used
* step_bin: nominal categorical, binned version of n_steps (e.g., "0-5", "6-10", etc)

Quantitative features: 2, Nominal features: 1

#### Preprocessing and Encoding

All preprocessing steps were implemented in a single sklearn Pipeline, as required:
Numerical features (n_steps, n_ingredients) were passed through unchanged (passthrough), since linear regression handles them naturally. Categorical feature (step_bin) was encoded with OneHotEncoder (handle_unknown="ignore"), producing binary indicator columns for each step range.
This ensures the model appropriately incorporates categorical information without imposing order where none exists.

#### Train/Test Split

To evaluate generalization performance:

* 80% of data was used for training

* 20% was held out as a test set

#### Baseline Model Performance

Using the cleaned dataset after removing outliers, the baseline model achieved:

* RMSE	37.41 minutes
* MAE	24.93 minutes

MAE indicates that on average, the model is off by about 25 minutes, while the RMSE, which penalizes larger errors, remains relatively high due to residual variation in prep times.


While the baseline captures some linear relationship between recipe complexity and prep time, it cannot model nonlinear relationships, which are likely present in cooking times. Variation in preparation time is still large, even within the same number of steps or ingredients. The substantial room for improvement motivates the development of a richer final model.

## **Final Model**

#### Feature Engineering

To improve predictive performance beyond the baseline model, I created two additional features based on the data-generating process of recipe preparation:

#### 1. `calories_per_ingredient`

**calories per ingredient = calories \ n_ingredients**


This measures how calorie-dense a recipe is relative to its size. Rich, dense recipes often require more involved cooking processes (e.g., baking or simmering), which can increase preparation time.

#### 2. `steps_per_ingredient`

**steps per ingredient = n_steps \ n_ingredients**


This captures procedural complexity. Recipes with many steps per ingredient often involve more precise or time-intensive techniques.

#### Other Features Included

| Feature | Type | Reason for Inclusion |
|---------|------|----------------------|
| `n_steps` | Quantitative | Reflects procedural depth |
| `n_ingredients` | Quantitative | Structural complexity |
| `calories`, `protein`, `sugar` | Quantitative | Relate to recipe richness and cooking methods |
| `calories_per_ingredient`, `steps_per_ingredient` | Engineered quantitative | Capture density & complexity |
| `step_bin` | Nominal | Allows nonlinear category differences |

These features together reflect real mechanisms that influence preparation time.


#### Modeling Algorithm

I selected **GradientBoostingRegressor** as the final model because it:

- captures nonlinear relationships  
- handles skewed numeric targets (like recipe time)  
- models interactions between features  
- generally outperforms linear models on structured tabular data  

This makes it well-suited for predicting preparation time.

#### Hyperparameter Tuning

I used **GridSearchCV with 3-fold cross-validation** to select hyperparameters.  
The grid included:

| Hyperparameter | Values Tested |
|----------------|----------------|
| `n_estimators` | [200, 300] |
| `learning_rate` | [0.05, 0.1] |
| `max_depth` | [2, 3, 4] |

#### Best Hyperparameters Found

{'model__learning_rate': 0.05,
 'model__max_depth': 4,
 'model__n_estimators': 200}

 
#### Final Model Performance


| Model | RMSE | MAE |
|---------|------|----------------------|
| Baseline (Linear Regression) | 37.41 | 24.93 |
| Final Model (Gradient Boosting) | 36.89 | 24.38 |

#### Interpretation

* RMSE improved, meaning the final model makes fewer large errors.
* MAE improved, showing typical errors are smaller.
* Improvements are modest but meaningful, given the natural variability in recipe preparation times.

Feature engineering captured more of the real cooking process, gradient boosting models nonlinear patterns that linear regression cannot, and cross-validated hyperparameter tuning ensures better generalization.

Overall, the final model aligns more closely with how recipes are actually prepared and shows measurable improvement over the baseline model.

## **Fairness Analysis**

For my fairness analysis, I compared model performance between **simple recipes** (those with ≤ the median number of ingredients) and **complex recipes** (those with > the median). I chose ingredient count because it meaningfully reflects recipe complexity and could influence how difficult prep-time prediction is. I evaluated fairness using **RMSE**, since my task is regression and RMSE appropriately captures prediction error magnitude.

**Null Hypothesis:** The model is fair. Its RMSE for simple and complex recipes is roughly the same, and any observed difference is due to random chance.  
**Alternative Hypothesis:** The model is unfair. Its RMSE for complex recipes is higher than its RMSE for simple recipes.  

#### Test Statistic:  

**RMSE_complex - RMSE_simple**


#### Significance Level: 0.05

The observed RMSE difference was **≈ 1.51**. I then ran a permutation test with 1000 shuffles of the group labels to simulate the distribution of RMSE differences under the null. The resulting **p-value was 0.036**.

Since 0.036 is slightly below 0.05, this suggests some evidence of a difference in performance. However, the effect size (about 1.5 minutes) is extremely small relative to recipe durations.

#### Conclusion: Although the permutation test yields a marginally significant result, the difference in RMSE is practically negligible. There is **no meaningful evidence of unfairness** in how the model predicts prep time for simple vs. complex recipes.

<iframe
  src="assets/rmse_diff.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

