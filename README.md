---
title: "30535 Applied Problem Set 2"
author: "Khristel Zavaleta"
date: "5/3/2022"
output: 
  pdf_document:
    number_sections: yes
  html_document:
    df_print: paged
urlcolor: blue
---

# Public Sector Application: Home Health Care (description)

The Department of Health and Human Services Office of the Inspector General is interested in identifying home health agency that are potentially over-billing Medicare. In prior years we have had a Harris alum give a guest lecture about these data. Here is the report that he would talk about. Unfortunately the guest lecturer has moved away, but this is still a great problem set for learning about cleaning messy data!
The overarching idea of the problem set is that home health agencies could also be billing a lot because they defrauding the government or because they have unusually sick patients. We wouldn’t want to punish agencies that are taking sicker patients! The data on provider utilization is organized by codes indicating how sick a patient is in terms of their Clinical, Functional, and Service Levels. A case-mix of 1 corresponds to a typical patient’s expected cost. A score of 0.75 indicates low expected costs and a score of 1.5 indicates high expected costs.

# Data Ingestion (10 points)

Some of the variable names in the dataset do not comply with the style guide. There is no need to change source variable names. However, if you create a new variable, that should comply with the style guide.

```{r setup, include=FALSE}
rm(list = ls())
setwd("/Users/khristelzavaleta/Documents/GitHub/applied-ps-2-khristel-ahmad")
library(tidyverse)
library(testthat)
library(readr)
library(ggplot2)
```


**1. Go to the Centers for Medicare and Medicaid Systems data portal. Find and directly click download the Medicare Post-Acute Care & Hospice by Provider and Service for only 2014. You should have downloaded an unformatted csv. Open the file in spreadsheet software. What does it mean when there is a "*" listed in this file? Read the file into R. You will get some warnings when you read it in. What are the warnings (hint: use problems() in readr)? Name the data frame provider. Use test_that to check that you have 31665 rows.**

- What does it mean when there is a "*" listed in this file?

Some aggregated records which are derived from 10 or fewer beneficiaries are excluded to protect the privacy of Medicare beneficiaries and are indicated with an asterisk. Cells are also suppressed when greater than 75% of beneficiaries have a given chronic condition or primary diagnosis, indicated using a double asterisk (**).

Source: https://data.cms.gov/resources/medicare-post-acute-care-hospice-by-provider-and-service-methodology

- Reading the file: 

```{r}
provider <-
  read_csv("unformatted_medicare_post_acute_care_hospice_by_provider_and_service_2014_12_31.csv")
```

```{r}
problems(provider)
```

- What are the warnings (hint: use problems() in readr)?

The warning appears because R identifies a column with as a given type (double) but there are some rows where we find other type of data, for example a character. In this data set, we can that we have some asterisk in the dataframe to protect privacy of Medicare beneficiaries.

- Using test_that: 

```{r}
test_that(
  "we have the right number of rows",
  expect_equal(nrow(provider), 31665)
)
```

\pagebreak

**2. Now, when before downloading the data, click dataset. You should have downloaded a zip file. Open Provider_by_HHRG_PUF_2014 in Excel. Read the file into R using the readxl package and name it provider_hhrg. Use test_that to check that you have 111904 rows.**

- Reading the file into R: 

```{r}
library(readxl)

provider_hhrg <- read_excel("Provider_by_HHRG_PUF_2014.xlsx", sheet = "Data")
```

- Using test_that to check 111904:

```{r}
test_that(
  "we have the right number of rows",
  expect_equal(nrow(provider_hhrg), 111904)
)
```


**3. Download Medicare’s 2014 case-mix weights (CY 2014 HH PPS case-mix weights) from this website. Name the data frame case_mix_weight. Name the variable for 2014 casemix casemix_2014. Drop the column named “2013 HH PPS Case-Mix Weights”. Use test_that to check that you have 153 rows.**

- Naming the data frame, the variable and dropping the column:

```{r}
case_mix_weight <- read_csv("CY 2014 Final HH PPS Case-Mix Weights.csv")

case_mix_weight <- case_mix_weight %>%
  rename(
    casemix_2014 =
      `2014 Final HH PPS Case-Mix Weights`
  ) %>% # naming the variable
  select(-`2013 HH PPS Case-Mix Weights`) # Dropping the column
```

- Using test_that to check 153 rows:

```{r}
test_that(
  "we have the right number of rows",
  expect_equal(nrow(case_mix_weight), 153)
)
```

\pagebreak

# Data Orientation and Validation (15 points)

**1. What are the five types of service categories in provider (this will require some googling)?**

The five types of service categories are: 

```{r}
provider %>%
  distinct(Srvc_Ctgry)
```

Also, from the data from the Centers for Medicare & Medicaid Services, we can conclude know that the codes refers to:

HH: Home Health
HOS: Hospice
SNF: Skilled Nursing Facility
IRF:Inpatient Rehabilitation Facility
LTC: Long-Term Care Hospital

Definitions of the service categories:

a. Skilled nursing facility (SNF): A nursing facility with the staff and equipment to give skilled nursing care and/or skilled rehabilitation services and other related health services

b. Inpatient rehabilitation facility (IRF): IRFs combine hospital-level care with intensive rehabilitation to treat patients and help them regain their usual functioning. It is basically a care center for people who need rehabilitation and have medical needs that require round the clock nursing care

c. Home health agency (HH): means a public agency or private organization that provides skilled nursing services and at least one other home health service

d. Long-term care hospital (LTC): Services that include medical and non-medical care provided to people who are unable to perform basic activities of daily living such as dressing or bathing. Long-term supports and services can be provided at home, in the community, in assisted living or in nursing homes

e. The Medicare Health Outcomes Survey (HOS) is the first patient-reported outcomes measure used in Medicare managed care

Source: https://data.cms.gov/resources/medicare-post-acute-care-hospice-by-provider-and-service-data-dictionary

\pagebreak

**2. provider and provider_hhrg contain information at three different levels of aggregation. What are they?**

The three levels are the nation, state, and provider.

- Using provider_hhrg data frame:

```{r}
provider_hhrg %>%
  distinct(Smry_Ctgry)
```

- Using provider data frame:

```{r}
provider %>%
  distinct(Smry_Ctgry)
```

**3. Using public documents, calculate how many people received home health care benefits (Srvc_Ctgry == "HH") from Medicare in calendar year 2014. Compare this to the total number of beneficiaries in provider. Do the numbers from these sources align? If not, why do you think they might not align?**

```{r}
provider %>%
  filter(Srvc_Ctgry == "HH") %>%
  filter(Smry_Ctgry == "NATION") %>%
  select(Smry_Ctgry, Srvc_Ctgry, Bene_Dstnct_Cnt)
```
Two resources have been found that refer to the number of patients treated at home health care:

- From the article "The Future of Home Health Care: A Strategic Framework for Optimizing Value", we can conclude that the data is aligned, given that they also provide an approximate of 3.4 million people served: "Approximately 3.4 million people receive Medicare skilled home health care, which supports homebound patients by providing coverage for intermittent skilled nursing and therapy services that are provided by Medicare-certified physician home health agencies subject to a plan of care."

Source: https://journals.sagepub.com/doi/full/10.1177/1084822316666368

- According to CDC data for 2015 the total number of patients was 4,455,651. In this regard, we can comment that this difference occurs in (1) because they are different years and (2) because a different source is used to calculate the data on patients served (it is mentioned in the article that it is extracted from a merge file).

Source: https://www.cdc.gov/nchs/fastats/home-health-care.htm

**4. Compare the total number of episodes in provider for home health care and the number in provider_hhrg. Do the numbers from these sources align? If not, why do you think they might not align? Hint: read_xlsx does not automatically parse data types when loading the data, as contrast to read_csv. For our data, you may find readr::parse_number helpful in converting variables to a numerical type.** 


```{r}
provider %>%
  filter(Srvc_Ctgry == "HH", Smry_Ctgry == "NATION") %>%
  select(Smry_Ctgry, Srvc_Ctgry, Tot_Epsd_Stay_Cnt)
```

```{r}
provider_hhrg_v1 <- provider_hhrg %>%
  mutate(Tot_Epsd_Stay_Cnt = parse_number(Tot_Epsd_Stay_Cnt))

provider_hhrg_v1 %>%
  filter(Smry_Ctgry == "NATION") %>%
  summarise(sum(Tot_Epsd_Stay_Cnt))
```


- Do the numbers from these sources align? If not, why do you think they might not align?

The numbers from provider and provider_hhrg don't align. In the first one we got 6558889 total episodes while in the second one we got 5988839 total episodes. They may not align because the providers could be inflating the numbers of episodes when reporting for the first source. The two data frame have difference data source. 

\pagebreak

**5. Focus on just the provider-level rows in provider_hhrg. Within this subset of rows, what column(s) uniquely identify each row? Use a test_that statement to document your answer.** 

- Total number of rows that should be uniquely identify: 

```{r}
provider_hhrg %>%
  filter(Smry_Ctgry == "PROVIDER") %>%
  nrow()
```

- Testing the columns that uniquely identifes ghe rows "Prvdr_ID" and "Grpng":

```{r}
test <- provider_hhrg %>%
  filter(Smry_Ctgry == "PROVIDER") %>%
  distinct(Prvdr_ID, Grpng)

test_that(
  "we have the right number of rows",
  expect_equal(nrow(test), 105245)
)
```

- Example:

```{r}
test_2 <- provider_hhrg %>%
  filter(Prvdr_ID == "027002", Grpng == "1AFK")

test_that(
  "we have the right number of rows",
  expect_equal(nrow(test_2), 1)
)
```

\pagebreak

# Merge Provider Costs with Provider Case-Mix (40 points)

To assess whether a provider is overbilling, we need join the weights in case_mix_weight on to provider_hhrg. This requires some syntax that we haven’t covered yet, so we will walk through it step-by-step.

**1. Google to find the stringr vignette and read it (or read R4DS chapter 14). What does the str_sub command do? What are the required arguments? What does the str_trim command do?**

- str_sub: extract character of the string from a start to an end position. The require arguments are the string (a character or the specific column in the data set), and an integer that gives the position: start (gives the position of the first character) and end (gives the position of the last).

- str_trim: it removes white spaces from a string. The require argument is the string (the character vector or the specific column in the data set). We can also add the side from which we want to remove the whitespace (left, right or both).

**2. There is no common identifer between the two datasets. However, there is common information that can be used to link the two datasets. Review both datasets using the View statement. What five types of information are available in both datasets? Explain using plain English (not copy-pasting a definition that you found online) what these five types of information are.**

- Reviewing the datasets:

```{r}
view(provider_hhrg)
view(case_mix_weight)
```

- Information Available in both datasets: Episodes, Therapy, Clinical Severity Level, Functional Severity Level 1, Service Severity Level 1

*Data frame provider_hhrg*

```{r}
tibble(provider_hhrg$Grpng_Desc) %>%
  distinct()
```

*Data frame case_mix_weight*

```{r}
tibble(
  case_mix_weight$`Clinical, Functional, and Service Levels`,
  case_mix_weight$Description
)
```

This information addresses the characteristics of the illness and the type of care, the description of the HHRG. Five time of information:

- Episodes (Early or late episode): *Early episode* covers the and first and second episodes in an illness and the *Late episode* implies the third episode and beyond in an illness.
- Therapy: Number of therapy visits
- The following are measurement of the impact in the patient: Clinical Severity Level, Functional Severity Level, Service Severity Level

**3. Which column(s) in provider_hhrg do you plan to use for joining? How many distinct HHRG groups are there using this (these) column(s)?**

We plan to use the column Grpng_Desc from the provider data frame.

- Distinct HHRG groups:

```{r}
tibble(provider_hhrg$Grpng_Desc) %>%
  distinct() %>%
  nrow()
```

There are 153 distinct HHRG groups.

**4. Take the column(s) you chose from provider_hhrg and separate them into five new columns – one for each of the information types you listed in #1 above. Be sure to apply str_trim() to any columns which contain text.**

- Separating into five new columns: 

```{r}
provider_hhrg_v2 <- provider_hhrg %>%
  separate(Grpng_Desc,
    c("Episode", "Therapies", "CSL", "FSL", "SVL"),
    sep = ","
  ) %>%
  mutate(
    Episode = str_trim(Episode),
    Therapies = str_trim(Therapies),
    CSL = str_trim(CSL), FSL = str_trim(FSL), SVL = str_trim(SVL)
  )
```


**5. R will likely throw a warning " Additional pieces discarded in. . . " followed by a series of row numbers. List three of the row numbers returned in the warning message. Use filter(row_number() == xx) to check these rows by hand. Why did you get a warning for these rows? Do you think it makes sense to drop these rows? Why or why not?**

```{r}
provider_hhrg %>%
  filter(row_number() == 122 | row_number() == 5580 | row_number() == 5581) %>%
  select(Grpng_Desc)
```

- Why did you get a warning for these rows? Do you think it makes sense to drop these rows? Why or why not?

I get a warning message because these columns terminate in a comma and do not have a character after them, the algorithm expects 5 parts and discards the "sixth" separator. It's not removing the rows, but it is rejecting the new columns that would be created (the sixth column), thus dropping the extra column makes sense. We will clean the data, with the final comma removed from this column. This is what we'll do next:

- Number of rows with the problem:

```{r}
provider_hhrg %>%
  filter(endsWith(Grpng_Desc, ",")) %>%
  nrow()
```

- Deleting the commas that are located at the end of the string

```{r}
clean_provider_hhrg <- provider_hhrg %>%
  mutate(
    Grpng_Desc = if_else(
      grepl(",$", provider_hhrg$Grpng_Desc),
      str_sub(Grpng_Desc, 1, nchar(Grpng_Desc) - 1),
      Grpng_Desc
    )
  )
```

Source: https://stackoverflow.com/questions/47026374/ifelse-starting-with-ends-with-includes

- Separating into five columns, no warning message:

```{r}
provider_hhrg_v3 <- clean_provider_hhrg %>%
  separate(Grpng_Desc,
    c("Episode", "Therapies", "CSL", "FSL", "SVL"),
    sep = ","
  ) %>%
  mutate(
    Episode = str_trim(Episode),
    Therapies = str_trim(Therapies),
    CSL = str_trim(CSL), FSL = str_trim(FSL), SVL = str_trim(SVL)
  )
```


**6. Which column(s) in case_mix_weight do you plan to use for merging? How many distinct HHRG groups are there using this (these) column(s)?**

We plan to use the column Description" and "Clinical, Functional, and Service Levels" from the case_mix_weight data frame.

- Distinct HHRG groups:

```{r}
case_mix_weight %>%
  distinct(Description, `Clinical, Functional, and Service Levels`) %>%
  nrow()
```

There are 153 distinct groups.

\pagebreak

**7. Take the column(s) you chose from case_mix_weight and again separate to add five new columns – one for each of the information types you listed in #1 above. Be sure to apply str_trim to any columns which contain text. Be sure to use the same five column names as you did in the previous question.**

- Separating into five new columns:

```{r}
case_mix_weight_v2 <- case_mix_weight %>%
  separate(Description,
    c("Episode", "Therapies"),
    sep = ","
  ) %>%
  mutate(
    Episode = str_trim(Episode),
    Therapies = str_trim(Therapies)
  ) %>%
  separate(`Clinical, Functional, and Service Levels`,
    c("CSL", "FSL", "SVL"),
    sep = c(2, 4)
  ) %>%
  mutate(
    CSL = str_trim(CSL), FSL = str_trim(FSL), SVL = str_trim(SVL)
  )

case_mix_weight_v2
```

**8. A successful join requires both datasets to have the same values in addition to the same column names. For each of the five new columns, run count() in both datasets. Which of the column(s) have the same values in both datasets? Which of the column(s) have similar values, but require further cleanup? Read about str_sub() command in Section 14 as well as fct_recode() and fct_collapse() commands in section 15.5. Use these commands to fix the columns in case_mix_weight to ensure that the five columns have identical values to provider_hhrg.**

- Running count for each of the five new columns in the provider_hhrg data frame:

```{r}
provider_hhrg_v3 %>%
  count(Episode)
```

```{r}
provider_hhrg_v3 %>%
  count(Therapies)
```


```{r}
provider_hhrg_v3 %>%
  count(CSL)
```

```{r}
provider_hhrg_v3 %>%
  count(FSL)
```

```{r}
provider_hhrg_v3 %>%
  count(SVL)
```

- Running count for each of the five new columns in the case_mix_weight data frame:

```{r}
case_mix_weight_v2 %>%
  count(Episode)
```

```{r}
case_mix_weight_v2 %>%
  count(Therapies)
```

```{r}
case_mix_weight_v2 %>%
  count(CSL)
```

```{r}
case_mix_weight_v2 %>%
  count(FSL)
```

```{r}
case_mix_weight_v2 %>%
  count(SVL)
```

- Which of the column(s) have the same values in both datasets? Which of the column(s) have similar values, but require further cleanup?

Although no two columns have precisely the same data, CSL, FSL, and SVL contain highly similar data that can be readily standardized with the corresponding provider hhrg data frame. Episodes and Therapies, on the other hand, will need data modification in order to be equivalent.

- Fixing columns in case_mix_weight:

*Making uniform columns CSL, FSL and SVL:*

```{r}
case_mix_weight_v3 <- case_mix_weight_v2 %>%
  mutate(CSL = fct_recode(CSL,
    "Clinical Severity Level 1" = "C1",
    "Clinical Severity Level 2" = "C2",
    "Clinical Severity Level 3" = "C3"
  ), FSL = fct_recode(FSL,
    "Functional Severity Level 1" = "F1",
    "Functional Severity Level 2" = "F2",
    "Functional Severity Level 3" = "F3"
  ), SVL = fct_recode(SVL,
    "Service Severity Level 1" = "S1",
    "Service Severity Level 2" = "S2",
    "Service Severity Level 3" = "S3",
    "Service Severity Level 4" = "S4",
    "Service Severity Level 5" = "S5"
  ))
```


```{r}
# Output:

case_mix_weight_v3 %>%
  count(CSL)

case_mix_weight_v3 %>%
  count(FSL)

case_mix_weight_v3 %>%
  count(SVL)
```

*Fixing column "Therapies":*

```{r}
case_mix_weight_v4 <- case_mix_weight_v3 %>%
  mutate(Therapies = fct_collapse(Therapies,
    "0-13 therapies" = c(
      "0 to 5 Therapy Visits",
      "10 Therapy Visits",
      "11 to 13 Therapy Visits",
      "6 Therapy Visits",
      "7 to 9 Therapy Visits"
    ), "14-19 therapies" = c(
      "14 to 15 Therapy Visits",
      "16 to 17 Therapy Visits",
      "18 to 19 Therapy Visits"
    ),
    "20+ therapies" = c("20+ Therapy Visits")
  ))
```

```{r}
case_mix_weight_v4 %>%
  count(Therapies)
```

*Fixing column "Episode":*

For this column with take into consideration the following information: (a) Early episodes are the first and second episodes in an illness. (b) Late episodes are any episode after the second episode in an illness. 

Source:https://www.cms.gov/Research-Statistics-Data-and-Systems/Statistics-Trends-and-Reports/Medicare-Provider-Charge-Data/Downloads/PAC-Methodology.pdf

```{r}
case_mix_weight_v5 <- case_mix_weight_v4 %>%
  mutate(Episode = fct_recode(Episode,
    "Early Episode" = "1st and 2nd Episodes",
    "Late Episode" = "3rd+ Episodes",
    "Early or Late Episode" = "All Episodes"
  ))
```

```{r}
case_mix_weight_v5 %>%
  count(Episode)
```

**9. Create a new data frame called provider_hhrg_wt by joining case_mix_weight to provider_hhrg. Here are two tests to check that your join worked: (a) use test_that to check that provider_hhrg_wt has 111904 rows, (b) show that casemix_2014 is non-missing for all the rows.**

- Creating the new data frame:

```{r}
provider_hhrg_wt <- provider_hhrg_v3 %>%
  left_join(case_mix_weight_v5, by = c(
    "Episode", "Therapies",
    "CSL", "FSL", "SVL"
  ))

view(provider_hhrg_wt)
```

- (a) Testing for having 111904 rows:

```{r}
test_that(
  "we have the right number of rows",
  expect_equal(nrow(provider_hhrg_wt), 111904)
)
```

- (b) Showing that casemix_2014 doesn't has missing values:

```{r}
sum(is.na(provider_hhrg_wt$casemix_2014))
```

\pagebreak

# Billing Outlier Analysis (25 points)

Construct a data frame provider_sum with one row per Provider ID. Each provider serves many groups of patients (each with their own row in provider_hhrg). You will need to describe the provider’s “average billing behavior. The new data frame should have the following columns:

Provider ID
• provider name
• state
• mean medicare payment amount per episode (weighted by total episodes within each group)
• mean patient case mix (weighted by total episodes within each group)
• total number of total episodes.

Hint: mean puts equal weight on each observation so you will need to find a different summary function to construct this dataset.

For each question below, please follow the four-part approach laid out in lecture. I have given you the question (step 1). You should write out your query (step 2), show the plot or table that results from this query (step 3), and write out the answer to the question in a sentence (step 4).

```{r}
provider_hhrg_wt_v1 <- provider_hhrg_wt %>%
  filter(Smry_Ctgry == "PROVIDER") %>% # We are just going to work with PROVIDER
  distinct(Prvdr_ID, Prvdr_Name, State)

provider_hhrg_wt_v1
```

- Creating the mean medicare payment amount per episodw, the mean patient case mix and the total number of total episodes:

```{r}
provider_sum <- provider_hhrg_wt %>% # Converting to numeric values
  mutate(
    Avg_Pymt_Amt_Per_Epsd = parse_number(Avg_Pymt_Amt_Per_Epsd),
    Tot_Epsd_Stay_Cnt = parse_number(Tot_Epsd_Stay_Cnt)
  ) %>%
  filter(Smry_Ctgry == "PROVIDER") %>%
  group_by(Prvdr_ID) %>%
  summarise(
    Mean_Pymt_Epsd_weighted =
      sum(Avg_Pymt_Amt_Per_Epsd * Tot_Epsd_Stay_Cnt) / sum(Tot_Epsd_Stay_Cnt),
    Mean_Casemix_weighted =
      sum(casemix_2014 * Tot_Epsd_Stay_Cnt) / sum(Tot_Epsd_Stay_Cnt),
    Total_number_episodes = sum(Tot_Epsd_Stay_Cnt)
  ) %>%
  left_join(provider_hhrg_wt_v1, by = "Prvdr_ID") %>%
  select(
    Prvdr_ID, Prvdr_Name, State, Mean_Pymt_Epsd_weighted,
    Mean_Casemix_weighted, Mean_Casemix_weighted, Total_number_episodes
  )

provider_sum
```

**1. Question: How much variation is there in average cost per episode by home-health agency?**

Assuming that for home-health agency it is referring to "by provider":

```{r, warning=FALSE, fig.height = 2}
ggplot(provider_sum) +
  geom_histogram(aes(
    x = Prvdr_ID,
    y = Mean_Pymt_Epsd_weighted
  ), stat = "identity") +
  ggtitle("Variation in average cost per episode by home-health agency") +
  theme(plot.title = element_text(hjust = 0.5)) +
  labs(y = "Average cost per episode", x = "Home-health agencies")
```


```{r, warning=FALSE, message=FALSE, fig.height = 3}
ggplot(provider_sum) +
  geom_histogram(aes(x = Mean_Pymt_Epsd_weighted)) +
  ggtitle("Variation in average cost per episode by home-health agency") +
  theme(plot.title = element_text(hjust = 0.5)) +
  labs(y = "Count", x = "Average cost per episode")
```

We can observe from the graphs that the average cost of each episode varies considerably on the axis. For example, in figure two, we can see that the data ranges from slightly over 1,000 to more than 6,000. However, we can observe that there is a data concentration between 2200 and 3200. Therefore, we can say that the data distribution is bell-shaped and near to "normal." Figure 1 shows that several providers have significant data spikes, indicating that the high values are not due to a single source or provider. 

**2. Question: How much variation is there in average cost after accounting for how sick patients are via casemix? Show three different ways to depict the covariation of these two variables. Then explain which plot you prefer to answer the question and why.**

```{r, include=FALSE}
library(hexbin)
library(binsreg)
```


```{r, warning=FALSE, message=FALSE, fig.height = 2}
ggplot(provider_sum) +
  geom_point(aes(x = Mean_Pymt_Epsd_weighted, y = Mean_Casemix_weighted)) +
  ggtitle("Variation in average cost per episode by Casemix") +
  theme(plot.title = element_text(hjust = 0.5)) +
  labs(y = "Average Casemix", x = "Average cost per episode")
```

- Three different ways to depict the covariation:

*Using geom_hex*

```{r, warning=FALSE, message=FALSE, fig.height = 3}
ggplot(provider_sum) +
  geom_hex(aes(x = Mean_Pymt_Epsd_weighted, y = Mean_Casemix_weighted)) +
  ggtitle("Variation in average cost per episode by Casemix") +
  theme(plot.title = element_text(hjust = 0.5)) +
  labs(y = "Average Casemix", x = "Average cost per episode")
```

*Using boxplot*

```{r, warning=FALSE, message=FALSE, fig.height = 2}
ggplot(provider_sum) +
  geom_boxplot(aes(
    x = Mean_Casemix_weighted, y = Mean_Pymt_Epsd_weighted,
    group = cut_width(Mean_Casemix_weighted, 0.5)
  )) +
  ggtitle("Variation in average cost per episode by Casemix") +
  theme(plot.title = element_text(hjust = 0.5)) +
  labs(y = "Average cost per episode", x = "Average Casemix")
```

*Using binscatter*


```{r, warning=FALSE, message=FALSE, fig.height = 2}
binsreg(
  provider_sum$Mean_Casemix_weighted,
  provider_sum$Mean_Pymt_Epsd_weighted
)
```

To answer the question we prefer to use the binscatter because this graph shows us in a more direct way the direct relationship between the average cost and the how sick patients are. 

- How much variation is there in average cost after accounting for how sick patients are via casemix?

Taking into consideration the severity of the patients' illnesses by casemix, we may conclude that there is very little (almost non) variation in the data. The price and the severity of the patient's illness are inextricably linked. We can see a positive correlation between the two factors, and the more serious the illness, the higher the average price of each episode will be.

\pagebreak


**3. For each HHA, construct a new cost_normalized variable which is the ratio of average cost to the average case-mix. Question: Is there more dispersion in average cost or cost_normalized? Why does one have less dispersion than the other? Answer this question with a plot, not a statistic. You might find it helpful to find and link to a stack overflow thread on overlaying histograms with ggplot2.)**

- Creating the new variable cost_normalized: 

```{r}
provider_sum_v1 <- provider_sum %>%
  mutate(cost_normalized = Mean_Pymt_Epsd_weighted / Mean_Casemix_weighted)
```

```{r, message=FALSE, warning=FALSE, fig.height = 3}
ggplot(provider_sum_v1) +
  geom_histogram(aes(x = Mean_Pymt_Epsd_weighted), fill = "red", alpha = 0.3) +
  geom_histogram(aes(x = cost_normalized), fill = "purple", alpha = 0.3) +
  ggtitle("Dispersion of average cost and cost_normalized") +
  theme(plot.title = element_text(hjust = 0.5)) +
  labs(y = "Count", x = "Average cost per episode")
```

Source: https://stackoverflow.com/questions/6957549/overlaying-histograms-with-ggplot2-in-r

- Is there more dispersion in average cost or cost_normalized? Why does one have less dispersion than the other?

There is more dispersion in the average cost. Because the average cost variable does not account for the severity of the patient's sickness, we may claim that the average cost variable has higher dispersion than cost_normalized. For the construction of the variable cost_normalized, the average cost was divided by case_mix and therefore when the disease is more severe it pushes the cost normalized variable lower, whereas if the disease is less severe it drives it higher. That is, the data are normalized based on the severity of the illness. Therefore, we may infer that the variable cost normalized is a preferable source of analysis.

\pagebreak

**4. Question: What are the top 5 HHAs with the highest billing per episode in Illinois? What might happen if OIG decided to try to push down costs at the 5 HHAs with the highest billing per episode in Illinois (unadjusted for case mix)? What are the top 5 HHAs with the highest markups (billing per episode after normalizing for case mix) in Illinois? Is there any overlap between these two lists?**

- Top 5 HHAs with the highest billing per episode in Illinois

```{r}
provider_sum_v1 %>%
  filter(State == "IL") %>%
  arrange(desc(Mean_Pymt_Epsd_weighted)) %>%
  head(5) %>%
  select(-Total_number_episodes, -cost_normalized)
```

- What might happen if OIG decided to try to push down costs at the 5 HHAs with the highest billing per episode in Illinois (unadjusted for case mix)?

```{r}
provider_sum_v1 %>%
  filter(State == "IL") %>%
  arrange(desc(Mean_Casemix_weighted)) %>%
  head(5) %>%
  select(Prvdr_ID, Prvdr_Name, Mean_Casemix_weighted)
```

The chart above shows that the providers with the highest billing per episode are also the ones who handle the most complex cases of illness. As a result, decreasing expenses may lead to these providers refusing to treat the sickest patients since it is inconvenient for them to do so.

- Top 5 HHAs with the highest markups in Illinois:

```{r}
provider_sum_v1 %>%
  filter(State == "IL") %>%
  arrange(desc(cost_normalized)) %>%
  head(5)
```


We can see that there is no overlap between these two constructions of the top5, therefore, the theory that decisions cannot be made from the first table is stronger.