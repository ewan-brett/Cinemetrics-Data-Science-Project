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

``` r
cinemetrics_messy <- history %>%
  left_join(titles, by = "movie_id") %>%
  left_join(users, by = "user_id")
```

    Warning in left_join(., titles, by = "movie_id"): Detected an unexpected many-to-many relationship between `x` and `y`.
    ℹ Row 14 of `x` matches multiple rows in `y`.
    ℹ Row 85 of `y` matches multiple rows in `x`.
    ℹ If a many-to-many relationship is expected, set `relationship =
      "many-to-many"` to silence this warning.

``` r
dim(cinemetrics)[1]
```

    [1] 19965

``` r
dim(cinemetrics_messy)[1]
```

    [1] 22558

<!--- WRITE YOUR WRITTEN ANSWER BELOW THIS LINE --->

#### Written Interpretation

*Removing duplicates in history was necessary because otherwise
duplicate rows would remain after the join, which would skew the
analysis of the variables. Performing the join without cleaning the data
first results in a dataset with 22558 rows, whereas with cleaned data we
have 19965 rows.*

### Task 2: Missing Values Handling

<!--- DO NOT DELETE THIS LINE - TASK 2 ANCHOR --->

``` r
# Write your Task 2 code here:

cinemetrics %>% 
  is.na() %>% 
  colSums() %>% 
  kable(col.names = c("Variable", "Missing_Count"))
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

``` r
cinemetrics$user_rating_100 %>% 
  replace_na(0) %>% 
  median()
```

    [1] 70

<!--- WRITE YOUR WRITTEN ANSWER BELOW THIS LINE --->

#### Written Interpretation

*summary() gives a general overview of the data, but may not account for
missing values, especially if the variable is a str or a factor. We
cannot be sure how it treats missing values, and calculates statistics
like the median. Hence it is much more reliable to use is.na() to
programmatically calculate the missing values in our data, and this
shows us that there are 985 for user_rating_100. That way we have the
freedom over how these are handled, e.g. whether you remove missing
values, or assign them a specific value, this can be decided based on
context of the data which summary() wouldn’t handle as effectively. For
example the median after removing the missing values of user_rating_100
is 71, however if we knew that users don’t bother rating films they
didn’t like, then it would make more sense to assign missing values as
0, giving us a median of 70. *

### Task 3: Genre Grouping

<!--- DO NOT DELETE THIS LINE - TASK 3 ANCHOR --->

``` r
# Write your Task 3 code here:
head(cinemetrics,20)
```

    # A tibble: 20 × 13
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
    11    1114       37 2025-09-29                  42               36
    12     119       60 NA                         115               63
    13    1767       90 2025-03-12                  28               19
    14    2293       35 2025-12-16                  22               19
    15     199       88 NA                          52               12
    16    1380        5 0009-11-25                  37               26
    17    2258       92 2025-06-25                  32               35
    18    2526       85 2025-08-11                  89               56
    19    1571       49 2025-03-08                  21               26
    20    2215       86 2025-11-16                  19               40
    # ℹ 8 more variables: user_rating_100 <int>, review_snippet <chr>,
    #   movie_title <chr>, release_year <int>, budget_usd <int>, genre_tags <chr>,
    #   age <int>, subscription_type <chr>

``` r
unique(cinemetrics$genre_tags)
```

     [1] "animation"                   "comedy, family"             
     [3] "action"                      "Science fiction"            
     [5] "animated"                    "drama, emotional"           
     [7] "comedy"                      "sci-fi"                     
     [9] "drama, critically-acclaimed" "horror"                     
    [11] "romantic comedy"             "action, high-octane"        
    [13] "romantic"                    "rom-com"                    
    [15] "Drama"                       "sci-fi, fantasy"            
    [17] "terror"                      "scary"                      
    [19] "animation, kids"             "Romance"                    
    [21] "documentary, real-world"     "Documentary"                

``` r
cinemetrics <- cinemetrics %>%
  mutate(
    primary_genre = factor(genre_tags) %>%
      fct_recode(
        "Drama" = "drama, emotional",
        "Drama" = "drama, critically-acclaimed",
        "Romance" = "romantic comedy",
        "Romance" = "rom-com",
        "Comedy" = "comedy, family",
        "Comedy" = "comedy",
        "Sci-Fi" = "Science fiction",
        "Sci-Fi" = "sci-fi",
        "Sci-Fi" = "sci-fi, fantasy",
        "Animation" = "animated",
        "Animation" = "animation, kids",
        "Animation" = "animation",
        "Horror" = "terror",
        "Horror" = "scary",
        "Documentary" = "documentary, real-world",
        "Romance" = "romantic",
        "Action" = "action",
        "Action" = "action, high-octane",
        "Horror" = "horror") %>%
      fct_lump_n(n = 5, other_level = "Other"))


unique(cinemetrics$primary_genre)
```

    [1] Animation Comedy    Other     Sci-Fi    Drama     Romance  
    Levels: Animation Comedy Drama Romance Sci-Fi Other

``` r
cinemetrics %>%
  count(primary_genre, sort = TRUE) %>%
  kable()
```

| primary_genre |    n |
|:--------------|-----:|
| Animation     | 4207 |
| Other         | 3728 |
| Sci-Fi        | 3543 |
| Drama         | 3307 |
| Comedy        | 2885 |
| Romance       | 2295 |

``` r
# Write your verification code here:

cinemetrics %>%
  group_by(genre_tags) %>%
  summarise(
    avg_rating = mean(user_rating_100, na.rm = TRUE),  
    movie_count = n()) %>%
  arrange(desc(movie_count)) %>%
  filter(genre_tags %in% c("Romance", "romantic", "rom-com", "romantic comedy")) %>% 
  kable(caption = "Romance vs Rom-Com")
```

| genre_tags      | avg_rating | movie_count |
|:----------------|-----------:|------------:|
| rom-com         |   69.80269 |        1009 |
| Romance         |   69.82660 |         991 |
| romantic        |   69.04196 |         150 |
| romantic comedy |   66.35036 |         145 |

Romance vs Rom-Com

<!--- WRITE YOUR WRITTEN ANSWER BELOW THIS LINE --->

#### Written Interpretation

NEED TO CHANGE THIS:

*This table shows us the original “genre_tags” has inconsistent labeling
which makes it unsuitable to perform proper analysis. We see there are
larger categories such as “rom-com” (1009 movies) and “romance” (991
movies), however we also have smaller categories representing the same
group but differently names, such as “romantic” (150) and “romantic
comedy” (145). Clearly “rom-com” and “romantic comedy” should be put in
the same group and “romance” and “romantic” should also be the same
group. However we then encounter a potential error which could effect
our analysis, as “romantic” is just an adjective, and potentially there
could be Rom-Coms in that category, which would be mislabeled if we kept
Rom-Com and Romance as seperate categories. Further, I found that when
we keep them as seperate categories, and group by the most frequent five
categories, “Other” becomes the largest category with 4225 entries. It
does not make sense to have such a large “Other” category when these can
be grouped into bigger categories with similar properties. When we group
them together, “Other” has only 2295 entries, hence we have more
insightful categories. Hence, I have decided grouping Rom-coms and
Romances as one category will be the best for our analysis, and allow
for better interpretability.*

### Task 4: Genre Ratings Table

<!--- DO NOT DELETE THIS LINE - TASK 4 ANCHOR --->

``` r
# Write your Task 4 code here:
cinemetrics %>%
  group_by(primary_genre) %>%
  summarise(
    avg_rating = mean(user_rating_100, na.rm = TRUE),  
    movie_count = n()) %>%
  arrange(desc(avg_rating)) %>%
  kable(caption = "Romance vs Rom-Com")
```

| primary_genre | avg_rating | movie_count |
|:--------------|-----------:|------------:|
| Other         |   74.34502 |        3728 |
| Comedy        |   70.07359 |        2885 |
| Drama         |   69.83692 |        3307 |
| Animation     |   69.69912 |        4207 |
| Sci-Fi        |   69.68587 |        3543 |
| Romance       |   69.54708 |        2295 |

Romance vs Rom-Com

``` r
cinemetrics_test <- cinemetrics %>%
  mutate(primary_genre = factor(genre_tags) %>%
     fct_recode(
                 "Drama" = "drama, emotional",
                 "Drama" = "drama, critically-acclaimed",
                 "Rom-Com" = "romantic comedy",
                 "Rom-Com" = "rom-com",
                 "Comedy" = "comedy, family",
                 "Comedy" = "comedy",
                 "Sci-Fi" = "Science fiction",
                 "Sci-Fi" = "sci-fi",
                 "Sci-Fi" = "sci-fi, fantasy",
                 "Animation" = "animated",
                "Animation" = "animation, kids",
                "Animation" = "animation",
                 "Horror" = "terror",
                 "Horror" = "scary",
                 "Documentary" = "documentary, real-world",
                 "Romance" = "romantic",
                 "Action" = "action",
                 "Action" = "action, high-octane",
                 "Horror" = "horror"))

     
cinemetrics_test %>%
count(primary_genre, sort = TRUE) %>%
knitr::kable()
```

| primary_genre |    n |
|:--------------|-----:|
| Animation     | 4207 |
| Sci-Fi        | 3543 |
| Drama         | 3307 |
| Comedy        | 2885 |
| Horror        | 1798 |
| Action        | 1585 |
| Rom-Com       | 1154 |
| Romance       | 1141 |
| Documentary   |  345 |

``` r
# Write your verification code here:
cinemetrics %>%
  group_by(primary_genre) %>%
  summarise(
    avg_rating = mean(user_rating_100, na.rm = TRUE), 
    avg_budget_USD = mean(budget_usd, na.rm = TRUE),
    movie_count = n()) %>%
  arrange(desc(avg_rating)) %>%
  kable(caption = "Budget (USD) vs Average rating (categories lumped)")
```

| primary_genre | avg_rating | avg_budget_USD | movie_count |
|:--------------|-----------:|---------------:|------------:|
| Other         |   74.34502 |       58846173 |        3728 |
| Comedy        |   70.07359 |       48351193 |        2885 |
| Drama         |   69.83692 |       22857593 |        3307 |
| Animation     |   69.69912 |      122994708 |        4207 |
| Sci-Fi        |   69.68587 |      139588009 |        3543 |
| Romance       |   69.54708 |       56494292 |        2295 |

Budget (USD) vs Average rating (categories lumped)

``` r
cinemetrics_test %>%
  group_by(primary_genre) %>%
  summarise(
    avg_rating = mean(user_rating_100, na.rm = TRUE), 
    avg_budget_USD = mean(budget_usd, na.rm = TRUE),
    movie_count = n()) %>%
  arrange(desc(avg_rating)) %>%
  filter(primary_genre %in% c("Horror", "Sci-Fi", "Action")) %>% 
  kable(caption = "Budget (USD) vs Average rating (categories not lumped)")
```

| primary_genre | avg_rating | avg_budget_USD | movie_count |
|:--------------|-----------:|---------------:|------------:|
| Horror        |   79.77719 |       10181094 |        1798 |
| Sci-Fi        |   69.68587 |      139588009 |        3543 |
| Action        |   69.21661 |      115543533 |        1585 |

Budget (USD) vs Average rating (categories not lumped)

<!--- WRITE YOUR WRITTEN ANSWER BELOW THIS LINE --->

#### Written Interpretation

*Sci-Fi has the largest budget (139588009USD), however it has the second
lowest average rating. We see here that Horror and Action are not part
of the top 5 categories, and therefore come under “Other”. The average
budget for “Other” (58846173USD) is roughly half of the Sci-Fi budget,
however the average ratings (74.35) are much higher. Hence larger budget
does not guarantee higher average rating. We can look at the unlumped
categories, which further reinforces this as Horror has by far the best
ratings (79.78), however has the smallest budget (10181094USD), and
Action has the lowest rating (69.22) and a much larger budget
(115543533USD). A very broad category like “Action” can mean that
performance of many different sub-categories balance out, so we cannot
really make good inferences from the data. For example, more niche
categories which don’t perform well won’t show in our inferences as they
will be averaged out by more successful subcategories within “Action”.*

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

    **Prose Word Count:** 727 words
