# Optimizing an ML Pipeline in Azure

## Overview
This project is part of the Udacity Azure ML Nanodegree.
In this project, we build and optimize an Azure ML pipeline using the Python SDK and a provided Scikit-learn model.
This model is then compared to an Azure AutoML run.

## Summary
The dataset contains data about marketing campaigns based on phonecalls from banking institutions. The data contains client data (numeric, categorical), info about the last campaign, other attributes about previous campaign and time aspects, social and economic context.
The prediction excercise was understanding whether or not a client subscribed for term deposit, this is a classification problem.
There were 2 approaches used: 
* Building a baseline Logistic regression model, with a careful manual feature engineering, use hyperparameter tuning on the model and observe the best we can do with the model
* Allowing AutoML do featurization and deifferent combinations of transformations and models on the data, to see, whether better model output can be achieved along a metric which is the same for automl and logistic regression - in this case accuracy

The best performing model vas a voting ensemble, generated by autoML

## Scikit-learn Pipeline
### Overview
The solution contained 2 major components
* A python file, containing the data cleaning logic, the data ingestion code and the model specification, also used for outputting the model in each run into a pickle file
* An ipython notebook responsible for configuring and setting up the environment, the hyperparameter space and the sampling logic to go through that, an estimator, which defines the model code and compute target link, a hyperdrive config, which defines the goal of the optimization effort
* The ipython notebook also contains a widget, that can show the progress of the hyperdrive optimization and different child   runs in a widget. These can be observed in the AML workspace too, when checking on experiments
* The thirs function of the notebook is model saving and registration

### Data ingestion
Data is ingested in the model definition file through a TabularDatasetFactory class. This ingestion happens at every child run, from a centralized blob storage. The factory can handle many datasources, and the result is an azureml specific data object, which can be transformed into a pandas dataframe. The data contains more than 1million rows in total

### Cleaning and featurization
Also contained in the train.py file, this does a couple of things like transforming categorical variables to binary numeric or dummies. Also gets rid of orginal columns where dummies were created, and mapping non-binary categoricals to numeric ones.

### Model training script 
There are 2 aspects of this
* Splitting the dataset 
* Actual modeling with the parameters passed over to the model

### Algorythm
The LogisticRegression of scikit learn is a common baseline model for prediction excercises, because of the easy explainability. 
It can take a number of parameters, here Ithe parameter space was focusing mainly on the regularization strength.
The default solver us lbfgs and it uses l2 regularization.

### Hyperparameter tuning and model management
The hyperdrive process went through the parameter space using the RandomParameterSampling sampler. This goes through the parameter space, which in this case was a discrete one, so choice based selection was used.
The best model was chosen through a programmatic call from to geth the best run ID and the pkl file for the best performing model, which can be registered to AML Workspace model store.

## Sampler selection
There are a number of samplers that can be selected
* Random sampling
* Grid sampling
* Bayesian sampling
I have opted for Random sampling, because it goes through an initial selection of hyperparameter space, with early termination and low performance runs. As I was doing baselining, for this project I was focusing on getting early result and see whether the auto ML can perform somewhat or significantly better.

## Early stopping policy selection
There are a number of early stopping policies
* Bandit policy
* Median stopping 
* Truncation selection

Median  stopping policy is amore conservative approach, which can provide 25-35% of the savings
Bandit with smaller allowable slack and truncation selection with larger truncation percentage can cause better results in saving. 
With 4 processing nodes the truncation selection with 50% seemed well defined use case for saving in case of baselining.

## AutoML
**In 1-2 sentences, describe the model and hyperparameters generated by AutoML.**


## Pipeline comparison
### Model comparison
The hyperdrive approach focuses only on the hyperparameters of one specific model, it does not go through a wide range of possible models and transformation combinations.
This can be useful, if we want to work with a specific model version and we want to have a full control over the featurization process. This approach is good when tuning a small number of selected models or models in production already.
The auto ML approach is useful if we want to explore a wide range of models and transoformations, i.e. in the initial exploration phase, also to get some hints about the possible best transformations. This approach is good when we are enriching our understanding about the data and the solution and want to narrow down the modeling efforts to a smaller set of vvariants.
Both modeling tecniques can obtain high performance models, in this specific case automl was able to generate slightly better model, than the baseline, even after optimization.

### Architecture comparison
The hyperdrive architecture and the automl architecture are different in a sense, that hyperdrive separates the model code from the orchestration and optimization code. This is useful in some cases, when the modeling and the model operationalization is done by different teams.





## Future work
The hyperoptimization effort was largely run as a baselining and a proof, that the necessary utilities (sampler, stopping) can be used. In case of more sophisticated baselining this part need to be adjusted to the modeling effort which delivers a more mature baseline.
Accuracy may not be the only metric important in a classification. Tuning the model along differnt metrics would be an important use case, especially if Type1 and Type2 errors have different pricetags.
From the other side I have not used allowed_models/blocked_models features here, these can be important, if we have specific business requirements about the type of model we need to use (i.e. client interpretability.)

## Proof of cluster clean up
During running
![image](https://user-images.githubusercontent.com/81808810/113709673-12acf700-96e3-11eb-9e7a-567e1e62dd4b.png)

