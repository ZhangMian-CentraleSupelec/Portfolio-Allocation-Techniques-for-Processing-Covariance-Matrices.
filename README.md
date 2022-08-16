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
 
  ## Portfolio Metrics
  - Some metrics can be used to measure the diversification and leverage of the portfolio. For instance, we define the following metrics:

  - The effective portfolio diversification $N_{eff} = \frac{1}{||\mathbf{w}||_2^2}$. It represents the effective number of stocks with a significant amount of money invested,

  - The number of stocks that account for q percent of the total amount of money invested $N_q = \arg\underset{l}{\min}\Sigma_{i=1}^l|w_i| \geq q ||\mathbf{w}||_1^2$,

  - The gross leverage $G = ||\mathbf{w}||_1^2$. When no short selling is allowed $G=1$. Portfolios with $G > 1$ have an additional intrinsic risk due to high level of short-selling.

  ## Creating in and out samples

  - First, we will use the first 1000 days in sample and 252 days that follow out sample to build the efficient frontiere. For all the following, we will note $in\_data$ when talking about the in sample data, and $out\_data$ when talking about the out sample data.

  - Note that, when importing the data, we need to make some changes. Indeed, we drop all the columns in $in\_data$ that contain a $NaN$ value. Then, we compute the vanilla returns. For $out\_data$, we keep the same columns as in_data, compute the vanilla returns, and finally replace $NaN$ values with 0.

  - Reminder: if $r_X^t$ is the log return of an asset $X$ at a time $t$, then $e^{r_X^t} - 1$ is the corresponding vanilla return.

  ## Building the efficient frontiers
  - Imagine a simple portfolio made up of only two assets A and B.  
  - You can plot all possible combinations of asset weights (50% asset A, 50% asset B, 75% asset A, 25% asset B etc...) and calculate the associated risk and return.

  - The “Efficient frontier” traces all optimal combinations i.e. which offer the highest return per unit of risk. Any portfolio combination that lies below this frontier is not optimal because a higher return can be obtained for the same level of risk (or the same return can be obtained for a lower level of risk).

  - To obtain the optimal portfolio for a given level of risk, we need to solve the following equation:

  $$\begin{equation}
    \begin{cases}
          \arg\max \mathbf{\mu}^T\cdot\mathbf{w} & ,\\
          s.c \quad \mathbf{w}^T\cdot\Sigma\cdot\mathbf{w} \leq risk\_level & .
    \end{cases}
  \end{equation}$$

  - This problem is equivalent to minimising the risk for a given level of return:

  $$\begin{equation}
    \begin{cases}
          \arg\min \mathbf{w}^T\cdot\Sigma\cdot\mathbf{w} & ,\\
          s.c \quad \mathbf{\mu}^T\cdot\mathbf{w} \geq return\_level & .
    \end{cases}
  \end{equation}$$

  - We use the latter in the algorithm below to compute an efficient portfolio.

  ## Efficient Frontier in sample

  - Among all efficient portfolios, there are some that stand out and are considered as optimal portfolios.

  - In this work, we will consider the Minimum Variance (MV) Portfolio, and the Maximum Sharpe Ratio (MSR) Portfolio. These two portfolios can be computed with analytical formulas:
  
    $\mathbf{w}_{MV} = \frac{\Sigma^{-1}\cdot\mathbf{1}}{\mathbf{1}^T\cdot\Sigma^{-1}\cdot\mathbf{1}}\quad,$

    $\mathbf{w}_{MSR} = \frac{\Sigma^{-1}\cdot\mathbf{\mu}}{\mathbf{1}^T\cdot\Sigma^{-1}\cdot\mathbf{\mu}}\quad.$
  
  - The MV Portfolio is the portfolio that minimizes the risk without a return constraint. Thus, the portfolio doesn't depend on returns.  
  - The MSR Portfolio is the portfolio that maximizes the return (in excess of the risk-free rate) per unit of risk. It is also the portfolio at the intersection of the efficient frontier and the capital market line.


  - Once we have computed the weights of the portfolios, we can compute the expected risk and return using the following formulas:
  
    $r = \mathbf{\mu}^T\cdot\mathbf{w}$

    $risk = \sqrt{252*\mathbf{w}^T\cdot\Sigma\cdot\mathbf{w}}$
  
  ## Efficient frontier out sample
  
  - The risks and returns of the $out\_data$ are computed using the weights from $in\_sample$. Indeed the out-sample data reprensents the future. That is to say that such data is not available when we build our portfolios. Hence, we are trying to see how the portfolios built from $in\_data$ performed.
  
  ![image](https://user-images.githubusercontent.com/110284601/184860710-db46cc29-8f33-41cc-ac93-0adedc53495a.png)
  
  Is seems there is a mistake in the computing of the MSR Portfolio. It should have been on the curve.
  
  In the above plots, we can notice the following:
  
  The efficient frontier in the out-of-sample shrinks and shifts to the right,
  
  The portfolios built with in sample data have lower performances when we can use the out sample data. For a same level of risk, the return for the out sample could be 5 times lower than the return of the in sample,
  
  There is also a clear difference in the shape of the sharpe ratio curve. We notice that for the in sample, the sharpe ratio increases dramatically and then keeps at a steady level. For the out sample data, the sharpe ratio reaches a peak, after which it decreases gradually while the risk keeps increasing.
  
  A few reasons to explain the deviation:
  
  In this case, because the time window for the in sample is 1000 days, so it could be quite normal that there is some significant change among those equities and so it will influence the portfolio allocation. In the second part, we will see how the time window affects the portfolios,
  
  In the construction of the efficient line, we suppose that we could both short and long an asset, so it will give our model the ability to create a more complex model to get a better result than traditional allocation (only long no short). It is more likely to overfit our training set and that will cause a deviation when we realize it on out sample data set.
  
  An explanation for the sharpe ratio:
  
  Because in our model, we can short or long the asset as we want, so our algorithm can find the optimal allocation of the asset and also multiply it. That's the reason why the sharpe ratio could stay at the highest level and not like traditional sharpe ratio, which ofter decrease as the risk increase. And also that's the form when we put the allocation in the out sample case.

  
  ## In-sample eigenvalue clipping
  
  In this part, we will work on covariance cleaning. When using the in sample covariance matrix, some directions contain more robust information than others. Lower eigenvalues are under-estimated while medium-higher are over-estimated due to some noise component. To avoid over-trusting in a few eigen-directions, and under-trusting in other orthogonal eigen-directions. The simplest filtering approach is clipping the eigenvalue distribution of the correlation matrix.

The eigenvalue clipping consists of averaging the directions on the bulk to reduce the sample-size error. It is done through the following steps:  
1. Determine $\Lambda$ and $\mathbf{V}$ such as $\Lambda$ is diagonal and $\Sigma = \mathbf{V}\cdot\Lambda\cdot\mathbf{V}^T$,  
2. Compute $\Lambda^{(c)}$ a diagonal matrix with:
$$\lambda_i^{(c)} = \frac{\overset{N}{\underset{j=1}{\Sigma}}\lambda_j\delta(\lambda_j<\lambda_{Max})}{\overset{N}{\underset{j=1}{\Sigma}}\delta(\lambda_j<\lambda_{Max})}$$
where $(\lambda_j)_{j\in[1,N]}$ are the eigenvalues of $\Sigma$, and $\delta$ is the Heaviside function,  
3. Compute $\mathbf{C}^{(t)} = \mathbf{V}\cdot\Lambda^{(c)}\cdot\mathbf{V}^T$,  
4. For $i,j\in[1,N]$, compute $C_{i,j}^{(c)} = \frac{C_{i,j}^{(t)}}{C_{i,i}^{(t)}C_{j,j}^{(t)}}$,  
5. Finally, compute the eigenclipped covariance matrix $\Sigma^{(c)}$ knowing that $\forall i,j\in[1,N]$, $\Sigma_{i,j}^{(c)} = C_{i,j}^{(c)}\Sigma_{i,i}\Sigma_{j,j}$.

We can therefore use the in sample eigenvalue clipped covariance matrix when determining our portfolio.
  
![image](https://user-images.githubusercontent.com/110284601/184862602-2bb8b25c-3eec-44ee-9009-fde8ddae7179.png)

In the portfolio we have 468 stocks and therefore 468 eigenvalues. With the method of eigenvalue clipping, we adjust the eigenvalues by keeping the largest eigenvalue, and averaging the other ones (cf orange curve). We use then the clipped eigenvalues to adjust the covariance matrix (cf green curve).

This method decreases the "noise" in the in data eigenvalues and thus is more likely to output a more adapted covariance matrix.

In the portfolio we have 468 stocks and therefore 468 eigenvalues. With the method of eigenvalue clipping, we adjust the eigenvalues by keeping the largest eigenvalue, and averaging the other ones (cf orange curve). We use then the clipped eigenvalues to adjust the covariance matrix (cf green curve).

This method decreases the "noise" in the in data eigenvalues and thus is more likely to output a more adapted covariance matrix.

From the plots above, we can observe the following:

- In both cases, the efficient frontier in the out-of-sample shrinks and shifts to the right,
- The eigenvalue clipping method decreases the global risk for given level of returns,
- For the sharpe ratio, with eigenvalue clipping, the sharpe ratio will decrease gradually after reaching a peak. The shape of the curve of the in sample starts to fit the shape of the curve of the out sample.

A few explanations about the efficient frontiers and sharpe ratio:

- After the eigenvalue clipping, we can allocate a portfolio with very little risk(volatility), therefore we can get a sharpe ratio far more higher than the unfiltered version,
- The efficient frontiers for the unfiltered and filtered covariance matrices look the same for high levels of risk. It is therefore understandable that the sharpe ratio converges to a similar level.


