---
title: "Pump it Up: Data Mining the Water Table"
date: 2020-11-06T20:52:31-05:00
slug: ""
description: "Project write up."
keywords: [Data Science, Classification]
draft: true
tags: [Data Science, Project, Classification]
math: false
toc: false
---
## Project Description
In this post, I will tell you about building my submission to the [Pump it Up: Data Mining the Water Table](https://www.drivendata.org/competitions/7/pump-it-up-data-mining-the-water-table/) competition hosted by [DrivenData](https://www.drivendata.org/). The competition problem uses data about water pumps in Tanzania collected by a partnership between [Taarifa](http://taarifa.org/) and the [United Republic of Tanzania Ministry of Water](https://www.maji.go.tz/) to predict if they are currently in need of repair. Specifically, the goal is to classify a water pump as 'functional', 'functional needs repair', or 'non functional' given the available data. By better understanding what factors predict issues with water pumps, we hope to improve access to potable water in Tanzania and reduce operational costs associated with maintaining water infrastructure. 

For the purposes of the competition, the model is assessed in terms of the classification rate, also commonly called [accuracy](https://en.wikipedia.org/wiki/Accuracy_and_precision).
The classification rate is a number between zero and one with zero indicating that all prediction were wrong, one indicating all predictions were correct, and 0.7 indicating that, on average, seven out of ten predictions are correct.

My model produced an accuracy score of 0.8217. As of the time of writing, the best reported score for the competition is 0.8294 and my model ranks 490 out of 10,304 submissions.

## Main Takeaway
I was surprised by the performance that I was able to achieve without any feature engineering or manual data cleaning. This model is an exercise in applying off-the-shelf tools with minimal effort to great effect. 

## Modeling Approach

Initial experiments with unoptimized models showed that the [XGBClassifier](https://xgboost.readthedocs.io/en/latest/python/index.html) performed best on this data.
With this in mind, I wanted to build an [sklearn pipeline](https://scikit-learn.org/stable/modules/generated/sklearn.pipeline.Pipeline.html) to implement preprocessing and model fitting and then use an [sklearn grid search](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GridSearchCV.html) to select the best hyper-parameters for the model and perform five-fold [cross validation](https://en.wikipedia.org/wiki/Cross-validation_(statistics)).

If you would like to build the model yourself, you can access this project at my [github repo](https://github.com/sethchart/Pump-it-Up-Data-Mining-the-Water-Table).

### Initial Setup
My goal in this project was to use as many off-the-shelf tools as possible and avoid writing unnecessary custom code to achieve data processing or modeling.

#### Import Required Modules
Everything that is imported below gets used at least once.
```python
#Basics
import pandas as pd

#Plotting
import matplotlib.pyplot as plt
import seaborn as sns

#Train Test Split
from sklearn.model_selection import train_test_split

# Imputer
from sklearn.impute import SimpleImputer

# Preprocessing
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import OneHotEncoder, 
	StandardScaler, 
	FunctionTransformer

# Classifiers
from xgboost import XGBClassifier
from sklearn.multiclass import OneVsRestClassifier

#Pipeline
from sklearn.pipeline import Pipeline

#Grid Search
from sklearn.model_selection import GridSearchCV

# Model evaluation
from sklearn.metrics import plot_confusion_matrix
```

#### Set Random State
The random state to be used whenever a randomized process is initiated.


```python
random_state = 42
```

#### Select Columns to Drop from the Model
Provide a list of variables that should be dropped from the model. We have not observed any improvement in model performance from dropping data, measured in terms of accuracy. Training time is obviously improved by dropping columns but there seems to be a small price to pay in terms of accuracy for reducing the number of available features.


```python
drop_cols = []
```

#### Import Data
Because of the submission format requirements for the competition, it is vital that we retain the index column through out modeling so that we are able to produce predictions that can be validated using the competition's validation data.


```python
features = pd.read_csv('../data/training_features.csv', 
		       index_col='id')
targets = pd.read_csv('../data/training_labels.csv', 
		      index_col='id')

df = features.join(targets, how='left')
X = df.drop('status_group', axis=1)
y = df['status_group']
```

#### Make Test Train Split
For the purposes of model tuning we hold 10% of the data out for local testing.


```python
X_train, X_test, y_train, y_test = train_test_split(
					X, 
					y, 
					test_size=0.1, 
					random_state=random_state
				   )
```

#### Data Validation
We experimented with both manual and automated feature selection, however neither approach improved model performance. Initially, we had issues with mixed data types in both the `public_meeting` and `permit` columns. The function below converts all categorical variables to strings to eliminate thoes errors. 


```python
def convert_categorical_to_string(data):
    return pd.DataFrame(data).astype(str)

CategoricalTypeConverter = FunctionTransformer(
				convert_categorical_to_string
			   )
```

#### Classify Variables
We will need to pre-process our data in preparation for classification. Pre-processing is different for categorical and numerical variables. In order to implement different pre-pricessing flows, we must first classify all of our variables as categorical or numerical. The function below separates columns into these two classes and excludes any variables that will be dropped from the model.


```python
def classify_columns(df, drop_cols):
    """Takes a dataframe and a list of columns 
       to drop and returns:
        - cat_cols: A list of categorical columns.
        - num_cols: A list of numerical columns.
    """
    cols = df.columns
    keep_cols = [col for col in cols if col not in drop_cols]
    cat_cols = []
    num_cols = []
    for col in keep_cols:
        if df[col].dtype == object:
            cat_cols.append(col)
        else:
            num_cols.append(col)
    return cat_cols, num_cols
```


```python
cat_cols, num_cols = classify_columns(X_train, drop_cols)
```

#### Build Preprocessor
Below we build a preprocessing step for our pipeline which handles all data processing.

##### Categorical Preprocessing Pipeline
The pipeline below executes the following three steps for all of our categorical data.
1. Convert all values in categorical columns to strings. This avoids data type errors in the following steps.
2. Fill all missing values with the string `missing`.
3. One-hot encode all categorical variables. Because this data contains categorical variables with many possible values, it is possible to encounter values in testing data that was not present in the training data. For this reason, we need to set `handel_unknown` to `ignore` so that the encoder will simply ignore unknown values in testing data.


```python
categorical_pipeline = Pipeline(steps=[
    ('typeConverter', 
     CategoricalTypeConverter),
    ('imputer', 
     SimpleImputer(strategy='constant', fill_value='missing')),
    ('standardizer', 
     OneHotEncoder(handle_unknown='ignore',dtype=float))
])
```

##### Numerical Preprocessing Pipeline
The pipeline below executes two steps:
1. Imputes missing values in any numerical column with the median value from that column.
2. Scales each variable to have mean zero and standard deviation one.


```python
numerical_pipeline = Pipeline(steps=[
    ('imputer', SimpleImputer(strategy='median')),
    ('standardizer', StandardScaler())
])
```

#### Preprocessing Pipeline
The column transformer below implements each of the three possible pre-processing behaviors. 
1. Apply the categorical pipeline.
2. Apply the numerical pipeline.
3. Drop the specified columns.
The if-then statement below ensures that the drop processor is only implemented if there are columns to drop. This is needed since passing an empty `drop_col` list throws an error.


```python
if len(drop_cols) > 0:
    preprocessor = ColumnTransformer(
    transformers=[
        ('numericalPreprocessor', 
	 numerical_pipeline, 
	 num_cols),
        ('categoricalPreprocessor', 
	 categorical_pipeline, 
 	 cat_cols),
        ('dropPreprocessor', 
	 'drop', 
	 drop_cols)
    ])
else:
    preprocessor = ColumnTransformer(
    transformers=[
        ('numericalPreprocessor', 
		 numerical_pipeline, 
		 num_cols),
        ('categoricalPreprocessor', 
	 categorical_pipeline, 
	 cat_cols)
    ])
```

#### Build Model Pipeline
Below we build our main pipeline which executes two steps.
1. Apply preprocessing to the raw data.
2. Fit a one vs rest classifier to the processed data using an eXtreme Gradient Boosted forest model.


```python
pipeline = Pipeline(
    steps=[
        ('preprocessor', preprocessor),
        ('classifier', OneVsRestClassifier(estimator='passthrough'))
    ]
)
```

#### Building Parameter Grid
Below we define a grid of hyper-parameters for our pipeline that will be tested in a grid search below.


```python
parameter_grid = [
    {
        'classifier__estimator': [XGBClassifier()],
        'classifier__estimator__max_depth': [5, 10, 15, 20],
        'classifier__estimator__n_estimators': [100, 150, 200, 250]
    }]
```

#### Instantiate Grid Search
Below we instantiate a grid search object which will fit our pipeline for every combination of the parameters defined above. Since the competition uses accuracy as it's measure of model quality, we sill evaluate model performance in terms of accuracy. For each parameter combination, the grid search will also execute five-fold cross validation. 

In order to maximize performance, we will fit our grid search on the full provided training data set and select our best hyper-parameters based on the results of cross validation. For the purposes of local model evaluation, we will then refit the best model on our local training data and use our local testing data to produce a confusion matrix.


```python
grid_search = GridSearchCV(
    estimator=pipeline, 
    param_grid=parameter_grid, 
    scoring='accuracy', 
    cv=5, 
    verbose=2, 
    n_jobs=-2,
    refit=True
)
```

#### Fit Grid Search
Below we fit our grid search on the full training set and select the best model hyper-parameters. This step takes an Extremely long time to run.


```python
grid_search.fit(X, y)
model = grid_search.best_estimator_
```

    Fitting 5 folds for each of 16 candidates, totalling 80 fits


    [Parallel(n_jobs=-2)]: Using backend LokyBackend with 3 concurrent workers.


## Display Results of Grid Search
Below we display the results of our grid search. We pay particular attention to `std_test_score` which will become larger if the model is over-fit. 


```python
grid_search_results = pd.DataFrame(grid_search.cv_results_)
grid_search_results.to_csv('../reports/grid_search_results.csv')
```

#### Predict on Validation Data
Below we import the testing data provided by the competition. To maximize performance we refit our model on the full training data set. Predictions are formatted and saved to CSV for submission.


```python
model.fit(X,y)
X_validate = pd.read_csv('../data/testing_features.csv', index_col='id')
y_validate = model.predict(X_validate)
df_predictions = pd.DataFrame(y_validate, index=X_validate.index, columns=['status_group'])
df_predictions.to_csv('../predictions/final_model.csv')
```

#### Produce Confusion Matrix
Below we fit the model on our local training data and produce a confusion matrix using the local test data. This provides a reasonable indication of how the model performs. Because the model needs to be fit before producing the matrix, this step will take a long time to run.


```python
model.fit(X_train, y_train)
fig, ax = plt.subplots()
fig.set_figheight(10)
fig.set_figwidth(10)
plot_confusion_matrix(model, X_test, y_test, ax=ax, normalize='true', include_values=True)
fig.savefig('../images/Confusion_Matrix.png', bbox_inches='tight')
```

##### Confusion Matrix Analysis
