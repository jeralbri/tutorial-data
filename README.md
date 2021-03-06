Example Data Generation
================
Jeremy Albright
12 January, 2020

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

The one-sample and independent samples t-test examples assume that the
200 observations are independent. Create this file and save as an SPSS,
SAS, and Stata file.

``` r
iq_long <- iq %>%
  mutate(id = row_number()) %>% 
  select(id, gender, iq) %>%
  expss::apply_labels(id     = "Subject ID",
                      gender = "Gender",
                      iq     = "IQ") 

iq_long %>%
  write_sav("data/iq_long.sav")

iq_long %>% 
  write_sas("data/iq_long.sas7bdat")

iq_long %>% 
  write_dta("data/iq_long.dta")
```

The paired samples t-test example assumes the data come from 100 paired
observations. Create this file in wide format in SPSS.

``` r
iq_wide <- iq %>%
  select(id, time, iq) %>%
  spread(time, iq) %>%
  rename(Time_1 = `Time 1`, Time_2 = `Time 2`) %>%
  expss::apply_labels(id     = "Subject ID",
                      Time_1 = "IQ at Time 1",
                      Time_2 = "IQ at Time 2") 

iq_wide %>%
  write_sav("data/iq_wide.sav")

iq_wide %>% 
  write_sas("data/iq_wide.sas7bdat")

iq_wide %>% 
  write_dta("data/iq_wide.dta")
```

## One-Way ANOVA

The ANOVA data are fake data generated years ago for a university
course. The code (and random seed) are long gone, but the file remains.
It’s saved in this repo as an SPSS file, `anova.sav`. To transform to a
SAS and Stata file:

``` r
anova <- read_spss("data/anova.sav")

anova %>% 
  write_sas("data/anova.sas7bdat")

anova %>% 
  write_dta("data/anova.dta")
```

## Two-Way ANOVA

The data used for the two-way factorial ANOVA are available in the
`carData` package. Create SPSS, SAS, and Stata versions:

``` r
library(carData)

Moore <- Moore %>% 
  mutate(partner.status = fct_relevel(partner.status, c("low",
                                                        "high")),
         fcategory      = fct_relevel(fcategory, c("low",
                                                   "medium",
                                                   "high")))

Moore %>% 
  write_sav("data/moore.sav")

Moore  %>% 
  dplyr::rename(partner_status = partner.status) %>% 
  write_sas("data/moore.sas7bdat")

Moore %>% 
  dplyr::rename(partner_status = partner.status) %>% 
  write_dta("data/moore.dta")
```

## Cross-Sectional Mixed (Multilevel) Models

Replicating the results from Raudenbush and Bryk’s (2002) textbok
requires the High School and Beyond data. These data are available in
the `merTools` package from Jared Knowles and Carl Frederick. We provide
examples in both SPSS and Stata.

``` r
library(merTools)

write_sav(hsb, "data/hsb.sav")

write_sas(hsb, "data/hsb.sas7bdat")

write_dta(hsb, "data/hsb.dta")
```

## 2016 ANES

The 2016 American National Election Studies data are used in multiple
blog posts and tutorials. Accessing the original data requires
[registering](https://electionstudies.org/data-center/) to agree to the
terms of use (no fee, no annoying emails). Assuming you have registered,
the following syntax will prepare the data for the analysis we present.

``` r
anes <- read_delim("data/anes_timeseries_2016_rawdata.txt",
                   delim = "|", guess_max = 5000) 

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
  mutate_if(is.character, as.numeric) %>% 
  mutate(income = if_else(income < 1, NA_real_, income),
         age    = if_else(age    < 1, NA_real_, age)) %>% 
  filter(vote %in% c(1,2) & gender %in% c(1,2)) %>% # only 11 non-binary, too small for inference
  mutate(vote = factor(vote, levels = 1:2, labels = c("Clinton", "Trump")),
         educ = case_when(
           educ %in% 1:8 ~ 1,
           educ %in% 9 ~ 2,
           educ %in% 10:12 ~ 3,
           educ %in% 13 ~ 4,
           educ %in% 14:16 ~ 5,
           TRUE ~ -999),
         gender = factor(gender, levels = 1:2, labels = c("Male", "Female"))) %>%
  mutate(educ = factor(educ, level = 1:5, labels = c("HS Not Completed",
                                                     "Completed HS",
                                                     "College < 4 Years",
                                                     "College 4 Year Degree",
                                                     "Advanced Degree"))) %>% 
  mutate(income = factor(income, levels = 1:28, labels = c("Under $5,000",
                                                           "$5,000-$9,999",  
                                                           "$10,000-$12,499",
                                                           "$12,500-$14,999",
                                                           "$15,000-$17,499",
                                                           "$17,500-$19,999",
                                                           "$20,000-$22,499",
                                                           "$22,500-$24,999",
                                                           "$25,000-$27,499",
                                                           "$27,500-$29,999",
                                                           "$30,000-$34,999",
                                                           "$35,000-$39,999",
                                                           "$40,000-$44,999",
                                                           "$45,000-$49,999",
                                                           "$50,000-$54,999",
                                                           "$55,000-$59,999",
                                                           "$60,000-$64,999",
                                                           "$65,000-$69,999",
                                                           "$70,000-$74,999",
                                                           "$75,000-$79,999",
                                                           "$80,000-$89,999",
                                                           "$90,000-$99,999",
                                                           "$100,000-$109,999",
                                                           "$110,000-$124,999",
                                                           "$125,000-$149,999",
                                                           "$150,000-$174,999",
                                                           "$175,000-$249,999",
                                                           "$250,000 or more"))) %>% 
  mutate(educ_cont = as.numeric(educ))
```

Save the cleaned data file.

``` r
write_sav(anes, "data/cleaned-anes.sav")

write_dta(anes, "data/cleaned-anes.dta")

write_sas(anes, "data/cleaned-anes.sas7bdat")
```

## Latent Variable Models

The CFA and SEM examples in the tutorials use the illustrations from
Bollen’s (1989) classic textbook, *Structural Equations with Latent
Variables*, which, after thirty years, remains among the best resources
for understanding SEM estimators. The data used for his examples are
available in the `sem` package. We’ll save the data in SPSS format as
well.

``` r
library(sem)
data("Bollen")

write_sav(Bollen, "data/bollen_sem.sav")
```

## GSS Data

The GSS data have been helpfully organized into an R package by [Kieran
Healy](https://kjhealy.github.io/gssr/). It can be installed directly
from Github as follows:

``` r
devtools::install_github("kjhealy/gssr")
```

Load data.

``` r
library(gssr)
data(gss_all)
```

For the analysis of sexual behavior by party identification and
religion, filter to `year == 2018` and save the following variables:

  - vstrat: Variance Stratum
  - vpsu: Variance PSU
  - wtssall: Weight
  - partnrs5: Number of sex partners last five years
  - partyid: party identification (0 = strong democrdat, 6 = strong
    republican, 7 = other party)
  - feelrel: how religious are you? (1 = extremely religious, 7 =
    extremely non-religious)
  - sex: sex (1 = male, 2 = female)

Save as `.rds` file.

``` r
gss_all %>% 
  filter(year == 2018) %>% 
  select(vstrat, vpsu, wtssall, partnrs5, partyid, feelrel, sex) %>% 
  saveRDS("data/gss_sex.rds")
```

## Regression

The data for the regression examples come from the replication materials
for DiGrazia, J. McKelvey, K. Bollen, J. Rojas, F. More Tweets, More
Votes: Social Media as a Quantitative Indicator of Political Behavior.
PLoS ONE. November 2013. The original files are available
[here](http://dx.doi.org/10.7910/DVN/23103) and were accessed January
12, 2020.

The data were collected to assess whether Twitter mentions of candidates
in the 2010 and 2012 US Congressional elections were associated with the
candidates’ electoral performance. The data will be used to demonstrate
how to perform regression analysis in common statistical software. The
results in the tutorials are not to be taken as a commentary on the
article, nor as having been endorsed by the authors of the original
study.

The data come in csv format. The first column is the row number and can
be dropped.

``` r
twitter_tbl <- read_csv("data/mtmv_data_10_12.csv") %>% 
  select(-X1)
```

    ## Warning: Missing column names filled in: 'X1' [1]

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_double()
    ## )

    ## See spec(...) for full column specifications.

The original article included a control for district partisanship using
the district’s vote share for John McCain in the 2008 presidential
election. To demonstrate the use of dummy variables, this variable will
be converted into tertiles.

``` r
twitter_tbl <- twitter_tbl %>% 
  mutate(mccain_tert = gtools::quantcut(mccain, q = 3)) %>% 
  mutate(mccain_tert = factor(as.numeric(mccain_tert), levels = 1:3, labels = c("Bottom Tertile",
                                                                                "Middle Tertile",
                                                                                "Top Tertile")))
```

Check recodes.

``` r
twitter_tbl %>% 
  ggplot(aes(x = mccain_tert, y = mccain)) +
  geom_boxplot(color = "black", fill = "firebrick") +
  labs(x = "", y = "McCain Vote Share")
```

![](README_files/figure-gfm/unnamed-chunk-15-1.png)<!-- -->

Write out rds, SPSS and Stata.

``` r
saveRDS(twitter_tbl, "data/twitter_data.rds")
write_sav(twitter_tbl, "data/twitter_data.sav")
write_dta(twitter_tbl, "data/twitter_data.dta")
```
