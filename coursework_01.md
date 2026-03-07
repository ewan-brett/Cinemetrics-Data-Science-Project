# Coursework 1: MA22019, Introduction to Data Science
Ewan Brett

<!--- DO NOT DELETE THIS LINE --->

<!--- METADATA ANCHOR: MA22019-CW1-2026 --->

Please read the `coursework_01_instructions.pdf` for the full context,
data dictionary, and detailed task requirements before starting.

``` r
library(tidyverse)
library(tidytext)
library(knitr)
```

## Part A: Data Wrangling & EDA

### Task 1: Data Joining

<!--- DO NOT DELETE THIS LINE - TASK 1 ANCHOR --->

``` r
# Write your Task 1 code here:
titles <- read_csv("data/titles.csv")
users <- read_csv("data/users.csv")
history <- read_csv("data/history.csv")

# cleaning the titles data
clean_titles <- distinct(titles)
dim(titles)[1] - dim(clean_titles)[1]
```

    [1] 10

``` r
# this has removed 10 duplicate rows

# cleaning the history data
clean_history <- distinct(history) %>% 
  mutate("watch_date"= gsub("/","-",watch_date))
dim(history)[1] - dim(clean_history)[1]
```

    [1] 311

``` r
# this has removed 311 duplicate rows

# joining the data
cinemetrics <- clean_history %>%
  left_join(clean_titles, by = "movie_id") %>%
  left_join(users, by = "user_id") %>% 
  mutate("release_year"= as.integer(release_year), 
         "budget_usd" = as.integer(budget_usd),
         "age"= as.integer(age),
         "user_rating_100" = as.integer(user_rating_100),
         "watch_date" = as.Date(watch_date),
         "watch_duration_mins" = as.integer(watch_duration_mins),
         "engagement_score"= as.integer(engagement_score))

head(cinemetrics,10)
```

    # A tibble: 10 × 13
       user_id movie_id watch_date watch_duration_mins engagement_score
         <dbl>    <dbl> <date>                   <int>            <int>
     1     134       76 2025-03-22                 117               NA
     2    1353       25 2026-02-21                 103               65
     3    1367        1 2025-01-16                  15               24
     4    2154       16 2025-10-21                 130               74
     5     529       75 NA                          15               27
     6     201       50 2025-11-16                  52               41
     7     923       43 2026-02-22                  74               54
     8    2192       23 0012-04-25                  15               30
     9    2129       14 2025-04-11                 133               63
    10    2187       85 2025-11-06                  68               39
    # ℹ 8 more variables: user_rating_100 <int>, review_snippet <chr>,
    #   movie_title <chr>, release_year <int>, budget_usd <int>, genre_tags <chr>,
    #   age <int>, subscription_type <chr>

<!--- WRITE YOUR WRITTEN ANSWER BELOW THIS LINE --->

#### Written Interpretation

*Removing duplicates in history was necessary because otherwise
duplicate rows would remain after the join, which would skew the
analysis of the variables. Performing the join without cleaning the data
first leads to having 2593 additional rows. *

### Task 2: Missing Values Handling

<!--- DO NOT DELETE THIS LINE - TASK 2 ANCHOR --->

``` r
# Write your Task 2 code here:

cinemetrics %>% 
  is.na() %>% 
  colSums() %>% 
  knitr::kable(col.names = c("Variable", "Missing_Count"))
```

| Variable            | Missing_Count |
|:--------------------|--------------:|
| user_id             |             0 |
| movie_id            |             0 |
| watch_date          |          5110 |
| watch_duration_mins |           574 |
| engagement_score    |           582 |
| user_rating_100     |           985 |
| review_snippet      |          1939 |
| movie_title         |             0 |
| release_year        |             0 |
| budget_usd          |           980 |
| genre_tags          |             0 |
| age                 |            50 |
| subscription_type   |            50 |

``` r
cinemetrics$user_rating_100 %>% 
  median(na.rm = TRUE)
```

    [1] 71

<!--- WRITE YOUR WRITTEN ANSWER BELOW THIS LINE --->

#### Written Interpretation

*summary() gives a general overview of the data, but may not account for
missing values, especially if the variable is a str or a factor. We
cannot be sure how it treats missing values, and calculates statistics
like the median. Hence it is much more reliable to use is.na() to
programmatically calculate the missing values in our data. That way we
have the freedom over how these are handled, e.g. whether you remove
missing values, or assign them a specific value, this can be decided
based on context of the data which summary() wouldn’t handle as
effectively. *

### Task 3: Genre Grouping

<!--- DO NOT DELETE THIS LINE - TASK 3 ANCHOR --->

``` r
# Write your Task 3 code here:
```

``` r
# Write your verification code here:
```

<!--- WRITE YOUR WRITTEN ANSWER BELOW THIS LINE --->

#### Written Interpretation

*Replace this text with your empirical finding regarding the ‘romantic
comedy’ sub-genre.*

### Task 4: Genre Ratings Table

<!--- DO NOT DELETE THIS LINE - TASK 4 ANCHOR --->

``` r
# Write your Task 4 code here:
```

``` r
# Write your verification code here:
```

<!--- WRITE YOUR WRITTEN ANSWER BELOW THIS LINE --->

#### Written Interpretation

*Replace this text with your empirical finding regarding the budget
disparity between Horror and the blockbusters.*

### Task 5: Ratings and Engagement Scores

<!--- DO NOT DELETE THIS LINE - TASK 5 ANCHOR --->

``` r
# Write your Task 5 code here:
```

<!--- WRITE YOUR WRITTEN ANSWER BELOW THIS LINE --->

#### Written Interpretation

*Replace this text with your interpretation of the boxplots’ visual
footprints.*

``` r
# Write your verification code here:
```

#### Empirical Verification

*Replace this text with your empirical finding regarding the highest and
lowest IQRs.*

### Task 6: Engagement Score and Subscription Type

<!--- DO NOT DELETE THIS LINE - TASK 6 ANCHOR --->

``` r
# Write your Task 6 code here:
```

``` r
# Write your verification code here:
```

#### Empirical Verification

*Replace this text with your empirical finding regarding the
distribution of users across age and subscription tiers.*

<!--- WRITE YOUR WRITTEN ANSWER BELOW THIS LINE --->

#### Written Interpretation

*Replace this text with your explanation of the mathematical illusion.*

------------------------------------------------------------------------

## Part B: Text Data Analysis

### Task 7: Tokenization & Word Frequency

<!--- DO NOT DELETE THIS LINE - TASK 7 ANCHOR --->

``` r
# Write your Task 7 code here:
```

<!--- WRITE YOUR WRITTEN ANSWER BELOW THIS LINE --->

#### Written Interpretation

*Replace this text with your explanation of why the top two words are
unhelpful for analysis.*

### Task 8: Sentiment Analysis

<!--- DO NOT DELETE THIS LINE - TASK 8 ANCHOR --->

``` r
# Write your Task 8 code here:
```

<!--- WRITE YOUR WRITTEN ANSWER BELOW THIS LINE --->

#### Written Interpretation

*Replace this text with your explanation of the scatterplot outlier.*

### Task 9: Advanced Text Mining (TF-IDF)

<!--- DO NOT DELETE THIS LINE - TASK 9 ANCHOR --->

``` r
# Write your Task 9 code here: 
```

<!--- WRITE YOUR WRITTEN ANSWER BELOW THIS LINE --->

#### Written Interpretation

1.  *Replace this text with the TF-IDF score for the word “film”.*
2.  *Replace this text with your mathematical explanation using the IDF
    equation from the lecture.*
3.  *Replace this text with your comparison of TF-IDF vs frequency.*

### Task 10: Executive Recommendation

<!--- DO NOT DELETE THIS LINE - TASK 10 ANCHOR --->

<!--- WRITE YOUR WRITTEN ANSWER BELOW THIS LINE --->

#### Written Interpretation

*Replace this text with your one-paragraph executive recommendation (max
150 words), citing at least three specific findings from earlier tasks
and acknowledging one limitation.*

------------------------------------------------------------------------

<!--- WORD COUNT ANCHOR --->

    **Prose Word Count:** 321 words
