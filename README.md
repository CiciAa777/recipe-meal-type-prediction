# recipe-meal-type-prediction
**Author**: Cici Xu 

## Project Overview 
This data science project investigates different parts of a recipe and its cooking time, and constructs a classifer model to predict if a recipe type is a daily meal, event meal, or big meal. The recipe type is categorized with respect to cooking time.  This is a course project for DSC80- Practice and Application of Data Science at UCSD. 

Our exploratory data analysis on this dataset can be found [here](https://github.com/CiciAa777/recipe-dataset-investigation/tree/main).


## Framing the Problem 

**Investigation Problem**: This project aims to predict if a recipe type is a daily meal, event meal, or big meal by conducting a multiclass classification. The classification is based on time consumption level of a recipe, i.e. how long it takes to finish the recipe. 

**Response Variable**: Meal type column that indicates the time consumption level of a recipe. This column is created by setting threhold to the 'minutes' column in the dataframe: 
Daily Meal - 
Event Meal - 
Big Meal - 

Meal type is chosen as the response variable because it's a very useful and esasy category that allows people to look for recipes based on their needs. 

**Measureing Metrics**: the performance metric chosen for this classification model is f1-score(micro). This is because there exists a rather large class imbalance in observed dataset where majority of recipes have the type "Daily Meal", which makes accuracy not a good metric. The prediction model can get a pretty high accuracy simply by predicting 'daily meal' all the time. Thus, I chose micro f1-score as the evaluation metric because it's suitable for imbalanced datasets and is able to aggregate the performance metrics across all three classes into a single metric. 

**Information At Time of Prediction**: At the time of prediction, the user would have access to many details of the recipe including the recipe's ingredients, instructions, nutrition info, subtmitted, time, tags of the recipe post. 
	Number of steps, and number of ingredients are also through counting. The easiness of a recipe and whether it's a recipe for holidays can be determined from looking at recipe's tags. Similarly, calories of a meal and the year recipe posted can be determined from the nutriton info and submitted time of the recipe. Thus, seven features columns are available at the time of prediction: 'year','holiday','easiness','n_steps','n_ingredients','calories'


## Baseline Model 

**Features Used**: For the Baseline Model, three features are used as shown below:
'year','n_steps','n_ingredients'

difficulty (numerical feature): This feature extracted from the 'submitted' column indicates the year a recipe is posted. This feature is considered based on my previous investigation on the Trend of Cooking Time Over Years, where it's found that average cooking time seems to increase over the years. 

n_steps (numercial feature): This feature is included because usually a recipe that involves more steps takes a longer time to complete. 

n_ingredients (nuemrical feature): Similary, this feature is included because usually a recipe with more ingredients indicate that it's a more complex dish and is likely to take more time. 


**Feature engineering**: Since all of these three features are quantitative, and Random Forest Classifier, a tree-based algorithm, is utilized, I decide not to do transform these features for simplicity. 

**Model Construction**: Dataset used is split into training data and test data. RandomForestClassifier is utilized inside a pipepline object to fit training data and make predictions. 

**Model Performance**: The micro f1 score of baseline model on training set is 0.87701317715959 and the  micro f1 score of baseline model on testing set is 0.8594958464623317.

The performance of this baseline model is fine, but not excellent. This is because the f1 scores are very low for 'big meal' and 'event meal' classes. As shown in the confusion matrix below, the model tend to predict many 'big meal' classes or 'event meal' classes as 'daily meal' due to class imbalance in the observed dataset. 



## Final Model 
**Features Added**:
In the final model, to improve model performance and aaddress the class imbalance issue, three more features are added into the model to better classify meal type: 'holiday','easiness','calories'

'holiday': This feature is extracted from the 'tags' column in the dataset. Tags that contain the word 'thanksgiving','christmas' or 'holiday' are marked as 'holiday',and the rest are marked as 'not_holiday' in this added feature. 

'easiness': This feature is extracted from the 'tags' column in the dataset Tags that contain the word 'easy' are marked as 'easy',and the rest are marked as 'not_easy' in this added feature. 

'calories': This feature is extracted from the 'nutrition' column in the dataset. This feature is added because usually meals with higher total calories can usually be bigger and more complex meals. 

**Feature Engineering**: With added features, there are in total sixth features, in which two features 'holiday' and 'easiness' are categorical features, and the rest frou are quantitative features. Thus, 'holiday' and 'easiness' are hot encoded, and three quantitative features 'n_steps','n_ingredients' and 'calories' are standardized for computatin efficiency, while 'year' is not transformed for interpretation purpose. 


**Model Constructio and Choice of Hyperparameter**: With the above transformation for each of the feature, a ColumnTransformer is used to allocate a OneHotEncoder transformer to 'holiday' and 'easiness' and a StandardScaler transformer 'n_ingredients','n_steps', and 'calories'. The 'year' feature is passed through without any transformation. The ColumnTransformer is then combined with a RandomForestClassifier as the multi-class classifier in one Pipeline object as the final model.

To address the class imbalance issue and improve the performance model, Grid Search is used to find out the fine tune the model and find the optimized hyperparameters, which inclues 'classifier__n_estimators', 'classifier__max_depth', and 'classifier__class_weight'. Multiple trials are conducted to find the best combination of hyperparameters. 

**Model Performance**: The final model is fitted with hyperparameters returned by GridSearch on the same training set to test its performance. Hyperparameters returned are: 
classifier__class_weight': None, 'classifier__max_depth': 7, 'classifier__n_estimators': 10. 

 The micro f1 score of the final model on training set is (change) and the  micro f1 score of the final model on testing set is (change)。

The performance of the final model is better, since the minor f1 score of our final model improved by (change) compared to our baseline model's accuracy. f1 scores of the the two models are comparable because the same training set and testing set are used for the two models.

As shown by the confusion matrix below, while the perfomrance of the final model improved with respect to the minor f1-score, the issue with class imbalance still exists, as 'big meal' and 'event meal' classes are still mostly mispredicted as 'dail meal'. 


## Fairness Analysis 

To teset whether the final model is fair, a fairness analysis is conducted by splitting the dataset into two groups based on 'month' column and conducting a permutation test on precision score of the prediction. Test statististics chosen is the absolute difference of the precision socres from the two groups. 

**Group A and Group B**: The two groups used in this permutation test are  extracted from the column 'month', in which a Binarizer is used to separate the two groups: 
Group A: Recipes posted between January and June

Group B: Recipes posted between July and December

**Null Hypothesis**: The model is fair. Its precision for recipes posted between January and June and Recipes posted between July and December
are roughly the same, and any differences are due to random chance. 

**Alternative Hypothesis**: The model is unfair. Its precision for recipes posted between January and June is different from its precision for Recipes posted between July and December. 

**Test Statistic**: Absolute difference in group means.

Significance Level: The significance level we set for this permutation test is 0.01. That being said, we will perform a relatively strong permutation test with confidence level of 99%.

Evaluation Metrics and Test Statistics: Since we used accuracy as the performance measurement in the models before, we will continue to use this as the evaluation metrics in our permutations. As for test statistics, because we suspect the accuracy might be different for testing on older recipes as compared to on more recent recipes, we use the absolute difference in accuracy scores of the predictions on two groups using our final model.

P-Value Result: After 100 permutation, we get a p-value of 0.6, which is greater than our test statistics of 0.01. This suggests we fail to reject out null hypothesis.

Below is a graphic visualization of the outcome of our permutation test:

<iframe src="permutation_test_dist.html" width=800 height=600 frameBorder=0></iframe>
Conclusion: According to the result from our permutation test above, it shows evidence that suggests our final model on predicting time consumption based on recipe contents is very likely to be a fair model in terms of the time for both old recipes and the relatively recent recipes.