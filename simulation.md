Simulation
================

## Let’s simulate something

Have a funtion:

``` r
sim_mean_sd = function(samp_size, mu = 3, sigma = 4){
  
 sim_data = 
  tibble(
    x = rnorm(n = samp_size, mean = mu, sd = sigma)
  )

sim_data %>% 
  summarize(
    mean = mean(x),
    sd = sd(x)
  ) 
}
```

Can simulate by running this line:

``` r
sim_mean_sd(30)
```

    ## # A tibble: 1 x 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  3.33  3.70

Let’s simulate a lot. Start with a for loop:

``` r
output = vector("list", length = 100)

for (i in 1:100) {
  output[[i]] = sim_mean_sd(samp_size = 30)
}

bind_rows(output)
```

    ## # A tibble: 100 x 2
    ##     mean    sd
    ##    <dbl> <dbl>
    ##  1  3.53  3.18
    ##  2  3.44  3.84
    ##  3  3.45  3.53
    ##  4  1.68  3.69
    ##  5  3.95  4.22
    ##  6  3.27  4.34
    ##  7  2.05  4.05
    ##  8  3.10  3.72
    ##  9  3.55  4.11
    ## 10  3.87  3.79
    ## # … with 90 more rows

Let’s use a loop function:

``` r
sim_results =
  rerun(100, sim_mean_sd(samp_size = 30)) %>% 
  bind_rows()
```

Let’s look at results:

``` r
sim_results %>% 
  ggplot(aes(x = mean)) + geom_density()
```

<img src="simulation_files/figure-gfm/unnamed-chunk-5-1.png" width="90%" />

``` r
sim_results %>% 
  summarize(
    avsamp_mean = mean(mean),
    sdsamp_mean = sd(mean)
  )
```

    ## # A tibble: 1 x 2
    ##   avsamp_mean sdsamp_mean
    ##         <dbl>       <dbl>
    ## 1        2.98       0.756

``` r
sim_results %>% 
  ggplot(aes(x = sd)) + geom_density()
```

<img src="simulation_files/figure-gfm/unnamed-chunk-5-2.png" width="90%" />

## Let’s try other sample sizes

``` r
n_list =
  list(
    "n = 30"  = 30, 
    "n = 60"  = 60, 
    "n = 120" = 120, 
    "n = 240" = 240
  )

output = vector("list", length = 4)

output[[1]] = rerun(100, sim_mean_sd(samp_size = n_list[[1]])) %>% 
  bind_rows()

for (i in 1:4) {
  output[[i]] = rerun(100, sim_mean_sd(samp_size = n_list[[i]])) %>% 
  bind_rows()
}
```

``` r
sim_results =
  tibble(
  sample_size = c(30, 60, 120, 240)
) %>% 
  mutate(
    output_lists = map(.x = sample_size, ~ rerun(100, sim_mean_sd(.x))),
    estimate_df = map(output_lists, bind_rows)
    ) %>% 
  select(-output_lists) %>% 
  unnest(estimate_df)
```

Do data frame things:

``` r
sim_results %>%
  mutate(
    sample_size = str_c("n = ", sample_size),
    sample_size = fct_inorder(sample_size)
  ) %>% 
  ggplot(aes(x = sample_size, y = mean)) + 
  geom_violin()
```

<img src="simulation_files/figure-gfm/unnamed-chunk-8-1.png" width="90%" />

``` r
sim_results %>% 
  group_by(sample_size) %>% 
  summarize(
    avsamp_mean = mean(mean),
    sdsamp_mean = sd(mean)
  )
```

    ## `summarise()` ungrouping output (override with `.groups` argument)

    ## # A tibble: 4 x 3
    ##   sample_size avsamp_mean sdsamp_mean
    ##         <dbl>       <dbl>       <dbl>
    ## 1          30        2.93       0.696
    ## 2          60        3.01       0.551
    ## 3         120        3.00       0.364
    ## 4         240        3.00       0.211
