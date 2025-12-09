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

## **Assessment of Missingness**

## **Hypothesis Testing**

## **Framing a Prediction Problem**

## **Baseline Model**

## **Final Model**

## **Fairness Analysis**


