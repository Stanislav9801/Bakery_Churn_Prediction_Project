# Bakery customers churn prediction.  
  
## Project overview
Client: A large coffee and bakery chain.  
Objective: Based on the data, predict that the customer will go into churn.

Data description:  
Table `data`:  
- clnt_ID - unique user id, str   
- timestamp - date and time of purchase, datetime
- gest_Sum - sum of purchase, float
- gest_Discount - amount of discount, float
 
Table `target`.
- clnt_ID - unique user id, str
- target - churn flag, int: 1 if user is churned | 0 if not churned


## Solution algorithm:  
1. **Data preprocessing**  
    - Checking the data
    - Creation new features

2. **Exploratory data analysis**  
During the analysis, it was observed that:   

    - On average, spending by "non-churned" customers within a week, month, and year is higher than spending by "churned" customers.  At the same time, "non-churned" customers spend on average slightly less than "churned" customers within 1 day.  
    - Discounts for "non-churned" guests are higher than for "churned" guests. The same is true for the share of discounted purchases in the total number of purchases for the month.  
    - "Non-churned" customers on average visit the bakery more often and with less intrevals than the "churned" customers.

3. **Evaluation of generalization ability of machine learning algorithms with default settings**  
    - Data preparation for machine learning algorithms and creation of necessary preprocessing functions
    - Evaluation of Random Forest / Random Forest with polynomial features (intersections only)
    - Evaluation of K-Nearest Neighbors
    - Evaluation of Logistic Regression / Logistic Regression with polynomial features
    - Evaluation of Support Vector Machine / Support Vector Machine with polynomial features
    - Evaluation of Gradient Boosting / Gradient Boosting with polynomial features (intersections only)  

    The evaluation was done using the LeaveOneGroupOut cross-validation technique. The results of the models are presented in the table below:  

    |                                     |   ROC-AUC |   F1 |   Accuracy |   Precision |   Recall |
    |:------------------------------------|----------:|-----:|-----------:|------------:|---------:|
    | Random Forest                       |      68.5 | 66.7 |       63.8 |        65.5 |     68   |
    | Random Forest with Polynom          |      68.2 | 66.2 |       63.4 |        65.3 |     67.1 |
    | KNN                                 |      64.6 | 64.4 |       61.5 |        63.6 |     65.3 |
    | **Logistic Regression**             |  **72.2** |**70.2**|   **67** | **67.8**    | **72.7** |
    | Logistic_Regression with Polynom    |      72.5 | 70.6 |       67.3 |        67.8 |     73.6 |
    | SVM Classifier                      |      71.1 | 71.1 |       66.9 |        66.5 |     76.4 |
    | SVM Classifier with Polynom         |      71.3 | 70.3 |       67.1 |        67.8 |     72.9 |
    | CatBoost (max roc_auc)              |      71.3 | 68.5 |       66   |        67.8 |     69.3 |
    | CatBoost (max f1)                   |      71.3 | 68.5 |       66   |        67.8 |     69.3 |
    | CatBoost with Polynom (max roc_auc) |      70.9 | 67.6 |       65.4 |        67.6 |     67.7 |
    
    For the further work a logistic regression model was chosen. 

4. **Hyperperparameters optimization**  
At this step, the most optimal combination of hyperparameters `C` and `l1_ratio` was selected for the logistic regression model. Hyperparameters were optimized using the HyperOpt library.  
The best hyperparameters turned out to be C=0.001, l1_ratio=0.9

5. **Probability calibration**  
At this step, probability calibration was performed for the final classifier.  

6. **Checking the calibrated and uncalibrated classifiers on the test set**    
At this stage, the generalization ability of the algorithms was evaluated on the test sample. The results are presented below:   

    |                                |   ROC-AUC |   F1 |   Accuracy |   Precision |   Recall |
    |:-------------------------------|----------:|-----:|-----------:|------------:|---------:|
    | Logistic_Regression            |      82.6 | 21.5 |       60.5 |        12.2 |     88.8 |
    | Logistic_Regression calibrated |      82.6 | 22.4 |       63.5 |        12.8 |     86.3 |
    | Dummy-estimator                |      49.8 | 10.8 |       49.9 |         6   |     49.7 |

    On the one hand, one might think that our model was bad at finding patterns in the data. However, the F1 score on the cross-validation was close to 70%, and the accuracy of the models is twice as high as the one of the constant algorithm. Moreover, the fact that the balance of classes in the data changes a lot as we approach the end of the year makes us wonder if all the data were really dumped and labeled correctly, if there are any anomalies, and what could be the reason for such imbalance at all.  
  
    If everything is labeled properly, then one of the reasons why the behavior of "non-churned" guests for the model becomes similar to the behavior of "churned" can also be the fact that 2020 is the year when the covid restrictions on public catering were in effect. This could affect the frequency of cafe visits even on the part of "non-churned" clients (it should be taken into account that one of the most important features we have is related to the number of visits per month, as well as the number of days that passed from the last visit to the 31st day of the month).  
    Perhaps, this is partly the reason why the model showed such results on the October data. 

    However, the most important thing in this case is that we managed to achieve a ROC-AUC of 82.6%. This means that more than 82% of the pairs of objects of the species (object of class 1, object of class 0), our algorithm correctly ordered, i.e. the classifier prediction on the object of positive class will be higher than on the object of negative class. After all, it would be more interesting for the bakery to order customers by the probability of termination of visits and depending on this apply different retention options: someone to send a discount coupon from a partner, someone to offer a discount for the next month, and someone special service conditions. Thus, it is not the label itself that is important, but the right order of the objects.

7. **Model Interpretation**  
The interpretation of the logistic regression coefficients revealed that the more "days from the last guest visit to the end of the month" and the fewer "visits during the month" the customer made, the higher the probability that he will go into outflow. At the same time, the most significant feature is the "number of visits per month".