Instrumental Variables
================
null

This exercise uses

  - `dplyr` for data wrangling
  - `broom::tidy()` for extracting coefficient estimates and standard
    errors
  - `estimatr::lm_robust()` for linear regression with robust standard
    errors
  - `estimatr::iv_robust()` for 2SLS with robust standard errors
  - `car::linearHypothesis()` to run conditional F test for weak
    instruments

# College in the county

## First-stage

Regress the endogenous covariate on all exogenous
variables

``` r
model2 <- lm_robust(educ ~ nearc4 + exper + black + south + married + smsa,
                    data = card)

tidy(model2) %>% 
  mutate_at(vars(estimate, std.error), .funs = ~ round(., 3))
```

    ##          term estimate std.error  statistic      p.value   conf.low
    ## 1 (Intercept)   16.831     0.125 134.291382 0.000000e+00 16.5849556
    ## 2      nearc4    0.327     0.081   4.062970 4.969605e-05  0.1693387
    ## 3       exper   -0.404     0.008 -49.305184 0.000000e+00 -0.4205174
    ## 4       black   -0.948     0.089 -10.703388 2.908912e-26 -1.1211060
    ## 5       south   -0.297     0.079  -3.775431 1.628308e-04 -0.4517818
    ## 6     married   -0.073     0.018  -4.125456 3.800720e-05 -0.1072436
    ## 7        smsa    0.421     0.085   4.957298 7.545703e-07  0.2544185
    ##     conf.high   df outcome
    ## 1 17.07643745 2996    educ
    ## 2  0.48522649 2996    educ
    ## 3 -0.38835058 2996    educ
    ## 4 -0.77395019 2996    educ
    ## 5 -0.14292374 2996    educ
    ## 6 -0.03814362 2996    educ
    ## 7  0.58737061 2996    educ

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
    ##   Res.Df Df  Chisq Pr(>Chisq)    
    ## 1   2997                         
    ## 2   2996  1 16.508  4.845e-05 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

## Estimate OLS and 2SLS effects

The formula syntax for \`estimatr::iv\_robust()\`\`\` is:

``` r
iv_robust(Y ~ D + X | Z + X, data = dat)
```

where \(D\) is the endogenous variable of interest, \(X\) are controls
and \(Z\) is the
instrument.

``` r
model1 <- lm_robust(lwage ~ educ + exper + black + south + married + smsa, 
                    data = card) 
  
model1 %>% 
  tidy() %>% 
  mutate_at(vars(estimate, std.error), .funs = ~ round(., 3))
```

    ##          term estimate std.error  statistic      p.value    conf.low
    ## 1 (Intercept)    5.063     0.066  76.492228 0.000000e+00  4.93352651
    ## 2        educ    0.071     0.004  19.689232 2.952497e-81  0.06408509
    ## 3       exper    0.034     0.002  15.063133 1.754841e-49  0.02970630
    ## 4       black   -0.166     0.017  -9.539818 2.859334e-21 -0.20015172
    ## 5       south   -0.132     0.015  -8.653098 8.038781e-18 -0.16136086
    ## 6     married   -0.036     0.004 -10.037802 2.420439e-23 -0.04287760
    ## 7        smsa    0.176     0.015  11.639880 1.168602e-30  0.14617550
    ##     conf.high   df outcome
    ## 1  5.19310657 2996   lwage
    ## 2  0.07826061 2996   lwage
    ## 3  0.03859733 2996   lwage
    ## 4 -0.13190317 2996   lwage
    ## 5 -0.10174268 2996   lwage
    ## 6 -0.02886383 2996   lwage
    ## 7  0.20539874 2996   lwage

``` r
ivreg <- iv_robust(lwage ~ educ + exper + black + south + married +
            smsa  | nearc4 + exper + black + south + married + smsa,
          data = card, se_type = "classical") 
```

# Fulton fish markets

## OLS

``` r
lm_robust(log(quantity) ~ log(price) + mon + tues + wed + thurs + time, 
          data = fish) %>% 
  tidy()
```

    ##          term     estimate   std.error  statistic      p.value
    ## 1 (Intercept)  8.301070693 0.182810285 45.4081164 7.881881e-64
    ## 2  log(price) -0.548897230 0.168142075 -3.2644847 1.551515e-03
    ## 3         mon -0.317545597 0.242448162 -1.3097464 1.936151e-01
    ## 4        tues -0.683625817 0.205218636 -3.3312073 1.255447e-03
    ## 5         wed -0.535288090 0.214712251 -2.4930487 1.449391e-02
    ## 6       thurs  0.068269443 0.168733591  0.4045990 6.867323e-01
    ## 7        time -0.001249897 0.002421495 -0.5161675 6.070033e-01
    ##       conf.low    conf.high df       outcome
    ## 1  7.937886154  8.664255231 90 log(quantity)
    ## 2 -0.882940811 -0.214853650 90 log(quantity)
    ## 3 -0.799211188  0.164119994 90 log(quantity)
    ## 4 -1.091328456 -0.275923177 90 log(quantity)
    ## 5 -0.961851454 -0.108724727 90 log(quantity)
    ## 6 -0.266949287  0.403488173 90 log(quantity)
    ## 7 -0.006060618  0.003560825 90 log(quantity)

## 2SLS with average wave height instrument

``` r
fs_waveheight <- lm_robust(log(price) ~ wave2 + mon + tues + wed + thurs + time, 
                           data = fish)

fs_waveheight %>% 
  tidy() %>% 
  mutate_at(vars(estimate, std.error), .funs = ~ round(., 3))
```

    ##          term estimate std.error   statistic      p.value     conf.low
    ## 1 (Intercept)   -0.706     0.159 -4.43699065 2.577025e-05 -1.021587346
    ## 2       wave2    0.103     0.021  4.95586761 3.360941e-06  0.061579010
    ## 3         mon   -0.036     0.117 -0.31191215 7.558287e-01 -0.268846439
    ## 4        tues    0.007     0.126  0.05742528 9.543337e-01 -0.242417823
    ## 5         wed    0.083     0.116  0.71376882 4.772171e-01 -0.147141485
    ## 6       thurs    0.136     0.107  1.27705837 2.048666e-01 -0.075640254
    ## 7        time   -0.002     0.001 -1.60962027 1.109830e-01 -0.004706456
    ##       conf.high df    outcome
    ## 1 -0.3896859280 90 log(price)
    ## 2  0.1439835212 90 log(price)
    ## 3  0.1958829343 90 log(price)
    ## 4  0.2568492520 90 log(price)
    ## 5  0.3121576426 90 log(price)
    ## 6  0.3478917181 90 log(price)
    ## 7  0.0004934493 90 log(price)

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
    ## 2     90  1 24.561  7.201e-07 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

``` r
iv_robust(log(quantity) ~ log(price) + mon + tues + wed + thurs + time | 
            wave2 + mon + tues + wed + thurs + time, 
          data = fish) %>% 
  tidy() %>% 
  mutate_at(vars(estimate, std.error), .funs = ~ round(., 3))
```

    ##          term estimate std.error  statistic      p.value     conf.low
    ## 1 (Intercept)    8.271     0.189 43.8000031 1.760387e-62  7.895895167
    ## 2  log(price)   -0.960     0.477 -2.0151014 4.687770e-02 -1.907145891
    ## 3         mon   -0.322     0.245 -1.3113373 1.930795e-01 -0.809047039
    ## 4        tues   -0.687     0.212 -3.2473773 1.637331e-03 -1.107697310
    ## 5         wed   -0.520     0.224 -2.3168421 2.278222e-02 -0.965537191
    ## 6       thurs    0.106     0.184  0.5724644 5.684347e-01 -0.260699986
    ## 7        time   -0.003     0.003 -0.9795822 3.299183e-01 -0.008757412
    ##      conf.high df       outcome
    ## 1  8.646209694 90 log(quantity)
    ## 2 -0.013547539 90 log(quantity)
    ## 3  0.165669372 90 log(quantity)
    ## 4 -0.266806114 90 log(quantity)
    ## 5 -0.074076407 90 log(quantity)
    ## 6  0.471759797 90 log(quantity)
    ## 7  0.002973284 90 log(quantity)

## 2SLS with wind speed instrument

### First-stage

``` r
fs_waveheight <- lm_robust(log(price) ~ speed3 + mon + tues + wed + thurs + time, 
                           data = fish)

fs_waveheight %>% 
  tidy() %>% 
  mutate_at(vars(estimate, std.error), .funs = ~ round(., 3))
```

    ##          term estimate std.error   statistic    p.value     conf.low
    ## 1 (Intercept)   -0.444     0.182 -2.43471259 0.01687678 -0.805612535
    ## 2      speed3    0.017     0.006  2.62374293 0.01021550  0.004052217
    ## 3         mon   -0.031     0.130 -0.24155804 0.80967215 -0.288689582
    ## 4        tues   -0.086     0.131 -0.65714680 0.51276356 -0.345413087
    ## 5         wed   -0.001     0.130 -0.01040228 0.99172335 -0.258693692
    ## 6       thurs    0.098     0.126  0.78182397 0.43637003 -0.151320847
    ## 7        time   -0.003     0.001 -2.08707752 0.03970861 -0.005922261
    ##       conf.high df    outcome
    ## 1 -0.0816361598 90 log(price)
    ## 2  0.0293256012 90 log(price)
    ## 3  0.2260971126 90 log(price)
    ## 4  0.1737017071 90 log(price)
    ## 5  0.2559987482 90 log(price)
    ## 6  0.3477041456 90 log(price)
    ## 7 -0.0001459619 90 log(price)

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
    ## 2     90  1 6.884   0.008697 **
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

### 2SLS

``` r
iv_robust(log(quantity) ~ log(price) + mon + tues + wed + thurs + time | 
            speed3 + mon + tues + wed + thurs + time , 
          data = fish) %>% 
  tidy() %>% 
  mutate_at(vars(estimate, std.error), .funs = ~ round(., 3))
```

    ##          term estimate std.error  statistic      p.value    conf.low
    ## 1 (Intercept)    8.198     0.246 33.2917122 2.213221e-52  7.70890976
    ## 2  log(price)   -1.960     1.021 -1.9198010 5.805050e-02 -3.98797288
    ## 3         mon   -0.332     0.291 -1.1398530 2.573720e-01 -0.90997446
    ## 4        tues   -0.696     0.283 -2.4603594 1.578914e-02 -1.25810960
    ## 5         wed   -0.482     0.282 -1.7101369 9.068599e-02 -1.04237238
    ## 6       thurs    0.196     0.269  0.7290697 4.678521e-01 -0.33816525
    ## 7        time   -0.007     0.005 -1.3974585 1.657112e-01 -0.01666391
    ##      conf.high df       outcome
    ## 1  8.687352585 90 log(quantity)
    ## 2  0.068268715 90 log(quantity)
    ## 3  0.246467042 90 log(quantity)
    ## 4 -0.134010089 90 log(quantity)
    ## 5  0.077974013 90 log(quantity)
    ## 6  0.730253517 90 log(quantity)
    ## 7  0.002901377 90 log(quantity)
