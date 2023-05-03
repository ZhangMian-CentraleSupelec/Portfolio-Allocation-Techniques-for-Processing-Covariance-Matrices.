# Portfolio-Allocation-Techniques-for-Processing-Covariance-Matrices.


### Key Result

- The eigenvalue clipping is not useful.
- The oracle estimator could improve the performance very very little.




# SUMMARY

  ## 1. Introduction 
  The hypothesis: 
  - I take the sp500 stock price since 2016-01-01.
  - Assume risk free rate is zero.
  - No short position.
  
  ## 2. Framework
  - Build efficient frontiers 
  - The Minimum Variance Portfolio and the Maximum Sharpe Ratio Portfolio 
  - Eigenvalue clipping and Cross Validation Oracle Estimator.
  
 # RESULT PRESENTATION
 
 ## Max Sharpe ratio strategy
![image](https://user-images.githubusercontent.com/110284601/236056053-6e4e39f3-c445-4e12-9266-94d2fa5907fc.png)

From the chart above:
- The estimation of the return is very unstable. 

- The estimation of the variance is a little unstable. 

- The difference of the sharpe-ratio is mainly driven by estimation of the return.

 
  
  ## Results 
  
  - Eigenvalue clipping is useful ? It could decrease the return and increase the volatility. Sometimes it's a terrible techinique.
  
  ![image](https://user-images.githubusercontent.com/110284601/236056245-f5a69b93-ebcb-4171-a770-325a6cb0e682.png)

  - Oracle estimator is useful ? Very limited.
  
  ![image](https://user-images.githubusercontent.com/110284601/236056671-9e8b0621-c903-447e-a1a6-822b90d6c2c0.png)

  
