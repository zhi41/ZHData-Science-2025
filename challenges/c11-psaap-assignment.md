Regression Case Study: PSAAP II
================
Khor Zhi Hong
2020-4-17

- [Grading Rubric](#grading-rubric)
  - [Individual](#individual)
  - [Submission](#submission)
- [Orientation: Exploring Simulation
  Results](#orientation-exploring-simulation-results)
  - [**q1** Perform your “initial checks” to get a sense of the
    data.](#q1-perform-your-initial-checks-to-get-a-sense-of-the-data)
  - [**q2** Visualize `T_norm` against `x`. Note that there are multiple
    simulations at different values of the Input variables: Each
    simulation result is identified by a different value of
    `idx`.](#q2-visualize-t_norm-against-x-note-that-there-are-multiple-simulations-at-different-values-of-the-input-variables-each-simulation-result-is-identified-by-a-different-value-of-idx)
  - [Modeling](#modeling)
    - [**q3** The following code chunk fits a few different models.
      Compute a measure of model accuracy for each model on
      `df_validate`, and compare their
      performance.](#q3-the-following-code-chunk-fits-a-few-different-models-compute-a-measure-of-model-accuracy-for-each-model-on-df_validate-and-compare-their-performance)
    - [**q4** Interpret this model](#q4-interpret-this-model)
  - [Contrasting CI and PI](#contrasting-ci-and-pi)
    - [**q5** The following code will construct a predicted-vs-actual
      plot with your model from *q4* and add prediction intervals. Study
      the results and answer the questions below under
      *observations*.](#q5-the-following-code-will-construct-a-predicted-vs-actual-plot-with-your-model-from-q4-and-add-prediction-intervals-study-the-results-and-answer-the-questions-below-under-observations)
- [Case Study: Predicting Performance
  Ranges](#case-study-predicting-performance-ranges)
  - [**q6** You are consulting with a team that is designing a prototype
    heat transfer device. They are asking you to help determine a
    *dependable range of values* for `T_norm` they can design around for
    this *single prototype*. The realized value of `T_norm` must not be
    too high as it may damage the downstream equipment, but it must also
    be high enough to extract an acceptable amount of
    heat.](#q6-you-are-consulting-with-a-team-that-is-designing-a-prototype-heat-transfer-device-they-are-asking-you-to-help-determine-a-dependable-range-of-values-for-t_norm-they-can-design-around-for-this-single-prototype-the-realized-value-of-t_norm-must-not-be-too-high-as-it-may-damage-the-downstream-equipment-but-it-must-also-be-high-enough-to-extract-an-acceptable-amount-of-heat)
- [References](#references)

*Purpose*: Confidence and prediction intervals are useful for studying
“pure sampling” of some distribution. However, we can combine CI and PI
with regression analysis to equip our modeling efforts with powerful
notions of uncertainty. In this challenge, you will use fluid simulation
data in a regression analysis with uncertainty quantification (CI and
PI) to support engineering design.

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
library(modelr)
library(broom)
```

    ## 
    ## Attaching package: 'broom'
    ## 
    ## The following object is masked from 'package:modelr':
    ## 
    ##     bootstrap

``` r
## Helper function to compute uncertainty bounds
add_uncertainties <- function(data, model, prefix = "pred", ...) {
  df_fit <-
    stats::predict(model, data, ...) %>%
    as_tibble() %>%
    rename_with(~ str_c(prefix, "_", .))

  bind_cols(data, df_fit)
}
```

# Orientation: Exploring Simulation Results

*Background*: The data you will study in this exercise come from a
computational fluid dynamics (CFD) [simulation
campaign](https://www.sciencedirect.com/science/article/abs/pii/S0301932219308651?via%3Dihub)
that studied the interaction of turbulent flow and radiative heat
transfer to fluid-suspended particles\[1\]. These simulations were
carried out to help study a novel design of [solar
receiver](https://en.wikipedia.org/wiki/Concentrated_solar_power),
though they are more aimed at fundamental physics than detailed device
design. The following code chunk downloads and unpacks the data to your
local `./data/` folder.

``` r
## NOTE: No need to edit this chunk
## Download PSAAP II data and unzip
url_zip <- "https://ndownloader.figshare.com/files/24111269"
filename_zip <- "./data/psaap.zip"
filename_psaap <- "./data/psaap.csv"

curl::curl_download(url_zip, destfile = filename_zip)
unzip(filename_zip, exdir = "./data")
df_psaap <- read_csv(filename_psaap)
```

    ## Rows: 140 Columns: 22
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (22): x, idx, L, W, U_0, N_p, k_f, T_f, rho_f, mu_f, lam_f, C_fp, rho_p,...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

![PSAAP II irradiated core flow](./images/psaap-setup.png) Figure 1. An
example simulation, frozen at a specific point in time. An initial
simulation is run (HIT SECTION) to generate a turbulent flow with
particles, and that swirling flow is released into a rectangular domain
(RADIATED SECTION) with bulk downstream flow (left to right).
Concentrated solar radiation transmits through the optically transparent
fluid, but deposits heat into the particles. The particles then convect
heat into the fluid, which heats up the flow. The false-color image
shows the fluid temperature: Notice that there are “hot spots” where hot
particles have deposited heat into the fluid. The dataset `df_psaap`
gives measurements of `T_norm = (T - T0) / T0` averaged across planes at
various locations along the RADIATED SECTION.

### **q1** Perform your “initial checks” to get a sense of the data.

``` r
## TODO: Perform your initial checks

dim(df_psaap)   #rows vs column 
```

    ## [1] 140  22

``` r
names(df_psaap) # list of variables
```

    ##  [1] "x"      "idx"    "L"      "W"      "U_0"    "N_p"    "k_f"    "T_f"   
    ##  [9] "rho_f"  "mu_f"   "lam_f"  "C_fp"   "rho_p"  "d_p"    "C_pv"   "h"     
    ## [17] "I_0"    "eps_p"  "avg_q"  "avg_T"  "rms_T"  "T_norm"

``` r
summary(df_psaap) 
```

    ##        x               idx           L                W          
    ##  Min.   :0.2500   Min.   : 1   Min.   :0.1292   Min.   :0.03198  
    ##  1st Qu.:0.4375   1st Qu.: 9   1st Qu.:0.1448   1st Qu.:0.03539  
    ##  Median :0.6250   Median :18   Median :0.1623   Median :0.03983  
    ##  Mean   :0.6250   Mean   :18   Mean   :0.1631   Mean   :0.04022  
    ##  3rd Qu.:0.8125   3rd Qu.:27   3rd Qu.:0.1819   3rd Qu.:0.04482  
    ##  Max.   :1.0000   Max.   :35   Max.   :0.2009   Max.   :0.04960  
    ##       U_0             N_p               k_f               T_f       
    ##  Min.   :1.667   Min.   :1527347   Min.   :0.07954   Min.   :241.9  
    ##  1st Qu.:1.846   1st Qu.:1707729   1st Qu.:0.08674   1st Qu.:262.3  
    ##  Median :2.075   Median :1909414   Median :0.09822   Median :291.4  
    ##  Mean   :2.094   Mean   :1929614   Mean   :0.09964   Mean   :298.3  
    ##  3rd Qu.:2.340   3rd Qu.:2154872   3rd Qu.:0.11123   3rd Qu.:331.7  
    ##  Max.   :2.583   Max.   :2387055   Max.   :0.12360   Max.   :370.5  
    ##      rho_f             mu_f               lam_f              C_fp       
    ##  Min.   :0.9637   Min.   :1.519e-05   Min.   :0.02393   Min.   : 813.2  
    ##  1st Qu.:1.0728   1st Qu.:1.672e-05   1st Qu.:0.02642   1st Qu.: 922.2  
    ##  Median :1.1943   Median :1.893e-05   Median :0.02976   Median :1013.4  
    ##  Mean   :1.2059   Mean   :1.902e-05   Mean   :0.03033   Mean   :1025.0  
    ##  3rd Qu.:1.3358   3rd Qu.:2.126e-05   3rd Qu.:0.03352   3rd Qu.:1131.3  
    ##  Max.   :1.4871   Max.   :2.340e-05   Max.   :0.03762   Max.   :1262.9  
    ##      rho_p            d_p                 C_pv             h       
    ##  Min.   : 7159   Min.   :8.497e-06   Min.   :362.2   Min.   :4569  
    ##  1st Qu.: 8053   1st Qu.:9.493e-06   1st Qu.:413.9   1st Qu.:5134  
    ##  Median : 9058   Median :1.061e-05   Median :462.5   Median :5830  
    ##  Mean   : 9144   Mean   :1.068e-05   Mean   :464.8   Mean   :5820  
    ##  3rd Qu.:10339   3rd Qu.:1.185e-05   3rd Qu.:516.9   3rd Qu.:6414  
    ##  Max.   :11128   Max.   :1.308e-05   Max.   :565.0   Max.   :7056  
    ##       I_0              eps_p            avg_q             avg_T      
    ##  Min.   :5664363   Min.   :0.3193   Min.   : 335025   Min.   :291.4  
    ##  1st Qu.:6363488   1st Qu.:0.3540   1st Qu.: 619232   1st Qu.:423.0  
    ##  Median :6943899   Median :0.3958   Median : 689560   Median :491.3  
    ##  Mean   :7095833   Mean   :0.4018   Mean   : 777490   Mean   :513.0  
    ##  3rd Qu.:7953745   3rd Qu.:0.4427   3rd Qu.: 978892   3rd Qu.:582.3  
    ##  Max.   :8849196   Max.   :0.4950   Max.   :1498542   Max.   :938.2  
    ##      rms_T           T_norm      
    ##  Min.   :3.387   Min.   :0.1215  
    ##  1st Qu.:4.937   1st Qu.:0.3889  
    ##  Median :5.698   Median :0.6328  
    ##  Mean   :5.961   Mean   :0.7360  
    ##  3rd Qu.:6.948   3rd Qu.:0.9795  
    ##  Max.   :9.254   Max.   :2.2840

``` r
colSums(is.na(df_psaap))  
```

    ##      x    idx      L      W    U_0    N_p    k_f    T_f  rho_f   mu_f  lam_f 
    ##      0      0      0      0      0      0      0      0      0      0      0 
    ##   C_fp  rho_p    d_p   C_pv      h    I_0  eps_p  avg_q  avg_T  rms_T T_norm 
    ##      0      0      0      0      0      0      0      0      0      0      0

``` r
# unique number of those things
df_psaap %>% summarise(
  Simulation_run  = n_distinct(idx),
  Channel_location  = n_distinct(x)
)
```

    ## # A tibble: 1 × 2
    ##   Simulation_run Channel_location
    ##            <int>            <int>
    ## 1             35                4

``` r
#range
df_psaap %>% summarise(across(
  c(avg_q, avg_T, rms_T, T_norm),
  list(min = ~min(.x, na.rm=TRUE),
       max = ~max(.x, na.rm=TRUE))
))
```

    ## # A tibble: 1 × 8
    ##   avg_q_min avg_q_max avg_T_min avg_T_max rms_T_min rms_T_max T_norm_min
    ##       <dbl>     <dbl>     <dbl>     <dbl>     <dbl>     <dbl>      <dbl>
    ## 1   335025.  1498542.      291.      938.      3.39      9.25      0.122
    ## # ℹ 1 more variable: T_norm_max <dbl>

**Observations**:

- (Take notes here!)
- there are 22 columns and 140 rows
- there are 4 different kinds of channel run, 35 different kinds of
  simulation run
- there are no Na or missing data in the excel sheet
- summaries as indicated above
- all inputs are numerical
- the range of outputs are as show in the console which maybe useful

The important variables in this dataset are:

| Variable | Category | Meaning                           |
|----------|----------|-----------------------------------|
| `x`      | Spatial  | Channel location                  |
| `idx`    | Metadata | Simulation run                    |
| `L`      | Input    | Channel length                    |
| `W`      | Input    | Channel width                     |
| `U_0`    | Input    | Bulk velocity                     |
| `N_p`    | Input    | Number of particles               |
| `k_f`    | Input    | Turbulence level                  |
| `T_f`    | Input    | Fluid inlet temp                  |
| `rho_f`  | Input    | Fluid density                     |
| `mu_f`   | Input    | Fluid viscosity                   |
| `lam_f`  | Input    | Fluid conductivity                |
| `C_fp`   | Input    | Fluid isobaric heat capacity      |
| `rho_p`  | Input    | Particle density                  |
| `d_p`    | Input    | Particle diameter                 |
| `C_pv`   | Input    | Particle isochoric heat capacity  |
| `h`      | Input    | Convection coefficient            |
| `I_0`    | Input    | Radiation intensity               |
| `eps_p`  | Input    | Radiation absorption coefficient  |
| `avg_q`  | Output   | Plane-averaged heat flux          |
| `avg_T`  | Output   | Plane-averaged fluid temperature  |
| `rms_T`  | Output   | Plane-rms fluid temperature       |
| `T_norm` | Output   | Normalized fluid temperature rise |

The primary output of interest is `T_norm = (avg_T - T_f) / T_f`, the
normalized (dimensionless) temperature rise of the fluid, due to heat
transfer. These measurements are taken at locations `x` along a column
of fluid, for different experimental settings (e.g. different dimensions
`W, L`, different flow speeds `U_0`, etc.).

### **q2** Visualize `T_norm` against `x`. Note that there are multiple simulations at different values of the Input variables: Each simulation result is identified by a different value of `idx`.

``` r
## TODO: Visualize the data in df_psaap with T_norm against x;
##       design your visual to handle the multiple simulations,
##       each identified by different values of idx



ggplot(
  df_psaap, aes(x = x, y = T_norm, group = factor(idx), color = factor(idx))) +
  geom_line(alpha = 0.5, size = 0.8) +
  labs(
    x = "Channel location (x)",
    y = expression(T[norm]~"(normalized temperature rise)"),
    color = "Simulation\nidx"
  ) +
  theme_minimal() 
```

    ## Warning: Using `size` aesthetic for lines was deprecated in ggplot2 3.4.0.
    ## ℹ Please use `linewidth` instead.
    ## This warning is displayed once every 8 hours.
    ## Call `lifecycle::last_lifecycle_warnings()` to see where this warning was
    ## generated.

![](c11-psaap-assignment_files/figure-gfm/q2-task-1.png)<!-- -->

## Modeling

The following chunk will split the data into training and validation
sets.

``` r
## NOTE: No need to edit this chunk
# Addl' Note: These data are already randomized by idx; no need
# to additionally shuffle the data!
df_train <- df_psaap %>% filter(idx %in% 1:20)
df_validate <- df_psaap %>% filter(idx %in% 21:36)
```

One of the key decisions we must make in modeling is choosing predictors
(features) from our observations to include in the model. Ideally we
should have some intuition for why these predictors are reasonable to
include in the model; for instance, we saw above that location along the
flow `x` tends to affect the temperature rise `T_norm`. This is because
fluid downstream has been exposed to solar radiation for longer, and
thus is likely to be at a higher temperature.

Reasoning about our variables—at least at a *high level*—can help us to
avoid including *fallacious* predictors in our models. You’ll explore
this idea in the next task.

### **q3** The following code chunk fits a few different models. Compute a measure of model accuracy for each model on `df_validate`, and compare their performance.

``` r
## NOTE: No need to edit these models
fit_baseline <- 
  df_train %>% 
  lm(formula = T_norm ~ x)

fit_cheat <- 
  df_train %>% 
  lm(formula = T_norm ~ avg_T)

fit_nonphysical <- 
  df_train %>% 
  lm(formula = T_norm ~ idx)

## TODO: Compute a measure of accuracy for each fit above;

models <- list(
  baseline = fit_baseline,
  cheat = fit_cheat,
  nonphysical = fit_nonphysical
)

model_metric <- tibble(
  model = names(models),
  mse = map_dbl(models, ~ mse(.x, df_validate)),
  rsq = map_dbl(models, ~ rsquare(.x, df_validate))
)

model_metric
```

    ## # A tibble: 3 × 3
    ##   model          mse     rsq
    ##   <chr>        <dbl>   <dbl>
    ## 1 baseline    0.0809 0.475  
    ## 2 cheat       0.0537 0.637  
    ## 3 nonphysical 0.159  0.00190

**Observations**:

- Which model is *most accurate*? Which is *least accurate*?
  - Most accurate is the cheat model has the lowest MSE 0.0537 and
    highest rsq 0.6374. -Least accurate is the nonphysical model has the
    highest MSE 0.1591 and lowest rsq 0.0019.
- What *Category* of variable is `avg_T`? Why is it such an effective
  predictor?
  - It output plane‑averaged fluid temperature where T_norm = (avg_T -
    T_f) / T_f\`, the normalized (dimensionless) temperature rise of the
    fluid, due to heat transfer
- Would we have access to `avg_T` if we were trying to predict a *new*
  value of `T_norm`? Is `avg_T` a valid predictor?
  - No is the very temperature measurement you’d need use before
    computing T_norm
- What *Category* of variable is `idx`? Does it have any physical
  meaning?
  - It is a run simulation label, merely labels different runs

### **q4** Interpret this model

Interpret the following model by answering the questions below.

*Note*. The `-` syntax in R formulas allows us to exclude columns from
fitting. So `T_norm ~ . - x` would fit on all columns *except* `x`.

``` r
## TODO: Inspect the regression coefficients for the following model
fit_q4 <- 
  df_train %>% 
  lm(formula = T_norm ~ . - idx - avg_q - avg_T - rms_T)
  # lm(formula = T_norm ~ L + W + U_0 + N_p + k_f + T_f)
  # lm(formula = T_norm ~ L - W - U_0 - N_p - k_f - T_f)


tidy(fit_q4)
```

    ## # A tibble: 18 × 5
    ##    term        estimate std.error statistic  p.value
    ##    <chr>          <dbl>     <dbl>     <dbl>    <dbl>
    ##  1 (Intercept) -3.03e-3   1.66e+0  -0.00183 9.99e- 1
    ##  2 x            1.02e+0   5.61e-2  18.2     1.66e-26
    ##  3 L            3.96e+0   9.36e-1   4.23    7.81e- 5
    ##  4 W           -3.71e+1   6.39e+0  -5.81    2.33e- 7
    ##  5 U_0         -3.26e-1   1.14e-1  -2.87    5.63e- 3
    ##  6 N_p          2.72e-7   1.38e-7   1.97    5.33e- 2
    ##  7 k_f          2.55e+0   2.13e+0   1.20    2.36e- 1
    ##  8 T_f         -3.79e-4   1.17e-3  -0.323   7.48e- 1
    ##  9 rho_f       -5.62e-1   3.21e-1  -1.75    8.51e- 2
    ## 10 mu_f        -8.25e+3   1.47e+4  -0.562   5.76e- 1
    ## 11 lam_f       -4.68e+0   1.11e+1  -0.420   6.76e- 1
    ## 12 C_fp        -6.59e-4   3.20e-4  -2.06    4.39e- 2
    ## 13 rho_p        5.64e-6   1.84e-5   0.307   7.60e- 1
    ## 14 d_p          1.24e+5   4.49e+4   2.75    7.76e- 3
    ## 15 C_pv        -7.24e-4   3.72e-4  -1.94    5.64e- 2
    ## 16 h            1.41e-6   4.87e-5   0.0289  9.77e- 1
    ## 17 I_0          1.60e-7   4.29e-8   3.73    4.13e- 4
    ## 18 eps_p        1.11e+0   1.10e+0   1.01    3.18e- 1

``` r
glance(fit_q4)
```

    ## # A tibble: 1 × 12
    ##   r.squared adj.r.squared sigma statistic  p.value    df logLik   AIC   BIC
    ##       <dbl>         <dbl> <dbl>     <dbl>    <dbl> <dbl>  <dbl> <dbl> <dbl>
    ## 1     0.935         0.917 0.140      52.0 3.46e-30    17   53.9 -69.7 -24.5
    ## # ℹ 3 more variables: deviance <dbl>, df.residual <int>, nobs <int>

``` r
df_psaap %>% summarise(
  sd_x  = sd(x),
  sd_Tf = sd(T_f)
)
```

    ## # A tibble: 1 × 2
    ##    sd_x sd_Tf
    ##   <dbl> <dbl>
    ## 1 0.281  38.9

**Observations**:

- Which columns are excluded in the model formula above? What categories
  do these belong to? Why are these important quantities to leave out of
  the model?
  - for first model - idx - avg_q - avg_T - rms_T, label data and output
    variable, label have no physical data, the output variable would
    reuse the data you trying to predict, in sense “cheat” -for second
    model x is excluded label have no physical data -for third model
    drop everything except L and intercept, used to compare R^2 and mse
    of other models as it check for variability of T-norm based on
    channel length.
- Which inputs are *statistically significant*, according to the model?
  - data with p\<0.05, such as W,T_f and K_f which is just close to 0.05
- What is the regression coefficient for `x`? What about the regression
  coefficient for `T_f`?
  - nil x have no coefficient in this model. T_f r coefficient
    -4.783458e-03
- What is the standard deviation of `x` in `df_psaap`? What about the
  standard deviation of `T_f`?
  - standard deviation of `x` is 0.2805121, standard deviation of `T_f`
    is 38.94204
- How do these standard deviations relate to the regression coefficients
  for `x` and `T_f`?
  - The standard deviation of an input tells you how much x usually
    moves around.The regression coefficient tells you how much T_norm
    changes when x moves by one unit. If you multiply them to get the
    typical change in T_norm when x moves by its usual amount. same for
    T_f but in terms of temperature, degree change and the average
    change.
- Note that literally *all* of the inputs above have *some* effect on
  the output `T_norm`; so they are all “significant” in that sense. What
  does this tell us about the limitations of statistical significance
  for interpreting regression coefficients?
  - Statistical significance is not equal practical importance. With a
    sufficiently large sample, even small effects become significant,
    low p. This mean need to use more methods before drawing conclusions
    such as using confidence intervals.

## Contrasting CI and PI

Let’s revisit the ideas of confidence intervals (CI) and prediction
intervals (PI). Let’s fit a very simple model to these data, one which
only considers the channel location and ignores all other inputs. We’ll
also use the helper function `add_uncertainties()` (defined in the
`setup` chunk above) to add approximate CI and PI to the linear model.

``` r
## NOTE: No need to edit this chunk
fit_simple <-
  df_train %>%
  lm(data = ., formula = T_norm ~ x)

df_intervals <-
  df_train %>%
  add_uncertainties(fit_simple, interval = "confidence", prefix = "ci") %>%
  add_uncertainties(fit_simple, interval = "prediction", prefix = "pi")
```

The following figure visualizes the regression CI and PI against the
objects they are attempting to capture:

``` r
## NOTE: No need to edit this chunk
df_intervals %>%
  select(T_norm, x, matches("ci|pi")) %>%
  pivot_longer(
    names_to = c("method", ".value"),
    names_sep = "_",
    cols = matches("ci|pi")
  ) %>%

  ggplot(aes(x, fit)) +
  geom_errorbar(
    aes(ymin = lwr, ymax = upr, color = method),
    width = 0.05,
    size = 1
  ) +
  geom_smooth(
    data = df_psaap %>% mutate(method = "ci"),
    mapping = aes(x, T_norm),
    se = FALSE,
    linetype = 2,
    color = "black"
   ) +
  geom_point(
    data = df_validate %>% mutate(method = "pi"),
    mapping = aes(x, T_norm),
    size = 0.5
  ) +

  facet_grid(~method) +
  theme_minimal() +
  labs(
    x = "Channel Location (-)",
    y = "Normalized Temperature Rise (-)"
  )
```

    ## `geom_smooth()` using method = 'loess' and formula = 'y ~ x'

    ## Warning in simpleLoess(y, x, w, span, degree = degree, parametric = parametric,
    ## : pseudoinverse used at 0.24625

    ## Warning in simpleLoess(y, x, w, span, degree = degree, parametric = parametric,
    ## : neighborhood radius 0.50375

    ## Warning in simpleLoess(y, x, w, span, degree = degree, parametric = parametric,
    ## : reciprocal condition number 7.4302e-16

    ## Warning in simpleLoess(y, x, w, span, degree = degree, parametric = parametric,
    ## : There are other near singularities as well. 0.25376

![](c11-psaap-assignment_files/figure-gfm/data-simple-model-vis-1.png)<!-- -->

Under the `ci` facet we have the regression confidence intervals and the
mean trend (computed with all the data `df_psaap`). Under the `pi` facet
we have the regression prediction intervals and the `df_validation`
observations.

**Punchline**:

- Confidence intervals are meant to capture the *mean trend*
- Prediction intervals are meant to capture *new observations*

Both CI and PI are a quantification of the uncertainty in our model, but
the two intervals designed to answer different questions.

Since CI and PI are a quantification of uncertainty, they should tend to
*narrow* as our model becomes more confident in its predictions.
Building a more accurate model will often lead to a reduction in
uncertainty. We’ll see this phenomenon in action with the following
task:

### **q5** The following code will construct a predicted-vs-actual plot with your model from *q4* and add prediction intervals. Study the results and answer the questions below under *observations*.

``` r
## TODO: Run this code and interpret the results
## NOTE: No need to edit this chunk
## NOTE: This chunk will use your model from q4; it will predict on the
##       validation data, add prediction intervals for every prediction,
##       and visualize the results on a predicted-vs-actual plot. It will
##       also compare against the simple `fit_simple` defined above.
bind_rows(
  df_psaap %>% 
    add_uncertainties(fit_simple, interval = "prediction", prefix = "pi") %>% 
    select(T_norm, pi_lwr, pi_fit, pi_upr) %>% 
    mutate(model = "x only"),
  df_psaap %>% 
    add_uncertainties(fit_q4, interval = "prediction", prefix = "pi") %>% 
    select(T_norm, pi_lwr, pi_fit, pi_upr) %>% 
    mutate(model = "q4"),
) %>% 
  
  ggplot(aes(T_norm, pi_fit)) +
  geom_abline(slope = 1, intercept = 0, color = "grey80", size = 2) +
  geom_errorbar(
    aes(ymin = pi_lwr, ymax = pi_upr),
    width = 0
  ) +
  geom_point() +
  
  facet_grid(~ model, labeller = label_both) +
  theme_minimal() +
  labs(
    title = "Predicted vs Actual",
    x = "Actual T_norm",
    y = "Predicted T_norm"
  )
```

![](c11-psaap-assignment_files/figure-gfm/q5-task-1.png)<!-- -->

**Observations**:

- Which model tends to be more accurate? How can you tell from this
  predicted-vs-actual plot?
  - the q4 model. Its points hug the diagonal line much more tightly
    than in the x only and its predictions are closer to the actual
    T_norm values.
- Which model tends to be *more confident* in its predictions? Put
  differently, which model has *narrower prediction intervals*?
  - The q4 model.Its vertical bars are shorter on average than those of
    the “x only” model—meaning less uncertainty around each prediction.
- How many predictors does the `fit_simple` model need in order to make
  a prediction? What about your model `fit_q4`? -fit_simple only used x
  while fit_q4 used L + W + U_0 + N_p + k_f + T_f

Based on these results, you might be tempted to always throw every
reasonable variable into the model. For some cases, that might be the
best choice. However, some variables might be *outside our control*; for
example, variables involving human behavior cannot be fully under our
control. Other variables may be *too difficult to measure*; for example,
it is *in theory* possible to predict the strength of a component by
having detailed knowledge of its microstructure. However, it is
*patently infeasible* to do a detailed study of *every single component*
that gets used in an airplane.

In both cases—human behavior and variable material properties—we would
be better off treating those quantities as random variables. There are
at least two ways we could treat these factors: 1. Explicitly model some
inputs as random variables and construct a model that *propagates* that
uncertainty from inputs to outputs, or 2. Implicitly model the
uncontrolled the uncontrolled variables by not including them as
predictors in the model, and instead relying on the error term
$\epsilon$ to represent these unaccounted factors. You will pursue
strategy 2. in the following Case Study.

# Case Study: Predicting Performance Ranges

### **q6** You are consulting with a team that is designing a prototype heat transfer device. They are asking you to help determine a *dependable range of values* for `T_norm` they can design around for this *single prototype*. The realized value of `T_norm` must not be too high as it may damage the downstream equipment, but it must also be high enough to extract an acceptable amount of heat.

In order to maximize the conditions under which this device can operate
successfully, the design team has chosen to fix the variables listed in
the table below, and consider the other variables to fluctuate according
to the values observed in `df_psaap`.

| Variable | Value    |
|----------|----------|
| `x`      | 1.0      |
| `L`      | 0.2      |
| `W`      | 0.04     |
| `U_0`    | 1.0      |
| (Other)  | (Varies) |

Your task is to use a regression analysis to deliver to the design team
a *dependable range* of values for `T_norm`, given their proposed
design, and at a fairly high level `0.8`. Perform your analysis below
(use the helper function `add_uncertainties()` with the `level`
argument!), and answer the questions below.

*Hint*: This problem will require you to *build a model* by choosing the
appropriate variables to include in the analysis. Think about *which
variables the design team can control*, and *which variables they have
chosen to allow to vary*. You will also need to choose between computing
a CI or PI for the design prediction.

``` r
# NOTE: No need to change df_design; this is the target the client
#       is considering
df_design <- tibble(x = 1, L = 0.2, W = 0.04, U_0 = 1.0)
# NOTE: This is the level the "probability" level customer wants
pr_level <- 0.8

## TODO: Fit a model, assess the uncertainty in your prediction, 
#        use the validation data to check your uncertainty estimates, and 
#        make a recommendation on a *dependable range* of values for T_norm
#        at the point `df_design`
fit_q6 <- lm(T_norm ~ x + L + W + U_0, data = df_train)

df_design <- df_design %>%
  add_uncertainties(
    fit_q6,
    interval = "prediction",
    level = pr_level,
    prefix = "pi"
  )

df_design
```

    ## # A tibble: 1 × 7
    ##       x     L     W   U_0 pi_fit pi_lwr pi_upr
    ##   <dbl> <dbl> <dbl> <dbl>  <dbl>  <dbl>  <dbl>
    ## 1     1   0.2  0.04     1   1.88   1.46   2.30

``` r
summary(fit_q6)$r.squared
```

    ## [1] 0.6626266

``` r
summary(fit_q6)$sigma
```

    ## [1] 0.2892852

``` r
df_validate_lah <- df_validate %>%
  add_uncertainties(
    fit_q6,
    interval = "prediction",
    level = pr_level,
    prefix = "pi"
  )

df_validate_lah %>%
  summarize(
    coverage = mean(pi_lwr <= T_norm & T_norm <= pi_upr)
  )
```

    ## # A tibble: 1 × 1
    ##   coverage
    ##      <dbl>
    ## 1    0.933

**Recommendation**:

- How much do you trust your model? Why?
  - my fitted model has an R^2 of about 0.6626266 and a standard
    deviation 0.2892852. That means it explains roughly 66.3% of the
    variation in T_norm, and typical prediction errors are on the order
    of +-0.2892 in normalized‑temperature units.Good enoug.
- What kind of interval—confidence or prediction—would you use for this
  task, and why?
  - since sizing a single prototype’s performance, i want a prediction
    interval and confidence interval would tell us about the mean T_norm
    for many replicas under these same conditions.
- What fraction of validation cases lie within the intervals you
  predict? (NB. Make sure to calculate your intervals *based on the
  validation data*; don’t just use one single interval!) How does this
  compare with `pr_level`?
  - it is wider of 0.933 than required 0.8?
- What interval for `T_norm` would you recommend the design team to plan
  around?
  - between 1.45685 and 2.296426
- Are there any other recommendations you would provide?
  - Collect more data at exactly the design conditions
  - Control or monitor high significant variables more closely

*Bonus*: One way you could take this analysis further is to recommend
which other variables the design team should tightly control. You could
do this by fixing values in `df_design` and adding them to the model. An
exercise you could carry out would be to systematically test the
variables to see which ones the design team should more tightly control.

# References

- \[1\] Jofre, del Rosario, and Iaccarino “Data-driven dimensional
  analysis of heat transfer in irradiated particle-laden turbulent
  flow” (2020) *International Journal of Multiphase Flow*,
  <https://doi.org/10.1016/j.ijmultiphaseflow.2019.103198>
