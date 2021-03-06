Properties of regression
================

<!-- README.md is generated from README.Rmd. Please edit that file -->

## OLS regression line

``` r
set.seed(1)

# construct the data
dat <- tibble(
  x = rnorm(1E4, 0, 1),
  u = rnorm(1E4, 0, 1),
  y = 5.5 * x + 12 * u
  )

# regress y on x 
reg <- lm(y ~ x, data = dat)

# create single-row data-frame of coefficient estimates
reg %>% 
  tidy() %>% # obtain coefs in tidy tibble
  select(term, estimate) %>% 
  spread(term, estimate) -> coefs
coefs 
```

    ## # A tibble: 1 x 2
    ##   `(Intercept)`     x
    ##           <dbl> <dbl>
    ## 1       -0.0499  5.56

``` r
# fitted values and residuals (two ways to recover them)
reg %>% 
  augment() %>% # add fitted values, resid to original data 
  mutate(yhat1 = .fitted,
         # take advantage of simplifying property of [[ here
         yhat2 = coefs[["(Intercept)"]] + coefs[["x"]] * dat[["x"]],
         uhat1 = .resid,
         uhat2 = y - yhat2) -> preds_resids

# check equality
preds_resids %>% 
  summarise(all.equal(yhat1, yhat2) & all.equal(uhat1, uhat2))
```

    ## # A tibble: 1 x 1
    ##   `all.equal(yhat1, yhat2) & all.equal(uhat1, uhat2)`
    ##   <lgl>                                              
    ## 1 TRUE

``` r
# figure 3
ggplot(preds_resids, aes(x, y)) +
  geom_point(alpha = 0.5) +
  geom_smooth(method = 'lm', col = ipsum_pal()(1)) +
  labs(title = 'OLS Regression Line') +
  theme_ipsum()
```

![](../fig/ols1a-1.png)<!-- -->

``` r
# figure 4
ggplot(preds_resids, aes(yhat1, uhat1)) +
  geom_point(alpha = 0.5) +
  labs(x = 'Fitted Values', y = 'Residuals') +
  theme_ipsum()
```

![](../fig/ols1b-1.png)<!-- -->

## Algebraic properties of OLS

### Show that the sum of the OLS residual always adds up to zero

``` r
set.seed(1234)

# construct the data
dat <- tibble(
  x = 9 * rnorm(10, 0, 1),
  u = 36 * rnorm(10, 0, 1),
  y = 3 + 2 * x + u
  )

reg <- lm(y ~ x, data = dat)

# fitted values and residuals (two ways to recover them)
reg %>% 
  augment() %>% 
  mutate(id = row_number(),
         yhat = .fitted,
         uhat = .resid,
         x_uhat = x * uhat, 
         yhat_uhat = yhat * uhat) %>% 
  select(id, x, uhat, y, yhat, uhat, x_uhat, yhat_uhat) %>% 
  gt(rowname_col = "id") %>% 
  tab_header(title = "OLS residuals always sum to zero") %>% 
  fmt_number(columns = vars(x, uhat, y, yhat, 
                            x_uhat, yhat_uhat), 
             decimals = 3) %>% 
  summary_rows(fns = list(~ sum(., na.rm = TRUE))) %>% 
  gtsave("zero_resid.png") -> gt_table
```

![](zero_resid.png)

``` r
means <- dat %>% summarise_all(mean)

tidy(reg) %>% 
  select(term, estimate) %>% 
  spread(term, estimate) %>% 
  mutate(av_pred = `(Intercept)` + means$x * x, 
         mean_y = means$y)
```

    ## # A tibble: 1 x 4
    ##   `(Intercept)`     x av_pred mean_y
    ##           <dbl> <dbl>   <dbl>  <dbl>
    ## 1         -3.85  1.25   -8.15  -8.15

## Unbiasness: Monte Carlo simulation of OLS

``` r
# ols function
ols <- function(...) {
  dat <- tibble(
    x = 9 * rnorm(1E4, 0, 1),
    u = 36 * rnorm(1E4, 0, 1),
    y = 3 + 2 * x + u
    ) %>% 
    lm(y ~ x, data = .)
}

betas_df <- map_df(1:1E3, ~ tidy(ols(.)), .id = "id") %>% 
  unnest() %>% 
  filter(term == "x") %>% 
  select(beta = estimate) 

skim(betas_df) %>% skimr::kable()
```

Skim summary statistics  
n obs: 1000  
n variables: 1

Variable type: numeric

<table>

<thead>

<tr>

<th>

variable

</th>

<th>

missing

</th>

<th>

complete

</th>

<th>

n

</th>

<th>

mean

</th>

<th>

sd

</th>

<th>

p0

</th>

<th>

p25

</th>

<th>

p50

</th>

<th>

p75

</th>

<th>

p100

</th>

<th>

hist

</th>

</tr>

</thead>

<tbody>

<tr>

<td>

beta

</td>

<td>

0

</td>

<td>

1000

</td>

<td>

1000

</td>

<td>

2

</td>

<td>

0.041

</td>

<td>

1.87

</td>

<td>

1.97

</td>

<td>

2

</td>

<td>

2.03

</td>

<td>

2.13

</td>

<td>

▁▁▃▇▇▆▁▁

</td>

</tr>

</tbody>

</table>

``` r
# figure 5
ggplot(betas_df, aes(x = beta, y = ..density..)) +
  geom_histogram() +
  geom_vline(xintercept = 2, col = "red") + 
  labs(title = "Distribution of coefficients from Monte Carlo simulation", 
       x = TeX("$\\beta_{x}$", ), y = 'Density') +
  theme_ipsum()
```

![](../fig/ols_value-1.png)<!-- -->

## Regression anatomy theorem

``` r
# auto dataset
auto <- read_dta('http://www.stata-press.com/data/r8/auto.dta') %>% 
  # cleaning up some of the Stata metadata
  zap_formats() %>% 
  zap_labels() 

# add the residuals to the data frame 
auto %<>% 
  mutate(length_resid = residuals(lm(length ~ weight + headroom + mpg, 
                                     data = .)))

# create a data frame of estimates from each regression in the list
coefs <- list(bivariate = price ~ length,
              multivariate = price ~ length + weight + headroom + mpg,
              aux1 = length ~ weight + headroom + mpg,
              aux2 = price ~ length_resid) %>% 
  map_df(.f = ~ tidy(lm(.x, data = auto)), 
         .id = "reg") 

# select the coefficients from the original regression and 
# the partialled-out version
coefs %>% 
  filter(term %in% c("length", "length_resid")) %>% 
  select(reg:estimate) %>% 
  knitr::kable() 
```

<table>

<thead>

<tr>

<th style="text-align:left;">

reg

</th>

<th style="text-align:left;">

term

</th>

<th style="text-align:right;">

estimate

</th>

</tr>

</thead>

<tbody>

<tr>

<td style="text-align:left;">

bivariate

</td>

<td style="text-align:left;">

length

</td>

<td style="text-align:right;">

57.20224

</td>

</tr>

<tr>

<td style="text-align:left;">

multivariate

</td>

<td style="text-align:left;">

length

</td>

<td style="text-align:right;">

\-94.49651

</td>

</tr>

<tr>

<td style="text-align:left;">

aux2

</td>

<td style="text-align:left;">

length\_resid

</td>

<td style="text-align:right;">

\-94.49651

</td>

</tr>

</tbody>

</table>

OLS slope estimate: \[\hat \beta_1 = \frac{C(x,y)}{Var(x)} \]

``` r
# OLS estimate 
auto %>% 
  summarise(beta = cov(price, length_resid) / var(length_resid))
```

    ## # A tibble: 1 x 1
    ##    beta
    ##   <dbl>
    ## 1 -94.5

``` r
pauto <- bind_rows(list(BV = auto, MV = auto), .id = "type") %>% 
  mutate(length = case_when(
    type == "BV" ~ length - mean(length), 
    TRUE ~ length_resid)) %>% 
  select(price, length, type)

# shift factor (mean adjustment of length requires adjustment of intercept)
s_factor <- coefs %>% 
  filter(reg == "bivariate", term == "length") %>% 
  pull(estimate) * mean(auto$length)
```

``` r
coefs_filt <- coefs %>% 
  filter(reg %in% c("bivariate", "aux2")) %>% 
  mutate(estimate = case_when(
    term == "(Intercept)" & reg == "bivariate" ~ estimate + s_factor,
    TRUE ~ estimate),
         term = case_when(
           term == "length_resid" ~ "length",
           TRUE ~ term)) %>% 
  select(reg:estimate) %>% 
  spread(term, estimate) 

ggplot(pauto) + 
  geom_point(aes(length, price, colour = type)) + 
  scale_colour_ipsum(name = 'Type')  + 
  geom_abline(data = coefs_filt, 
              aes(intercept = `(Intercept)`, 
                  slope = length, 
                  col = reg)) + 
  labs(title = 'Regression Anatomy', 
       x = 'Length', y = 'Price') +
  theme_ipsum() 
```

![](../fig/reganatomy-1.png)<!-- -->
