Linear Models
================

Load the NYC airbnb data

``` r
data(nyc_airbnb)
```

Look at the data / do some cleaning.

``` r
nyc_airbnb <- 
  nyc_airbnb |> 
  mutate(
    stars = review_scores_location / 2 
  ) |> 
  rename(
    borough = neighbourhood_group
  ) |> 
  filter(borough != "Staten Island") |> 
  select(price, stars, borough, room_type, neighbourhood)
```

Do regression!!!

``` r
fit = lm(formula = price ~ stars + borough, data = nyc_airbnb)
```

Do some additional cleaning then refit.

``` r
nyc_airbnb <-  
  nyc_airbnb |> 
  mutate(
    borough = fct_infreq(borough), 
    room_type = fct_infreq(room_type)
  ) # fct_infreq = puts categories in order of how frequently they appear in the dataset

fit = lm(formula = price ~ stars + borough, data = nyc_airbnb)
```

Look at `lm` stuff (DO NOT USE)

``` r
summary(fit)
names(summary(fit))
summary(fit)[["coefficients"]]
names(summary(fit))
summary(fit)[["df"]]

fitted.values(fit)
```

Look at cleaner `lm` stuff

``` r
fit |> 
  broom::tidy() |> 
  mutate(
    term = str_replace(term, "borough", "Borough: ")
  ) |> 
  select(term, estimate, p.value) |> 
  knitr::kable(digits = 3)
```

| term              | estimate | p.value |
|:------------------|---------:|--------:|
| (Intercept)       |   19.839 |   0.104 |
| stars             |   31.990 |   0.000 |
| Borough: Brooklyn |  -49.754 |   0.000 |
| Borough: Queens   |  -77.048 |   0.000 |
| Borough: Bronx    |  -90.254 |   0.000 |

``` r
fit |> 
  broom::glance()
```

    ## # A tibble: 1 × 12
    ##   r.squared adj.r.squared sigma statistic   p.value    df   logLik    AIC    BIC
    ##       <dbl>         <dbl> <dbl>     <dbl>     <dbl> <dbl>    <dbl>  <dbl>  <dbl>
    ## 1    0.0342        0.0341  182.      271. 6.73e-229     4 -202113. 4.04e5 4.04e5
    ## # ℹ 3 more variables: deviance <dbl>, df.residual <int>, nobs <int>

## Diagnostic

Look at residuals.

``` r
nyc_airbnb |> 
  modelr::add_residuals(fit) |> 
  modelr::add_predictions(fit) |> 
  filter(resid < 1000) |> 
  ggplot(aes(x = resid)) +
  geom_histogram()
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

<img src="linear_models_files/figure-gfm/unnamed-chunk-7-1.png" width="90%" />

``` r
nyc_airbnb |> 
  modelr::add_residuals(fit) |> 
  modelr::add_predictions(fit) |> 
  filter(resid < 1000) |> 
  ggplot(aes(x = borough, y = resid)) +
  geom_violin()
```

<img src="linear_models_files/figure-gfm/unnamed-chunk-7-2.png" width="90%" />

``` r
nyc_airbnb |> 
  modelr::add_residuals(fit) |> 
  modelr::add_predictions(fit) |> 
  filter(resid < 1000) |> 
  ggplot(aes(x = stars, y = resid)) +
  geom_point()
```

<img src="linear_models_files/figure-gfm/unnamed-chunk-7-3.png" width="90%" />
