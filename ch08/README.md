Instrumental Variables
================
null

# College in the county

``` r
model1 <- lm(lwage ~ educ + exper + black + south + married + smsa, data = card) 
  
model1 %>% 
  tidy() %>% 
  mutate_at(vars(estimate, std.error), .funs = ~ round(., 3))
```

    ## # A tibble: 7 x 5
    ##   term        estimate std.error statistic  p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)    5.06      0.064     79.4  0.      
    ## 2 educ           0.071     0.003     20.4  5.08e-87
    ## 3 exper          0.034     0.002     15.4  1.07e-51
    ## 4 black         -0.166     0.018     -9.43 8.24e-21
    ## 5 south         -0.132     0.015     -8.79 2.51e-18
    ## 6 married       -0.036     0.003    -10.5  1.47e-25
    ## 7 smsa           0.176     0.015     11.4  2.28e-29

``` r
ivreg <- iv_robust(lwage ~ educ + exper + black + south + married +
            smsa  | nearc4 + exper + black + south + married + smsa,
          data = card, se_type = "classical") 

# terms(ivreg) %>% 
#   tidy() %>% 
#   mutate_at(vars(estimate, std.error), .funs = ~ round(., 3))

model2 <- lm(educ ~ nearc4 + exper + black + south + married + smsa,
             data = card)

tidy(model2)
```

    ## # A tibble: 7 x 5
    ##   term        estimate std.error statistic  p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)  16.8      0.131      129.   0.      
    ## 2 nearc4        0.327    0.0824       3.97 7.33e- 5
    ## 3 exper        -0.404    0.00894    -45.2  0.      
    ## 4 black        -0.948    0.0905     -10.5  3.32e-25
    ## 5 south        -0.297    0.0791      -3.76 1.73e- 4
    ## 6 married      -0.0727   0.0177      -4.10 4.31e- 5
    ## 7 smsa          0.421    0.0849       4.96 7.47e- 7

``` r
coeftest(model2, vcov = vcovHC, type = "HC3")
```

    ## 
    ## t test of coefficients:
    ## 
    ##               Estimate Std. Error  t value  Pr(>|t|)    
    ## (Intercept) 16.8306965  0.1255116 134.0968 < 2.2e-16 ***
    ## nearc4       0.3272826  0.0806651   4.0573 5.091e-05 ***
    ## exper       -0.4044340  0.0082166 -49.2218 < 2.2e-16 ***
    ## black       -0.9475281  0.0886590 -10.6873 < 2.2e-16 ***
    ## south       -0.2973528  0.0788613  -3.7706  0.000166 ***
    ## married     -0.0726936  0.0176452  -4.1197 3.896e-05 ***
    ## smsa         0.4208945  0.0850215   4.9504 7.814e-07 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

``` r
linearHypothesis(model2, "nearc4 = 0")
```

    ## Linear hypothesis test
    ## 
    ## Hypothesis:
    ## nearc4 = 0
    ## 
    ## Model 1: restricted model
    ## Model 2: educ ~ nearc4 + exper + black + south + married + smsa
    ## 
    ##   Res.Df   RSS Df Sum of Sq      F    Pr(>F)    
    ## 1   2997 11304                                  
    ## 2   2996 11245  1    59.176 15.767 7.334e-05 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

# Fulton fish markets

## OLS

``` r
lm(log(quantity) ~ log(price) + mon + tues + wed + thurs + time, data = fish) %>% 
  tidy()
```

    ## # A tibble: 7 x 5
    ##   term        estimate std.error statistic  p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)  8.30      0.203      40.9   5.81e-60
    ## 2 log(price)  -0.549     0.184      -2.98  3.70e- 3
    ## 3 mon         -0.318     0.227      -1.40  1.66e- 1
    ## 4 tues        -0.684     0.224      -3.06  2.94e- 3
    ## 5 wed         -0.535     0.221      -2.42  1.74e- 2
    ## 6 thurs        0.0683    0.221       0.308 7.59e- 1
    ## 7 time        -0.00125   0.00264    -0.473 6.37e- 1

``` r
fish
```

    ## # A tibble: 97 x 11
    ##    quantity price   mon  tues   wed thurs speed2 wave2 speed3 wave3  time
    ##       <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>  <dbl> <dbl>  <dbl> <dbl> <dbl>
    ##  1     4080 0.700     1     0     0     0     15   7.5     20   9       1
    ##  2     3466 1.01      0     0     1     0     10   5       20   7.5     2
    ##  3     2295 1.39      0     0     0     1     10   6       20   4       3
    ##  4     1870 1.78      0     0     0     0     15   6       20   5       4
    ##  5     6885 0.827     1     0     0     0     10   3.5     20   3.5     5
    ##  6     5605 0.890     0     1     0     0     15   4.5     15   3.5     6
    ##  7     4959 0.792     0     0     1     0     10   4.5     20   3.5     7
    ##  8     2185 0.926     0     0     0     1     10   4       20   4.5     8
    ##  9     9320 0.700     0     0     0     0     10   4       15   4.5     9
    ## 10     6793 1.49      1     0     0     0     20  12.5     20   3.5    10
    ## # ... with 87 more rows

## 2SLS with average wave height instrument

``` r
fs_waveheight <- lm_robust(log(price) ~ wave2 + mon + tues + wed + thurs + time, 
                           data = fish, 
                           se_type = "HC3")

tidy(fs_waveheight) %>% mutate_at(vars(estimate, std.error), .funs = ~ round(., 3))
```

    ##          term estimate std.error   statistic      p.value     conf.low
    ## 1 (Intercept)   -0.706     0.166 -4.24413327 5.321713e-05 -1.035944441
    ## 2       wave2    0.103     0.022  4.69845832 9.372087e-06  0.059321707
    ## 3         mon   -0.036     0.122 -0.29961885 7.651588e-01 -0.278380318
    ## 4        tues    0.007     0.131  0.05522983 9.560778e-01 -0.252341049
    ## 5         wed    0.083     0.120  0.68800464 4.932193e-01 -0.155741331
    ## 6       thurs    0.136     0.111  1.22918563 2.222062e-01 -0.083887843
    ## 7        time   -0.002     0.001 -1.53424334 1.284778e-01 -0.004834191
    ##       conf.high df    outcome
    ## 1 -0.3753288327 90 log(price)
    ## 2  0.1462408240 90 log(price)
    ## 3  0.2054168128 90 log(price)
    ## 4  0.2667724778 90 log(price)
    ## 5  0.3207574879 90 log(price)
    ## 6  0.3561393072 90 log(price)
    ## 7  0.0006211843 90 log(price)

``` r
linearHypothesis(fs_waveheight, "wave2 = 0")
```

    ## Linear hypothesis test
    ## 
    ## Hypothesis:
    ## wave2 = 0
    ## 
    ## Model 1: restricted model
    ## Model 2: log(price) ~ wave2 + mon + tues + wed + thurs + time
    ## 
    ##   Res.Df Df  Chisq Pr(>Chisq)    
    ## 1     91                         
    ## 2     90  1 22.076  2.621e-06 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

``` r
iv_robust(log(quantity) ~ log(price) + mon + tues + wed + thurs + time | 
            wave2 + mon + tues + wed + thurs + time, 
          data = fish, 
          se_type = "HC3") %>% 
  tidy() %>% 
  mutate_at(vars(estimate, std.error), .funs = ~ round(., 3))
```

    ##          term estimate std.error  statistic      p.value     conf.low
    ## 1 (Intercept)    8.271     0.198 41.8037697 9.646505e-61  7.877980480
    ## 2  log(price)   -0.960     0.518 -1.8529963 6.715982e-02 -1.989974440
    ## 3         mon   -0.322     0.257 -1.2513781 2.140395e-01 -0.832398596
    ## 4        tues   -0.687     0.220 -3.1257866 2.387651e-03 -1.124052315
    ## 5         wed   -0.520     0.232 -2.2361852 2.781161e-02 -0.981614218
    ## 6       thurs    0.106     0.192  0.5488613 5.844598e-01 -0.276449267
    ## 7        time   -0.003     0.003 -0.9320401 3.538094e-01 -0.009056595
    ##      conf.high df       outcome
    ## 1  8.664124381 90 log(quantity)
    ## 2  0.069281010 90 log(quantity)
    ## 3  0.189020929 90 log(quantity)
    ## 4 -0.250451109 90 log(quantity)
    ## 5 -0.057999381 90 log(quantity)
    ## 6  0.487509078 90 log(quantity)
    ## 7  0.003272467 90 log(quantity)

## 2SLS with wind speed instrument

``` r
fs_waveheight <- lm_robust(log(price) ~ speed3 + mon + tues + wed + thurs + time, 
                           data = fish, 
                           se_type = "HC3")

tidy(fs_waveheight) %>% mutate_at(vars(estimate, std.error), .funs = ~ round(., 3))
```

    ##          term estimate std.error   statistic    p.value     conf.low
    ## 1 (Intercept)   -0.444     0.190 -2.33111890 0.02198095 -0.821699097
    ## 2      speed3    0.017     0.007  2.49398285 0.01445834  0.003394740
    ## 3         mon   -0.031     0.135 -0.23256081 0.81663051 -0.298647526
    ## 4        tues   -0.086     0.136 -0.63278602 0.52847790 -0.355405441
    ## 5         wed   -0.001     0.134 -0.01003709 0.99201391 -0.268057082
    ## 6       thurs    0.098     0.130  0.75471436 0.45239075 -0.160283424
    ## 7        time   -0.003     0.002 -2.00317334 0.04816693 -0.006043233
    ##       conf.high df    outcome
    ## 1 -6.554960e-02 90 log(price)
    ## 2  2.998308e-02 90 log(price)
    ## 3  2.360551e-01 90 log(price)
    ## 4  1.836941e-01 90 log(price)
    ## 5  2.653621e-01 90 log(price)
    ## 6  3.566667e-01 90 log(price)
    ## 7 -2.498995e-05 90 log(price)

``` r
linearHypothesis(fs_waveheight, "speed3 = 0")
```

    ## Linear hypothesis test
    ## 
    ## Hypothesis:
    ## speed3 = 0
    ## 
    ## Model 1: restricted model
    ## Model 2: log(price) ~ speed3 + mon + tues + wed + thurs + time
    ## 
    ##   Res.Df Df Chisq Pr(>Chisq)  
    ## 1     91                      
    ## 2     90  1  6.22    0.01263 *
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

``` r
# with windspeed instrument 
iv_robust(log(quantity) ~ log(price) + mon + tues + wed + thurs + time | 
            speed3 + mon + tues + wed + thurs + time , 
          data = fish, 
          se_type = "stata") %>% 
  tidy() %>% 
  mutate_at(vars(estimate, std.error), .funs = ~ round(., 3))
```

    ##          term estimate std.error  statistic      p.value    conf.low
    ## 1 (Intercept)    8.198     0.245 33.4352478 1.546359e-52  7.71100996
    ## 2  log(price)   -1.960     1.005 -1.9510690 5.415929e-02 -3.95547000
    ## 3         mon   -0.332     0.291 -1.1389773 2.577352e-01 -0.91041904
    ## 4        tues   -0.696     0.281 -2.4747262 1.520765e-02 -1.25484666
    ## 5         wed   -0.482     0.282 -1.7084121 9.100676e-02 -1.04293793
    ## 6       thurs    0.196     0.269  0.7292884 4.677190e-01 -0.33800504
    ## 7        time   -0.007     0.005 -1.4088552 1.623236e-01 -0.01658477
    ##      conf.high df       outcome
    ## 1  8.685252385 90 log(quantity)
    ## 2  0.035765832 90 log(quantity)
    ## 3  0.246911620 90 log(quantity)
    ## 4 -0.137273024 90 log(quantity)
    ## 5  0.078539561 90 log(quantity)
    ## 6  0.730093303 90 log(quantity)
    ## 7  0.002822241 90 log(quantity)
