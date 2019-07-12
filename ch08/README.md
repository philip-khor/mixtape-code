Instrumental Variables
================
null

    ## Warning: package 'estimatr' was built under R version 3.6.1

    ## Warning: package 'car' was built under R version 3.6.1

    ## Warning: package 'sandwich' was built under R version 3.6.1

    ## Warning: package 'lmtest' was built under R version 3.6.1

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

Regress the endogenous covariate on all exogenous variables

``` r
model2 <- lm_robust(educ ~ nearc4 + exper + 
                      black + south + married + smsa,
                    data = card)

tidy_round(model2)
```

| term        | estimate | std.error | p.value |
| :---------- | -------: | --------: | ------: |
| (Intercept) |   16.831 |     0.125 |       0 |
| nearc4      |    0.327 |     0.081 |       0 |
| exper       |  \-0.404 |     0.008 |       0 |
| black       |  \-0.948 |     0.089 |       0 |
| south       |  \-0.297 |     0.079 |       0 |
| married     |  \-0.073 |     0.018 |       0 |
| smsa        |    0.421 |     0.085 |       0 |

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

The formula syntax for `estimatr::iv_robust()` is

``` r
iv_robust(Y ~ D + X | Z + X, data = dat)
```

where \(D\) is the endogenous variable of interest, \(X\) are controls
and \(Z\) is the instrument.

<table>

<tr>

<td>

``` r
lm_robust(lwage ~ educ + exper +
            black + south + married + smsa,
          data = card) -> model1
  
tidy_round(model1)
```

<table class="table" style="margin-left: auto; margin-right: auto;">

<thead>

<tr>

<th style="text-align:left;">

term

</th>

<th style="text-align:right;">

estimate

</th>

<th style="text-align:right;">

std.error

</th>

<th style="text-align:right;">

p.value

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

(Intercept)

</td>

<td style="text-align:right;">

5.063

</td>

<td style="text-align:right;">

0.066

</td>

<td style="text-align:right;">

0

</td>

</tr>

<tr>

<td style="text-align:left;">

educ

</td>

<td style="text-align:right;">

0.071

</td>

<td style="text-align:right;">

0.004

</td>

<td style="text-align:right;">

0

</td>

</tr>

<tr>

<td style="text-align:left;">

exper

</td>

<td style="text-align:right;">

0.034

</td>

<td style="text-align:right;">

0.002

</td>

<td style="text-align:right;">

0

</td>

</tr>

<tr>

<td style="text-align:left;">

black

</td>

<td style="text-align:right;">

\-0.166

</td>

<td style="text-align:right;">

0.017

</td>

<td style="text-align:right;">

0

</td>

</tr>

<tr>

<td style="text-align:left;">

south

</td>

<td style="text-align:right;">

\-0.132

</td>

<td style="text-align:right;">

0.015

</td>

<td style="text-align:right;">

0

</td>

</tr>

<tr>

<td style="text-align:left;">

married

</td>

<td style="text-align:right;">

\-0.036

</td>

<td style="text-align:right;">

0.004

</td>

<td style="text-align:right;">

0

</td>

</tr>

<tr>

<td style="text-align:left;">

smsa

</td>

<td style="text-align:right;">

0.176

</td>

<td style="text-align:right;">

0.015

</td>

<td style="text-align:right;">

0

</td>

</tr>

</tbody>

</table>

</td>

<td>

``` r
iv_robust(lwage ~ educ + exper +
            black + south + married +
            smsa  | nearc4 + exper +
            black + south + married + smsa,
          data = card, 
          se_type = "classical") %>%  
  tidy_round()
```

<table class="table" style="margin-left: auto; margin-right: auto;">

<thead>

<tr>

<th style="text-align:left;">

term

</th>

<th style="text-align:right;">

estimate

</th>

<th style="text-align:right;">

std.error

</th>

<th style="text-align:right;">

p.value

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

(Intercept)

</td>

<td style="text-align:right;">

4.162

</td>

<td style="text-align:right;">

0.850

</td>

<td style="text-align:right;">

0.000

</td>

</tr>

<tr>

<td style="text-align:left;">

educ

</td>

<td style="text-align:right;">

0.124

</td>

<td style="text-align:right;">

0.050

</td>

<td style="text-align:right;">

0.013

</td>

</tr>

<tr>

<td style="text-align:left;">

exper

</td>

<td style="text-align:right;">

0.056

</td>

<td style="text-align:right;">

0.020

</td>

<td style="text-align:right;">

0.006

</td>

</tr>

<tr>

<td style="text-align:left;">

black

</td>

<td style="text-align:right;">

\-0.116

</td>

<td style="text-align:right;">

0.051

</td>

<td style="text-align:right;">

0.023

</td>

</tr>

<tr>

<td style="text-align:left;">

south

</td>

<td style="text-align:right;">

\-0.113

</td>

<td style="text-align:right;">

0.023

</td>

<td style="text-align:right;">

0.000

</td>

</tr>

<tr>

<td style="text-align:left;">

married

</td>

<td style="text-align:right;">

\-0.032

</td>

<td style="text-align:right;">

0.005

</td>

<td style="text-align:right;">

0.000

</td>

</tr>

<tr>

<td style="text-align:left;">

smsa

</td>

<td style="text-align:right;">

0.148

</td>

<td style="text-align:right;">

0.031

</td>

<td style="text-align:right;">

0.000

</td>

</tr>

</tbody>

</table>

</td>

</tr>

</table>

# Fulton fish markets

</table>

<tr>

<td>

``` r
lm_robust(log(quantity) ~ log(price) + 
            mon + tues + wed + thurs + time, 
          data = fish) %>% 
  tidy_round()
```

<table class="table" style="margin-left: auto; margin-right: auto;">

<thead>

<tr>

<th style="text-align:left;">

term

</th>

<th style="text-align:right;">

estimate

</th>

<th style="text-align:right;">

std.error

</th>

<th style="text-align:right;">

p.value

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

(Intercept)

</td>

<td style="text-align:right;">

8.301

</td>

<td style="text-align:right;">

0.183

</td>

<td style="text-align:right;">

0.000

</td>

</tr>

<tr>

<td style="text-align:left;">

log(price)

</td>

<td style="text-align:right;">

\-0.549

</td>

<td style="text-align:right;">

0.168

</td>

<td style="text-align:right;">

0.002

</td>

</tr>

<tr>

<td style="text-align:left;">

mon

</td>

<td style="text-align:right;">

\-0.318

</td>

<td style="text-align:right;">

0.242

</td>

<td style="text-align:right;">

0.194

</td>

</tr>

<tr>

<td style="text-align:left;">

tues

</td>

<td style="text-align:right;">

\-0.684

</td>

<td style="text-align:right;">

0.205

</td>

<td style="text-align:right;">

0.001

</td>

</tr>

<tr>

<td style="text-align:left;">

wed

</td>

<td style="text-align:right;">

\-0.535

</td>

<td style="text-align:right;">

0.215

</td>

<td style="text-align:right;">

0.014

</td>

</tr>

<tr>

<td style="text-align:left;">

thurs

</td>

<td style="text-align:right;">

0.068

</td>

<td style="text-align:right;">

0.169

</td>

<td style="text-align:right;">

0.687

</td>

</tr>

<tr>

<td style="text-align:left;">

time

</td>

<td style="text-align:right;">

\-0.001

</td>

<td style="text-align:right;">

0.002

</td>

<td style="text-align:right;">

0.607

</td>

</tr>

</tbody>

</table>

</td>

<td>

``` r
fs_waveheight <- lm_robust(log(price) ~ wave2 + 
                             mon + tues + wed + thurs + time, 
                           data = fish)

tidy_round(fs_waveheight)
```

<table class="table" style="margin-left: auto; margin-right: auto;">

<thead>

<tr>

<th style="text-align:left;">

term

</th>

<th style="text-align:right;">

estimate

</th>

<th style="text-align:right;">

std.error

</th>

<th style="text-align:right;">

p.value

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

(Intercept)

</td>

<td style="text-align:right;">

\-0.706

</td>

<td style="text-align:right;">

0.159

</td>

<td style="text-align:right;">

0.000

</td>

</tr>

<tr>

<td style="text-align:left;">

wave2

</td>

<td style="text-align:right;">

0.103

</td>

<td style="text-align:right;">

0.021

</td>

<td style="text-align:right;">

0.000

</td>

</tr>

<tr>

<td style="text-align:left;">

mon

</td>

<td style="text-align:right;">

\-0.036

</td>

<td style="text-align:right;">

0.117

</td>

<td style="text-align:right;">

0.756

</td>

</tr>

<tr>

<td style="text-align:left;">

tues

</td>

<td style="text-align:right;">

0.007

</td>

<td style="text-align:right;">

0.126

</td>

<td style="text-align:right;">

0.954

</td>

</tr>

<tr>

<td style="text-align:left;">

wed

</td>

<td style="text-align:right;">

0.083

</td>

<td style="text-align:right;">

0.116

</td>

<td style="text-align:right;">

0.477

</td>

</tr>

<tr>

<td style="text-align:left;">

thurs

</td>

<td style="text-align:right;">

0.136

</td>

<td style="text-align:right;">

0.107

</td>

<td style="text-align:right;">

0.205

</td>

</tr>

<tr>

<td style="text-align:left;">

time

</td>

<td style="text-align:right;">

\-0.002

</td>

<td style="text-align:right;">

0.001

</td>

<td style="text-align:right;">

0.111

</td>

</tr>

</tbody>

</table>

</td>

</tr>

</table>

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
          data = fish, se_type = "classical") %>% 
  tidy_round()
```

<table class="table" style="margin-left: auto; margin-right: auto;">

<thead>

<tr>

<th style="text-align:left;">

term

</th>

<th style="text-align:right;">

estimate

</th>

<th style="text-align:right;">

std.error

</th>

<th style="text-align:right;">

p.value

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

(Intercept)

</td>

<td style="text-align:right;">

8.271

</td>

<td style="text-align:right;">

0.210

</td>

<td style="text-align:right;">

0.000

</td>

</tr>

<tr>

<td style="text-align:left;">

log(price)

</td>

<td style="text-align:right;">

\-0.960

</td>

<td style="text-align:right;">

0.422

</td>

<td style="text-align:right;">

0.025

</td>

</tr>

<tr>

<td style="text-align:left;">

mon

</td>

<td style="text-align:right;">

\-0.322

</td>

<td style="text-align:right;">

0.233

</td>

<td style="text-align:right;">

0.172

</td>

</tr>

<tr>

<td style="text-align:left;">

tues

</td>

<td style="text-align:right;">

\-0.687

</td>

<td style="text-align:right;">

0.230

</td>

<td style="text-align:right;">

0.004

</td>

</tr>

<tr>

<td style="text-align:left;">

wed

</td>

<td style="text-align:right;">

\-0.520

</td>

<td style="text-align:right;">

0.227

</td>

<td style="text-align:right;">

0.025

</td>

</tr>

<tr>

<td style="text-align:left;">

thurs

</td>

<td style="text-align:right;">

0.106

</td>

<td style="text-align:right;">

0.230

</td>

<td style="text-align:right;">

0.647

</td>

</tr>

<tr>

<td style="text-align:left;">

time

</td>

<td style="text-align:right;">

\-0.003

</td>

<td style="text-align:right;">

0.003

</td>

<td style="text-align:right;">

0.354

</td>

</tr>

</tbody>

</table>

## 2SLS with wind speed instrument

<table>

<tr>

<td>

### First-stage

``` r
lm_robust(log(price) ~ speed3 + mon +
            tues + wed + thurs + time,
          data = fish) -> fs_waveheight

tidy_round(fs_waveheight)
```

<table class="table" style="margin-left: auto; margin-right: auto;">

<thead>

<tr>

<th style="text-align:left;">

term

</th>

<th style="text-align:right;">

estimate

</th>

<th style="text-align:right;">

std.error

</th>

<th style="text-align:right;">

p.value

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

(Intercept)

</td>

<td style="text-align:right;">

\-0.444

</td>

<td style="text-align:right;">

0.182

</td>

<td style="text-align:right;">

0.017

</td>

</tr>

<tr>

<td style="text-align:left;">

speed3

</td>

<td style="text-align:right;">

0.017

</td>

<td style="text-align:right;">

0.006

</td>

<td style="text-align:right;">

0.010

</td>

</tr>

<tr>

<td style="text-align:left;">

mon

</td>

<td style="text-align:right;">

\-0.031

</td>

<td style="text-align:right;">

0.130

</td>

<td style="text-align:right;">

0.810

</td>

</tr>

<tr>

<td style="text-align:left;">

tues

</td>

<td style="text-align:right;">

\-0.086

</td>

<td style="text-align:right;">

0.131

</td>

<td style="text-align:right;">

0.513

</td>

</tr>

<tr>

<td style="text-align:left;">

wed

</td>

<td style="text-align:right;">

\-0.001

</td>

<td style="text-align:right;">

0.130

</td>

<td style="text-align:right;">

0.992

</td>

</tr>

<tr>

<td style="text-align:left;">

thurs

</td>

<td style="text-align:right;">

0.098

</td>

<td style="text-align:right;">

0.126

</td>

<td style="text-align:right;">

0.436

</td>

</tr>

<tr>

<td style="text-align:left;">

time

</td>

<td style="text-align:right;">

\-0.003

</td>

<td style="text-align:right;">

0.001

</td>

<td style="text-align:right;">

0.040

</td>

</tr>

</tbody>

</table>

</td>

<td>

### 2SLS

``` r
iv_robust(log(quantity) ~ log(price) + mon + tues + 
            wed + thurs + time | 
            speed3 + mon + tues + wed + thurs + time , 
          data = fish, se_type = "classical") %>% 
  tidy_round()
```

<table class="table" style="margin-left: auto; margin-right: auto;">

<thead>

<tr>

<th style="text-align:left;">

term

</th>

<th style="text-align:right;">

estimate

</th>

<th style="text-align:right;">

std.error

</th>

<th style="text-align:right;">

p.value

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

(Intercept)

</td>

<td style="text-align:right;">

8.198

</td>

<td style="text-align:right;">

0.268

</td>

<td style="text-align:right;">

0.000

</td>

</tr>

<tr>

<td style="text-align:left;">

log(price)

</td>

<td style="text-align:right;">

\-1.960

</td>

<td style="text-align:right;">

0.907

</td>

<td style="text-align:right;">

0.033

</td>

</tr>

<tr>

<td style="text-align:left;">

mon

</td>

<td style="text-align:right;">

\-0.332

</td>

<td style="text-align:right;">

0.292

</td>

<td style="text-align:right;">

0.259

</td>

</tr>

<tr>

<td style="text-align:left;">

tues

</td>

<td style="text-align:right;">

\-0.696

</td>

<td style="text-align:right;">

0.288

</td>

<td style="text-align:right;">

0.018

</td>

</tr>

<tr>

<td style="text-align:left;">

wed

</td>

<td style="text-align:right;">

\-0.482

</td>

<td style="text-align:right;">

0.286

</td>

<td style="text-align:right;">

0.095

</td>

</tr>

<tr>

<td style="text-align:left;">

thurs

</td>

<td style="text-align:right;">

0.196

</td>

<td style="text-align:right;">

0.295

</td>

<td style="text-align:right;">

0.509

</td>

</tr>

<tr>

<td style="text-align:left;">

time

</td>

<td style="text-align:right;">

\-0.007

</td>

<td style="text-align:right;">

0.005

</td>

<td style="text-align:right;">

0.161

</td>

</tr>

</tbody>

</table>

</td>

</tr>

</table>

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
