Statistical assignment 4
================
\[Krystof Burysek\]
\[add date here\]

In this assignment you will need to reproduce 5 ggplot graphs. I supply
graphs as images; you need to write the ggplot2 code to reproduce them
and knit and submit a Markdown document with the reproduced graphs (as
well as your .Rmd file).

First we will need to open and recode the data. I supply the code for
this; you only need to change the file paths.

Data8 \<-
read\_tsv(“C:/Users/kryst/OneDrive/Documents/UK.U.zip/UKDA-6614-tab/tab/ukhls\_w8/h\_indresp.tab”)

    ```r
    library(tidyverse) 
    
    Data8 <- read_tsv("C:/Users/kryst/OneDrive/Documents/UK.U/UKDA-6614-tab/tab/ukhls_w8/h_indresp.tab")
    
    
    Data8 <- Data8 %>%
        select(pidp, h_age_dv, h_payn_dv, h_gor_dv)
    
    Stable <- read_tsv("C:/Users/kryst/OneDrive/Documents/UK.U/UKDA-6614-tab/tab/ukhls_wx/xwavedat.tab")
    
    Stable <- Stable %>%
        select(pidp, sex_dv, ukborn, plbornc)
    Data <- Data8 %>% left_join(Stable, "pidp")
    rm(Data8, Stable)
    Data <- Data %>%
        mutate(sex_dv = ifelse(sex_dv == 1, "male",
                           ifelse(sex_dv == 2, "female", NA))) %>%
        mutate(h_payn_dv = ifelse(h_payn_dv < 0, NA, h_payn_dv)) %>%
        mutate(h_gor_dv = recode(h_gor_dv,
                         `-9` = NA_character_,
                         `1` = "North East",
                         `2` = "North West",
                         `3` = "Yorkshire",
                         `4` = "East Midlands",
                         `5` = "West Midlands",
                         `6` = "East of England",
                         `7` = "London",
                         `8` = "South East",
                         `9` = "South West",
                         `10` = "Wales",
                         `11` = "Scotland",
                         `12` = "Northern Ireland")) %>%
        mutate(placeBorn = case_when(
                ukborn  == -9 ~ NA_character_,
                ukborn < 5 ~ "UK",
                plbornc == 5 ~ "Ireland",
                plbornc == 18 ~ "India",
                plbornc == 19 ~ "Pakistan",
                plbornc == 20 ~ "Bangladesh",
                plbornc == 10 ~ "Poland",
                plbornc == 27 ~ "Jamaica",
                plbornc == 24 ~ "Nigeria",
                TRUE ~ "other")
        )
    ```

Reproduce the following graphs as close as you can. For each graph,
write two sentences (not more\!) describing its main message.

1.  Univariate distribution (20 points).
    
    ``` r
    Data <- Data %>%
        filter(!is.na(h_payn_dv)) 
    
    Data %>%  
    ggplot(aes(x = h_payn_dv)) +
    geom_freqpoly() +
    xlab("Net monthly pay") +
    ylab("Number of respondents")
    ```
    
    ![](assignment4_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->
    
    ``` r
    ## Interpratation: The net monthly pay does not follow a normal distribution with the majority of 
    ## incomes concetrated from 0 to over 4000. This points unequal distribution of wealth in the UK.
    ```

2.  Line chart (20 points). The lines show the non-parametric
    association between age and monthly earnings for men and women.
    
    ``` r
     By.age.sex <- Data %>%
        group_by(h_age_dv, sex_dv) %>%
        summarise(mean_pay = mean(h_payn_dv))
    
    
     By.age.sex %>%
    ggplot(aes(x = h_age_dv, y = mean_pay, linetype = sex_dv), colour = "black") +
    geom_smooth(colour = "black") +
    xlim(c(16, 65)) +
    ylim(c(100, 2100)) +
    xlab("Age") +
    ylab("Monthly earnings") +
       scale_fill_distiller(name = "Sex")
    ```
    
    ![](assignment4_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->
    
    ``` r
    ## Interpretation: Both sexes pay earnings increase until 50 years of age but then their salary 
    ## starts to decrease. The discrepancy between women and men also increase rapidly when young with
    ## men earning conistently more throughtout the lifetime.
    ```

3.  Faceted bar chart (20 points).
    
    ``` r
    by.born.sex <- Data %>%
        group_by(sex_dv, placeBorn) %>%
        summarise(median_pay = median(h_payn_dv, na.rm = TRUE)) %>%
        filter(!is.na(sex_dv)) %>%
        filter(!is.na(placeBorn))
    
    
     by.born.sex %>%
    ggplot(aes(x =(sex_dv), y = median_pay )) +
    geom_bar(stat = "identity") +
    facet_wrap(vars(placeBorn)) +
    xlab("Sex") +
    ylab("Median monthly pay")
    ```
    
    ![](assignment4_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->
    
    ``` r
    ## Interpretation: The lowest discrepancy between mean and women have those of Bangaldeshi whoch is
    ## is the group with the lowest median monthly income. The most pronopunced differences betwen those ## born in Ireland and UK. Men overall receive more in all groups and the group erecives the highest ## pay. This indicates the better the group's pay the higher te income inequality between genders.
    ```

4.  Heat map (20 points).
    
    ``` r
     by.region.born.mean_age <- Data %>%
        group_by(h_gor_dv, placeBorn) %>%
        summarise(mean_age = mean(h_age_dv, na.rm = TRUE)) %>%
        filter(!is.na(h_gor_dv)) %>%
        filter(!is.na(placeBorn))
    
    
    by.region.born.mean_age %>%
    ggplot(aes(x = h_gor_dv, y = placeBorn, fill = mean_age)) +
    theme_classic()+
    geom_bin2d(colour = "blue", linetype = "blank") +
    theme(axis.text.x = element_text(angle = 90)) +
    xlab("Region") +
    ylab("Country of origin") +
    theme(axis.line.x = element_blank())+
    theme(axis.line.y = element_blank())
    ```
    
    ![](assignment4_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->
    
    ``` r
    ## Interpretation: The graph shows the distribution of age across UK by region and respodents' 
    ## country of origin. Whereas British population seems to have quite a stable age distribution 
    ## regions, other groups show more variation like those originating in Bangladesh or Nigeria which  ## could also be said to have the lowest age cohorts across the UK. White spots indicate there are ## no data for ages of groups living in those regions. Region with the youngest population could be ## said that London or Yorkshire.
    ```

5.  Population pyramid (20 points).
    
    ``` r
    ## Interpretation: The biggest age groups have the age of approximately 50 years old. Women are also
    ## slightly more numerous across age groups. The least numerous group is in their late twenties 
    ## indictaing the decline in the most productive population.
    ```
