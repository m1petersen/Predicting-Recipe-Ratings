# Predicting Recipe Ratings
A Machine Learning Project for DSC80 class at UCSD 

## Introduction: 
In this project we will explore a dataset of recipes.

This dataset has information on 83782 different recipes; within its 13 columns it contains nutrition information, preparation time, number of steps, the text for the steps needed to make the recipe, a description as well as tags for each recipe, and the ratings of the recipe.

The question for this dataset becomes: ** What types of recipes tend to have higher ratings? ** 
What drives ratings more? Are quick and easy recipes with high sugar and fats favored (Convenience and Comfort)? Or are healthy, more laborious recipes preferred (Quality and Effort)?

With this question we could get a better sense of what users usually prefer in their recipes, taking a look at what food components drive recipe ratings (Sugar, Carbs, Fat), if higher calories (Usually in recipes with higher Fat and Sugar content) mean higher ratings, and if simple recipes are preferred over more complex ones.

The most relevant columns from the dataset and our question are: 

**'minutes'** : Minutes to prepare recipe.

**'tags'** : Food.com tags for recipe.

**'nutrition'** : Nutrition information for the recipe, including calories, fat, protein, carbohydrates, sugar, and sodium. 

**'n_steps'** : Number of steps in recipe.

**'description'** : A user provided description of the recipe.

**'rating'** : The rating given to each recipe (Out of 5 stars)

## Data Cleaning: 
Data is messy, so as with most if not all datasets, we have to perform some cleaning of the data and colmns in order to begin our analysis. 

First we extracted each piece of information from the nutrition column and made it into its own column. This was done in order to transform what was a messy list of values into an informative collection of numeric columns describing the amount of nutrition values each recipe contained (calories, protein, fat, carbohydrates, sugar, etc).

We also did some work on the ratings column. First we replaced values of 0 with NaN's, then we calculated the average rating of each recipe and used that number instead for our analysis. This was done to ignore 0 values when calculating the mean rating and have a much cleaner and concise representation of the ratings.

Here are the first couple of rows and the most important columns our cleaned recipes dataframe:

|   minutes |   n_steps |   n_ingredients |   calories |   total_fat |   sugar |   carbohydrates |   average_rating |
|----------:|----------:|----------------:|-----------:|------------:|--------:|----------------:|-----------------:|
|        40 |        10 |               9 |      138.4 |          10 |      50 |               6 |                4 |
|        45 |        12 |              11 |      595.1 |          46 |     211 |              26 |                5 |
|        40 |         6 |               9 |      194.8 |          20 |       6 |               3 |                5 |
|       120 |         7 |               7 |      878.3 |          63 |     326 |              39 |                5 |
|        90 |        17 |              13 |      267   |          30 |      12 |               2 |                5 |

### Univariate Analysis:
<iframe
  src="assets/Ratings_Dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
The plot above shows the distribution of the Average Ratings of the recipes. From this plot we can tell that most recipes are given a 5 star rating while very few are given anything less than a 4. 

### Bivariate Analysis:
<iframe
  src="assets/calories_ratings.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
The plot above shows the distribution of ratings in relation to the amount of calories of the recipes, as we can see most recipes are rated 5 stars, which agrees with the univariate plot above this one, but we can also tell from the plot that most High Calorie recipes are rated highly. This gives us a sense of what the dataset looks like, higher calorie recipes might actually lead to higher ratings.

### Interesting Aggregates:
--- Average Rating by Calorie Content Quartile ---

| calorie_category     |   count |    mean |   median |      std |
|:---------------------|--------:|--------:|---------:|---------:|
| Q1 (Lowest Calorie)  |   20285 | 4.63705 |        5 | 0.638628 |
| Q2 (Low Calorie)     |   20297 | 4.62205 |        5 | 0.648937 |
| Q3 (High Calorie)    |   20290 | 4.61981 |        5 | 0.633557 |
| Q4 (Highest Calorie) |   20301 | 4.62255 |        5 | 0.641732 |

Here we have grouped the recipes by Calorie Content and averaged the ratings of each group. As we can see, the average rating is almost the same, no matter the amount of calories. These aggregates are not giving us much information due to rating inflation (users rarely take the time to rate a recipe they disliked), the mean of most recipes is close to 5, making mean comparisons less informative.

## Assessment of Missingness:

The average_rating column of our dataset has the most significant missingness, with 2609 missing values. This column is important for our analysis as that is the column we want to make predictions on. We have to stop and think of why this column is missing so much data.

### Is the data Not Missing at Random (NMAR)? 

I do not believe so, the most likely cause of the missingness is just that users have not interacted with those specific recipes, therefore they don't have any ratings. But one might argue that the missingness of the column is NMAR as people who would write a bad review on a recipe, probably wouldn't even take the time to do so, then the missingness of that rating would depend on the value of the rating itself.

### Tests

We will now run some tests to verify if the missingness of the column depends on antoher variable:

We ran a permutation test to verufy if the missingness of the average_rating column depended on the minutes column.

Hypothesis: Recipes that take longer might get MORE ratings (people who invest more time may be more motivated to rate)

Null Hypothesis (H₀): 
The missingness of average_rating does NOT depend on minutes. (The distribution of minutes is the same for recipes with and without ratings.)

Alternative Hypothesis (H₁): 
The missingness of average_rating DOES depend on minutes.

#### OBSERVED STATISTICS

Mean minutes (rating missing):  228.72

Mean minutes (rating present):  111.38

Absolute difference:            117.34

#### PERMUTATION TEST RESULTS

Observed difference:  117.3422

P-value:              0.0372

Significance level:   α = 0.05

#### REJECT the null hypothesis (p = 0.0372 < 0.05)
The missingness of average_rating DEPENDS on minutes.

Recipes with missing ratings have significantly different prep times.

This suggests MAR (Missing At Random) - the missingness is related to an observable feature (preparation time).

Here is a plot that visualizes our test and the distribution of the minutes column both when average_rating is present and when it is missing.

<iframe
  src="assets/missingness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


## Hypothesis Testing 
We have 2 main hypothesis that we want to test with our data:

### Hypothesis 1: Convenience vs Success
We compare the success rate (recipes rated >= 4.5) of the "Quick" recipe category (<30 minutes) against the "Long" recipe category (> 60 minutes).

#### Null Hypothesis: 
The success rate of quick recipes is equal to the success rate of long recipes.

#### Alternative Hypothesis: 
The success rate of quick recipes is greater than the success rate of long recipes.

#### Test Statistic and Significance Level:
Two-sample Z-statistic for Proportions, with a significance level of 0.05.

#### p-value
p-value = 1.7117854451669185e-12 much smaller than 0.05.

#### Conclusion
This extremely small p-value provides statistical evidence to **reject the null hypothesis**.

The data supports the alternate hypothesis, 
recipes requiring quick preparation (<= 30 minutes) have a significantly higher success rate (>=4.5 stars) 
than those requiring long preparation (>60 minutes).

### Hypothesis 2: Indulgence vs. Success
We compare the success rate of the highest calorie quartile (Q4) against the lowest calorie quartile (Q1).

#### Null Hypothesis: 
The success rate of highest calorie recipes (Q4) is equal to the success rate of lowest calorie recipes (Q1).

#### Alternate Hypothesis:
The success rate of highest calorie recipes (Q4) is greater than the success rate of lowest calorie recipes (Q1).

#### Test Statistic and Significance Level:
Two-sample Z-statistic for Proportions, with a significance level of 0.05.

#### p-value
p-value = 0.9999044824904763, much larger than 0.05.

#### Conclusion
In this case, since the p-value is large, we **Fail to Reject the Null Hypothesis**.

There is no statistical evidence to support the claim that the success rate of the highest calorie recipes is greater than the lowest calorie recipes.

## Framing a Prediction Problem
The goal of this project is to predict whether a newly submitted recipe will be highly rated by users. This prediction is made using only the intrinsic characteristics of the recipe, such as its ingredients, preparation time, and nutritional content, which are known at the moment the recipe is submitted.

#### Prediction Problem Type: Binary Classification.

#### Response Variable: The variable being predicted is is_top_rated.
This is a binary variable defined as:

1 (Success): If the recipe's average_rating is greater than or equal to 4.5 stars (out of 5).

0 (Failure): If the recipe's average_rating is less than 4.5 stars.

#### Justification for Response Variable:
While the raw data contains a continuous average_rating, the analytical goal is to identify recipes that are "successful" or "top-performing". Converting the continuous rating into a binary target with a cutoff of 4.5 allows the model to focus specifically on the factors that drive exceptional user satisfaction.

#### Model Evaluation Metric:
The primary metric used to evaluate the model's performance is the Area Under the Receiver Operating Characteristic Curve (ROC AUC).

#### Justification for ROC AUC:
Class Imbalance: 

In real-world data science problems, the number of successful (top-rated) recipes is often much smaller than the number of average or low-rated recipes (class imbalance). Accuracy can be misleading in such cases; a model that simply predicts the majority class can still achieve high accuracy. ROC AUC provides a measure of performance that is less sensitive to class imbalance.

#### Information Known at the Time of Prediction:
To ensure the model is practical for predicting the success of a new recipe **before** it is rated by users, the model is only trained on features available at the moment of submission.

The features used are all intrinsic properties of the recipe description and instructions.

The only invalid feature would be the average rating itself, as this is what we are trying to predict and a new recipe would not have a rating.

## Baseline Model:
For the Baseline Model, we will be trying to predict if a recipe will be highly rated (>=4.5 stars) using the 'is_top_rated' column.

#### Features used:
minutes (Numerical): The preparation time, which showed statistical significance in the hypothesis test.

time_category (Ordinal): A feature derived from minutes that simplifies the convenience factor into discrete groups.

#### Necessary Encodings:
The target variable needed to be encoded, we binarized the average_rating data by creating a new column (is_top_rated). 
This new column contained a 1 if the recipe had 4.5 stars or higher and a 0 otherwise.

The time_categoy variable also needed to be encoded, we separated the minutes variable into 3 categories, 'Quick' if the recipe took <= 30 minutes,
'Moderate' if it took between 30 and 60 minutes, and 'Long' if it took more than an hour.

#### Model Performance:
From the results of the baseline model performance:

Accuracy was 0.7507, meaning that the model correctly classified 75.07%of the recipes in the unseen test set.

**BUT** ROC AUC Score was only 0.5190. The model's ability to distinguish between the positive class (top-rated, 1) and the negative class (not top-rated, 0) is only slightly better than random guessing (AUC = 0.5).

In this case, the seemingly high accuracy is misleading, it is likely due to imbalance in the target variable. Since most recipes are top-rated (i.e., most values are 5), a model that simply predicts 5 stars for everything would still achieve an accuracy close to the proportion of top-rated recipes (likely around 75%).

The low ROC AUC is very informative in this case, A perfect model would score 1.0, and a model that performs no better than random chance scores 0.5.

Given that our model has a ROC AUC of only 0.519, we know that despite the seemingly high accuracy, it actually has little to no predictive power for distinguishing which recipes will truly be top-rated.

## Final Model:
