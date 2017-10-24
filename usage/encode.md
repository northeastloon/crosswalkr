---
layout: page
title: Encoding
---


## Command

```r
encodefrom(.data, var, cw_file, raw, clean, label, delimiter = NULL,
           sheet = NULL, case_ignore = TRUE, ignore_tibble = FALSE)

encodefrom_(.data, var, cw_file, raw, clean, label, delimiter = NULL,
            sheet = NULL, case_ignore = TRUE, ignore_tibble = FALSE)
```

#### NB

`encodefrom_()` is the standard evaluation version of `encodefrom`
(`var`, `raw`, `clean`, and `label` must be strings when using this
version)

## Arguments

* `.data` Data frame or tbl_df  
*  Column name of vector to be encoded
* `cw_file` Either data frame object or string with path to external
    crosswalk file, including path, which has columns representing
    `raw` (current) vector values, `clean` (new) vector values, and
    `label`s for values. Values in `raw` and `clean` columns must be
    unique (1:1 match)
    or an error will be thrown. Acceptable file types include:  
	* delimited (`.csv`, `.tsv`, or other)  
	* R (`.rda`, `.rdata`, `.rds`)  
	* Stata (`.dta`).
* `raw` Name of column in `cw_file` that contains values in current
     vector.
* `clean` Name of column in `cw_file` that contains new values for
  vector.
* `label` Name of column in `cw_file` with labels for new values.
* `delimiter` String delimiter used to parse
    `cw_file`. Only necessary if using a delimited file that
    isn't a comma-separated or tab-separated file (guessed by
    function based on file ending).
* `sheet` Specify sheet if `cw_file` is an Excel file and
    required sheet isn't the first one.
* `case_ignore` Ignore case when matching current `raw`
    vector names with new (`clean`) column names.
* `ignore_tibble` Ignore `.data` status as tbl_df and return vector as
     a factor rather than labelled vector.

## Returns

Vector that is either a factor or labelled, depending on data input
and options

## Examples
```r
## init data to be encoded
df <- data.frame(state = c('Kentucky','Tennessee','Virginia'),
                 stfips = c(21,47,51),
                 cenregnm = c('South','South','South'))

## create separate tbl_df
df_tbl <- tibble::as_data_frame(df)

## read in state crosswalk from package
cw <- get(data(stcrosswalk))

## encode using non-standard evaluation version as factor
df$state2 <- encodefrom(df, state, cw, stname, stfips, stabbr)

## ...as labelled vector b/c df_tbl is a tibble
df_tbl$state2 <- encodefrom(df_tbl, state, cw, stname, stfips, stabbr)

## ...override to encode as factor anyway
df_tbl$state3 <- encodefrom(df_tbl, state, cw, stname, stfips, stabbr, ignore_tibble = TRUE)

## show with labels
haven::as_factor(df_tbl)

## show without labels
haven::zap_labels(df_tbl)
```
