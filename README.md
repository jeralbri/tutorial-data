Fake Data Generation
================
Jeremy Albright
07 May, 2019

# Tutorials Data

The purpose of this repo is to document the creation of fake data used
in the Methods Consultants tutorials. This will provide full
replicability as well as public access to the data files for users
interested in playing around with other software. The files can be
accessed from
[here](https://github.com/jeralbri/tutorial-data/tree/master/data).

## t-tests

The t-test tutorials rely on fake IQ scores generated as follows:

``` r
set.seed(12345)
gender <- rbinom(100, size = 1, p = .5)

iq    <- tibble(id = rep(1:100, 2),
                 gender = rep(gender, 2), 
                 time = c(rep(0, 100), rep(1, 100))) %>%
  mutate(iq     = rnorm(n(), mean = 100 + 2*gender + 3*time + 3*gender*time, sd = 15)) %>%
  mutate(iq     = round(iq, 2)) %>%
  mutate(gender = factor(gender, levels = 0:1, labels = c("Male", "Female")),
         time   = factor(time,   levels = 0:1, labels = c("Time 1", "Time 2")))


head(iq)
```

<div class="kable-table">

| id | gender | time   |     iq |
| -: | :----- | :----- | -----: |
|  1 | Female | Time 1 |  93.89 |
|  2 | Female | Time 1 | 131.22 |
|  3 | Female | Time 1 | 102.80 |
|  4 | Female | Time 1 | 107.27 |
|  5 | Male   | Time 1 |  89.94 |
|  6 | Male   | Time 1 | 104.17 |

</div>

The one-sample and independent samples t-test examples assume that the
200 observations are independent. Create this file and save as an SPSS
file.

``` r
iq %>%
  mutate(id = row_number()) %>% 
  select(id, gender, iq) %>%
  expss::apply_labels(id     = "Subject ID",
                      gender = "Gender",
                      iq     = "IQ") %>%
  write_sav("data/iq_long.sav")
```

The paired samples t-test example assumes the data come from 100 paired
observations. Create this file in wide format in SPSS.

``` r
iq %>%
  select(id, time, iq) %>%
  spread(time, iq) %>%
  rename(Time_1 = `Time 1`, Time_2 = `Time 2`) %>%
  expss::apply_labels(id     = "Subject ID",
                      Time_1 = "IQ at Time 1",
                      Time_2 = "IQ at Time 2") %>%
  write_sav("data/iq_wide.sav")
```

## Cross-Sectional Mixed (Multilevel) Models

Replicating the results from Raudenbush and Bryk’s (2002) textbok
requires the High School and Beyond data. These data are available in
the `merTools` package from Jared Knowles and Carl Frederick. We provide
examples in both SPSS and Stata.

``` r
library(merTools)

write_sav(hsb, "data/hsb.sav")

write_dta(hsb, "data/hsb.dta")
```

## 2016 ANES

The 2016 American National Election Studies data are used in multiple
blog posts and tutorials. Accessing the original data requires
[registering](https://electionstudies.org/data-center/) to agree to the
terms of use. Assuming you have registered, downloaded the data in SPSS
format, and saved the file locally to the `data` directory, the
following syntax will prepare the data for the analysis we present.

``` r
anes <- read_sav("data/anes_timeseries_2016.sav") 

anes <- anes %>%
  select(pre_weight     = V160101,
         post_weight    = V160102,
         strat          = V160201,
         psu            = V160202,
         voted_2012     = V161005,
         get_news       = V161008,
         reg_vote       = V161011,
         right_track    = V161081,
         pres_approv    = V161082x,
         therm_dem      = V161086,
         therm_rep      = V161087,
         pers_finance   = V161110,
         has_hlth_care  = V161112,
         ideology_1     = V161126,
         ideology_2     = V161127,
         econ_now       = V161139,
         econ_change    = V161140x,
         vote_duty      = V161151x,
         party_id       = V161158x,
         govt_services  = V161178,
         guns           = V161187,
         immigrants     = V161192,
         build_wall     = V161196,
         help_blacks    = V161198,
         environment    = V161201,
         soc_sec_fund   = V161205,
         school_fund    = V161206,
         science_fund   = V161207,
         crime_fund     = V161208,
         childcare_fund = V161210,
         welfare_fund   = V161209,
         aid_poor_fund  = V161211,
         environ_fund   = V161212,
         trust_govt     = V161215,
         govt_corrupt   = V161218,
         trust_people   = V161219,
         warming_real   = V161221,
         warming_cause  = V161222, 
         relig_exempt   = V161227x,
         trans_bathroom = V161228x,
         protect_gays   = V161229x,
         abortion       = V161232,
         death_penalty  = V161233x,
         retro_econ     = V161235x,
         relig_import   = V161241,
         attend_church  = V161245,
         born_again     = V161263,
         age            = V161267,
         married        = V161268,
         occupation     = V161276x,
         work_now       = V161277,
         educ           = V161270, 
         union          = V161302,
         latino         = V161309,
         race           = V161310x,
         own_home       = V161334,
         gender         = V161342,
         own_stock      = V161350,
         income         = V161361x,
         numb_guns      = V161496,
         sex_orient     = V161511,
         vote           = V162034a 
         ) %>%
  mutate_at(vars(-starts_with("therm")), ~as_factor(.))
```

``` r
  filter(vote %in% c(1,2) & gender %in% c(1,2)) %>%
  mutate(vote = factor(vote, levels = 1:2, labels = c("Clinton", "Trump")),
         educ = case_when(
           V161270 %in% 1:8 ~ 1,
           V161270 %in% 9 ~ 2,
           V161270 %in% 10:12 ~ 3,
           V161270 %in% 13 ~ 4,
           V161270 %in% 14:16 ~ 5,
           TRUE ~ -999),
         gender = factor(gender, levels = 1:2, labels = c("Male", "Female"))) %>%
  mutate(educ = factor(educ, level = 1:5, labels = c("HS Not Completed",
                                                     "Completed HS",
                                                     "College < 4 Years",
                                                     "College 4 Year Degree",
                                                     "Advanced Degree"))) %>%
  filter(!is.na(educ) & age >= 18) %>%
  dplyr::select(-V161270)
```

Note that all of the questions with the exception of the actual vote
were taken from the pre survey. If modeling the vote, use the
`post_weight` to account for attrition, otherwise use `pre_weight`.
