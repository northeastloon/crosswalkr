---
layout: default
title: crosswalkr
---

# About 

This package offers a pair of functions, `renamefrom()` and
`encodefrom()`, for renaming and encoding data frames using external
crosswalk files. It is especially useful when constructing master data
sets from multiple smaller data sets that do not name or encode
variables consistently across files. Based on `renamefrom` and
`encodefrom` [Stata commands written by Sally Hudson and
team](https://github.com/slhudson/rename-and-encode).

## GitHub

Install the latest development version from Github with

```{r}
devtools::install_github('btskinner/crosswalkr')
```

## Dependencies

This package relies on the following packages, available in CRAN:

-   haven
-   labelled
-   readr
-   readxl
-   tibble

