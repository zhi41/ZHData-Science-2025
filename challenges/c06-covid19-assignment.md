COVID-19
================
(Khor Zhi Hong)
2020-3-9

- [Grading Rubric](#grading-rubric)
  - [Individual](#individual)
  - [Submission](#submission)
- [The Big Picture](#the-big-picture)
- [Get the Data](#get-the-data)
  - [Navigating the Census Bureau](#navigating-the-census-bureau)
    - [**q1** Load Table `B01003` into the following tibble. Make sure
      the column names are
      `id, Geographic Area Name, Estimate!!Total, Margin of Error!!Total`.](#q1-load-table-b01003-into-the-following-tibble-make-sure-the-column-names-are-id-geographic-area-name-estimatetotal-margin-of-errortotal)
  - [Automated Download of NYT Data](#automated-download-of-nyt-data)
    - [**q2** Visit the NYT GitHub repo and find the URL for the **raw**
      US County-level data. Assign that URL as a string to the variable
      below.](#q2-visit-the-nyt-github-repo-and-find-the-url-for-the-raw-us-county-level-data-assign-that-url-as-a-string-to-the-variable-below)
- [Join the Data](#join-the-data)
  - [**q3** Process the `id` column of `df_pop` to create a `fips`
    column.](#q3-process-the-id-column-of-df_pop-to-create-a-fips-column)
  - [**q4** Join `df_covid` with `df_q3` by the `fips` column. Use the
    proper type of join to preserve *only* the rows in
    `df_covid`.](#q4-join-df_covid-with-df_q3-by-the-fips-column-use-the-proper-type-of-join-to-preserve-only-the-rows-in-df_covid)
- [Analyze](#analyze)
  - [Normalize](#normalize)
    - [**q5** Use the `population` estimates in `df_data` to normalize
      `cases` and `deaths` to produce per 100,000 counts \[3\]. Store
      these values in the columns `cases_per100k` and
      `deaths_per100k`.](#q5-use-the-population-estimates-in-df_data-to-normalize-cases-and-deaths-to-produce-per-100000-counts-3-store-these-values-in-the-columns-cases_per100k-and-deaths_per100k)
  - [Guided EDA](#guided-eda)
    - [**q6** Compute some summaries](#q6-compute-some-summaries)
    - [**q7** Find and compare the top
      10](#q7-find-and-compare-the-top-10)
  - [Self-directed EDA](#self-directed-eda)
    - [**q8** Drive your own ship: You’ve just put together a very rich
      dataset; you now get to explore! Pick your own direction and
      generate at least one punchline figure to document an interesting
      finding. I give a couple tips & ideas
      below:](#q8-drive-your-own-ship-youve-just-put-together-a-very-rich-dataset-you-now-get-to-explore-pick-your-own-direction-and-generate-at-least-one-punchline-figure-to-document-an-interesting-finding-i-give-a-couple-tips--ideas-below)
    - [Ideas](#ideas)
    - [Aside: Some visualization
      tricks](#aside-some-visualization-tricks)
    - [Geographic exceptions](#geographic-exceptions)
- [Notes](#notes)

*Purpose*: In this challenge, you’ll learn how to navigate the U.S.
Census Bureau website, programmatically download data from the internet,
and perform a county-level population-weighted analysis of current
COVID-19 trends. This will give you the base for a very deep
investigation of COVID-19, which we’ll build upon for Project 1.

<!-- include-rubric -->

# Grading Rubric

<!-- -------------------------------------------------- -->

Unlike exercises, **challenges will be graded**. The following rubrics
define how you will be graded, both on an individual and team basis.

## Individual

<!-- ------------------------- -->

| Category | Needs Improvement | Satisfactory |
|----|----|----|
| Effort | Some task **q**’s left unattempted | All task **q**’s attempted |
| Observed | Did not document observations, or observations incorrect | Documented correct observations based on analysis |
| Supported | Some observations not clearly supported by analysis | All observations clearly supported by analysis (table, graph, etc.) |
| Assessed | Observations include claims not supported by the data, or reflect a level of certainty not warranted by the data | Observations are appropriately qualified by the quality & relevance of the data and (in)conclusiveness of the support |
| Specified | Uses the phrase “more data are necessary” without clarification | Any statement that “more data are necessary” specifies which *specific* data are needed to answer what *specific* question |
| Code Styled | Violations of the [style guide](https://style.tidyverse.org/) hinder readability | Code sufficiently close to the [style guide](https://style.tidyverse.org/) |

## Submission

<!-- ------------------------- -->

Make sure to commit both the challenge report (`report.md` file) and
supporting files (`report_files/` folder) when you are done! Then submit
a link to Canvas. **Your Challenge submission is not complete without
all files uploaded to GitHub.**

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.4     ✔ readr     2.1.5
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.5.1     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

*Background*:
[COVID-19](https://en.wikipedia.org/wiki/Coronavirus_disease_2019) is
the disease caused by the virus SARS-CoV-2. In 2020 it became a global
pandemic, leading to huge loss of life and tremendous disruption to
society. The New York Times (as of writing) publishes up-to-date data on
the progression of the pandemic across the United States—we will study
these data in this challenge.

*Optional Readings*: I’ve found this [ProPublica
piece](https://www.propublica.org/article/how-to-understand-covid-19-numbers)
on “How to understand COVID-19 numbers” to be very informative!

# The Big Picture

<!-- -------------------------------------------------- -->

We’re about to go through *a lot* of weird steps, so let’s first fix the
big picture firmly in mind:

We want to study COVID-19 in terms of data: both case counts (number of
infections) and deaths. We’re going to do a county-level analysis in
order to get a high-resolution view of the pandemic. Since US counties
can vary widely in terms of their population, we’ll need population
estimates in order to compute infection rates (think back to the
`Titanic` challenge).

That’s the high-level view; now let’s dig into the details.

# Get the Data

<!-- -------------------------------------------------- -->

1.  County-level population estimates (Census Bureau)
2.  County-level COVID-19 counts (New York Times)

## Navigating the Census Bureau

<!-- ------------------------- -->

**Steps**: Our objective is to find the 2018 American Community
Survey\[1\] (ACS) Total Population estimates, disaggregated by counties.
To check your results, this is Table `B01003`.

1.  Go to [data.census.gov](data.census.gov).
2.  Scroll down and click `View Tables`.
3.  Apply filters to find the ACS **Total Population** estimates,
    disaggregated by counties. I used the filters:

- `Topics > Populations and People > Counts, Estimates, and Projections > Population Total`
- `Geography > County > All counties in United States`

5.  Select the **Total Population** table and click the `Download`
    button to download the data; make sure to select the 2018 5-year
    estimates.
6.  Unzip and move the data to your `challenges/data` folder.

- Note that the data will have a crazy-long filename like
  `ACSDT5Y2018.B01003_data_with_overlays_2020-07-26T094857.csv`. That’s
  because metadata is stored in the filename, such as the year of the
  estimate (`Y2018`) and my access date (`2020-07-26`). **Your filename
  will vary based on when you download the data**, so make sure to copy
  the filename that corresponds to what you downloaded!

### **q1** Load Table `B01003` into the following tibble. Make sure the column names are `id, Geographic Area Name, Estimate!!Total, Margin of Error!!Total`.

*Hint*: You will need to use the `skip` keyword when loading these data!

``` r
## TASK: Load the census bureau data with the following tibble name.
df_pop <- "./data/ACSDT5Y2018.B01003-Data.csv"

df_pop <- read_csv(df_pop, skip = 1) %>%
  select(
    id = 1, 
    `Geographic Area Name` = 2, 
    `Estimate!!Total` = 3, 
    `Margin of Error!!Total` = 4
  )
```

    ## New names:
    ## Rows: 3220 Columns: 5
    ## ── Column specification
    ## ──────────────────────────────────────────────────────── Delimiter: "," chr
    ## (3): Geography, Geographic Area Name, Margin of Error!!Total dbl (1):
    ## Estimate!!Total lgl (1): ...5
    ## ℹ Use `spec()` to retrieve the full column specification for this data. ℹ
    ## Specify the column types or set `show_col_types = FALSE` to quiet this message.
    ## • `` -> `...5`

``` r
head(df_pop)
```

    ## # A tibble: 6 × 4
    ##   id             `Geographic Area Name` `Estimate!!Total` Margin of Error!!Tot…¹
    ##   <chr>          <chr>                              <dbl> <chr>                 
    ## 1 0500000US01001 Autauga County, Alaba…             55200 *****                 
    ## 2 0500000US01003 Baldwin County, Alaba…            208107 *****                 
    ## 3 0500000US01005 Barbour County, Alaba…             25782 *****                 
    ## 4 0500000US01007 Bibb County, Alabama               22527 *****                 
    ## 5 0500000US01009 Blount County, Alabama             57645 *****                 
    ## 6 0500000US01011 Bullock County, Alaba…             10352 *****                 
    ## # ℹ abbreviated name: ¹​`Margin of Error!!Total`

*Note*: You can find information on 1-year, 3-year, and 5-year estimates
[here](https://www.census.gov/programs-surveys/acs/guidance/estimates.html).
The punchline is that 5-year estimates are more reliable but less
current.

## Automated Download of NYT Data

<!-- ------------------------- -->

ACS 5-year estimates don’t change all that often, but the COVID-19 data
are changing rapidly. To that end, it would be nice to be able to
*programmatically* download the most recent data for analysis; that way
we can update our analysis whenever we want simply by re-running our
notebook. This next problem will have you set up such a pipeline.

The New York Times is publishing up-to-date data on COVID-19 on
[GitHub](https://github.com/nytimes/covid-19-data).

### **q2** Visit the NYT [GitHub](https://github.com/nytimes/covid-19-data) repo and find the URL for the **raw** US County-level data. Assign that URL as a string to the variable below.

``` r
## TASK: Find the URL for the NYT covid-19 county-level data
url_counties <- 'https://raw.githubusercontent.com/nytimes/covid-19-data/refs/heads/master/us-counties.csv'
```

Once you have the url, the following code will download a local copy of
the data, then load the data into R.

``` r
## NOTE: No need to change this; just execute
## Set the filename of the data to download
filename_nyt <- "./data/nyt_counties.csv"

## Download the data locally
curl::curl_download(
        url_counties,
        destfile = filename_nyt
      )

## Loads the downloaded csv
df_covid <- read_csv(filename_nyt)
```

    ## Rows: 2502832 Columns: 6
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (3): county, state, fips
    ## dbl  (2): cases, deaths
    ## date (1): date
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

You can now re-run the chunk above (or the entire notebook) to pull the
most recent version of the data. Thus you can periodically re-run this
notebook to check in on the pandemic as it evolves.

*Note*: You should feel free to copy-paste the code above for your own
future projects!

# Join the Data

<!-- -------------------------------------------------- -->

To get a sense of our task, let’s take a glimpse at our two data
sources.

``` r
## NOTE: No need to change this; just execute
df_pop %>% glimpse
```

    ## Rows: 3,220
    ## Columns: 4
    ## $ id                       <chr> "0500000US01001", "0500000US01003", "0500000U…
    ## $ `Geographic Area Name`   <chr> "Autauga County, Alabama", "Baldwin County, A…
    ## $ `Estimate!!Total`        <dbl> 55200, 208107, 25782, 22527, 57645, 10352, 20…
    ## $ `Margin of Error!!Total` <chr> "*****", "*****", "*****", "*****", "*****", …

``` r
df_covid %>% glimpse
```

    ## Rows: 2,502,832
    ## Columns: 6
    ## $ date   <date> 2020-01-21, 2020-01-22, 2020-01-23, 2020-01-24, 2020-01-24, 20…
    ## $ county <chr> "Snohomish", "Snohomish", "Snohomish", "Cook", "Snohomish", "Or…
    ## $ state  <chr> "Washington", "Washington", "Washington", "Illinois", "Washingt…
    ## $ fips   <chr> "53061", "53061", "53061", "17031", "53061", "06059", "17031", …
    ## $ cases  <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, …
    ## $ deaths <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, …

To join these datasets, we’ll need to use [FIPS county
codes](https://en.wikipedia.org/wiki/FIPS_county_code).\[2\] The last
`5` digits of the `id` column in `df_pop` is the FIPS county code, while
the NYT data `df_covid` already contains the `fips`.

### **q3** Process the `id` column of `df_pop` to create a `fips` column.

``` r
## TASK: Create a `fips` column by extracting the county code
df_q3 <- df_pop %>%
  mutate(fips = str_sub(id, -5, -1))  


df_q3 %>% glimpse()
```

    ## Rows: 3,220
    ## Columns: 5
    ## $ id                       <chr> "0500000US01001", "0500000US01003", "0500000U…
    ## $ `Geographic Area Name`   <chr> "Autauga County, Alabama", "Baldwin County, A…
    ## $ `Estimate!!Total`        <dbl> 55200, 208107, 25782, 22527, 57645, 10352, 20…
    ## $ `Margin of Error!!Total` <chr> "*****", "*****", "*****", "*****", "*****", …
    ## $ fips                     <chr> "01001", "01003", "01005", "01007", "01009", …

Use the following test to check your answer.

``` r
## NOTE: No need to change this
## Check known county
assertthat::assert_that(
              (df_q3 %>%
              filter(str_detect(`Geographic Area Name`, "Autauga County")) %>%
              pull(fips)) == "01001"
            )
```

    ## [1] TRUE

``` r
print("Very good!")
```

    ## [1] "Very good!"

### **q4** Join `df_covid` with `df_q3` by the `fips` column. Use the proper type of join to preserve *only* the rows in `df_covid`.

``` r
## TASK: Join df_covid and df_q3 by fips.
df_q4 <- df_covid %>%
  left_join(df_q3, by = "fips")

df_q4 %>% glimpse()
```

    ## Rows: 2,502,832
    ## Columns: 10
    ## $ date                     <date> 2020-01-21, 2020-01-22, 2020-01-23, 2020-01-…
    ## $ county                   <chr> "Snohomish", "Snohomish", "Snohomish", "Cook"…
    ## $ state                    <chr> "Washington", "Washington", "Washington", "Il…
    ## $ fips                     <chr> "53061", "53061", "53061", "17031", "53061", …
    ## $ cases                    <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, …
    ## $ deaths                   <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, …
    ## $ id                       <chr> "0500000US53061", "0500000US53061", "0500000U…
    ## $ `Geographic Area Name`   <chr> "Snohomish County, Washington", "Snohomish Co…
    ## $ `Estimate!!Total`        <dbl> 786620, 786620, 786620, 5223719, 786620, 3164…
    ## $ `Margin of Error!!Total` <chr> "*****", "*****", "*****", "*****", "*****", …

Use the following test to check your answer.

``` r
## NOTE: No need to change this
if (!any(df_q4 %>% pull(fips) %>% str_detect(., "02105"), na.rm = TRUE)) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Your df_q4 contains a row for the Hoonah-Angoon Census Area (AK),",
    "which is not in df_covid. You used the incorrect join type.",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
if (any(df_q4 %>% pull(fips) %>% str_detect(., "78010"), na.rm = TRUE)) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Your df_q4 does not include St. Croix, US Virgin Islands,",
    "which is in df_covid. You used the incorrect join type.",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
print("Very good!")
```

    ## [1] "Very good!"

For convenience, I down-select some columns and produce more convenient
column names.

``` r
## NOTE: No need to change; run this to produce a more convenient tibble
df_data <-
  df_q4 %>%
  select(
    date,
    county,
    state,
    fips,
    cases,
    deaths,
    population = `Estimate!!Total`
  )
```

# Analyze

<!-- -------------------------------------------------- -->

Now that we’ve done the hard work of loading and wrangling the data, we
can finally start our analysis. Our first step will be to produce county
population-normalized cases and death counts. Then we will explore the
data.

## Normalize

<!-- ------------------------- -->

### **q5** Use the `population` estimates in `df_data` to normalize `cases` and `deaths` to produce per 100,000 counts \[3\]. Store these values in the columns `cases_per100k` and `deaths_per100k`.

``` r
## TASK: Normalize cases and deaths
df_normalized <-
  df_data %>%
  mutate(
    cases_per100k = (cases/population) * 100000,
    deaths_per100k = (deaths/population) * 100000
  )


# df_normalized %>% glimpse()
# df_normalized%>%
#   filter(fips == "48301")
```

You may use the following test to check your work.

``` r
## NOTE: No need to change this
## Check known county data
if (any(df_normalized %>% pull(date) %>% str_detect(., "2020-01-21"))) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Date 2020-01-21 not found; did you download the historical data (correct),",
    "or just the most recent data (incorrect)?",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
if (any(df_normalized %>% pull(date) %>% str_detect(., "2022-05-13"))) {
  assertthat::assert_that(TRUE)
} else {
  print(str_c(
    "Date 2022-05-13 not found; did you download the historical data (correct),",
    "or a single year's data (incorrect)?",
    sep = " "
  ))
  assertthat::assert_that(FALSE)
}
```

    ## [1] TRUE

``` r
## Check datatypes
assertthat::assert_that(is.numeric(df_normalized$cases))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$deaths))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$population))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$cases_per100k))
```

    ## [1] TRUE

``` r
assertthat::assert_that(is.numeric(df_normalized$deaths_per100k))
```

    ## [1] TRUE

``` r
## Check that normalization is correct
assertthat::assert_that(
              abs(df_normalized %>%
               filter(
                 str_detect(county, "Snohomish"),
                 date == "2020-01-21"
               ) %>%
              pull(cases_per100k) - 0.127) < 1e-3
            )
```

    ## [1] TRUE

``` r
assertthat::assert_that(
              abs(df_normalized %>%
               filter(
                 str_detect(county, "Snohomish"),
                 date == "2020-01-21"
               ) %>%
              pull(deaths_per100k) - 0) < 1e-3
            )
```

    ## [1] TRUE

``` r
print("Excellent!")
```

    ## [1] "Excellent!"

## Guided EDA

<!-- ------------------------- -->

Before turning you loose, let’s complete a couple guided EDA tasks.

### **q6** Compute some summaries

Compute the mean and standard deviation for `cases_per100k` and
`deaths_per100k`. *Make sure to carefully choose **which rows** to
include in your summaries,* and justify why!

``` r
## TASK: Compute mean and sd for cases_per100k and deaths_per100k


df_summary <- df_normalized %>%
  filter(date == max(date), !is.na(population), population > 0) %>%
  summarise(
    mean_cases_per100k = mean(cases_per100k, na.rm = TRUE),
    sd_cases_per100k   = sd(cases_per100k, na.rm = TRUE),
    mean_deaths_per100k = mean(deaths_per100k, na.rm = TRUE),
    sd_deaths_per100k   = sd(deaths_per100k, na.rm = TRUE)
  )



df_summary
```

    ## # A tibble: 1 × 4
    ##   mean_cases_per100k sd_cases_per100k mean_deaths_per100k sd_deaths_per100k
    ##                <dbl>            <dbl>               <dbl>             <dbl>
    ## 1             24774.            6233.                375.              160.

``` r
# df_summary2
# df_summary3
```

- Which rows did you pick?
  - Rows where population is greater than 0
  - Filter out rows where there is a Na value
  - only the latest date is used for each county
- Why?
  - when doing mean and standard deviation the missing population such
    as 0 will greatly distort the calculation, giving infinite result
    -There are NA values in the population and using the NA values into
    the calculation will return NA therefore NA data are removed -the
    data is the collection of covid cases and death and is update by the
    date, thus most data have repeats, starting from very low and
    accumulative. Since the data is accumulative, it seems to be better
    to just use the latest date for comparison

### **q7** Find and compare the top 10

Find the top 10 counties in terms of `cases_per100k`, and the top 10 in
terms of `deaths_per100k`. Report the population of each county along
with the per-100,000 counts. Compare the counts against the mean values
you found in q6. Note any observations.

``` r
## TASK: Find the top 10 max cases_per100k counties; report populations as well

# df_pop %>%
#   filter(str_detect(`Geographic Area Name`, "Loving County"))
# df_covid %>%
#   filter(fips == "48301")
# huh somehow the data just have more covid cases than the population


top_10_cases <- df_normalized %>%
  arrange(desc(cases_per100k)) %>%
  distinct(county, .keep_all = TRUE) %>%
  slice(1:10) %>%
  select(
    county, 
    state, 
    date, 
    cases, 
    cases_per100k, 
    population)

top_10_cases
```

    ## # A tibble: 10 × 6
    ##    county                   state      date       cases cases_per100k population
    ##    <chr>                    <chr>      <date>     <dbl>         <dbl>      <dbl>
    ##  1 Loving                   Texas      2022-05-12   196       192157.        102
    ##  2 Chattahoochee            Georgia    2022-05-11  7486        69527.      10767
    ##  3 Nome Census Area         Alaska     2022-05-11  6245        62922.       9925
    ##  4 Northwest Arctic Borough Alaska     2022-05-11  4837        62542.       7734
    ##  5 Crowley                  Colorado   2022-05-13  3347        59449.       5630
    ##  6 Bethel Census Area       Alaska     2022-05-11 10362        57439.      18040
    ##  7 Dewey                    South Dak… 2022-03-30  3139        54317.       5779
    ##  8 Dimmit                   Texas      2022-05-12  5760        54019.      10663
    ##  9 Jim Hogg                 Texas      2022-05-12  2648        50133.       5282
    ## 10 Kusilvak Census Area     Alaska     2022-05-11  4084        49817.       8198

``` r
## TASK: Find the top 10 deaths_per100k counties; report populations as well

top_10_deaths <- df_normalized %>%
  arrange(desc(deaths_per100k)) %>%        
  distinct(county, .keep_all = TRUE) %>%     
  slice(1:10) %>%                          
  select(
    county, 
    state, 
    population, 
    deaths, 
    deaths_per100k)

top_10_deaths
```

    ## # A tibble: 10 × 5
    ##    county            state        population deaths deaths_per100k
    ##    <chr>             <chr>             <dbl>  <dbl>          <dbl>
    ##  1 McMullen          Texas               662      9          1360.
    ##  2 Galax city        Virginia           6638     78          1175.
    ##  3 Motley            Texas              1156     13          1125.
    ##  4 Hancock           Georgia            8535     90          1054.
    ##  5 Emporia city      Virginia           5381     55          1022.
    ##  6 Towns             Georgia           11417    116          1016.
    ##  7 Jerauld           South Dakota       2029     20           986.
    ##  8 Loving            Texas               102      1           980.
    ##  9 Robertson         Kentucky           2143     21           980.
    ## 10 Martinsville city Virginia          13101    124           946.

**Observations**: 24773.98 6232.789 375.1242 159.7369  
- (Note your observations here!) -There is significantly higher
cases_per100k than the mean_cases_per100k, and also for deaths_per100k
against mean_deaths_per100k .the reason maybe due top 10 county have
much smaller population compared to other county, with a lower number of
population, even a small number of case/deaths can result in an
extremely high per 100k rate

-loving have a the highest cases_per100k which maybe due to the
relatively small population size and having more cases than the
population. this maybe due to having recurring covid cases in the
population or people outside the population are also counted when they
visit the hospital in the county. -furthermore loving have top 10
death_per_100k despite only 1 death

- When did these “largest values” occur?
  - At the latest date as this is due to the data being accumulative,
    updated according to the date and adds up.

## Self-directed EDA

<!-- ------------------------- -->

### **q8** Drive your own ship: You’ve just put together a very rich dataset; you now get to explore! Pick your own direction and generate at least one punchline figure to document an interesting finding. I give a couple tips & ideas below:

``` r
latest_data <- df_normalized %>%
  filter(date == max(date)) %>% 
  filter(!is.na(population) & population > 0)

ggplot(latest_data, aes(x = population, y = cases_per100k)) +
  geom_point(alpha = 0.5, color = "blue") +
  scale_x_log10() +
  labs(
    title = "County Population vs COVID Cases_per_100k (Latest date)",
    x = "County Population (log scale)",
    y = "COVID Cases_per_100k"
  ) +
  theme_minimal()
```

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

``` r
ggplot(latest_data, aes(x = population, y = cases)) +
  geom_point(alpha = 0.5, color = "red") +
  scale_x_log10() +
  labs(
    title = "County Population vs COVID Cases (All Data)",
    x = "County Population (log scale)",
    y = "COVID Cases") +
  theme_minimal()
```

![](c06-covid19-assignment_files/figure-gfm/unnamed-chunk-1-2.png)<!-- -->

``` r
high_case_counties <- df_normalized %>%
  filter(cases_per100k > 50000) %>%
  arrange(desc(cases_per100k)) %>%
  distinct(county, .keep_all = TRUE)

case_counties <- df_normalized %>%
  filter(cases > 100000) %>%
  arrange(desc(cases)) %>%
  distinct(county, .keep_all = TRUE)

lowest_10_cases <- latest_data %>%
  arrange(cases) %>%
  distinct(county, .keep_all = TRUE) %>%
  slice(1:10) %>%
  select(
    county, 
    state, 
    date, 
    cases, 
    cases_per100k, 
    population)

lowest_10_cases
```

    ## # A tibble: 10 × 6
    ##    county    state        date       cases cases_per100k population
    ##    <chr>     <chr>        <date>     <dbl>         <dbl>      <dbl>
    ##  1 Kalawao   Hawaii       2022-05-13     1         1333.         75
    ##  2 Arthur    Nebraska     2022-05-13    31         7416.        418
    ##  3 Petroleum Montana      2022-05-13    35         8102.        432
    ##  4 King      Texas        2022-05-13    51        22368.        228
    ##  5 Slope     North Dakota 2022-05-13    63         8949.        704
    ##  6 Blaine    Nebraska     2022-05-13    65        13542.        480
    ##  7 McPherson Nebraska     2022-05-13    68        14978.        454
    ##  8 Sioux     Nebraska     2022-05-13    68         5371.       1266
    ##  9 Loup      Nebraska     2022-05-13    79        13504.        585
    ## 10 Harding   New Mexico   2022-05-13    85        18519.        459

``` r
high_case_counties
```

    ## # A tibble: 9 × 9
    ##   date       county            state fips  cases deaths population cases_per100k
    ##   <date>     <chr>             <chr> <chr> <dbl>  <dbl>      <dbl>         <dbl>
    ## 1 2022-05-12 Loving            Texas 48301   196      1        102       192157.
    ## 2 2022-05-11 Chattahoochee     Geor… 13053  7486     22      10767        69527.
    ## 3 2022-05-11 Nome Census Area  Alas… 02180  6245      5       9925        62922.
    ## 4 2022-05-11 Northwest Arctic… Alas… 02188  4837     13       7734        62542.
    ## 5 2022-05-13 Crowley           Colo… 08025  3347     30       5630        59449.
    ## 6 2022-05-11 Bethel Census Ar… Alas… 02050 10362     41      18040        57439.
    ## 7 2022-03-30 Dewey             Sout… 46041  3139     42       5779        54317.
    ## 8 2022-05-12 Dimmit            Texas 48127  5760     51      10663        54019.
    ## 9 2022-05-12 Jim Hogg          Texas 48247  2648     22       5282        50133.
    ## # ℹ 1 more variable: deaths_per100k <dbl>

``` r
case_counties
```

    ## # A tibble: 153 × 9
    ##    date       county        state   fips   cases deaths population cases_per100k
    ##    <date>     <chr>         <chr>   <chr>  <dbl>  <dbl>      <dbl>         <dbl>
    ##  1 2022-05-13 Los Angeles   Califo… 06037 2.91e6  32022   10098052        28802.
    ##  2 2022-05-13 New York City New Yo… <NA>  2.42e6  40267         NA           NA 
    ##  3 2022-05-11 Maricopa      Arizona 04013 1.28e6  17326    4253913        30174.
    ##  4 2022-05-06 Miami-Dade    Florida 12086 1.21e6  10917    2715516        44533.
    ##  5 2022-05-13 Cook          Illino… 17031 1.19e6  14936    5223719        22856.
    ##  6 2022-05-13 Harris        Texas   48201 1.03e6  10972    4602523        22439.
    ##  7 2022-05-13 San Diego     Califo… 06073 8.25e5   5271    3302833        24966.
    ##  8 2022-05-13 Riverside     Califo… 06065 6.27e5   6527    2383286        26295.
    ##  9 2022-05-06 Broward       Florida 12011 6.14e5   5841    1909151        32184.
    ## 10 2022-05-13 Orange        Califo… 06059 6.00e5   7023    3164182        18974.
    ## # ℹ 143 more rows
    ## # ℹ 1 more variable: deaths_per100k <dbl>

### Ideas

<!-- ------------------------- -->

- Look for outliers.
- Try web searching for news stories in some of the outlier counties.
- Investigate relationships between county population and counts.
- Do a deep-dive on counties that are important to you (e.g. where you
  or your family live).
- Fix the *geographic exceptions* noted below to study New York City.
- Your own idea!

**DO YOUR OWN ANALYSIS HERE**

I could not think of a punchline but i was curious about if there is a
correlation between small population size and variance of cases_per100k
due to with a lower number of population, even a small number of
case/deaths can result in an extremely high per 100k. But it turns out
that other loving being an extreme outlier, most of the data shows a
relatively similar range of cases_per100k regardless of the population
size. Where the medium sized county having quite a few showing greater
variance than those in the smaller population sized county.

I guess SIZE ISNT EVERYTHING, IT ALSO DEPENDS ON THE TECHNIQUE and the
efficiency of the covid policy put in place to curb the spread and also
many other factors like population density.

### Aside: Some visualization tricks

<!-- ------------------------- -->

These data get a little busy, so it’s helpful to know a few `ggplot`
tricks to help with the visualization. Here’s an example focused on
Massachusetts.

``` r
## NOTE: No need to change this; just an example
# df_normalized %>%
#   filter(
#     state == "Massachusetts", # Focus on Mass only
#     !is.na(fips), # fct_reorder2 can choke with missing data
#   ) %>%
# 
#   ggplot(
#     aes(date, cases_per100k, color = fct_reorder2(county, date, cases_per100k))
#   ) +
#   geom_line() +
#   scale_y_log10(labels = scales::label_number_si()) +
#   scale_color_discrete(name = "County") +
#   theme_minimal() +
#   labs(
#     x = "Date",
#     y = "Cases (per 100,000 persons)"
#   )
```

*Tricks*:

- I use `fct_reorder2` to *re-order* the color labels such that the
  color in the legend on the right is ordered the same as the vertical
  order of rightmost points on the curves. This makes it easier to
  reference the legend.
- I manually set the `name` of the color scale in order to avoid
  reporting the `fct_reorder2` call.
- I use `scales::label_number_si` to make the vertical labels more
  readable.
- I use `theme_minimal()` to clean up the theme a bit.
- I use `labs()` to give manual labels.

### Geographic exceptions

<!-- ------------------------- -->

The NYT repo documents some [geographic
exceptions](https://github.com/nytimes/covid-19-data#geographic-exceptions);
the data for New York, Kings, Queens, Bronx and Richmond counties are
consolidated under “New York City” *without* a fips code. Thus the
normalized counts in `df_normalized` are `NA`. To fix this, you would
need to merge the population data from the New York City counties, and
manually normalize the data.

# Notes

<!-- -------------------------------------------------- -->

\[1\] The census used to have many, many questions, but the ACS was
created in 2010 to remove some questions and shorten the census. You can
learn more in [this wonderful visual
history](https://pudding.cool/2020/03/census-history/) of the census.

\[2\] FIPS stands for [Federal Information Processing
Standards](https://en.wikipedia.org/wiki/Federal_Information_Processing_Standards);
these are computer standards issued by NIST for things such as
government data.

\[3\] Demographers often report statistics not in percentages (per 100
people), but rather in per 100,000 persons. This is [not always the
case](https://stats.stackexchange.com/questions/12810/why-do-demographers-give-rates-per-100-000-people)
though!
