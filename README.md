# 🍽️ Recipe Time Predictor
### Author: Nainika Neerukonda

---

## Introduction

This project analyzes recipe data from Food.com, a large dataset containing over **83,000 recipes** and their associated metadata. Each recipe includes details such as ingredients, number of steps, nutritional information, tags, user ratings, and more. The dataset provides a rich opportunity to study how recipe characteristics relate to cooking behavior and user experience.

### 🔍 Project Question  
The central question of this project is:

**“Can we predict how long a recipe will take to prepare based on its ingredients, steps, and nutritional attributes?”**

Preparation time is one of the most important factors for home cooks when deciding whether they can make a dish. Understanding which recipe characteristics drive cooking time can help users:

- Choose recipes that match their available time  
- Estimate meal-planning needs more accurately  
- Understand which aspects of a recipe make it more time-consuming  

### 📊 Dataset Overview  
After cleaning, the dataset contains **83,782 rows and 21 columns**.  
The columns most relevant to our prediction problem are:

| Column | Description |
|--------|-------------|
| **minutes** | Total time required to prepare the recipe (target variable). |
| **n_steps** | Number of steps in the recipe’s instructions; reflects complexity. |
| **n_ingredients** | Number of unique ingredients; measures recipe size. |
| **description** | Optional text written by the recipe author; often missing. |
| **tags** | Categorical list describing properties (e.g., “quick”, “dessert”). |
| **calories, sugar, protein, sodium, fat** | Nutritional attributes related to effort and cooking method. |
| **step_bin** | A derived categorical version of step count (e.g., “0–5”, “6–10”). |
| **desc_missing** | Indicator of whether the description is missing. |

These columns provide a mix of quantitative and categorical information that can be used to model cooking time.

---

## Why This Question Matters

Cooking time is central to food preparation. Many people filter recipes by how long they take, yet actual time often varies from expectations. If we can **predict preparation time accurately from recipe structure**, we can make recipe platforms more informative and help cooks plan meals more effectively.

This project combines statistical inference, missingness analysis, and predictive modeling to answer this question using real-world recipe data.


## Data Cleaning and Exploratory Data Analysis

## Assessment of Missingness

## Hypothesis Testing

## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis


