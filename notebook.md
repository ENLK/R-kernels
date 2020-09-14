---
title: "Statistical analysis"
author: "KT"
date: "14/09/2020"
output: html_document
---

---
title: "Stats and factor analysis in R"
output: rmarkdown::github_document
---

```{r}
pacman::p_load(tidyverse, 
               FactoMineR,factoextra,
               gridExtra, plotly)

data <- read.csv("real_estate_db.csv")
colnames(data)
```
Given the high dimensionality of the data, then it would be useful to apply factor analysis to see which variables capture the highest level of variance in the dataset. 

Skip on using summary() on the data since the output would be massively bulky and unkind to the naked eye.

Find those variables that are numeric first and observe some of their distributions.

```{r}

num <- select_if(data, is.numeric)
colnames(num)

```
```{r}
# Function to calculate % of missing data 
pMiss <- function(x) {sum(is.na(x))/length(x)*100}

# Identify columns with NA 
apply(num, 2, pMiss)

```
## filter out the missing observations on variables of interest

```{r}

f1 <- num %>%
  filter(!is.na(rent_mean)) %>%
  filter(!is.na(debt)) %>%
  filter(!is.na(male_age_mean)) %>%
  filter(!is.na(female_age_mean))
```

```{r}
x <- f1[, c("rent_mean", "debt",
            "male_age_mean", "female_age_mean")]

summary(x)
```


## plot the complete data

```{r}
x <- ggplot(f1, aes(x = rent_mean)) + 
  geom_histogram(aes(y = ..density..),
                 bins = 50, fill = "turquoise") + 
  stat_function(fun = dnorm, 
                color = "black",
                args = list(mean = mean(f1$rent_mean),
                            sd = sd(f1$rent_mean))) +
  theme_light() +
  labs(x = "rent", y = "freq")

y <- ggplot(f1, aes(x = debt)) +
  geom_histogram(aes(y = ..density..),
                 bins = 50, fill = "tomato3") + 
  stat_function(fun = dnorm,
                color = "black",
                args = list(mean = mean(f1$debt),
                            sd = sd(f1$debt))) + 
  theme_light() + 
  labs(x = "debt", y = "freq")

grid.arrange(x, y)

```
```{r}
w <- ggplot(f1, aes(x = female_age_mean))+ 
  geom_histogram(aes(y = ..density..),
                 bins = 50,
                 fill = "orchid") + 
  stat_function(fun = dnorm,
                color = "black",
                args = list(mean = mean(f1$female_age_mean),
                            sd = sd(f1$female_age_mean))) + 
  labs(x = "female age",
       y = "freq")

z <- ggplot(f1, aes(x = male_age_mean))+ 
  geom_histogram(aes(y = ..density..),
                 bins = 50,
                 fill = "gold2") + 
  stat_function(fun = dnorm,
                color = "black",
                args = list(mean = mean(f1$male_age_mean),
                            sd = sd(f1$male_age_mean))) + 
  labs(x = "male age",
       y = "freq")

grid.arrange(w, z)

```

```{r}

sums <- num %>% select(rent_mean, debt, 
                        female_age_mean,
                        male_age_mean) %>%
  filter(!is.na(rent_mean), !is.na(debt),
         !is.na(female_age_mean),
         !is.na(male_age_mean))

#do.call(cbind, lapply(sums, summary))
lapply(sums, summary)

```

## means and medians 

```{r}

par(mfrow = c(2, 1))
# debt

hist(sums$debt,
           col = "tomato2", 
           xlab = "debt",
           main = "Debt distribution") %>%
abline(v = mean(sums$debt),
       col = "slateblue") %>%
abline(v = median(sums$debt),
       col = "black")

# rent
hist(sums$rent_mean,
           col = "cyan", xlab = "rent",
           main = "rent distribution") %>%
abline(v = mean(sums$rent_mean),
       col = "red") %>%
abline(v = median(sums$rent_mean),
       col = "black") 
```


```{r}

rent_type <- data[, c("rent_mean", "type")] %>%
  filter(!is.na("rent_mean"), 
         !is.na("type"))


ggplot(rent_type, 
       aes(x = type, 
       y = rent_mean)) +
  geom_boxplot(fill = "gray",
               color = "black",
               outlier.color = "tomato", 
               outlier.shape = 2) + 
  theme_light()

```

```{r}
# Confidence interval plots 
fm_type <- data[, c("female_age_mean",
                    "male_age_mean",
                    "type")] %>%
  filter(!is.na("female_age_mean"), !is.na("male_age_mean"), !is.na("type"))

x2 <- ggplot(fm_type, aes(x = type, 
                          y = female_age_mean)) +
  stat_summary(fun.data = mean_cl_normal,
               color = "tomato") + 
  theme_light() + 
  labs(title = "Average female age by zone type (Confidence interval)",
       x = "type",
       y = "age")
  
y2 <- ggplot(fm_type, aes(x = type, 
                          y = male_age_mean)) +
  stat_summary(fun.data = mean_cl_normal,
               color = "dodgerblue") + 
  theme_light() + 
  labs(title = "Average male age by zone type (Confidence interval)",
       x = "type",
       y = "age")
  
grid.arrange(x2, y2)

```


## Inferential statistics
- Confidence intervals = how certain a value exist between two points
- P-value = probability of obtaining extreme results, provided the null hypothesis is true. High p-value = strong evidence against null hypothesis. 
- significance level = P(rejecting null hypothesis). 

Calculate 95% confidence interval for population mean 
```{r}
# sample size 
n <- sums %>% select(male_age_mean) %>%
  nrow()

# sd
male_std <- sums %>% summarize(std = sd(male_age_mean))

# 
x_bar <- sums %>% summarize(avg = mean(male_age_mean))

standard_error <- -qnorm(0.025)*male_std/sqrt(n)

# lower and upper
low <- x_bar - standard_error
high <- x_bar + standard_error

# female sample size

nf <- sums %>% select(female_age_mean) %>%
  nrow()

female_std <- sums %>%  filter(!is.na(female_age_mean)) %>% summarize(std = sd(female_age_mean))

x_bar <- sums %>%  filter(!is.na(female_age_mean)) %>%
  summarize(avg = mean(female_age_mean))

standard_error <- -qnorm(0.025)*female_std/sqrt(nf)

# lower and upper
low <- x_bar - standard_error
high <- x_bar + standard_error
```

## 95% of random samples from 38, 283 (male and female) will have confidence intervals that have the true population age mean of male and females between 'high' and 'low'

## H0: the true population age mean of females is within the high and low range
## H1: true population age mean of females is NOT within the high and low
range 

```{r}
set.seed(42)
f_age <- data %>% select(female_age_mean) %>%
  filter(!is.na(female_age_mean))

mu <- mean(f_age$female_age_mean)
n <- 100

x_bar <- mean(sample(f_age$female_age_mean,
                     n))

std <- sd(sample(f_age$female_age_mean,
                 n))

# 95% CI 
st_error <- qnorm(0.025)*std/sqrt(n)

# Z-score
z_score <- (x_bar - mu)/st_error
print(paste("Z-score:", z_score))

# P-value (one & two tailed)
two_tail_pval <- 2*pnorm(-abs(z_score))
one_tail_pval <- two_tail_pval / 2

print(paste0("Two tailed p-value:", two_tail_pval))
print(paste0("One tailed p-value:", one_tail_pval))
```
## Insufficient evidence to reject H0 

## Contigency tables are used here to summarize the association between various variables 

- find if two variables are independent or dependent i.e. their relationship.
- independence is a condition for the central limit theorem. 
- Central limit theorem = increasing sample size results in the sample mean approximating a normal distribution. 
- Increasing sample size causes the measured sample means to be more closely distributed around the population mean. 

```{r}
# Chi-squared test of independence on categorical variables 
cats <- select_if(data, is.factor)

colnames(cats)

```

```{r}
tab <- table(cats$type, cats$state)
#summary(tab)
chisq.test(tab)
```
## p-val is less than 0.05, indicating a strong association
```{r}
# Chisq test by unique observation in each variable  
contingency_tab <- cats %>%
  select(state, type) %>%
  table() %>% 
  prop.table() %>%
  round(4)

contingency_tab
```
## T-distribution

- Used for experimenting with small sample sizes
- n < 30 

```{r}
states <- data %>% select(state, rent_mean) %>%
  filter(state == "Texas" | state == "Georgia") %>%
  filter(!is.na(rent_mean))

set.seed(42)
n <- 21
sample <- sample_n(states, n)

Tex <- sample %>%
  filter(state == "Texas")

# Texas test 
Tex_n <- Tex %>%
  nrow()

Tex_x_bar <- Tex %>% summarize(avg = mean(rent_mean))

t_score <- abs(qt(0.025, df = 11))

Tex_sd <- Tex %>%
  summarize(sd = sd(rent_mean))

upper <- Tex_x_bar + t_score * (Tex_sd / sqrt(Tex_n))
lower <- Tex_x_bar - t_score * (Tex_sd / sqrt(Tex_n))
upper
lower
```
## 95% sure the average rent for Texas is between the lower and upper bounds 

## Assume the average rent for Georgia is 888
- Find the p-value i.e. amount of evidence to reject null hypothesis 
```{r}
Geo <- states %>% filter(state == "Georgia")

Geo_n <- Geo %>%
  nrow()

Geo_x_bar <- Geo %>% summarize(avg = mean(rent_mean))

Geo_sd <- Geo %>% summarize(sd = sd(rent_mean))

standard_error <- Geo_sd / sqrt(Geo_n)

mu = 888
t_score <- (Geo_x_bar - mu) / standard_error
print(paste0(t_score))

2*pt(6.371, df = 11, lower.tail = FALSE)

```
## p-value < 0.05, sufficient evidence to reject null hypothesis


## Difference between two means of independent groups 
- Independent groups = 2 separate subject groups 
- Assumes 2 groups (populations) have homogeneity of variance
- The groups are normally distributed 
- Each value is sampled independently

```{r}
states <- data %>%
  filter(state == "New York" | state == "Texas") %>%
  filter(!is.na(rent_mean))

ggplot(states, aes(x=state, y = rent_mean)) + 
  geom_boxplot()+
  stat_summary(fun.y = mean,
               color = "red",
               geom = "point",
               size = 2) + 
  theme_light() +
  labs(x = "State", y = "rent")
```

- H0 = no difference between average rent in NY and Texas
- H1 = difference exists
```{r}
# p-value

NY_n <- states %>%
  filter(state == "New York") %>%
  nrow()

NY_x_bar <- states %>% filter(state == "New York") %>%
  summarize(avg = mean(rent_mean))

NY_sd <- states %>% filter(state == "New York") %>%
  summarize(sd = sd(rent_mean))

# standard error
Tex_se <- Tex_sd/sqrt(Tex_n)
NY_se <- NY_sd/sqrt(NY_n)

standard_error <- sqrt(Tex_se)^2 + sqrt(NY_se)^2

# Point estimate
p_est <- (Tex_x_bar - NY_x_bar)

t_score <- abs((p_est - 0)/standard_error)
t_score <- as.numeric(t_score)

p_val <- 2*pt(t_score, df = 21, lower.tail = FALSE)
p_val

```
## No significant evidence available to reject null hypothesis (no difference between average rent in Texas and New York)


## ANOVA tests
- H0 = mean of rent is the same across the sampled group of states
- H1 = at least 2 states are different from each other 

```{r}
state4 <- data %>%
  select(state, rent_mean) %>%
  group_by(state) %>%
  filter(!is.na(rent_mean)) %>%
  filter(state == "New York"| state == "Texas" | state == "Washington" | state == "California") %>%
  summarize(avg = mean(rent_mean), sd = sd(rent_mean), count = n())

state4
```

```{r}
state4 <- data %>%
  select(state, rent_mean) %>%
  filter(!is.na(rent_mean)) %>%
  filter(state == "New York"| state == "Texas" | state == "Washington" | state == "California") %>%
  group_by(state)

ggplot(state4, aes(x = state, y = rent_mean,
                   fill = state)) +
  geom_boxplot() +
  stat_summary(fun.y = mean,
               color = "red",
               geom = "point",
               size = 2) + 
  labs(x = "States", y = "Rent") + 
  theme_light()
  
```
```{r}
ungroup4 <- state4 %>%
  ungroup()

anova_state4 <- aov(rent_mean ~ state,
                    data = ungroup4)

summary(anova_state4)
```
## Significant evidence to believe that the mean rent is different for at least 2 states.