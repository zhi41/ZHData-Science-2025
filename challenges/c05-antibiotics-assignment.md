Antibiotics
================
Khor Zhi Hong
2020-30-4

*Purpose*: Creating effective data visualizations is an *iterative*
process; very rarely will the first graph you make be the most
effective. The most effective thing you can do to be successful in this
iterative process is to *try multiple graphs* of the same data.

Furthermore, judging the effectiveness of a visual is completely
dependent on *the question you are trying to answer*. A visual that is
totally ineffective for one question may be perfect for answering a
different question.

In this challenge, you will practice *iterating* on data visualization,
and will anchor the *assessment* of your visuals using two different
questions.

*Note*: Please complete your initial visual design **alone**. Work on
both of your graphs alone, and save a version to your repo *before*
coming together with your team. This way you can all bring a diversity
of ideas to the table!

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

``` r
library(ggrepel)
```

*Background*: The data\[1\] we study in this challenge report the
[*minimum inhibitory
concentration*](https://en.wikipedia.org/wiki/Minimum_inhibitory_concentration)
(MIC) of three drugs for different bacteria. The smaller the MIC for a
given drug and bacteria pair, the more practical the drug is for
treating that particular bacteria. An MIC value of *at most* 0.1 is
considered necessary for treating human patients.

These data report MIC values for three antibiotics—penicillin,
streptomycin, and neomycin—on 16 bacteria. Bacteria are categorized into
a genus based on a number of features, including their resistance to
antibiotics.

``` r
## NOTE: If you extracted all challenges to the same location,
## you shouldn't have to change this filename
filename <- "./data/antibiotics.csv"

## Load the data
df_antibiotics <- read_csv(filename)
```

    ## Rows: 16 Columns: 5
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): bacteria, gram
    ## dbl (3): penicillin, streptomycin, neomycin
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
df_antibiotics %>% knitr::kable()
```

| bacteria                        | penicillin | streptomycin | neomycin | gram     |
|:--------------------------------|-----------:|-------------:|---------:|:---------|
| Aerobacter aerogenes            |    870.000 |         1.00 |    1.600 | negative |
| Brucella abortus                |      1.000 |         2.00 |    0.020 | negative |
| Bacillus anthracis              |      0.001 |         0.01 |    0.007 | positive |
| Diplococcus pneumonia           |      0.005 |        11.00 |   10.000 | positive |
| Escherichia coli                |    100.000 |         0.40 |    0.100 | negative |
| Klebsiella pneumoniae           |    850.000 |         1.20 |    1.000 | negative |
| Mycobacterium tuberculosis      |    800.000 |         5.00 |    2.000 | negative |
| Proteus vulgaris                |      3.000 |         0.10 |    0.100 | negative |
| Pseudomonas aeruginosa          |    850.000 |         2.00 |    0.400 | negative |
| Salmonella (Eberthella) typhosa |      1.000 |         0.40 |    0.008 | negative |
| Salmonella schottmuelleri       |     10.000 |         0.80 |    0.090 | negative |
| Staphylococcus albus            |      0.007 |         0.10 |    0.001 | positive |
| Staphylococcus aureus           |      0.030 |         0.03 |    0.001 | positive |
| Streptococcus fecalis           |      1.000 |         1.00 |    0.100 | positive |
| Streptococcus hemolyticus       |      0.001 |        14.00 |   10.000 | positive |
| Streptococcus viridans          |      0.005 |        10.00 |   40.000 | positive |

``` r
df_antibiotics
```

    ## # A tibble: 16 × 5
    ##    bacteria                        penicillin streptomycin neomycin gram    
    ##    <chr>                                <dbl>        <dbl>    <dbl> <chr>   
    ##  1 Aerobacter aerogenes               870             1       1.6   negative
    ##  2 Brucella abortus                     1             2       0.02  negative
    ##  3 Bacillus anthracis                   0.001         0.01    0.007 positive
    ##  4 Diplococcus pneumonia                0.005        11      10     positive
    ##  5 Escherichia coli                   100             0.4     0.1   negative
    ##  6 Klebsiella pneumoniae              850             1.2     1     negative
    ##  7 Mycobacterium tuberculosis         800             5       2     negative
    ##  8 Proteus vulgaris                     3             0.1     0.1   negative
    ##  9 Pseudomonas aeruginosa             850             2       0.4   negative
    ## 10 Salmonella (Eberthella) typhosa      1             0.4     0.008 negative
    ## 11 Salmonella schottmuelleri           10             0.8     0.09  negative
    ## 12 Staphylococcus albus                 0.007         0.1     0.001 positive
    ## 13 Staphylococcus aureus                0.03          0.03    0.001 positive
    ## 14 Streptococcus fecalis                1             1       0.1   positive
    ## 15 Streptococcus hemolyticus            0.001        14      10     positive
    ## 16 Streptococcus viridans               0.005        10      40     positive

# Visualization

<!-- -------------------------------------------------- -->

### **q1** Prototype 5 visuals

To start, construct **5 qualitatively different visualizations of the
data** `df_antibiotics`. These **cannot** be simple variations on the
same graph; for instance, if two of your visuals could be made identical
by calling `coord_flip()`, then these are *not* qualitatively different.

For all five of the visuals, you must show information on *all 16
bacteria*. For the first two visuals, you must *show all variables*.

*Hint 1*: Try working quickly on this part; come up with a bunch of
ideas, and don’t fixate on any one idea for too long. You will have a
chance to refine later in this challenge.

*Hint 2*: The data `df_antibiotics` are in a *wide* format; it may be
helpful to `pivot_longer()` the data to make certain visuals easier to
construct.

#### Visual 1 (All variables)

In this visual you must show *all three* effectiveness values for *all
16 bacteria*. This means **it must be possible to identify each of the
16 bacteria by name.** You must also show whether or not each bacterium
is Gram positive or negative.

``` r
# WRITE YOUR CODE HERE


df_antibiotics %>% 
  pivot_longer(
    cols  = c(penicillin, streptomycin, neomycin),
    names_to  = "antibiotic",
    values_to = "MIC"
  ) %>% 
  mutate(
    gram_status = if_else(gram == "positive", "Gram-positive", "Gram-negative")
  ) %>% 
  ggplot(aes(bacteria, MIC, fill = antibiotic)) +
  geom_bar(
    stat     = "identity",
    position = position_dodge(width = 1)
  ) +
  scale_y_log10(
    labels = scales::label_log(),
    breaks = scales::log_breaks()
  ) +
  geom_hline(
    yintercept = 0.1,
    linetype   = "dashed",
    colour     = "red",
  ) +
  scale_fill_manual(
    values = c(
      penicillin   = "green",
      streptomycin = "blue",
      neomycin     = "orange"
    )
  ) +
  facet_wrap(~ gram_status) +
  labs(
    title = "MIC of Three Antibiotics Against Different Bacteria",
    x     = "Bacterium",
    y     = "MIC (log scale)",
    fill  = "Antibiotic"
  ) +
  theme_minimal() +
  theme(
    axis.text.x = element_text(angle = 90, vjust = 0.5, hjust = 1)
  )
```

![](c05-antibiotics-assignment_files/figure-gfm/q1.1-1.png)<!-- -->

#### Visual 2 (All variables)

In this visual you must show *all three* effectiveness values for *all
16 bacteria*. This means **it must be possible to identify each of the
16 bacteria by name.** You must also show whether or not each bacterium
is Gram positive or negative.

Note that your visual must be *qualitatively different* from *all* of
your other visuals.

``` r
# WRITE YOUR CODE HERE

df_antibiotics_2 <- df_antibiotics %>%
  mutate(gram_status = ifelse(gram == "positive", "Gram-positive", "Gram-negative")) %>%
  pivot_longer(
    cols = c(penicillin, 
    streptomycin, 
    neomycin), 
    names_to = "antibiotic", 
    values_to = "MIC"
    )

df_antibiotics_2 %>%
  ggplot(aes(x = bacteria, y = log10(MIC), color = gram_status)) +
  geom_point(aes(shape = antibiotic), size = 3) +
  scale_color_manual(values = c("Gram-positive" = "red", "Gram-negative" = "blue")) +
  theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust = 1)) +
  labs(
  title = "MIC of Antibiotics on Different Bacteria",
  x = "Bacteria",
  y = "log 10 MIC",
  color = "Gram Status",
  shape = "Antibiotic"
)
```

![](c05-antibiotics-assignment_files/figure-gfm/q1.2-1.png)<!-- -->

#### Visual 3 (Some variables)

In this visual you may show a *subset* of the variables (`penicillin`,
`streptomycin`, `neomycin`, `gram`), but you must still show *all 16
bacteria*.

Note that your visual must be *qualitatively different* from *all* of
your other visuals.

``` r
library(tidyverse)

df_antibiotics %>% 
  pivot_longer(
    c(penicillin, streptomycin, neomycin),
    names_to  = "antibiotic",
    values_to = "MIC"
  ) %>% 
  mutate(gram_status = if_else(gram == "positive", "Gram-positive", "Gram-negative")) %>% 
  ggplot(aes(MIC, fct_rev(bacteria), fill = antibiotic)) +
  geom_col(position = position_dodge(width = 0.8)) +     
  scale_x_log10(                                          
    breaks = c(0.01, 0.1, 1, 10, 100),
    labels = c("0.01", "0.1", "1", "10", "100")
  ) +

  scale_fill_manual(values = c(
    penicillin   = "orange",
    streptomycin = "green",
    neomycin     = "blue"
  )) +
  facet_wrap(~ gram_status, scales = "free_y") +            
  labs(
    title = "MIC of Three Antibiotics Across Bacteria (log scale)",
    x     = "MIC (log scale)",
    y     = "Bacterium",
    fill  = "Antibiotic"
  ) +
  theme_minimal()
```

![](c05-antibiotics-assignment_files/figure-gfm/q1.3-1.png)<!-- -->

#### Visual 4 (Some variables)

In this visual you may show a *subset* of the variables (`penicillin`,
`streptomycin`, `neomycin`, `gram`), but you must still show *all 16
bacteria*.

Note that your visual must be *qualitatively different* from *all* of
your other visuals.

``` r
# WRITE YOUR CODE HERE


df_antibiotics %>% 
  pivot_longer(
    c(penicillin, streptomycin, neomycin),
    names_to  = "antibiotic",
    values_to = "MIC"
  ) %>% 
  mutate(log_MIC = log10(MIC)) %>% 
  ggplot(aes(antibiotic, fct_rev(bacteria), fill = log_MIC)) +
  geom_tile(colour = "white") +
  labs(
    title = "MIC of Three Antibiotics Across Bacteria (log scale)",
    x     = "Antibiotic",
    y     = "Bacterium"
  ) +
  theme_minimal()
```

![](c05-antibiotics-assignment_files/figure-gfm/q1.4-1.png)<!-- -->

#### Visual 5 (Some variables)

In this visual you may show a *subset* of the variables (`penicillin`,
`streptomycin`, `neomycin`, `gram`), but you must still show *all 16
bacteria*.

Note that your visual must be *qualitatively different* from *all* of
your other visuals.

``` r
# WRITE YOUR CODE HERE

df_antibiotics_5 <- df_antibiotics %>%
  pivot_longer(cols = c(penicillin, streptomycin, neomycin), names_to = "antibiotic", values_to = "MIC") %>%
  mutate(gram_status = ifelse(gram == "positive", "Gram-positive", "Gram-negative"))

df_antibiotics_5 %>%
  ggplot(aes(x = log10(MIC), y = bacteria, color = antibiotic)) +
  geom_point(size = 2) +
  scale_color_manual(values = c("penicillin" = "orange", "streptomycin" = "green", "neomycin" = "blue")) +
  facet_wrap(~ gram_status, scales = "free_y") +
  labs(
  title = "MIC of Antibiotics on Different Bacteria",
  x = "log 10 MIC",
  y = "Bacteria",
  color = "Antibiotic") 
```

![](c05-antibiotics-assignment_files/figure-gfm/q1.5-1.png)<!-- -->

### **q2** Assess your visuals

There are **two questions** below; use your five visuals to help answer
both Guiding Questions. Note that you must also identify which of your
five visuals were most helpful in answering the questions.

*Hint 1*: It’s possible that *none* of your visuals is effective in
answering the questions below. You may need to revise one or more of
your visuals to answer the questions below!

*Hint 2*: It’s **highly unlikely** that the same visual is the most
effective at helping answer both guiding questions. **Use this as an
opportunity to think about why this is.**

#### Guiding Question 1

> How do the three antibiotics vary in their effectiveness against
> bacteria of different genera and Gram stain?

*Observations* - What is your response to the question above? - The
effectiveness of the three antibiotics (penicillin, streptomycin, and
neomycin) varies significantly depending on the genera and Gram stain
status.

-Penicillin tends to be less effective against Gram-neative bacteria,
and especially ineffective against Aerobacter aerogenes, Klebsiella
pneumoniae, Mycobacterium tuberculosis and Pseudomonas aeruginosa with
MIC values ~800 and above.

-Penicillin is very effective against Gram-positive bacteria but doesn’t
do much against Gram-negative ones. Streptomycin and neomycin, in
contrast, work on both types: streptomycin’s strength is fairly even no
matter the bug, while neomycin can hit some hard-to-treat Gram-negative
bacteria especially well, though it may be slightly less potent than
streptomycin on a few Gram-positive strains.

- Which of your visuals above (1 through 5) is **most effective** at
  helping to answer this question?
  - Visual 2, df_antibiotics_2: Dot Plot with shapes showing the type
    antibodies
- Why?
  - df_antibiotics_3 is most effective to show which of antibiotics MIC
    is below 0.1 but is lacking the Gram stain categorization thus not
    chosen. The Dot Plot with shapes showing antibodies,
    df_antibiotics_2 is most effective out of all 5 for this analysis
    because it shows the MIC of all three antibiotics for each bacterium
    and distinguishes between Gram-positive and Gram-negative bacteria
    using color coding. The shapes allows to compare the effectiveness
    of antibiotics within and between Gram-positive and Gram-negative
    groups.

#### Guiding Question 2

In 1974 *Diplococcus pneumoniae* was renamed *Streptococcus pneumoniae*,
and in 1984 *Streptococcus fecalis* was renamed *Enterococcus fecalis*
\[2\].

> Why was *Diplococcus pneumoniae* was renamed *Streptococcus
> pneumoniae*?

*Observations* - What is your response to the question above? - From
googling, In 1886, S.pneumoniae was first known as pneumococcus due to
its close association with pneumonia. In 1920, it was renamed as
Diplococcus pneumoniae as it was usually observed in pairs. In 1974, the
name was changed to Streptococcus pneumoniae due to its similarities in
forming chains like other Streptococcus spp. S. pneumoniae can be
distinguished from other species using rMLST.

- Which of your visuals above (1 through 5) is **most effective** at
  helping to answer this question?
  - 3
- Why? from the visualisation S. pneumoniae responds to penicillin
  similarly Streptococcus bacteria, with low MIC values showing it is
  sensitive which maybe used to link it to be like Streptococcus than
  other bacteria,and maybe the reason to move it into that group.

# References

<!-- -------------------------------------------------- -->

\[1\] Neomycin in skin infections: A new topical antibiotic with wide
antibacterial range and rarely sensitizing. Scope. 1951;3(5):4-7.

\[2\] Wainer and Lysen, “That’s Funny…” *American Scientist* (2009)
[link](https://www.americanscientist.org/article/thats-funny)
