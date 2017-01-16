---
layout: post
title: User Experience Survey about Two visualization Form (Histogram vs. Scatterplot )
---

### 0.Motivation

Different forms of data visualization convey different insights to certain dataset. We designed a survey about obtaining information between a histogram and a scatterplot, then collected 23 MDS classmates' survey feedback. We mainly focus on presenting two variables(intentional walk & home runs) sourced from the batting table of the Lahman R package (<http://cran.us.r-project.org/web/packages/Lahman/Lahman.pdf>) for the Toronto Blue Jays team for all years in the dataset (1977 - 2015).

### 1.Preparation: Import and clean the survey data

```
# convert character variables into factor variables
survey_data <- survey_data %>%
  mutate(plot = as.factor(plot),
         distribution_iw = as.factor(distribution_iw),
         distribution_hr = as.factor(distribution_hr))

# present the top 5 rows of the tidy dataset
survey_data %>%
  head(5)
```

### 2.Visualization: encoding the users' answers into graphs

![comparing trend accessibility](../../images/q1_q3.png)

![comparing distribution accessibility](../../images/q5_q6.png)

![comparing estimates of  the number of outlier](../../images/q7_q8.png)

![comparing estimates of the mean](../../images/q9_12_1.png)

![comparing estimates of the mode and minimum](../../images/q9_12_2.png)

![comparing outlier recognition](../../images/q2.png)


### 3.Conclusions

- Histogram is more effective in conveying distribution/trend and mode.

- Scatterplot is more effective in addressing outlier and relationship.
