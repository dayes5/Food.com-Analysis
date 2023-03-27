# Recipes and Food Calorie Prediction

## Framing the Problem

For our prediction problem, we decided to predict the number of calories for each recipe. This is a regression problem because calories is a continuous, quantitative variable. 

For our data cleaning, the process is detailed on [this website](https://nimenon24.github.io/Recipe-and-Ratings-from-Food.com-EDA/). The site also has our EDA.

Our response variable is the number of calories a recipe has. We chose calories because it has become one of the defining characteristics of a recipe or food, and we thought it would be interesting to see what other factors or nutrient qualities in food can influence calories. In addition, we have already started to explore it earlier in our EDA, and have a good familiarity of the variable. 

We chose to use R^2 as our metric, since we are using a regression model. R^2 is the proportion of the variance in the response variable that is explained by the predictor variables, and since that is between 0 and 1, we thought it was easier to understand and work with. In addition, it is not dependent on sample size or range like root mean squared error is. 

At the current time, we know the features that were given to us in the data frames that we had merged together from Food.com. Some of these features include nutrients, minutes, and number of steps. We can use some of these features in our prediction as the date for all these recipes have come before 2022. 


## Baseline Model


We decided to use a linear regression model as our baseline model. The features we chose were `carbohydrates`, `protein`, and `n_ingredients`. 

In our baseline model, we have 2 continuous quantitative variables: `protein` and `carbohydrates`. We also have 1 discrete quantitative variable, `n_ingredients`. 

When we plotted `protein` and `calories`, as well as `carbohydrates` and `calories`, we saw that the relationship seemed to appear linear and was positive. Protein and carbohydrates are also things that are typically positively correlated with calories. `n_ingredients` also made sense for us to include, since the more ingredients, the more calories there typically are.

<iframe src="assets/protein_cal.html" width=800 height=600 frameBorder=0></iframe>

<iframe src="assets/carbs_cal.html" width=800 height=600 frameBorder=0></iframe>

The performance of our model had an R^2 value of 0.8025781014688295. Based on this score, this suggests that our model is pretty “good” in predicting a linear relationship. We decided this because our score for R^2 is fairly close to 1, which is where the R^2 score tells us that it is a good linear fit. 


## Final Model

When we were building upon our baseline model, we decided to include all variables that could influence the number of calories for a food. We decided to include `sugar`, `total fat`, and `rating` (in addition to our original `carbohydrates`, `protein`, and `n_ingredients`).

We included these because we thought that they also influenced the calorie content of each recipe, in the case of `sugar` and `total fat`, or could provide us a hint in regards to how good it tasted (as in the case of `rating`). Typically, as fat and sugar increase, so does the number of calories in a recipe–they are positively correlated. We also thought that higher tastiness could mean that the recipe had a higher calorie content.

We decided to transform sugar by using the StandardScalar transformer as we wanted to make sure that the measurements were contributing equally and were on the same scale to make a fair prediction later on. 

We decided to use the QuantileTransformer on total fat as our distribution was right-skewed, and we wanted to spread out very common fat values. The QuantileTransformer would allow us to do this as it would transform the feature into a uniform distribution. 

For `rating`, we decided to use one-hot encoding because it was a categorical variable. 

We tried three different regression models: linear regression, decision tree regression, and random forest regression. For each model, we tried different combinations of our features, and passed the rest of the variables through. Then, we found the average R^2 value across all folds for each combination.

Linear regression combinations and their mean R^2 values across all folds:

| Factor Combination                          | Means    |
|:--------------------------------------------|---------:|
| quant fat + quant sugar                     | 0.839824 |
| quant fat + std sugar                       | 0.84275  |
| quant fat + std sugar + rating              | 0.84276  |
| quant fat + quant carb + std sugar + rating | 0.772188 |
| all                                         | 0.758055 |


Decision Tree combinations and their mean R^2 values across all folds:

| Factor Combination                          | Means    |
|:--------------------------------------------|---------:|
| quant fat + quant sugar                     | 0.969414 |
| quant fat + std sugar                       | 0.97254  |
| quant fat + std sugar + rating              | 0.96695  |
| quant fat + quant carb + std sugar + rating | 0.975035 |
| all                                         | 0.973825 |


Random Forest combinations and their mean R^2 values across all folds:

| Factor Combination                          | Means    |
|:--------------------------------------------|---------:|
| quant fat + quant sugar                     | 0.965452 |
| quant fat + std sugar                       | 0.967322 |
| quant fat + std sugar + rating              | 0.963469 |
| quant fat + quant carb + std sugar + rating | 0.965212 |
| all                                         | 0.966005 |

Here, “quant” refers to how we used a Quantile Transformer for those factors. “std” means that the variable was transformed using a Standard Scaler, and we one-hot encoded rating. “all” meant that we had `rating` one-hot encoded, `sugar` transformed with Standard Scaler, and `total fat`, `protein`, and `n_ingredients` transformed with Quantile Transformer.

We found the optimal combination to be using total fat (with Quantile Transformer) and sugar (with Standard Scaler). Although our Decision Tree Regressor with all variables (Standard Scaler for sugar, and Quantile Transformer for `total fat`, `protein`, and `n_ingredients`) had a higher score, we decided to go with the simpler model since the difference was quite small–only around 0.01.

After we found the optimal combination, we then used `GridSearchCV` to find the best parameters (such as maximum depth and the minimum sample size) for our Decision Tree and Random Forest Regressions. 

After we applied train test split, we found the following training and test R^2 scores for each model:

Linear Regression: 
Training: 0.8480349912087494
Testing: 0.8347615240995385

Our best Decision Tree, with `max_depth= None` and `min_samples_split = 2`: 
Training: 0.9993726305010362
Testing: 0.9667203773211627

Our best Random Forest, with parameters `max_depth = 7`, `min_samples_split = 3`, and `n_estimators = 7`: 
Training: 0.9835020497209663
Testing: 0.9791101623648127

We found that using a decision tree regression, with parameters `max_depth= None` and `min_samples_split = 2`, yielded the highest R^2 values for both our training and test dataset. 


## Fairness Analysis

We decided to explore if our model performed similarly for dessert recipes and non-dessert recipes. 

For our evaluation metric, we decide to continue to use R^2.

Our null hypothesis was:
The model's score is the same for both dessert recipes and non-dessert recipes, and any differences are due to chance.

Our alternative hypothesis was: 
The model's score is higher for dessert recipes.

Our test statistic was the difference in R^2 scores (dessert - non-dessert), and we got around 0.00832 for our observed statistic. 

Our significance level was 0.01. Our resulting p-value was very close to 0, which is significant. 

This means that our model appears to not be fair when looking at its performance between dessert recipes and non-dessert recipes; we can say with 99% confidence that our model does a better job predicting the number of calories for dessert recipes than for non-dessert recipes. 
