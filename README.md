## Welcome itdr GitHub Page

## Overview

**itdr** is a system for estimating a basis of the central and central mean subspaces in regression by using integral transformation methods. This  `vignette` demonstrate the usage of functions in **itdr** package over `automobile`, `Recumbent` and `PDB` datasets.

## Chapter 1: Installation 
### 1.1: Install **itdr** package

Installation can be done for **itdr** R package in three ways.

* From the Comprehensive R Archive Network (CRAN): Use `install.packages()` function in R. Then, import `itdr` package into working session using `library()` function. That is,
```{r eval=FALSE, include=TRUE}
install.packages("itdr")
library(itdr)
```

* From binary source package: Use `intall.package()` function in R. Then, import **itdr** package into working session using `library()` function. That is,
```{r eval=FALSE, include=TRUE}
install.packages("~/itdr.zip")
library(itdr)
```

* From GitHub: Use `install_github()` function in R `devtools` package as follows.
 
```{r eval=FALSE, include=TRUE}
library(devtools)
install_github("TharinduPDeAlwis/itdr")
library(itdr)
```

## Chpater 2: Functions related to the Fourier and Convolution transformation methods to estimate Sufficient Dimension Reduction (SDR) subspaces

In this section, we describe the functions in **itdr** package which use Fourier transformation method to estimate sufficient dimension reduction (SDR) subspaces in regression. We only demonstrate the Fourier transformation method. However, by passing the argument `method="CM"` to the `itdr()` function, convolution transformation method can be obtained. 

Before estimating the SDR subspaces, it is required to estimate the dimension (d) of the SDR subspace and tuning parameters `sw2`, and `st2`.  In Section 2.1, the estimation of dimension (d) is demonstrated.  The estimation of the tuning parameter `sw2` for both subspaces, i.e., for the central subspace (CS) and the central mean subspace (CMS), is explained in Section 2.2.1. Moreover, the estimation of `st2` for the central subspace (CS) is explained in Section 2.2.2. Finally, the use of `itdr()` function to estimate the central subspace is demonstrated in Section 2.3.

### 2.1: Estimating the dimension (d) of sufficient dimension reduction (SDR) subspaces

Bootstrap estimation procedure is used to estimate the unknown dimension (d) of sufficient dimension reduction subspaces, for more details see Zhu and Zeng (2006).  The `d.boots()` function can be used to estimate the dimension (d). Now, let's estimate the dimension `d` of the central subspace of the `automobile` dataset, here we use the response variable y, and the predictor variables x as mentioned in Zhu and Zeng (2006). We  need to pass the arguments to the `d.boots()` function as `space="pdf"` to estimate the CS, `xdensity="normal"` for assuming normal density for the predictors, and `method="FM"` for useing the Fourier transformation method.
```{r eval=FALSE, include=TRUE}
#Install package
library(itdr)
# Use dataset available in itdr package
data(automobile)
head(automobile)
automobile.na=na.omit(automobile)
# prepare response and predictor variables 
auto_y=log(automobile.na[,26])
auto_xx=automobile.na[,c(10,11,12,13,14,17,19,20,21,22,23,24,25)]
auto_x=scale(auto_xx) # Standardize the predictors
# call to the d.boots() function with required arguments
d_est=d.boots(auto_y,auto_x,plot=TRUE,space="pdf",xdensity = "normal",method="FM")
auto_d=d_est$d.hat
# Estimated d_hat=2
```

Here, the estimate of the dimension of the central subspace for 'automobile' data is 2, i.e., d_hat=2.

### 2.2: Estimating tuning parameters and bandwidth parameters for Gaussian kernel density estimation

There are two tuning parameters that need to be estimated in the process of estimating SDR subspaces using the Fourier method: namely `sw2` and `st2`. The `sw2` required in both the central mean (CMS) and the central subspace (CS). However, the `st2` required only in the central subspace. The code in Section 2.2.1 demonstrates the use of function `wx()` to estimate the tuning parameter `sw2`, and the use of the function `wy()` to estimate the tuning parameter `st2` is described in Section 2.2.2.

### 2.2.1: Estimate `sw2`

To estimate the tuning parameter `sw2`, we can use `wx()` function with the subspace option either `space="pdf"` for the CS and `space="mean"` for the CMS. During the estimation process, the other parameters are fixed. The following R code chunk demonstrates the estimation of `sw2` for the central subspace.

```{r eval=FALSE, include=TRUE}
auto_d=2 #The estimated value from Section 2.1
auto_sw2=wx(auto_y,auto_x,auto_d,wx_seq=seq(0.05,1,by=0.01),space="pdf",method="FM")
auto_sw2$wx.hat # we get the estimator for sw2 as 0.14
```

### 2.2.2: Estimate `st2`

To estimate the tuning parameter `st2`, we can use `wy()` function. Here, the other parameters are fixed. Notice that we do not need to specify the `space`, because the tuning parameter `st2` only required for the central subspace (CS).
```{r eval=FALSE, include=TRUE}
auto_d=2 # Estimated value from Section 2.1
auto_st2=wy(auto_y,auto_x,auto_d,wx=0.1,wy_seq=seq(0.1,1,by=0.1),xdensity="normal",method="FM")
auto_st2$wy.hat # we get the estimator for st2=0.9 
```
### 2.2.3: Estimate the bandwidth (`h`) of the Gaussian kernel density function

If the distribution function of the predictor variables is unknown, then we use the Gaussian kernel density estimation to approximate the density function of the predictor variables. However, the bandwidth parameter needs to be estimated when `xdensity="kernel"` is used. The `wh()` function uses the bootstrap estimator to estimate the bandwidth of the Gaussian kernel density estimation. 
```{r eval=FALSE, include=TRUE}
h_hat=wh(auto_y,auto_x,auto_d,wx=5,wy=0.1,wh_seq=seq(0.1,2,by=.1),B=50,space = "pdf",xdensity = "kernel",method="FM")
#Bandwidth estimator for Gaussian kernel density estimation for central subspace
h_hat$h.hat #we have the estimator as h_hat=1
```

### 2.3: Estimate SDR subspaces

We have described the estimation procedure of the tunning parameters in the Fourier method in Sections 2.1-2.2. Now, we are ready to estimate the SDR subspaces. Zhu and Zeng (2006) used the Fourier method to facilitate the estimation of the SDR subspaces when the predictors are following a multivariate normal distribution. However, when the predictor variables is following an elliptical distribution  or more generally when the distribution of the predictors is unknown, the predictors' distribution function is approximated by using the Gaussian kernel density estimation (Zeng and Zhu, 2010). The `itdr()` function can be used to estimate the SDR subspaces under `FM` method as follows.  Since the default setting of the `itdr()` function has `method="FM"`, It is optional to specify the method as "FM". 

```{r eval=FALSE, include=TRUE}
library(itdr)
data(automobile)
head(automobile)
df=cbind(automobile[,c(26,10,11,12,13,14,17,19,20,21,22,23,24,25)])
dff=as.matrix(df)
automobi=dff[complete.cases(dff),]
d=2; # Estimated value from Section 2.1
wx=.14 # Estimated value from Section 2.2.1
wy=.9  # Estimated value from Section 2.2.2
wh=1.5  # Estimated value from Section 2.2.3
p=13  # Estimated value from Section 2.3
y=automobi[,1]
x=automobi[,c(2:14)]
xt=scale(x)
#Distribution of the predictors is a normal distribution
fit.F_CMS=itdr(y,xt,d,wx,wy,wh,space="pdf",xdensity = "normal",method="FM")
round(fit.F_CMS$eta_hat,2)
#Distribution of the predictors is a unknown (using kernel method)
fit.F_CMS=itdr(y,xt,d,wx,wy,wh,space="pdf",xdensity = "kernel",method="FM")
round(fit.F_CMS$eta_hat,2)
```

## Chapter 3: Functions related with estimating the central mean subspace using Iterative Hessian Transformation (IHT)

The `itdr()` function also can be used to estimate the central mean subspace in regression by using iterative Hessian transformation (IHT) method as described in the following R chunk on `Recumbent` dataset available in **itdr** package. Notice that the inputs are the method of estimation (`method=iht`), the response vector (y), design matrix of the predictors (x), and the dimension (d) of the CMS. The response and predictors are chosen as same as Cook and Li (2002).  

```{r eval=FALSE, include=TRUE}
library(itdr)
data("Recumbent")
Recumbent.df=na.omit(Recumbent)
y=Recumbent.df$outcome
X1=log(Recumbent.df$ast)
X2=log(Recumbent.df$ck)
X3=log(Recumbent.df$urea)
p=3
x=matrix(c(X1,X2,X3),ncol=p)
d=2
fit.iht_CMS=itdr(y,x,2,method="iht")
fit.iht_CMS$eta_hat
```

## Chapter 4: Functions related with estimating the central subspace in regression using Fourier transformation approach on inverse dimension reduction

In this section, we demonstrate the functions in **itdr** package, which related to the Fourier transformation approach on inverse dimension reduction. In Section 4.1, we demonstrate the use of function to estimate the dimension of the central subspace using the Fourier transformation approach on inverse dimension reduction. The estimation of the CS is described in Section 4.2.

### 4.1: Estimating d

The estimation of the dimension of the CS can be achieved using  `d.test()` function which gives outputs of three different p-values for three different test statistics; Weighted Chi-square test statistic (Weng and Yin, 2018), Scaled test statistic (Bentler and Xie, 2000), and Adjusted test statistic (Bentler and Xie, 2000). Suppose `m` is the candidate dimension of the CS to be tested `(H_0: d=m)`, then, the following R code shows the testing a candidate value `m` (<p) for dimension of the CS of the planning database (PDB).

```{r eval=FALSE, include=TRUE}
library(itdr)
data(PDB)
colnames(PDB)=NULL
p=15
#select predictor vecotr (y) and response variables (X) according to Weng and Weng and Yin, (2018).
df=PDB[,c(79,73,77,103,112,115,124,130,132,145,149,151,153,155,167,169)]
dff=as.matrix(df)
#remove the NA rows
planingdb=dff[complete.cases(dff),]
y=planingdb[,1] #n-dimensionl response vector
x=planingdb[,c(2:(p+1))] # raw desing matrix
x=x+0.5
# desing matrix after tranformations
xt=cbind(x[,1]^(.33),x[,2]^(.33),x[,3]^(.57),x[,4]^(.33),x[,5]^(.4),
x[,6]^(.5),x[,7]^(.33),x[,8]^(.16),x[,9]^(.27),x[,10]^(.5),
x[,11]^(.5),x[,12]^(.33),x[,13]^(.06),x[,14]^(.15),x[,15]^(.1))
m=1
W=sapply(50,rnorm)
#run the hypothsis tests
d.test(y,x,m)
```

### 4.2: Estimating central subspace

After selecting the dimension of the CS as described in Section 4.1, then, an estimator for the CS can be obtained by using `invFM()` function. The following R chunk shows the use of `invFM()` to estimate the CS for planning database (PDB).

```{r eval=FALSE, include=TRUE}
library(itdr)
data(PDB)
colnames(PDB)=NULL
p=15
#select predictor vecotr (y) and response variables (X) according to Weng and Weng and Yin, (2018).
df=PDB[,c(79,73,77,103,112,115,124,130,132,145,149,151,153,155,167,169)]
dff=as.matrix(df)
#remove the NA rows
planingdb=dff[complete.cases(dff),]
y=planingdb[,1] #n-dimensionl response vector
x=planingdb[,c(2:(p+1))] # raw desing matrix
x=x+0.5
# desing matrix after tranformations give in Weng and Yin, (2018).
xt=cbind(x[,1]^(.33),x[,2]^(.33),x[,3]^(.57),x[,4]^(.33),x[,5]^(.4),
x[,6]^(.5),x[,7]^(.33),x[,8]^(.16),x[,9]^(.27),x[,10]^(.5),
x[,11]^(.5),x[,12]^(.33),x[,13]^(.06),x[,14]^(.15),x[,15]^(.1))
W=sapply(50,rnorm)
d=1 # estimated dimension of the CS from Section 4.1
betahat <-invFM(xt,y,d,W,F)$beta # estimated basis
betahat
```

## Acknowledgment

The codes for the Fourier transformation and the convolution transformation methods are adapted from the codes provided by Zhu and Zeng (2006). Moreover, those for the elliptically contoured distributed variables and the kernel density estimation methods are essentially a modification of the program provided by Zeng and Zhu (2010). The code for Fourier transforms approach for the inverse dimension reduction method is adapted from the code provided by Weng and Yin (2018).

## References 

* Bentler, P.M., and Xie, J. (2000). Corrections to Test Statistics in Principal Hessian Directions.
_Statistics and Probability Letters_. 47, 381-389.

* Cook R. D., and Li, B., (2002).
Dimension Reduction for Conditional Mean in Regression. 
_The Annals of Statitics_, 30, 455-474.
 
* Weng J. and Yin X. (2018). Fourier Transform Approach for Inverse Dimension Reduction Method. _Journal of Nonparametric Statistics_. 30, 4, 1029-0311.

* Zeng P. and Zhu Y. (2010).
An Integral Transform Method for Estimating the Central Mean and Central Subspaces. _Journal of Multivariate Analysis_. 101, 271--290.
 
* Zhu Y. and Zeng P. (2006). Fourier Methods for Estimating the Central Subspace and Central Mean Subspace in Regression. _Journal of the American Statistical Association_. 101, 1638--1651.
