# Portfolio-Allocation-Techniques-for-Processing-Covariance-Matrices.
This is the project based on the 3A CentraleSupélec - Portfolio Allocation.

# SUMMARY

  ## 1. Introduction 
  The hypothesis: 
  - We consider the day 1000 as today 
  - We took a risk-free rate of 1% 
  - We don’t allow short selling 
  - We take only 10 assets in order to simplify the lecture of the result and reduce the time of computing 
  - Time metrics were calculated on a 30 or 90 day basis 
  ## 2. Optimisation
  - Build efficient frontiers 
  - The Minimum Variance Portfolio and the Maximum Sharpe Ratio Portfolio 
  - Robust Optimisation 
  ## 3. The techniques for Covariance Matrics
  - Eigenvalue Clipping 
  - Oracle Estimator 
  - Cross Validate Eigenvalue Shrinkage 
  ## 4. Comparison
  - Eigenvalue Distribution 
  - The cost of transaction, turnover and cumulative return
  ## 5. Adding a risk-free asset
  - The Capital Market Line 
  - The metrics with free-risk assets 
  ## 6. Conclusion
  
 # RESULT PRESENTATION
 
 ## Porfolio metrics
 
  - Some metrics can be used to measure the diversification and leverage of the portfolio. For instance, we define the following metrics:

  - The effective portfolio diversification $N_{eff} = \frac{1}{||\mathbf{w}||_2^2}$. It represents the effective number of stocks with a significant amount of money invested,

  - The number of stocks that account for q percent of the total amount of money invested $N_q = \arg\underset{l}{\min}\Sigma_{i=1}^l|w_i| \geq q ||\mathbf{w}||_1$,

  - The gross leverage $G = ||\mathbf{w}||_1^2$. When no short selling is allowed $G=1$. Portfolios with $G > 1$ have an additional intrinsic risk due to high level of short-selling.
 
 ## Covariance Cleaning and Noise Filtering Functions
 
  ### Eigenvalue clipping
  
  When using the in sample covariance matrix, some directions contain more robust information than others. Lower eigenvalues are under-estimated while medium-higher are over-estimated due to some noise component. To avoid over-trusting in a few eigen-directions, and under-trusting in other orthogonal eigen-directions. The simplest filtering approach is clipping the eigenvalue distribution of the correlation matrix.

  The eigenvalue clipping consists of averaging the directions on the bulk to reduce the sample-size error. It is done through the following steps:  
  1. Determine $\Lambda$ and $\mathbf{V}$ such as $\Lambda$ is diagonal and $\Sigma = \mathbf{V}\cdot\Lambda\cdot\mathbf{V}^T$,  
  2. Compute $\Lambda^{(c)}$ a diagonal matrix with:
  $$\lambda_i^{(c)} = \frac{\overset{N}{\underset{j=1}{\Sigma}}\lambda_j\delta(\lambda_j<\lambda_{Max})}{\overset{N}{\underset{j=1}{\Sigma}}\delta(\lambda_j<\lambda_{Max})}$$
  where $(\lambda_j)_{j\in[1,N]}$ are the eigenvalues of $\Sigma$, and $\delta$ is the Heaviside function,  
  3. Compute $\mathbf{C}^{(t)} = \mathbf{V}\cdot\Lambda^{(c)}\cdot\mathbf{V}^T$,  
  4. For $i,j\in[1,N]$, compute $C_{i,j}^{(c)} = \frac{C_{i,j}^{(t)}}{C_{i,i}^{(t)}C_{j,j}^{(t)}}$,  
  5. Finally, compute the eigenclipped covariance matrix $\Sigma^{(c)}$ knowing that $\forall i,j\in[1,N]$, $\Sigma_{i,j}^{(c)} = C_{i,j}^{(c)}\Sigma_{i,i}\Sigma_{j,j}$.

  We can therefore use the in sample eigenvalue clipped covariance matrix when determining our portfolio.
  
  ### Oracle Estimator
  
  The oracle estimator is another way to clean the in sample covariance matrix. Assuming we know the out sample covariance matrix, we would like to adapt the in sample filtered covariance matrix $\Theta$ in order to minimize $d(\Sigma_{out}, \Theta)$. It is obtained with the following steps:  
  1. Determine $\Lambda$ and $\mathbf{V}$ such as $\Lambda$ is diagonal and $\Sigma = \mathbf{V}\cdot\Lambda\cdot\mathbf{V}^T$,  
  2. Compute $\mathbf{O} = diag((\mathbf{V}^T\cdot\Sigma_{out}\cdot\mathbf{V})_{diagonal}$,  
  3. Compute $\Theta = \mathbf{V}\cdot\mathbf{O}\cdot\mathbf{V}^T$.
 
  ### Cross validate eigenvalue shrinkage
  
  *One* of the problems encountered during the previous work is eigenvalues equal to $0$. This occurs in low dimensional regime ($T<N$) or when stocks are highly correlated. The former is very common since having $T>>N$ may induce including some historical patterns that are totally unrelated to the current market states. 

  This part addresses the zero eigenvalues issue using the Cross-Validate Eigenvalue Shrinkage method. It is a cross validation mehtod in which the in-sample is split in two sets ($in\_sample^*$ and $out\_sample^*$) that don't preserve the temporal order. Let's see how to implement it.  
  
  1. In iteration $i$, split the in-sample in $in\_sample^*$ and $out\_sample^*$:  
    - $in\_sample^*$ is a sorted sub-sample of in-sample and $out\_sample^*$ is the left-out,  
    - $out\_sample^*$ should be more than 10\% of in-sample and should be too small,  

  2. In iteration $i$, determine $\Lambda_{in^*_i}$ and $\mathbf{V}_{in^*_i}$ such as $\Lambda_{in^*_i}$ is diagonal and $\Sigma_{in^*_i} = \mathbf{V}_{in^*_i}\cdot\Lambda_{in^*_i}\cdot\mathbf{V}_{in^*_i}^T$,
  
  3. In iteration $i$, compute $\Lambda_{CV_{i}}^{(t)} = (\mathbf{V}_{in^*_i}^T\cdot\Sigma_{out^*_i}\cdot\mathbf{V}_{in^*_i})_{diagonal}$,  
  
  4. Compute $\Lambda_{CV}^{(t)} = \overline{\Lambda_{CV}^{(t)}}$. It is the average of the $\Lambda_{CV_{i}}^{(t)}$.

  At this stage, the average computed eigenvalues are not monotonically decreasing. Therefore, we use the *Isotonic Regression* from the *scikit learn* library. In brief, it is a technique of fitting a free-form line to a sequence of observations such that the fitted line is non-decreasing (or non-increasing) everywhere, and lies as close to the observations as possible.

  5. Compute $\Lambda_{CV}^{(ISO)}$ with the *Isotonic Regression* function,  
  
  6. Determine $\Lambda_{in}$ and $\mathbf{V}_{in}$ such as $\Lambda_{in}$ is diagonal and $\Sigma_{in} = \mathbf{V}_{in}\cdot\Lambda_{in}\cdot\mathbf{V}_{in}^T$,  
  
  7. The CV estimator is defined as follows: $$\Sigma_{CV} = \mathbf{V}_{in}\cdot diag(\Lambda_{CV}^{(ISO)})\cdot\mathbf{V}_{in}^T\quad .$$

  If the sample-size error is the only source of noise, then the estimator converges to the oracle estimator.
  
  
  
  ### Results
  
  - Portfolios with unfiltered covariance matrix
  
  ![image](https://user-images.githubusercontent.com/110284601/184870132-11b4917c-d317-46cf-8baf-5dad530221e0.png)

  ```
    The MV portfolio has the following weights
  [0.22 0.25 0.   0.08 0.09 0.12 0.   0.14 0.   0.08],
  has a risk of 11.26% and a return of 6.78%.
  It also has an effective diversification of 5.84 and 90% of weight are in  6 stocks.

  The maximum SR portfolio has the following weights
  [0.   0.   0.   0.   0.   0.   0.18 0.82 0.   0.  ],
  has a risk of 18.32% and a return of 57.99%.
  It also has an effective diversification of 1.41 and 90% of weight are in  2 stocks.

  The robust portfolio has the following weights
  [0.   0.34 0.   0.   0.03 0.   0.09 0.51 0.03 0.  ],
  has a risk of 14.67% and a return of 42.38%.
  It also has an effective diversification of 2.59 and 90% of weight are in  3 stocks.
  ```
  
  ### Portfolios with eigenvalue clipped covariance
  
  ![image](https://user-images.githubusercontent.com/110284601/184870294-61ac3708-c377-4b45-8cb8-883a0847d130.png)
  
  ```
  The MV portfolio has the following weights
  [0.2  0.29 0.   0.01 0.12 0.03 0.   0.17 0.01 0.16],
  has a risk of 5.92% and a return of 11.85%.
  It also has a effective diversification of 5.03 and 90% of weight are in  5 stocks.

  The maximum SR portfolio has the following weights
  [0.   0.37 0.   0.   0.   0.   0.03 0.6  0.   0.  ],
  has a risk of 9.16% and a return of 42.25%.
  It also has a effective diversification of 2.0 and 90% of weight are in  2 stocks.

  The robust portfolio has the following weights
  [0.   0.39 0.   0.   0.   0.   0.03 0.58 0.   0.  ],
  has a risk of 9.06% and a return of 41.76%.
  It also has a effective diversification of 2.04 and 90% of weight are in  2 stocks.
  ```
  
  ```
  Using this portfolios in the future, we get: 
   - A return equal to 33.45% and a risk equal to 11.99% for the robust Portolio.
   - A return equal to 28.4% and a risk equal to 7.58% for the MV Portolio.
   - A return equal to 32.7% and a risk equal to 12.02% for the maximum SR Portolio.
  ```
  
  ### Portfolios with the oracle estimator
  
  ![image](https://user-images.githubusercontent.com/110284601/184870426-9e9b8362-eb58-4f48-b215-e87cc46e2fbe.png)
  
  ```
  The MV portfolio has the following weights
  [0.24 0.17 0.   0.1  0.17 0.07 0.   0.16 0.01 0.08],
  has a risk of 12.59% and a return of 5.36%.
  It also has a effective diversification of 6.15 and 90% of weight are in  6 stocks.

  The maximum SR portfolio has the following weights
  [0.   0.1  0.   0.   0.   0.   0.24 0.66 0.   0.  ],
  has a risk of 20.85% and a return of 57.0%.
  It also has a effective diversification of 1.99 and 90% of weight are in  3 stocks.

  The robust portfolio has the following weights
  [0.07 0.3  0.   0.02 0.03 0.   0.06 0.46 0.05 0.  ],
  has a risk of 15.55% and a return of 35.47%.
  It also has a effective diversification of 3.14 and 90% of weight are in  5 stocks.
  ```
  
  ```
  Using this portfolios in the future, we get: 
   - A return equal to 26.91% and a risk equal to 10.12% for the robust Portolio.
   - A return equal to 14.32% and a risk equal to 7.38% for the MV Portolio.
   - A return equal to 29.41% and a risk equal to 12.65% for the maximum SR Portolio.
  ```
  
  ### Portfolios and Cross Validate Eigenvalue Shrinkage
  
  ![image](https://user-images.githubusercontent.com/110284601/184870595-d024f8fc-2660-4371-831c-0ce822fab30b.png)
  
  ```
  The MV portfolio has the following weights
  [0.2  0.2  0.   0.11 0.1  0.12 0.   0.15 0.01 0.12],
  has a risk of 10.55% and a return of 5.59%.
  It also has a effective diversification of 6.65 and 90% of weight are in  7 stocks.

  The maximum SR portfolio has the following weights
  [0.   0.   0.   0.   0.   0.   0.28 0.7  0.02 0.  ],
  has a risk of 17.91% and a return of 60.97%.
  It also has a effective diversification of 1.74 and 90% of weight are in  2 stocks.

  The robust portfolio has the following weights
  [0.   0.25 0.   0.   0.02 0.   0.21 0.45 0.07 0.  ],
  has a risk of 15.11% and a return of 48.62%.
  It also has a effective diversification of 3.22 and 90% of weight are in  3 stocks.
  ```
  
  ```
  Using this portfolios in the future, we get: 
   - A return equal to 27.51% and a risk equal to 10.79% for the robust Portolio.
   - A return equal to 13.27% and a risk equal to 7.38% for the MV Portolio.
   - A return equal to 26.37% and a risk equal to 12.97% for the maximum SR Portolio.
  ```
  
  ## Eigenvalue distribution
  
  ![image](https://user-images.githubusercontent.com/110284601/184870737-fcb6926a-a45e-412f-968e-fb6f9403021f.png)
  
   ### Time evolution
  
  ![image](https://user-images.githubusercontent.com/110284601/184870806-44751467-ef8a-4dc9-963f-1ac177bffdc4.png)

   ### With free-risk asset
   
   We want to calculate the Capital market line (CML) which is the tangent line drawn from the point of the risk-free asset to the feasible region for risky assets. The tangency point M represents the market portfolio, so named since all rational investors (minimum variance criterion) should hold their risky assets in the same proportions as their weights in the market portfolio.
  
   ![image](https://user-images.githubusercontent.com/110284601/184871028-5c917bdb-97f2-432a-80c5-cb729c6d4294.png)

   ![image](https://user-images.githubusercontent.com/110284601/184871086-8c77f99a-7b79-49ad-a789-0d3afd4bcdc3.png)
   
   We remark that the cumulative return, the risk and the turnover are linear because of the additivity of these metrics for independant stocks (the risk-free asset is not correlated with the market). The growth of the sharpe ratio could be explained by no volatility for the risk-free asset.

   ![image](https://user-images.githubusercontent.com/110284601/184871155-37222988-c307-4750-a624-241c546e2b01.png)

  
  

  
  
