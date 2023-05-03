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
  
  - Eigenvalue clipping is useful ? It could decrease the return and increase the volatility. Sometimes it's a terrible techinique.
  
  ![image](https://user-images.githubusercontent.com/110284601/236056245-f5a69b93-ebcb-4171-a770-325a6cb0e682.png)

  - Oracle estimator is useful ? Very limited.
  
  ![image](https://user-images.githubusercontent.com/110284601/236056671-9e8b0621-c903-447e-a1a6-822b90d6c2c0.png)

  
