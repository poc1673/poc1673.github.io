# Motivation for Firth's logistic regression:

A key problem which can throw off the performance of logistic regression is the possibility of bias in the data.  In a 1997 paper, David Firth suggested a tweak to the maximum likelihood function to compensate for the bias term. You can find it in [1]. 

Typically, we define the critical value of the maximum likelihood function $l$ as being:

<img src="http://latex.codecogs.com/gif.latex?\nabla l  = U(\theta) = 0" border="0"/>

We may change the maximum likelihood function as $U(\theta) = l'(\theta) = t - K(\theta)$. This additive term $t$ impacts the location of the optimal solution, but doesn't change the overall shape.

In the case of logistic regression, we can define the modified score function as:

$$ U(\beta_r)^* \equiv U(\beta_r) + \frac{1}{2}trace\Big[I(\beta)^{-1}  \frac{\partial I(\beta)}{\partial\beta_r} \Big] = 0 \;\;\; r \in [1,...,k)  $$ 
Where $I(\beta)$ is the information matrix.

While the original aim of this method was to simply decrease the bias (and act as a regularizer for logistic regression), it turns out that it provides a useful solution to a specific problem in classification: separation. 

# Separation of Data

## Definition:
The basis of separation in data is rooted in perfect multicollinearity between the dependent variable and the independent variable.

Separation occurs when we can define some hyperplane in the data which perfectly splits the two classes in a binary logistic regression. 

Consider the case where you have some variable $x \in \mathbb{R}$ as a predictor of a binary event $y \in \{0,1\}$. In this contrived example let:

$$ y =  \begin{cases} 
      0 & \text{if } x< 4 \\
      \frac{100-x}{100} & 4 \leq x 
   \end{cases}
 $$

Consider the case that you are fitting a model of the form:


$$ p = \frac{1}{1+exp(-(\beta_0 + \beta_1 x))} $$
IE - a very simple binary logistic regression model.

If this is the case, then $\beta_1$ will explode. The intuitive explanation for this is that the model should become more accurate as $\beta_1$ gets larger. Here's a trivial example with 10000 samples:


```r
set.seed(13)
y <- c(rep(0,5000),rep(1,5000))
x <- c(runif(5000,min=0,max = 3.999999),runif(5000,min=4,max = 10) )
example_model <- glm(formula = y~x,family = "binomial")
example_model$coefficients
```

```
## (Intercept)           x 
##   -46761.11    11691.04
```

## Quasi-complete Separation


If a data-set doesn't meet what was described earlier as separation, the data-set may still be quasi-complete separation if the value of $y$ splits the values of $x$. To reverse the example above:

$$ x \in  \begin{cases} 
      [0,4) & \text{if } y =0 \\
      [4,\infty) & \text{if } y =1   
   \end{cases}
 $$

would be an example of quasi-complete separation.


# An Example



```r
library(pacman)
p_load(logistf,data.table,knitr)
data(sex2)
firth_reg <- logistf(data = sex2,case~.)
normal_reg <- glm(data = sex2,case~.,family = "binomial")
```

Knowing what I already know about this data-set, let's look at the relationship of dia with censor:


```r
plot(x = sex2$case,sex2$dia)
```

![](2020-03-14-Firth_s_Logistic_Regression_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

We can see that we have quasi-separation for dia. $dia =0$ only if $case = 1$. Let's compare the results of a regression model with vanilla logistic regression, and Firth's logistic regression as implemented in George Heinze's *logistf* package:


```r
kable(data.frame("Logistic Regression" = normal_reg$coefficients,"Firth Logistic Regression" = firth_reg$coefficients),digits = 3)
```

               Logistic.Regression   Firth.Logistic.Regression
------------  --------------------  --------------------------
(Intercept)                  0.128                       0.120
age                         -1.164                      -1.106
oc                          -0.074                      -0.069
vic                          2.406                       2.269
vicl                        -2.246                      -2.111
vis                         -0.820                      -0.788
dia                         16.734                       3.096
We can see that the there is a marked difference in the coefficient for the dia parameter in the data-set above. Depending on how much we think the results above make sense, we may have more luck with Firth's logistic regression when trying to project case out of the training sample.


```r
kable(normal_reg$coefficients)
```

                        x
------------  -----------
(Intercept)     0.1283465
age            -1.1643939
oc             -0.0735641
vic             2.4059255
vicl           -2.2461950
vis            -0.8201143
dia            16.7342250

# Alternatives to Firth's Logistic Regression

While reviewing this methodology, it occurred to me that in many cases if you are compensating for separation you are compensating for theoretical problems. In that case, this is only appropriate if the modeler has seriously considered the theoretical reasons for why there is separation in the data in the first place. A good example is from the first example from [2] regarding the endometrial cancer study:

> Because the other two risk factors are continuous, exact logistic regression analysis is not available. As the medical investigators were interested in the effect of neovasculization, omission of this risk factor (option 1 of Section 1) cannot be considered here. 

In my specific domain (finance), there generally should be no hard and fast rules dictating the binary outcome. If there are, it's worth looking into the theory behind the model being fit.

# References:
1. Bias reduction of maximum likelihood estimates - David Firth, 1997
2. A solution to the problem of separation in logistic regression - George Heinz and Michael Schemper, 2002
3. Advances in Financial Machine Learning - Marcos Luiz Prado
4. [What is complete or quasi-Complete separation and How do we deal with them ](https://stats.idre.ucla.edu/other/mult-pkg/faq/general/faqwhat-is-complete-or-quasi-complete-separation-in-logisticprobit-regression-and-how-do-we-deal-with-them/)




