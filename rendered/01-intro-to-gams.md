01 Introduction to GAMs
================
Darren Wong
2024-07-20

- [Links](#links)
- [Introduction](#introduction)
- [GAM Basics](#gam-basics)
- [Motivation - assessing the linear
  approach](#motivation---assessing-the-linear-approach)
  - [Anatomy of a GAM](#anatomy-of-a-gam)
- [Basis functions and smoothing](#basis-functions-and-smoothing)
  - [Big Wiggly Style (Wiggliness)](#big-wiggly-style-wiggliness)
  - [Number of basis functions](#number-of-basis-functions)

# Links

- [Noam Ross’ GAMs in R course - Chapter
  1](https://noamross.github.io/gams-in-r-course/chapter1)

# Introduction

- We often face a tension between interpretability (think simple
  regression - easy to understand and use for inference) and flexibility
  (think neural networks - can model complex relationships but aren’t
  explainable).

  - **Generalised Linear Models** (GAMs) provide a middle ground; they
    can model complex, non-linear relationships - but can still be used
    for inferential statistics.

# GAM Basics

- We use **smooths/splines** in GAMs to fit data of a wide variety of
  shapes. To tell R we want to use a spline for a variable, we wrap it
  in `s()` when specifying a fitting formula.
- The flexible splines in GAMs are actually the sum of multiple smaller
  functions called **basis functions**. Each basis function has a
  coefficient and an intercept associated it - which are parameters in
  the model - providing fine control over each one.
  - This is more complex than linear models that only have one intercept
    and coefficient associated with each pair of dependent and
    independent variables
  - We can extract coefficients from GAM fitted objects using `coef()`
  - The figure below shows each basis function with an equal coefficient
    (a), and the same functions with trained coefficients (b)

![](images/clipboard-2542440245.png)

# Motivation - assessing the linear approach

- We’ll first fit a linear model to the data to assess its
  appropriateness for this data.

``` r
mcycle <- MASS::mcycle

# Fit LM
lm_mod <- lm(accel ~ times, data = mcycle)

mcycle %>% 
  ggplot(aes(x = times, y = accel)) +
  geom_point() +
  geom_smooth(method = 'lm')
```

    ## `geom_smooth()` using formula = 'y ~ x'

![](/Users/darrenwong/Documents/Projects/gam-course/rendered/01-intro-to-gams_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

- Now we’ll fit a GAM using the `mgcv` package, we can see that the
  fitted model is much closer to the data at more x-values.

``` r
# Fit GAM
gam_mod <- mgcv::gam(accel ~ s(times), data = mcycle)

# Convert to a mgcViz object
draw(gam_mod, residuals = T)
```

![](/Users/darrenwong/Documents/Projects/gam-course/rendered/01-intro-to-gams_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

## Anatomy of a GAM

- As mentioned earlier, GAMs are composed of basis functions
- We can extract the coefficients of these basis functions using the
  `coef()` function - we see there are 9 basis functions and one
  intercept term

``` r
coef(gam_mod)
```

    ## (Intercept)  s(times).1  s(times).2  s(times).3  s(times).4  s(times).5 
    ##  -25.545865  -63.718008   43.475644 -110.350132  -22.181006   35.034423 
    ##  s(times).6  s(times).7  s(times).8  s(times).9 
    ##   93.176458   -9.283018 -111.661472   17.603782

# Basis functions and smoothing

- GAMs can fit highly complex relationships, but this leads to potential
  for overfitting the data. In this section, we’ll look at how smoothing
  can help combat this issue.

  - To elaborate, we want a model that fits close to the *relationship*
    between the variables, but not the *noise* in the data.

## [Big Wiggly Style](https://www.youtube.com/watch?v=0pdH1Ba3A7Y&themeRefresh=1) (Wiggliness)

- How well a GAM fits the data is controlled by something called
  **likelihood**; how complex curves are allowed to be is controlled by
  **wiggliness**. Good fits (no overfitting) are achieved through a
  balance of the two. The relationship between the two can be shown as
  follows, with the $\lambda$ parameter controlling the balance (this is
  optimised during model fitting).

$$Fit = Likelihood - \lambda \times Wiggliness$$

![](images/clipboard-2115590685.png)

<center>
<i>We can see above the effects of changing λ. Too large a λ we get no
complexity, too small and we get clear overfitting. The optimised λ
value appropriately fits the relationship in the data, and not the
noise.</i>
</center>

- We generally allow R to choose the optimal smoothing parameter
  ($\lambda$) for us, but we can set it ourselves, either for the whole
  model, or per smooth as follows.
- The `mgcv` package provides for multiple methods of selecting the
  optimal smooth. Noam (as well as other GAM experts) recommends
  Restricted Maximum Likelihood (REML) as it is more likely to give you
  stable results. We can set this with the `method` argument.

``` r
# Setting the smoothing parameter
gam(y ~ s(x), data = dat, sp = 0.1)
gam(y ~ s(x, sp = 0.1), data = dat)

# Setting smoothing to restricted maximum likelihood
gam(y ~ s(x), data = dat, method = "REML")
```

## Number of basis functions

- The other factor that can affect how wiggly a GAM is is the number of
  basis functions used in each smooth function.

![](images/clipboard-4025323901.png)

- We can set this parameter for each smoothing function (`s()`) in the
  model manually with the `k` argument. If we set it too low, the model
  won’t capture all compexity in the model - but if it’s too high, the
  wiggliness parameter will help to dampen the overfitting. However, we
  need to be careful, as the more basis functions we have, the more
  parameters we need to estimate.

  - There is a way to test if we have the appropraite number of basis
    functions in each smooth which we’ll discuss later in the course.

``` r
gam(y ~ s(x, k = 3), data = dat, method = "REML")
gam(y ~ s(x, k = 10), data = dat, method = "REML")
```
