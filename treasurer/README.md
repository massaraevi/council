
Documentation for the Treasurer
===============================

Keep all transactions (income and expenses) in the `accounting.csv` file, with the date in the ISO 8601 standard format (*YYYY-MM-DD*, all numbers). Income should be positive, expenses should be negative (with a minus `-` sign in front).

Receipts should be scanned and placed in the `receipts/` folder, with the filename as the ISO 8601 date format (`YYYY-MM-DD`), followed by the item and then the location of purchase. For instance, food purchased on June 5, 2000 would be `2000-06-05-food-place.pdf`.

Given that this is a *coders* group, all summaries of the finances are done in R with R Markdown. Code is included in the output.

Accounting overview
===================

``` r
# Load the packages and data.
library(dplyr)
library(lubridate)
library(scales)
cad <- dollar_format(negative_parens = TRUE)
finances <- read.csv('accounting.csv') %>% 
    mutate(TransactionDate = ymd(TransactionDate))
```

Actual income and expenses
--------------------------

``` r
income <- sum(finances$Income, na.rm = TRUE)
expense <- sum(finances$Expense, na.rm = TRUE)

library(tidyr)
data_frame(
    Income = cad(income),
    Expense = cad(expense),
    `**Total**` = cad(income - expense)
    ) %>%
    gather(Type, Amount) %>%
    knitr::kable()
```

| Type      | Amount    |
|:----------|:----------|
| Income    | $4,362.85 |
| Expense   | $2,977.22 |
| **Total** | $1,385.63 |

``` r
snacks <- filter(finances, grepl('Snacks', Reason)) 
weeks <- difftime(Sys.Date(), min(snacks$TransactionDate), units = 'weeks')
weeks <- as.numeric(weeks)
per_week <- abs(sum(snacks$Expense) / weeks)
```

**Per session (weekly) expense for snacks**: $6.95

Projected income and expenses
-----------------------------

Values surrounded by brackets `()` denote negative values, as is often standard in accounting. Projected values are until end of *fiscal year*. Fiscal year starts from the month we first started using the BMO bank account (May):

Fiscal year: *May 1st-April 30th.*

``` r
# This needs date needs to be changed each year.
fiscal_year_end <- difftime('2018-05-01', Sys.Date(), units = 'weeks')
fiscal_year_end <- as.numeric(fiscal_year_end)

workshop_income <- finances %>% 
    filter(Tag == "SWC", !is.na(Income), !grepl("walk-in", Reason))

estimated_per_workshop_income <- workshop_income %>% 
    summarize(EstimatedIncome = sum(Income) / length(Income))
estimated_workshops_per_year <- 3

budget_estimate <- data_frame(
    Snacks = -per_week * (fiscal_year_end),
    Misc = -200, # such as stickers, equipment, etc
    Workshops = estimated_per_workshop_income[[1]] * estimated_workshops_per_year
    ) %>%
    mutate(Total = rowSums(.)) %>%
    mutate_all(funs(cad)) %>%
    gather(Item, Amount) %>% 
    as.data.frame() # fix error with pander

library(pander)
pander(budget_estimate, emphasize.strong.rows = nrow(budget_estimate), 
       style = 'rmarkdown', justify = c('left', 'right'))
```

| Item      |         Amount|
|:----------|--------------:|
| Snacks    |      ($337.92)|
| Misc      |         ($200)|
| Workshops |      $2,183.64|
| **Total** |  **$1,645.72**|
