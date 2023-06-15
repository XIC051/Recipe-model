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

#### 1. It is well-suited for regression tasks. 

#### 2. It tells us the percentage of the variability in the calories of recipes that can be accounted for by the features used in our prediction 
model. 

#### 3. It provides a meaningful measure of how well our model fits the data and explains the variability in the calories data of recipes.

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

#### features:

##### 1. `"n_ingredients"`

It is a **discrete numerical variable**.

We used **StandardScaler()** to standardize this feature, by subtracting the mean and dividing by the standard deviation. 

##### 2. `"tags"`

It is a **nominal categorical variable** in form of a list of strings.  

We first used a **FunctionTransformer()** by using a custom function **transform_tags()** defined to extract a specific category, "main-dish," from the 'tags' column. The resulting boolean values (True or False) indicate whether each recipe is tagged as a main dish or not. 

After the custom transformation, we used **OneHotEncoder(handle_unknown='ignore')** to convert the transformed categorical variable into a one-hot encoded representation. 

##### 3. `"carbohydrates"`

It is a **discrete numerical variable**.

We used **StandardScaler()** to standardize this feature, by subtracting the mean and dividing by the standard deviation. 

### Performance of our model

R-squared on train data: 0.5689925700915398
R-squared on test data: 0.5716317971225593




