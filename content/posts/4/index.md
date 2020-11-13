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

## Modeling Approach

Initial experiments with unoptimized models showed that the [XGBClassifier](https://xgboost.readthedocs.io/en/latest/python/index.html) performed best on this data. With this in mind we wish to build an [sklearn pipeline](https://scikit-learn.org/stable/modules/generated/sklearn.pipeline.Pipeline.html) to implement preprocessing and model fitting and then use an [sklearn grid search](https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.GridSearchCV.html) to select the best hyper-parameters for our model and perform five-fold [cross validation](https://en.wikipedia.org/wiki/Cross-validation_(statistics)).

## Preprocessing
We experimented with both manual and automated feature selection, however neither approach improved model performance. In the end we found separating data into numerical and categorical variables and using basic preprocessing techniques yielded the best results. 


### Classifying Variables
We will need to pre-process our data in preparation for classification. Pre-processing is different for categorical and numerical variables. In order to implement different pre-pricessing flows, we must first classify all of our variables as categorical or numerical. The function below separates columns into these two classes and excludes any variables that will be dropped from the model.


```python
def classify_columns(df, drop_cols):
    """Takes a dataframe and a list of columns to drop and returns:
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

### Building Preprocessor
Below we build a preprocessing step for our pipeline which handles all data processing.

#### Categorical Pipeline
The pipeline below executes the following three steps for all of our categorical data.
1. Convert all values in categorical columns to strings. This avoids data type errors in the following steps.
2. Fill all missing values with the string `missing`.
3. One-hot encode all categorical variables. Because this data contains categorical variables with many possible values, it is possible to encounter values in testing data that was not present in the training data. For this reason, we need to set `handel_unknown` to `ignore` so that the encoder will simply ignore unknown values in testing data.


```python
categorical_pipeline = Pipeline(steps=[
    ('typeConverter', CategoricalTypeConverter),
    ('imputer', SimpleImputer(strategy='constant', fill_value='missing')),
    ('standardizer', OneHotEncoder(handle_unknown='ignore',dtype=float))
])
```

#### Numerical Pipeline
The pipeline below executes two steps:
1. Imputes missing values in any numerical column with the median value from that column.
2. Scales each variable to have mean zero and standard deviation one.


```python
numerical_pipeline = Pipeline(steps=[
    ('imputer', SimpleImputer(strategy='median')),
    ('standardizer', PowerTransformer())
])
```

#### Preprocessor
The column transformer below implements each of the three possible pre-processing behaviors. 
1. Apply the categorical pipeline.
2. Apply the numerical pipeline.
3. Drop the specified columns.
The if-then statement below ensures that the drop processor is only implemented if there are columns to drop. This is needed since passing an empty `drop_col` list throws an error.


```python
if len(drop_cols) > 0:
    preprocessor = ColumnTransformer(
    transformers=[
        ('numericalPreprocessor', numerical_pipeline, num_cols),
        ('categoricalPreprocessor', categorical_pipeline, cat_cols),
        ('dropPreprocessor', 'drop', drop_cols)
    ])
else:
    preprocessor = ColumnTransformer(
    transformers=[
        ('numericalPreprocessor', numerical_pipeline, num_cols),
        ('categoricalPreprocessor', categorical_pipeline, cat_cols)
    ])
```

## Building Pipeline
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

## Building Parameter Grid
Below we define a grid of hyper-parameters for our pipeline that will be tested in a grid search below.


```python
parameter_grid = [
    {
        'classifier__estimator': [XGBClassifier()],
        'classifier__estimator__max_depth': [15],
        'classifier__estimator__n_estimators': [200]
    }]
```

## Instantiate Grid Search
Below we instantiate a grid search object which will fit our pipeline for every combination of the parameters defined above. Since the competition uses accuracy as it's measure of model quality, we sill evaluate model performance in terms of accuracy. For each parameter combination, the grid search will also execute five-fold cross validation. 

In order to maximize performance, we will fit our grid search on the full provided training data set and select our best hyper-parameters based on the results of cross validation. For the purposes of local model evaluation, we will then refit the best model on our local training data and use our local testing data to produce a confusion matrix.


```python
grid_search = GridSearchCV(
    estimator=pipeline, 
    param_grid=parameter_grid, 
    scoring='accuracy', 
    cv=5, 
    verbose=2, 
    n_jobs=-1,
    refit=True
)
```

## Fit Grid Search
Below we fit our grid search on the full training set and select the best model hyper-parameters. This step takes an Extremely long time to run.


```python
grid_search.fit(X, y)
model = grid_search.best_estimator_
```

    Fitting 5 folds for each of 1 candidates, totalling 5 fits


    [Parallel(n_jobs=-1)]: Using backend LokyBackend with 4 concurrent workers.
    [Parallel(n_jobs=-1)]: Done   5 out of   5 | elapsed: 31.0min finished


## Display Results of Grid Search
Below we display the results of our grid search. We pay particular attention to `std_test_score` which will become larger if the model is over-fit. 


```python
pd.DataFrame(grid_search.cv_results_)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean_fit_time</th>
      <th>std_fit_time</th>
      <th>mean_score_time</th>
      <th>std_score_time</th>
      <th>param_classifier__estimator</th>
      <th>param_classifier__estimator__max_depth</th>
      <th>param_classifier__estimator__n_estimators</th>
      <th>params</th>
      <th>split0_test_score</th>
      <th>split1_test_score</th>
      <th>split2_test_score</th>
      <th>split3_test_score</th>
      <th>split4_test_score</th>
      <th>mean_test_score</th>
      <th>std_test_score</th>
      <th>rank_test_score</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1173.294172</td>
      <td>326.280756</td>
      <td>4.297567</td>
      <td>1.072134</td>
      <td>XGBClassifier(max_depth=15, n_estimators=200)</td>
      <td>15</td>
      <td>200</td>
      <td>{'classifier__estimator': XGBClassifier(max_de...</td>
      <td>0.81633</td>
      <td>0.812879</td>
      <td>0.814141</td>
      <td>0.811785</td>
      <td>0.810354</td>
      <td>0.813098</td>
      <td>0.002042</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



## Make Predictions for Validation
Below we import the testing data provided by the competition. To maximize performance we refit our model on the full training data set. Predictions are formatted and saved to CSV for submission.


```python
X_validate = pd.read_csv('../data/testing_features.csv', index_col='id')
y_validate = model.predict(X_validate)
df_predictions = pd.DataFrame(y_validate, index=X_validate.index, columns=['status_group'])
df_predictions.to_csv('../predictions/final_model.csv')
```

## Produce Confusion Matrix
Below we fit the model on our local training data and produce a confusion matrix using the local test data. This provides a reasonable indication of how the model performs. Because the model needs to be fit before producing the matrix, this step will take a long time to run.


```python
fit_model = model.fit(X_train, y_train)
fig, ax = plt.subplots()
fig.set_figheight(10)
fig.set_figwidth(10)
plot_confusion_matrix(model, X_test, y_test, ax=ax, normalize='true', include_values=True)
fig.savefig('../images/Confusion_Matrix.png', bbox_inches='tight')
```

## Get Feature Importance


```python
fit_model.named_steps.classifier.estimator.feature_importances_
```


```python

```
