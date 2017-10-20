---
layout: page
title: Renaming
---

## Command

```r
renamefrom(.data, cw_file, raw, clean, label = NULL, delimiter = NULL, 
           sheet = NULL, drop_extra = TRUE, case_ignore = TRUE,
           keep_label = FALSE, name_label = FALSE)

renamefrom_(.data, cw_file, raw, clean, label = NULL, delimiter = NULL,
            sheet = NULL, drop_extra = TRUE, case_ignore = TRUE,
            keep_label = FALSE, name_label = FALSE)
```

#### NB

`renamefrom_()` is the standard evaluation version of `renamefrom`
(`raw`, `clean`, and `label` must be strings when using this version)

## Arguments

* `.data` Data frame or tbl_df
* `cw_file` Either data frame object or string with path to external
    crosswalk file, which has columns representing `raw` (current)
    column names, `clean` (new) column names, and labels
    (optional). Values in `raw` and `clean` columns must be unique
    (1:1 match) or an error
    will be thrown. Acceptable file types include:  
	* delimited (`.csv`, `.tsv`, or other)  
	* R (`.rda`, `.rdata`, `.rds`)  
	* Stata (`.dta`).
* `raw` Name of column in `cw_file` that contains column names of
    current data frame.
* `clean` Name of column in `cw_file` that contains new column names.
* `label` Name of column in `cw_file` with labels for columns.
* `delimiter` String delimiter used to parse `cw_file`. Only necessary
    if using a delimited file that isn't a comma-separated or
    tab-separated file (guessed by function based on file ending).
* `sheet` Specify sheet if `cw_file` is an Excel file and required
    sheet isn't the first one.
* `drop_extra` Drop extra columns in current data frame if they are
    not matched in `cw_file`.
* `case_ignore` Ignore case when matching current `raw` column names
    with new (`clean`) column names.
* `keep_label` Keep current label, if any, on data frame columns that
	aren't matched in `cw_file`. Default `FALSE` means that unmatched
	columns have any existing labels set to `NULL`.
* `name_label` Use old (`raw`) column name as new (`clean`) column
	name label. Cannot be used if `label` option is set.

## Returns

Data frame or tbl_df with new column names and labels.

## Examples
```r
## init starting data (to be renamed)
df <- data.frame(state = c('Kentucky','Tennessee','Virginia'),
                 fips = c(21,47,51),
                 region = c('South','South','South'))

## create crosswalk between raw names and clean names with labels
cw <- data.frame(old_name = c('state','fips'),
                 new_name = c('stname','stfips'),
                 label = c('Full state name', 'FIPS code'))

## rename using non-standard evaluation version
df1 <- renamefrom(df, cw, old_name, new_name, label)

## ...using old names as labels
df2 <- renamefrom(df, cw, old_name, new_name, name_label = TRUE)

## ...keep original columns that don't have match in crosswalk
df3 <- renamefrom(df, cw, old_name, new_name, drop_extra = FALSE)

## rename using standard evaluation version
df1 <- renamefrom_(df, cw, 'old_name', 'new_name', 'label')
df2 <- renamefrom_(df, cw, 'old_name', 'new_name', name_label = TRUE)
df3 <- renamefrom_(df, cw, 'old_name', 'new_name', drop_extra = FALSE)

```

