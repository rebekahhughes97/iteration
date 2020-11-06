Reading Data from the web
================

## Do something simple

``` r
x_vec = rnorm(30, mean = 5, sd = 3)

(x_vec - mean(x_vec)) / sd(x_vec)
```

    ##  [1]  0.09688842  0.21934268  1.74969265  0.67532944  0.41878980  1.40186033
    ##  [7] -1.08940271 -0.92248902 -0.49975040 -2.25658228  1.37053603 -0.74747346
    ## [13]  0.64249678 -1.04319441 -1.23762554  0.85210045 -0.39609001 -0.57027036
    ## [19]  0.07113848  0.12597516 -1.26815651  0.91058442 -0.29056550  0.11065692
    ## [25]  2.26379196  0.32422775  0.41540129  0.14453648 -0.44834705 -1.02340177

We want a function to compute z-scores:

``` r
z_scores = function(x){
  if (!is.numeric(x)){
    stop("Input must be numeric")
  }
  if (length(x) < 3){
    stop("Input must have at least 3 numbers")
  }
  z = (x - mean(x)) / sd(x)
  return(z)
}

z_scores(x_vec)
```

    ##  [1]  0.09688842  0.21934268  1.74969265  0.67532944  0.41878980  1.40186033
    ##  [7] -1.08940271 -0.92248902 -0.49975040 -2.25658228  1.37053603 -0.74747346
    ## [13]  0.64249678 -1.04319441 -1.23762554  0.85210045 -0.39609001 -0.57027036
    ## [19]  0.07113848  0.12597516 -1.26815651  0.91058442 -0.29056550  0.11065692
    ## [25]  2.26379196  0.32422775  0.41540129  0.14453648 -0.44834705 -1.02340177

Try the function on some other things: (they should give errors)

``` r
z_scores(3)
```

    ## Error in z_scores(3): Input must have at least 3 numbers

``` r
z_scores("my name is rebekah")
```

    ## Error in z_scores("my name is rebekah"): Input must be numeric

``` r
z_scores(mtcars)
```

    ## Error in z_scores(mtcars): Input must be numeric

``` r
z_scores(c(TRUE, TRUE, FALSE, TRUE))
```

    ## Error in z_scores(c(TRUE, TRUE, FALSE, TRUE)): Input must be numeric

## Multiple Outputs

``` r
mean_and_sd = function(x){
  if (!is.numeric(x)){
    stop("Input must be numeric")
  }
  if (length(x) < 3){
    stop("Input must have at least 3 numbers")
  }
  mean_x = mean(x)
  sd_x = sd(x)
  tibble(
    mean = mean_x,
    sd = sd_x
  )
}
```

Check that the function works:

``` r
mean_and_sd(x_vec)
```

    ## # A tibble: 1 x 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  4.91  3.00

## Multiple Inputs

``` r
sim_data = 
  tibble(
    x = rnorm(n = 100, mean = 4, sd = 3)
  )

sim_data %>% 
  summarize(
    mean = mean(x),
    sd = sd(x)
  )
```

    ## # A tibble: 1 x 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  4.32  2.91

Need to translate above into a function:

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

sim_mean_sd(samp_size = 100, mu = 6, sigma = 3)
```

    ## # A tibble: 1 x 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  5.48  3.05

``` r
sim_mean_sd(samp_size = 100)
```

    ## # A tibble: 1 x 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  2.82  3.93

## Review Napoleon Dynamite

``` r
url = "https://www.amazon.com/product-reviews/B00005JNBQ/ref=cm_cr_arp_d_viewopt_rvwer?ie=UTF8&reviewerType=avp_only_reviews&sortBy=recent&pageNumber=1"

dynamite_html = read_html(url)

review_titles = 
  dynamite_html %>%
  html_nodes(".a-text-bold span") %>%
  html_text()

review_stars = 
  dynamite_html %>%
  html_nodes("#cm_cr-review_list .review-rating") %>%
  html_text() %>%
  str_extract("^\\d") %>%
  as.numeric()

review_text = 
  dynamite_html %>%
  html_nodes(".review-text-content span") %>%
  html_text() %>% 
  str_replace_all("\n", "") %>% 
  str_trim()

reviews = tibble(
  title = review_titles,
  stars = review_stars,
  text = review_text
)
```

What about the next page of reviews? We need to turn it into a function
instead of continuously coppying and pasting.

``` r
read_page_reviews = function(url){
  
  html = read_html(url)

review_titles = 
  dynamite_html %>%
  html_nodes(".a-text-bold span") %>%
  html_text()

review_stars = 
  dynamite_html %>%
  html_nodes("#cm_cr-review_list .review-rating") %>%
  html_text() %>%
  str_extract("^\\d") %>%
  as.numeric()

review_text = 
  dynamite_html %>%
  html_nodes(".review-text-content span") %>%
  html_text() %>% 
  str_replace_all("\n", "") %>% 
  str_trim()

reviews = tibble(
  title = review_titles,
  stars = review_stars,
  text = review_text
)
reviews
}
```

Try the function:

``` r
dynamite_url = "https://www.amazon.com/product-reviews/B00005JNBQ/ref=cm_cr_arp_d_viewopt_rvwer?ie=UTF8&reviewerType=avp_only_reviews&sortBy=recent&pageNumber=2"

read_page_reviews(dynamite_url)
```

    ## # A tibble: 10 x 3
    ##    title                               stars text                               
    ##    <chr>                               <dbl> <chr>                              
    ##  1 Great Value                             5 "Great Value"                      
    ##  2 I LOVE THIS MOVIE                       5 "THIS MOVIE IS SO FUNNY ONE OF MY …
    ##  3 Don't you wish you could go back?       5 "Watch it 100 times. Never. Gets. …
    ##  4 Stupid, but very funny!                 5 "If you like stupidly funny '90s t…
    ##  5 The beat                                5 "The best"                         
    ##  6 Hilarious                               5 "Super funny! Loved the online ren…
    ##  7 Love this movie                         5 "We love this product.  It came in…
    ##  8 Entertaining, limited quality           4 "Entertainment level gets a 5 star…
    ##  9 Boo                                     1 "We rented this movie because our …
    ## 10 Movie is still silly fun....amazon…     1 "We are getting really frustrated …

Let’s read a few pages of reviews:

``` r
dynamite_url_base = "https://www.amazon.com/product-reviews/B00005JNBQ/ref=cm_cr_arp_d_viewopt_rvwer?ie=UTF8&reviewerType=avp_only_reviews&sortBy=recent&pageNumber="

dynamite_urls = str_c(dynamite_url_base, 1:5)

all_reviews = 
  bind_rows(
    read_page_reviews(dynamite_urls[1]),
    read_page_reviews(dynamite_urls[2]),
    read_page_reviews(dynamite_urls[3]),
    read_page_reviews(dynamite_urls[4]),
    read_page_reviews(dynamite_urls[5])
)
```

## Mean Scoping Example

``` r
f = function(x){
  z = x + y
  z
}

x = 2
y = 2

f(x = y)
```

    ## [1] 4

``` r
f = function(x1){
  z = x1 + x2
  z
}

x1 = 2

f(x1 = 2)
```

    ## Error in f(x1 = 2): object 'x2' not found

## Functions as arguements

``` r
my_summary = function(x, summ_func){
  summ_func(x)
}

x_vec = rnorm(100, 3, 7)

mean(x_vec)
```

    ## [1] 3.993701

``` r
my_summary(x_vec, IQR)
```

    ## [1] 9.140106
