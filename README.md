# Demand forecast demo project

__1. Project overview__

Public bicycle sharing (PBS) systems are the means of renting bicycles where the process of obtaining membership, rental, and bike return is automated via a network of kiosk locations throughout a city. Using these systems, people are able rent a bicycle from a one location and return it to a different place on an as-needed basis. Maintenance work in continuously functioning system such as PBS would require either shuting down the system for a period of time or performing work interupting the normal bicycle sharing usage- both at risk of customer dissatisfaction. The latter effect can be avoided with better maintenance planning utilizing methods of predictive modelling.   

In this demo exercise purpose has been to predict demand for PBS system services in Washington, D.C depending on meteorological data.
Of particular interest has been identification of conditions in which traffic would be low enough and weather mild enough to allow for PBS system maintenance. 

Solution utilized Jupyter Notebook/Azure Machine Learning; ML algorithms: multiple linear regression, random forest, gradient boosting stack ensemble and data analysis libraries: pandas, matplotlib, numpy, sklearn, seaborn, pickle. 

__2. Data collection and preparation__

_2.1. Data collection_

Original data has been taken from Kaggle competition resource: https://www.kaggle.com/c/bike-sharing-demand/data

_2.2. Exploratory Data Analysis (EDA)_

Loaded dataset has 11 dimensions and about 11k datapoints. Correlation heatmap revealed several variables likely to have predictive power for 'count' of customers using the PBS system. It also shows strong multicolinearity in a case of 'temp' (real temperature) and 'atemp' (apparent temperature):    

![image](https://user-images.githubusercontent.com/99291264/156817243-e011645f-5f30-44c6-b1af-ef1781d5519a.png)

As an example, the scatter for temperature versus customer count is considerable, yet both correlation coefficient and causal link indicate a positive relationship:

![image](https://user-images.githubusercontent.com/99291264/156817618-6d07a95b-e459-44d5-95fc-18d410ce2898.png)

_2.3. Data cleaning_

In order to prepare data for further analysis following actions have been performed:
  * Elimination of data points with missing values for 'count' variable,
  * Replacing incorrect values (customer count < 0) with mean value in a dataset, 
  * Removing duplicates based on 'datetime' variable,
  * Removing redundant variables: those that constitute output rather than input ('registered','casual'), those that are not meteorological
    ('datetime', 'season', 'holiday', 'workingday') and those that show strong dependency on another variable in a dataset ('atemp'- real temperature 
    considered primary to apparent temperature)      

Resulting cleaned dataset is as follows:

![image](https://user-images.githubusercontent.com/99291264/156821452-e4ec37ab-8639-459e-97c0-65048ac9c40e.png)

__3. Predictive Modelling__

_3.1. Baseline models_

Model response (target) variable ('count') as well as predictor variables ('temp', 'humidity', 'windspeed', 'weather') have been defined. Train-test split and then basic linear regression and random forrest models have been initiated and trained. As a null model prediction of mean 'count' for each data point has been taken. Root mean squared error (RMSE) has been used as model error metric. 

Random forest model consistently shows the best results. In an example run linear regression beats null model by 14% and random forrest beats it by 17%.

Prediction visualizations between response and predictor variables confirm validity of the selected random forrest baseline model. Despite predictive power of each individual predictor variable is limited by multivariable nature of the analyzed problem, both the predicted magnitude and relationship shape are generally close to observation:  


![image](https://user-images.githubusercontent.com/99291264/156817828-951154a0-96be-4c6b-b01c-553b8fe187bb.png)

![image](https://user-images.githubusercontent.com/99291264/156818505-9ed5d2c6-762b-476c-86b2-6de158302a67.png)

![image](https://user-images.githubusercontent.com/99291264/156819345-5d9faee5-9466-491a-a2b4-41482feda4df.png)

_3.2. Model optimization_

In order to further improve quality of the baseline model _feature selection_ and _hyperparameter tuning_ methods have been employed. 

Performing feature selection _for_ loop invoking _SelectKBest_ filtering method has been initiated. Number of predictor variables (features) k=3 consistently showed to minimize root mean squared error (RMSE). Therefore, the training dataset has been transformed accordingly and least predictive variable according to the method ('windspeed') has been removed. 

Then the hyperparameter tuning has been performed on a local machine using transformed dataset, cross validation grid search (_GridSearchCV_) and custom scoring function. Finally, best model has been found for hyperparameters- {'max_depth': 4, 'max_features': 'auto', 'n_estimators': 400} beating the null model by 36%.

_3.3. Model deployment and utilization_

The best performing local has been deployed as offline application using _pickle.dump_ method. Then it has been loaded back in order to make predictions. 

For the purpose of this exercise it is assumed that maintenance cannot be performed in temperature below 14 Celsius and in rainy/snowy conditions or at night. Furthermore, the maximum traffic suitable for maintenance has been set to 150.

Coldest month in Washington, D.C that typically allows for maintenance is then April with daily mean temperature of 14.6 Celsius. Inserting this value into the model it can be seen that for clear weather with few clouds humidity must go over 72 to consistently output demand of less than 150 customers. For July with daily mean temperature of 27.2 Celsius the same is true at humidity of 83. 

_3.4. Future work_

An automated _Azure Machine Learning_ dummy run has been performed on a raw dataset in which 42 different ML algorithms has been tried and tuned on a cloud compute cluster. The best performing _stack ensemble regressor_ combined 4 gradient boosting models. It parsed string variable 'datetime' in order to extract period of a day dependency. This improved the signal of meteorological data and allowed for better predictions of demand during daylight. In order to improve predictive power further this method should be used on a feature filtered dataset (as decribed above). Morever, full (T, RH) space could be mapped to see all coordinates yielding demand below assumed threshold. 

__4. Conclusions__

Approach taken here achieved 36% improvement compared to null model and good prediction curve shape fidelity. This suggests that ML model deployed as an independent application can, with few improvements, reasonably output maintenance feasibility at any given day with clear weather based on temperature and humidity data only. 

__Resources__

Jupyter notebook with code: https://github.com/TobiasMazan/Demand-forecast-demo-project/blob/main/PBS%20Demand%20Prediction.ipynb
