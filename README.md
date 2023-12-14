# recipe-meal-type-prediction
**Author**: Cici Xu 

## Project Overview 
This data science project investigates different parts of a recipe and its cooking time, aiming to construct a classifer model that predicts if a recipe type is a daily meal, event meal, or big meal. The recipe type is categorized with respect to cooking time.  This is a course project for DSC80- Practice and Application of Data Science at UCSD. 

My exploratory data analysis on this dataset can be found [here](https://ciciaa777.github.io/recipe-dataset-investigation/).


## Datasets in the study 
The main dataset used (Recipes) contains information on 83,782 recipes from 2008 to 2018 on food.com, with 10 columns as shown in the table below.

| Column          | Description                                     |
| --------------- | ----------------------------------------------- |
| name            | Recipe name                                     |
| id              | Recipe ID                                       |
| minutes         | Minutes to prepare recipe                       |
| contributor_id  | User ID who submitted this recipe               |
| submitted       | Date recipe was submitted                       |
| tags            | Food.com tags for recipe                        |
| nutrition       | Nutrition information in the form [calories, total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for "percentage of daily value" |
| n_steps         | Number of steps in recipe                       |
| steps           | Text for recipe steps, in order                 |
| description     | User-provided description                        |


Another dataset (Ratings) which contains people's ratings and comments on recipes is also used. This dataset has a total of 731,927 reviews, with 5 columns as listed below:

| Column      | Description             |
| ----------- | ----------------------- |
| user_id     | User ID                 |
| recipe_id   | Recipe ID               |
| date        | Date of interaction     |
| rating      | Rating given            |
| review      | Review text             |


## Data Cleaning 

1\. Convert columns in Recipes dataset into correct types: 'id' column is converted from integer to string, and 'date' column is converted from string to datetime object. 

2\. Data cleaning with Nutrition column. Split the nutrition column so that each nutrition has its own column, and change calories column data type to float. 

3\. Create a "year" and a "months" column.

4\. Create a 'holiday' column based on 'tags' column. If 'tags' column contains 'christmas'or'thanksgiving' or 'holiday', then the value of 'holiday' column is 'holiday', otherwise value is'not_holiday'. 

5\.create a 'level' column based on if 'tags' column contains 'easy'.Similarly, if 'tags' column contains 'easy', then the value of 'easiness' column is 'easy', otherwise value is'not_easy'.  

8\. retrieve needed columns 'meal_type','year','holiday','easiness',
'n_steps','n_ingredients','calories','months' and put them into a smaller dataframe df.

First five rows of the dataframe df after cleaning are displayed below. 

| meal_type   |   year | holiday     | easiness   |   n_steps | n_ingredients |   calories |   months |
|:------------|-------:|:------------|:-----------|----------:|----------------:|-----------:|---------:|
| daily meal  |   2008 | not_holiday | not_easy   |        10 |               9 |      138.4 |       10 |
| daily meal  |   2011 | not_holiday | not_easy   |        12 |              11 |      595.1 |        4 |
| daily meal  |   2008 | not_holiday | easy       |         6 |               9 |      194.8 |        5 |
| event meal  |   2008 | holiday     | not_easy   |         7 |               7 |      878.3 |        2 |
| daily meal  |   2012 | not_holiday | not_easy   |        17 |              13 |      267   |        3 |


## Framing the Problem 

**Investigation Problem**: This project aims to predict if a recipe type is a daily meal, event meal, or big meal by conducting a multiclass classification. The classification is based on time consumption level of a recipe, i.e. how long it takes to finish the recipe. 

**Response Variable**: Meal type column that indicates the time consumption level of a recipe. This column is created by setting threhold to the 'minutes' column in the dataframe.
Daily Meal - cooking time lower or equal to 1.5 hrs
Event Meal - cooking time between 1.5hrs to 4hrs
Big Meal - cooking time that takes more than 4hrs

Meal type is chosen as the response variable because it's a very useful and esasy category that allows people to look for recipes based on their needs. 

**Measureing Metrics**: The performance metric chosen for this classification model is micro f1-score.
This is because there exists a rather large class imbalance in observed dataset where majority of recipes have the type "Daily Meal", which makes accuracy not a good metric. The prediction model can get a pretty high accuracy simply by predicting 'daily meal' all the time. 
Thus, I chose micro f1-score as the evaluation metric because it's suitable for imbalanced datasets and is able to aggregate the performance metrics across all three classes into a single metric. 

**Information At Time of Prediction**: At the time of prediction, the user would have access to many details of the recipe including the recipe's ingredients, instructions, nutrition info, subtmitted, time, tags of the recipe post. 
Number of steps, and number of ingredients are accessible through counting. The easiness of a recipe and whether it's a recipe for holidays can be determined from looking at recipe's tags. Similarly, calories of a meal and the year recipe posted can be determined from the nutriton info and submitted time of the recipe. Thus, sisxthfeatures columns are available at the time of prediction: 'year','holiday','easiness','n_steps','n_ingredients','calories'


## Baseline Model 

**Features Used**: For the Baseline Model, three features are used as shown below:'year','n_steps','n_ingredients'

**year** (discrete quantitative feature): This feature extracted from the 'submitted' column indicates the year a recipe is posted. This feature is considered based on my previous investigation on the Trend of Cooking Time Over Years, where it's found that average cooking time seems to increase over the years. 

**n_steps** (discrete quantitative feature): This feature is included because a recipe that involves more steps usually takes a longer time to complete. 

**n_ingredients** (discrete quantitative feature): Similary, this feature is included because a recipe with more ingredients usually indicate that it's a more complex dish and is likely to take more time. 


**Feature engineering**: Since all of these three features are quantitative, and Random Forest Classifier, a tree-based algorithm, is utilized, I decide not to transform these features for simplicity. 

**Model Construction**: Dataset used is split into training data and test data. RandomForestClassifier is utilized inside a pipepline object to fit training data and make predictions. 

**Model Performance**: The micro f1 score of baseline model on training set is 0.8769654338277421 and the  micro f1 score of baseline model on testing set is 0.8594481046500525.

The performance of this baseline model is fine, but not excellent. This is because the f1 scores are very low for 'big meal' and 'event meal' classes. As shown in the confusion matrix below, the model tend to predict many 'big meal' classes or 'event meal' classes as 'daily meal' due to class imbalance in the observed dataset. 

|                 |   predicted big meal |   predicted daily meal |   predicted event meal |
|:----------------|---------------------:|-----------------------:|-----------------------:|
| true big meal   |                   22 |                   1035 |                     26 |
| true daily meal |                   59 |                  17920 |                    160 |
| true event meal |                   24 |                   1640 |                     60 |



## Final Model 
**Features Added**:
In the final model, to improve model performance and aaddress the class imbalance issue, three more features are added into the model to better classify meal type: 'holiday','easiness','calories'

'**holiday'**(categorical/qualitative): This feature is extracted from the 'tags' column in the dataset. Tags that contain the word 'thanksgiving','christmas' or 'holiday' are marked as 'holiday',and the rest are marked as 'not_holiday' in this added feature. This feature is chosen because recipes for holidays can usually be 'event meal'or big meal'.

**'easiness'**(categorical/qualitative): This feature is extracted from the 'tags' column in the dataset Tags that contain the word 'easy' are marked as 'easy',and the rest are marked as 'not_easy' in this added feature. This feature is chosen becase easy recipes are usually for daily meals.

**'calories'**(continuous quantitative): This feature is extracted from the 'nutrition' column in the dataset. This feature is added because meals with higher total calories are bigger and more complex meals and can take more time to cook.

**Feature Engineering**: With added features, there are in total sixth features, in which two features 'holiday' and 'easiness' are categorical features, and the rest features are quantitative features. Thus, 'holiday' and 'easiness' are hot encoded, and three quantitative features 'n_steps','n_ingredients' and 'calories' are standardized for computatin efficiency, while 'year' is not transformed for interpretation purpose. 


**Model Constructio and Choice of Hyperparameter**: With the above transformation for each of the feature, a ColumnTransformer is used to allocate a OneHotEncoder transformer to 'holiday' and 'easiness' and a StandardScaler transformer 'n_ingredients','n_steps', and 'calories'. The 'year' feature is passed through without any transformation. The ColumnTransformer is then combined with a RandomForestClassifier as the multi-class classifier in one Pipeline object as the final model.

To address the class imbalance issue and improve the performance model, Grid Search is used to fine tune the model and determine the optimized hyperparameters, which inclues 'classifier__n_estimators', 'classifier__max_depth', and 'classifier__class_weight'. Multiple trials are conducted to find the best combination of hyperparameters. 

**Model Performance**: The final model is fitted with hyperparameters returned by GridSearch on the same training set to test its performance. Hyperparameters returned are: 
classifier__class_weight': None, 'classifier__max_depth': 7, 'classifier__n_estimators': 10. 

The micro f1 score of the final model on training set is 0.8668279330320199 and the  micro f1 score of the final model on testing set is 0.866227441993698.

The performance of the final model is better, since the micro f1 score of our final model improved by around 0.79% compared to the baseline model's micro f1 score. f1 scores of the the two models are comparable because the same training set and testing set are used for the two models.

However, as shown by the confusion matrix below, while the perfomrance of the final model improved with respect to the micro f1-score, the issue with class imbalance still exists, as 'big meal' and 'event meal' classes are still mostly mispredicted as 'dail meal'. This issue can be solved by undersampling majority class or oversampling minority class. Overall, there's still a lot of space to improve this model when it comes to its precision and recall. 


|                 |   predicted big meal |   predicted daily meal |   predicted event meal |
|:----------------|---------------------:|-----------------------:|-----------------------:|
| true big meal   |                   2 |                   1072 |                     9  |
| true daily meal |                   1 |                  18124 |                     14 |
| true event meal |                   1 |                   1705 |                     18 |




## Fairness Analysis 

To test whether the final model is fair, a fairness analysis is conducted by splitting the dataset into two groups based on 'months' column and conducting a permutation test on precision score of the prediction. Test statististics chosen is the absolute difference of the precision socres from the two groups. 

**Group A and Group B**: The two groups used in this permutation test are  extracted from the column 'months', in which a Binarizer is used to separate the two groups: 
Group A: Recipes posted on odd months 
Group B: Recipes posted on even months

**Null Hypothesis**: The model is fair. Its precision for recipes posted on odd months and Recipes posted on even months are roughly the same, and any differences are due to random chance. 

**Alternative Hypothesis**: The model is unfair. Its precision for recipes posted on odd months is different from its precision for Recipes posted on even months. 

**Evaluation Metrics and Test Statistics**: Since micro f1-score is used as the performance measurement in the models before, it is ontinue used as the evaluation metrics in the permutation test. 

**Test Statistic**: Absolute difference in group means of micro f1-scores

**Significance Level**: The significance level set for this permutation test is 0.05. 


**P-Value Result and Conclusion**: After 100 permutation, we get a p-value of 0.65, much larger than 0.05, suggesting it fails to reject the null hypothesis. Thus, it's highly possible that the final model to predict meal type is a fair model comparing even months posted recipes and odd months posted recipes. 

