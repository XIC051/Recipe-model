Our exploratory data analysis on this dataset can be found [here](https://xic051.github.io/DSC/report.html)

# Framing the Problem

### prediction problem

For this project, we are trying to **predict the calories of recipes**.

The type of our prediction problem is **regression**.

### Response Variable
 
The response variable is **how much calories a single recipe contains**. We chose this variable because it plays a crucial role in nutritional analysis and dietary planning. Caloric intake is a fundamental aspect of maintaining a balanced and healthy diet. By predicting the calories of different recipes, we can provide people with valuable information for people to manage their calorie consumption effectively.

### Metric

R-squared (R2)

We chose R-squared because:

1. It is well-suited for regression tasks. 

2. It tells us the percentage of the variability in the calories of recipes that can be accounted for by the features used in our prediction 
model. 

3. It provides a meaningful measure of how well our model fits the data and explains the variability in the calories data of recipes.

### Justification of information known 

When training our model to predict calories, we are given all the information included in the recipes, but not ratings of each recipe yet as ratings are only given after recipes are published. Therefore, all the features we are using in our model are from information in the recipes, including `["total fat", "carbohydrates", "sugar", "n_ingredients", "steps", "tags"]`. 

### data cleaning

We have done some basic data cleaning when doing EDA, which are included [in this website](https://xic051.github.io/DSC/report.html). 
The followings are some additional data cleaning we did for the purpose of the prediction models. 

#### 1. We drop some unnecessary columns in the data frame

Columns being dropped: `["nutrition", "contributor_id", "user_id", "recipe_id", "submitted", "rating", "average rating", "review", "date"]`

#### 2. We drop some rows for some columns

- calories 

The original `merged_df` contains many receipes that have unreasonable calories, such as 45609.0. We set calories with less than 3000 as a resonable threshold. Therefore, we only want to focus on recipes with 3000 or less calories and filter out the rows with extreme calories values.

- total fat, sugar, sodium, protein, saturated fat, carrbohydrates 

Since the unit for these numiercal variables are **PDV**, which is **Percentage of Daily Value**, it is impossible for these columns to have a value that is greater than 100. So we filter out rows with a value that is greater than or equal to 100 among these columns in `merged_df`. 

# Baseline Model 

### description of our model

Our model utilizes three features, "total fat (PDV)", "published_year" and "tags", to predict the calorie content of recipes. After processing the features, where the detailed process is provided below, we built a pipeline using a linear regression model. Then the model's performance is evaluated using the R-squared metric on both the train and test data.

#### feature transforming and encoding:

##### 1. `"n_ingredients"`

It is a **discrete numerical variable**.

We used **StandardScaler()** to standardize this feature, by subtracting the mean and dividing by the standard deviation. 

##### 2. `"tags"`

It is a **nominal categorical variable** in form of a list of strings.  

We first used a **FunctionTransformer()** by using a custom function **transform_oven()** defined to tranform the `tags` column in the original data frame to a column of boolean series (True or False) checking whether the tag **"main-dish"** is included in the tags list of that recipe or not. The `tags` column now indicates whether each recipe is a main-dish or not. This transformation is based on the belief that main dishes typically have higher calorie contents compared to other types of dishes, such as appetizers or side dishes, as they often include protein-rich ingredients, larger portions, and more complex preparations.

After the custom transformation, we used **OneHotEncoder(handle_unknown='ignore')** to convert the transformed categorical variable (True or False) into a one-hot encoded representation. 

##### 3. `"carbohydrates"`

It is a **continuous numerical variable**.

We used **StandardScaler()** to standardize this feature, by subtracting the mean and dividing by the standard deviation. 

### Performance of our baseline model

R-squared on train data: 0.5689925700915398
R-squared on test data: 0.5716317971225593

Based on the performance, firstly, we can see that R-squared values for both train and test data are relatively close, indicating that our baseline model is not overfitting the training data and is performing consistently on unseen data.

Secondly, although the R-squared value isn't very close to 1, it is still a decent value, indicating that our baseline model is in the right direction for prediction and can perform better after improvement in the final model.  

# Final Model

### Features Changes

##### 1. We add a `oven` feature from transformation of column `steps`

It is a **nominal categorical variable** in the form of a list of strings. 

We first used a **FunctionTransformer()** by using a custom function **transform_oven()** defined to tranform the `steps` column in the original data frame to a column of boolean series (True or False) checking whether **"oven"** is included in the steps or not. The `steps` column now indicates whether each recipe has a step that using oven or not. 

**We used this transformation because** we believe that cooking methods that involve the use of an oven, such as baking or roasting, can lead to higher calories content as it often involves the use of added fats, oils, or high-calorie ingredients. So adding this feature allows our model to capture the potential calorie-enhancing effects of oven cooking, thus improving its ability to predict the calorie content of recipes that involve the "oven-based" cooking methods.

After the custom transformation, we used **OneHotEncoder(handle_unknown='ignore')** to convert the transformed categorical variable (True or False) into a one-hot encoded representation. 


##### 2. We add `total fat` 

It is a **continuous numerical variable**.

**We choose to add it because** we believe that fat content is a significant contributor to the overall calorie content of a recipe. Fat is known to have a higher calorie density compared to other nutrients. By incorporating this feature, the model can take into account the amount of fat present in a recipe and its potential effect on calorie estimation. 

After plotting the distribution of `total fat`, we see that the distribution is **right-skewed**, so we decided to use **QuantileTransformer(n_quantiles=100)** to spread out the data more evenly across the range while preserving the rank order of the values and reduce the impact of outliers and extreme values on our model's predictions. 

<iframe src="assets/fat.html" width=800 height=600 frameBorder=0></iframe>


##### 3. We add `sugar`

It is a **continuous numerical variable**.

**We choose to add it because we believe** that sugar is another ingredient that significantly contributes to the calorie content of recipes. It is a carbohydrate and provides a substantial amount of calories per gram. By adding it, the model can include the impact of sugar on the overall calories when making predition. Recipes with higher sugar levels are likely to have higher calorie content. 

After plotting the distribution of `sugar`, we see that the distribution is **right-skewed**, so we decided to use **QuantileTransformer(n_quantiles=100)** to spread out the data more evenly across the range while preserving the rank order of the values and reduce the impact of outliers and extreme values on our model's predictions. 

<iframe src="assets/sugar.html" width=800 height=600 frameBorder=0></iframe>

##### 4. We change (not adding new feature) how we process feature `carbohydrates` from StandardScaler() to QuantileTransformer(n_quantiles=100)

After plotting the distribution of `carbohydrates`, we see that the distribution is **right-skewed**, as the original StandardScaler() transformation assumes a normal distribution of the data, we decided to use **QuantileTransformer(n_quantiles=100)** instead which helps to spread out the data more evenly across the range while preserving the rank order of the values and reduce the impact of outliers and extreme values on our model's predictions. 

<iframe src="assets/carbohydrates.html" width=800 height=600 frameBorder=0></iframe>

**Note**

For the original feature `n_ingredients`, we continue to use **StandardScaler()** as the distribution of`n_ingredients` is a normal distribution. 

<iframe src="assets/n_ingredients.html" width=800 height=600 frameBorder=0></iframe>

Also, we keep the transformation and encoding of columns `tags` to check for "main-dish" tag from the baseline model the same in our final model. 

### Modeling Algorithm

We chose to use **RandomForestRegressor** as our final model, which is an ensemble method based on decision trees. The **hyperparameter** that ended up performing the best was 'max_depth' with a value of **10**, as determined by GridSearchCV.

To select the best hyperparameters, we performed a grid search using cross-validation with 5 folds. The range of 'max_depth' hyperparameter was set from 1 to 15, and the model was trained and evaluated on each combination of hyperparameters. The best performing hyperparameter value was selected based on the mean cross-validated score. 

The overall model consists of a preprocessing pipeline that includes feature transformations for the `tags` and `steps` columns using OneHotEncoder after applying custom functions to transform the data. For numerical features, the `total fat`, `carbohydrates`, and `sugar` columns are transformed using QuantileTransformer to handle the right-skewed distribution, while the 'n_ingredients' column is standardized using StandardScaler.


### Performance of our final model

|    | Hyperparameters              |    Score |\n|---:|:-----------------------------|---------:|\n|  0 | {'regressor__max_depth': 1}  | 0.451636 |\n|  1 | {'regressor__max_depth': 2}  | 0.6793   |\n|  2 | {'regressor__max_depth': 3}  | 0.818093 |\n|  3 | {'regressor__max_depth': 4}  | 0.878214 |\n|  4 | {'regressor__max_depth': 5}  | 0.906364 |\n|  5 | {'regressor__max_depth': 6}  | 0.923514 |\n|  6 | {'regressor__max_depth': 7}  | 0.931208 |\n|  7 | {'regressor__max_depth': 8}  | 0.935237 |\n|  8 | {'regressor__max_depth': 9}  | 0.936823 |\n|  9 | {'regressor__max_depth': 10} | 0.936893 |\n| 10 | {'regressor__max_depth': 11} | 0.936306 |\n| 11 | {'regressor__max_depth': 12} | 0.93512  |\n| 12 | {'regressor__max_depth': 13} | 0.933842 |\n| 13 | {'regressor__max_depth': 14} | 0.932574 |

Compared to the baseline model, our final model incorporates additional features, improves the preprocessing process by handling skewed data, and utilizes a more advanced algorithm to search for the best combinations of hyperparameters. The performance of the final model is improved in terms of R-squarted value from around 0.569 for training data and 0.572 for testing data, to 

# Fairness Analysis 
