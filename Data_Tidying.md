Data_Tidying
================
2023-03-12

``` r
library(tidyverse)
```

## pivot_longer

better read by r

``` r
pulse_data = haven::read_sas("./data/public_pulse_data.sas7bdat") %>% 
  janitor::clean_names()
pulse_data
```

    ## # A tibble: 1,087 × 7
    ##       id   age sex    bdi_score_bl bdi_score_01m bdi_score_06m bdi_score_12m
    ##    <dbl> <dbl> <chr>         <dbl>         <dbl>         <dbl>         <dbl>
    ##  1 10003  48.0 male              7             1             2             0
    ##  2 10015  72.5 male              6            NA            NA            NA
    ##  3 10022  58.5 male             14             3             8            NA
    ##  4 10026  72.7 male             20             6            18            16
    ##  5 10035  60.4 male              4             0             1             2
    ##  6 10050  84.7 male              2            10            12             8
    ##  7 10078  31.3 male              4             0            NA            NA
    ##  8 10088  56.9 male              5            NA             0             2
    ##  9 10091  76.0 male              0             3             4             0
    ## 10 10092  74.2 female           10             2            11             6
    ## # … with 1,077 more rows

wide to long

``` r
pluse_data_tidy = pulse_data %>% 
  pivot_longer(
    bdi_score_bl:bdi_score_12m,
    names_to = "visit",
    names_prefix = "bdi_score_",
    values_to = "bdi"
  )
pluse_data_tidy
```

    ## # A tibble: 4,348 × 5
    ##       id   age sex   visit   bdi
    ##    <dbl> <dbl> <chr> <chr> <dbl>
    ##  1 10003  48.0 male  bl        7
    ##  2 10003  48.0 male  01m       1
    ##  3 10003  48.0 male  06m       2
    ##  4 10003  48.0 male  12m       0
    ##  5 10015  72.5 male  bl        6
    ##  6 10015  72.5 male  01m      NA
    ##  7 10015  72.5 male  06m      NA
    ##  8 10015  72.5 male  12m      NA
    ##  9 10022  58.5 male  bl       14
    ## 10 10022  58.5 male  01m       3
    ## # … with 4,338 more rows

rewrite, combine and mutate

``` r
pulse_data = haven::read_sas("./data/public_pulse_data.sas7bdat") %>% 
  janitor::clean_names() %>% 
  pivot_longer(
    bdi_score_bl:bdi_score_12m,
    names_to = "visit",
    names_prefix = "bdi_score_",
    values_to = "bdi"
  ) %>% 
  relocate(id, visit) %>% 
  mutate(visit = recode(visit, "bl" = "00m"))
pulse_data
```

    ## # A tibble: 4,348 × 5
    ##       id visit   age sex     bdi
    ##    <dbl> <chr> <dbl> <chr> <dbl>
    ##  1 10003 00m    48.0 male      7
    ##  2 10003 01m    48.0 male      1
    ##  3 10003 06m    48.0 male      2
    ##  4 10003 12m    48.0 male      0
    ##  5 10015 00m    72.5 male      6
    ##  6 10015 01m    72.5 male     NA
    ##  7 10015 06m    72.5 male     NA
    ##  8 10015 12m    72.5 male     NA
    ##  9 10022 00m    58.5 male     14
    ## 10 10022 01m    58.5 male      3
    ## # … with 4,338 more rows

## pivot wider

make a more human readable table

``` r
analysis_result = tibble(
  group = c("treatment","treatment","placebo","placebo"),
  time = c("pre","post","pre","post"),
  mean = c(4,8,3.5,4)
)

analysis_result
```

    ## # A tibble: 4 × 3
    ##   group     time   mean
    ##   <chr>     <chr> <dbl>
    ## 1 treatment pre     4  
    ## 2 treatment post    8  
    ## 3 placebo   pre     3.5
    ## 4 placebo   post    4

``` r
analysis_result %>% 
  pivot_wider(
    names_from = "time",
    values_from = "mean"
  )
```

    ## # A tibble: 2 × 3
    ##   group       pre  post
    ##   <chr>     <dbl> <dbl>
    ## 1 treatment   4       8
    ## 2 placebo     3.5     4

## stack tables

``` r
fellowship_ring = readxl::read_excel("./data/LotR_Words.xlsx", range = "B3:D6") %>% 
  mutate(movie = "fellowship_ring") %>% 
  relocate(movie)
fellowship_ring
```

    ## # A tibble: 3 × 4
    ##   movie           Race   Female  Male
    ##   <chr>           <chr>   <dbl> <dbl>
    ## 1 fellowship_ring Elf      1229   971
    ## 2 fellowship_ring Hobbit     14  3644
    ## 3 fellowship_ring Man         0  1995

``` r
two_towers = readxl::read_excel("./data/LotR_Words.xlsx", range = "F3:H6") %>% 
  mutate(movie = "two_towers") %>% 
  relocate(movie)
two_towers
```

    ## # A tibble: 3 × 4
    ##   movie      Race   Female  Male
    ##   <chr>      <chr>   <dbl> <dbl>
    ## 1 two_towers Elf       331   513
    ## 2 two_towers Hobbit      0  2463
    ## 3 two_towers Man       401  3589

``` r
return_king = readxl::read_excel("./data/LotR_Words.xlsx", range = "J3:L6") %>% 
  mutate(movie = "return_king") %>% 
  relocate(movie)
return_king
```

    ## # A tibble: 3 × 4
    ##   movie       Race   Female  Male
    ##   <chr>       <chr>   <dbl> <dbl>
    ## 1 return_king Elf       183   510
    ## 2 return_king Hobbit      2  2673
    ## 3 return_king Man       268  2459

Bind all row, and clean the data to long

``` r
lotr_tidy = bind_rows(fellowship_ring, two_towers, return_king) %>% 
  janitor::clean_names() %>% 
  relocate(movie) %>% 
  pivot_longer(
    female:male,
    names_to = "gender",
    values_to = "words"
  )
lotr_tidy
```

    ## # A tibble: 18 × 4
    ##    movie           race   gender words
    ##    <chr>           <chr>  <chr>  <dbl>
    ##  1 fellowship_ring Elf    female  1229
    ##  2 fellowship_ring Elf    male     971
    ##  3 fellowship_ring Hobbit female    14
    ##  4 fellowship_ring Hobbit male    3644
    ##  5 fellowship_ring Man    female     0
    ##  6 fellowship_ring Man    male    1995
    ##  7 two_towers      Elf    female   331
    ##  8 two_towers      Elf    male     513
    ##  9 two_towers      Hobbit female     0
    ## 10 two_towers      Hobbit male    2463
    ## 11 two_towers      Man    female   401
    ## 12 two_towers      Man    male    3589
    ## 13 return_king     Elf    female   183
    ## 14 return_king     Elf    male     510
    ## 15 return_king     Hobbit female     2
    ## 16 return_king     Hobbit male    2673
    ## 17 return_king     Man    female   268
    ## 18 return_king     Man    male    2459

## joining datasets

import fas

``` r
pups_df = read_csv("./data//FAS_pups.csv") %>% 
  janitor::clean_names() %>% 
  mutate(sex = recode(sex,'1' = "male", '2' = "female"))
```

    ## Rows: 313 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
pups_df
```

    ## # A tibble: 313 × 6
    ##    litter_number sex   pd_ears pd_eyes pd_pivot pd_walk
    ##    <chr>         <chr>   <dbl>   <dbl>    <dbl>   <dbl>
    ##  1 #85           male        4      13        7      11
    ##  2 #85           male        4      13        7      12
    ##  3 #1/2/95/2     male        5      13        7       9
    ##  4 #1/2/95/2     male        5      13        8      10
    ##  5 #5/5/3/83/3-3 male        5      13        8      10
    ##  6 #5/5/3/83/3-3 male        5      14        6       9
    ##  7 #5/4/2/95/2   male       NA      14        5       9
    ##  8 #4/2/95/3-3   male        4      13        6       8
    ##  9 #4/2/95/3-3   male        4      13        7       9
    ## 10 #2/2/95/3-2   male        4      NA        8      10
    ## # … with 303 more rows

``` r
litters_df = read_csv("./data/FAS_litters.csv") %>% 
  janitor::clean_names() %>% 
  relocate(litter_number) %>% 
  separate(group, into = c("dose","day_of_tx"), sep = 3)
```

    ## Rows: 49 Columns: 8
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
litters_df
```

    ## # A tibble: 49 × 9
    ##    litter_number   dose  day_o…¹ gd0_w…² gd18_…³ gd_of…⁴ pups_…⁵ pups_…⁶ pups_…⁷
    ##    <chr>           <chr> <chr>     <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>
    ##  1 #85             Con   7          19.7    34.7      20       3       4       3
    ##  2 #1/2/95/2       Con   7          27      42        19       8       0       7
    ##  3 #5/5/3/83/3-3   Con   7          26      41.4      19       6       0       5
    ##  4 #5/4/2/95/2     Con   7          28.5    44.1      19       5       1       4
    ##  5 #4/2/95/3-3     Con   7          NA      NA        20       6       0       6
    ##  6 #2/2/95/3-2     Con   7          NA      NA        20       6       0       4
    ##  7 #1/5/3/83/3-3/2 Con   7          NA      NA        20       9       0       9
    ##  8 #3/83/3-3       Con   8          NA      NA        20       9       1       8
    ##  9 #2/95/3         Con   8          NA      NA        20       8       0       8
    ## 10 #3/5/2/2/95     Con   8          28.5    NA        20       8       0       8
    ## # … with 39 more rows, and abbreviated variable names ¹​day_of_tx, ²​gd0_weight,
    ## #   ³​gd18_weight, ⁴​gd_of_birth, ⁵​pups_born_alive, ⁶​pups_dead_birth,
    ## #   ⁷​pups_survive

join

``` r
fas_df = 
  left_join(pups_df,litters_df,by = "litter_number") %>% 
  arrange(litter_number)

fas_df
```

    ## # A tibble: 313 × 14
    ##    litter_…¹ sex   pd_ears pd_eyes pd_pi…² pd_walk dose  day_o…³ gd0_w…⁴ gd18_…⁵
    ##    <chr>     <chr>   <dbl>   <dbl>   <dbl>   <dbl> <chr> <chr>     <dbl>   <dbl>
    ##  1 #1/2/95/2 male        5      13       7       9 Con   7            27      42
    ##  2 #1/2/95/2 male        5      13       8      10 Con   7            27      42
    ##  3 #1/2/95/2 fema…       4      13       7       9 Con   7            27      42
    ##  4 #1/2/95/2 fema…       4      13       7      10 Con   7            27      42
    ##  5 #1/2/95/2 fema…       5      13       8      10 Con   7            27      42
    ##  6 #1/2/95/2 fema…       5      13       8      10 Con   7            27      42
    ##  7 #1/2/95/2 fema…       5      13       6      10 Con   7            27      42
    ##  8 #1/5/3/8… male        4      NA      NA       9 Con   7            NA      NA
    ##  9 #1/5/3/8… male        4      NA       7       9 Con   7            NA      NA
    ## 10 #1/5/3/8… male        4      NA       7       9 Con   7            NA      NA
    ## # … with 303 more rows, 4 more variables: gd_of_birth <dbl>,
    ## #   pups_born_alive <dbl>, pups_dead_birth <dbl>, pups_survive <dbl>, and
    ## #   abbreviated variable names ¹​litter_number, ²​pd_pivot, ³​day_of_tx,
    ## #   ⁴​gd0_weight, ⁵​gd18_weight
