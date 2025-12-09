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

   The raw data come in two files: `RAW_recipes.csv` (recipe metadata) and `RAW_interactions.csv` (user ratings and reviews).  
   On the website, each recipe can have many user interactions, but some recipes may have none.  
   To mirror this structure, I performed a **left merge** of recipes with interactions on `id` (recipe ID) and `recipe_id`.  
   A left merge preserves all recipes (even those without ratings), which is important because users can still view and cook unrated recipes.

2. **Treating 0-star ratings as missing (NaN)**

   On Food.com, users cannot actually give a 0-star rating. In practice, a `rating` of 0 in the interactions file indicates that the user left a review without selecting a star value.  
   Because a 0 here does **not** represent a true rating on the same scale as 1–5, I replaced all `rating = 0` values with `NaN`.  
   This avoids artificially pulling down average ratings and better reflects the true data-generating process: “no rating given” rather than “worst possible rating.”

3. **Computing average rating per recipe**

   After cleaning the ratings, I aggregated the interactions by recipe and computed an **average rating** for each `id`.  
   I then merged this average back into the recipes table as a new column, `average_rating`.  
   This creates a single, recipe-level dataset that summarizes user feedback and can be used directly in plots, hypothesis tests, and prediction models.

4. **Parsing the `nutrition` column into numeric features**

   The `nutrition` column originally stores a **string representation of a list** containing seven values:
   calories, total fat (%DV), sugar (%DV), sodium (%DV), protein (%DV), saturated fat (%DV), and carbohydrates (%DV).  
   Since these are meaningful numeric attributes that users see on recipe pages, I:
   - Converted each string to a real Python list,  
   - Split the list into seven separate numeric columns (`calories`, `total_fat`, `sugar`, `sodium`, `protein`, `saturated_fat`, `carbohydrates`), and  
   - Dropped the original `nutrition` string column.  
   This makes nutritional information much easier to analyze and include as features in models.

5. **Removing extreme outliers in `minutes`**

   The `minutes` column contains a few unrealistically large values (e.g., recipes recorded as taking tens of thousands of minutes).  
   These are likely data entry errors or artifacts of scraping, and they strongly distort visualizations and inflate error metrics like RMSE.  
   To obtain a more realistic and stable dataset, I restricted the data to recipes with preparation times between the **5th and 95th percentiles** of `minutes`.  
   This keeps the vast majority of plausible cooking times while trimming extreme outliers that do not reflect typical user experience.

Overall, these steps produce a cleaner, more interpretable recipe-level dataset that better reflects how the data arise on Food.com and supports meaningful exploratory analysis and modeling.


The cleaned dataframe ended up with 83782 rows and 19 columns. Because the full dataset contains many columns, I display only the most relevant ones for readability.Below is the head of the cleaned `recipes` DataFrame used for the rest of the analysis (scroll right to view more columns):  

|   minutes |   n_steps |   n_ingredients |   average_rating | description                                                                                                                                                                                                                                                                                                                                                                       |   calories |   sugar |   protein |
|----------:|----------:|----------------:|-----------------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------:|--------:|----------:|
|        40 |        10 |               9 |                4 | these are the most; chocolatey, moist, rich, dense, fudgy, delicious brownies that you'll ever make.....sereiously! there's no doubt that these will be your fav brownies ever for you can add things to them or make them plain.....either way they're pure heaven!                                                                                                              |      138.4 |      50 |         3 |
|        45 |        12 |              11 |                5 | this is the recipe that we use at my school cafeteria for chocolate chip cookies. they must be the best chocolate chip cookies i have ever had! if you don't have margarine or don't like it, then just use butter (softened) instead.                                                                                                                                            |      595.1 |     211 |        13 |
|        40 |         6 |               9 |                5 | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008 |      194.8 |       6 |        22 |
|       120 |         7 |               7 |                5 | why a millionaire pound cake?  because it's super rich!  this scrumptious cake is the pride of an elderly belle from jackson, mississippi.  the recipe comes from "the glory of southern cooking" by james villas.                                                                                                                                                                |      878.3 |     326 |        20 |
|        90 |        17 |              13 |                5 | ready, set, cook! special edition contest entry: a mediterranean flavor inspired meatloaf dish. featuring: simply potatoes - shredded hash browns, egg, bacon, spinach, red bell pepper, and goat cheese.                                                                                                                                                                         |      267   |      12 |        29 |


### Univariate Analysis


### Bivariate Analysis


### Interesting Aggregates



<iframe 
    src="assets/calories_dist.html"
    width="800"
    height="600"
    frameborder="0">
</iframe>


## **Assessment of Missingness**

## **Hypothesis Testing**

## **Framing a Prediction Problem**

## **Baseline Model**

## **Final Model**

## **Fairness Analysis**


