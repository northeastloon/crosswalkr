---
layout: default
title: vignette
---

Researchers often must compile master data sets from a number of smaller
data sets that are not consistent in terms of variable names or value
encodings. This can be especially true for large administrative data
sets that span multiple years and/or departments. Other times, teams of
researchers must work together to maintain a master data set and it is
important for replicability and future collaboration that the team rely
on consistent naming and encoding conventions.

For example, let's say there are three flat files of student information
that need to be merged into a single large data set for analysis.

### File 1

<table>
<thead>
<tr class="header">
<th align="left">sid</th>
<th align="left">lname</th>
<th align="left">state</th>
<th align="left">t_score</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">1</td>
<td align="left">Jackson</td>
<td align="left">VA</td>
<td align="left">74</td>
</tr>
<tr class="even">
<td align="left">2</td>
<td align="left">Harrison</td>
<td align="left">KY</td>
<td align="left">86</td>
</tr>
<tr class="odd">
<td align="left">3</td>
<td align="left">Nixon</td>
<td align="left">IL</td>
<td align="left">78</td>
</tr>
</tbody>
</table>

### File 2

<table>
<thead>
<tr class="header">
<th align="left">stu_id</th>
<th align="left">last_name</th>
<th align="left">st</th>
<th align="left">test_score</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">4</td>
<td align="left">Washington</td>
<td align="left">2</td>
<td align="left">92</td>
</tr>
<tr class="even">
<td align="left">5</td>
<td align="left">Roosevelt</td>
<td align="left">11</td>
<td align="left">67</td>
</tr>
<tr class="odd">
<td align="left">6</td>
<td align="left">Taylor</td>
<td align="left">47</td>
<td align="left">68</td>
</tr>
</tbody>
</table>

### File 3

<table>
<thead>
<tr class="header">
<th align="left">s_id</th>
<th align="left">name</th>
<th align="left">sta</th>
<th align="left">score</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">7</td>
<td align="left">Tyler</td>
<td align="left">North Dakota</td>
<td align="left">91</td>
</tr>
<tr class="even">
<td align="left">8</td>
<td align="left">Grant</td>
<td align="left">South Dakota</td>
<td align="left">82</td>
</tr>
<tr class="odd">
<td align="left">9</td>
<td align="left">Adams</td>
<td align="left">Illinois</td>
<td align="left">89</td>
</tr>
</tbody>
</table>

It is clear that these files contain the same basic information, but
neither the names nor encodings for `state` | `st` | `sta` are
consistent.

One solution is to just fix these one at a time before joining them. For
example:

    library(crosswalkr)
    library(dplyr)
    library(labelled)
    library(haven)

    df1 <- file_1 %>%
        rename(id = sid,
               last_name = lname,
               stabbr = stat,
               score = t_score)

    df2 <- file_2 %>%
        rename(id = stu_id,
               stabbr = st,
               score = test_score) %>%
        mutate(stabbr = as.character(stabbr))

    df3 <- file_3 %>%
        rename(id = s_id,
               stabbr = sta,
               last_name = name)

    df <- rbind(df1, df2, df3)
    df

    ##   id  last_name       stabbr score
    ## 1  1    Jackson           VA    74
    ## 2  2   Harrison           KY    86
    ## 3  3      Nixon           IL    78
    ## 4  4 Washington            2    92
    ## 5  5  Roosevelt           11    82
    ## 6  6     Taylor           47    89
    ## 7  7      Tyler North Dakota    91
    ## 8  8      Grant South Dakota    82
    ## 9  9      Adams     Illinois    89

The problem, of course, is there is a lot of room for error since the
renaming process has to be repeated for each data frame.

### Using a crosswalk file

Instead, it makes more sense to create a crosswalk data set that aligns
old (or raw) column names with new (or clean) column names and, if
desired, labels. The `crosswalk` to join these files could be:

<table>
<thead>
<tr class="header">
<th align="left">clean</th>
<th align="left">label</th>
<th align="left">file_1_raw</th>
<th align="left">file_2_raw</th>
<th align="left">file_3_raw</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">id</td>
<td align="left">Student ID</td>
<td align="left">sid</td>
<td align="left">stu_id</td>
<td align="left">s_id</td>
</tr>
<tr class="even">
<td align="left">last_name</td>
<td align="left">Student last name</td>
<td align="left">lname</td>
<td align="left">last_name</td>
<td align="left">name</td>
</tr>
<tr class="odd">
<td align="left">stabbr</td>
<td align="left">State abbreviation</td>
<td align="left">stat</td>
<td align="left">st</td>
<td align="left">sta</td>
</tr>
<tr class="even">
<td align="left">score</td>
<td align="left">Test score</td>
<td align="left">t_score</td>
<td align="left">test_score</td>
<td align="left">score</td>
</tr>
</tbody>
</table>

The crosswalk file (`cw_file`) could be:

1.  Data frame object already in memory  
2.  A string with path and name (*e.g.*, `'./path/to/crosswalk.csv'`) of
    a flat file of one of the following types:
    1.  Comma separated (`*.csv`)  
    2.  Tab separated (`*.tsv`)  
    3.  Other delimited (`*.txt`) with `delimiter` option set to
        delimiter string (*e.g.*, `delimiter = '|'`)  
    4.  Excel (`*.xls` or `*.xlsx`) with `sheet` option set to sheet
        number or string name (defaulting to the first sheet)  
    5.  R data (`*.rdata`, `*.rda`, `*.rds`)  
    6.  Stata data (`*.dta`)

If given a string to the `cw_file` argument, `renamefrom()` and
`encodefrom()` determine the type of file by its ending.

Renaming
--------

To rename using the `renamefrom()` command:

    df1 <- renamefrom(file_1,
                      cw_file = crosswalk,
                      raw = file_1_raw,
                      clean = clean,
                      label = label)
    df2 <- renamefrom(file_2, cw_file = crosswalk, raw = file_2_raw, clean = clean, label = label)
    df3 <- renamefrom(file_3, cw_file = crosswalk, raw = file_3_raw, clean = clean, label = label)

    df <- rbind(df1, df2, df3)
    df

    ##   id  last_name       stabbr score
    ## 1  1    Jackson           VA    74
    ## 2  2   Harrison           KY    86
    ## 3  3      Nixon           IL    78
    ## 4  4 Washington            2    92
    ## 5  5  Roosevelt           11    82
    ## 6  6     Taylor           47    89
    ## 7  7      Tyler North Dakota    91
    ## 8  8      Grant South Dakota    82
    ## 9  9      Adams     Illinois    89

And check out the labels:

    var_label(df)

    ## $id
    ## [1] "Student ID"
    ## 
    ## $last_name
    ## [1] "Student last name"
    ## 
    ## $stabbr
    ## [1] "State abbreviation"
    ## 
    ## $score
    ## [1] "Test score"

As new raw data files are added to the project, they could simply be
given a new column in the crosswalk file that mapped their raw column
names to the clean versions.

Encoding
--------

These same example files have inconsistent encodings for state: one uses
two-letter abbreviations, another the FIPS code, and another the full
name. Again, instead of fixing each one at a time, a separate crosswalk
for encoding these values could be used. The `crosswalkr` package
includes a state-level crosswalk, `stcrosswalk`:

    data(stcrosswalk)
    head(stcrosswalk)

    ## # A tibble: 6 x 7
    ##   stfips stabbr     stname cenreg cenregnm cendiv           cendivnm
    ##    <int>  <chr>      <chr>  <int>    <chr>  <int>              <chr>
    ## 1      1     AL    Alabama      3    South      6 East South Central
    ## 2      2     AK     Alaska      4     West      9            Pacific
    ## 3      4     AZ    Arizona      4     West      8           Mountain
    ## 4      5     AR   Arkansas      3    South      7 West South Central
    ## 5      6     CA California      4     West      9            Pacific
    ## 6      8     CO   Colorado      4     West      8           Mountain

The `encodefrom()` function works much like `renamefrom()`. The only
difference is that a vector of encoded values is returned that can be
added to an existing dataframe.

`encodefrom()` returns either base R factors or labels depending on
whether the input data frame is a tibble.

### return factor

    df1$state <- encodefrom(file_1, var = stat, stcrosswalk, raw = stabbr, clean = stfips, label = stname)
    df1

    ##   id last_name stabbr score    state
    ## 1  1   Jackson     VA    74 Virginia
    ## 2  2  Harrison     KY    86 Kentucky
    ## 3  3     Nixon     IL    78 Illinois

    sapply(df1, class)

    ##          id   last_name      stabbr       score       state 
    ## "character" "character" "character" "character"    "factor"

### return labelled vector

    file_1_ <- file_1 %>% tbl_df()
    df1$state <- encodefrom(file_1_, var = stat, stcrosswalk, raw = stabbr,
                            clean = stfips, label = stname)
    as_factor(df1)

    ##   id last_name stabbr score    state
    ## 1  1   Jackson     VA    74 Virginia
    ## 2  2  Harrison     KY    86 Kentucky
    ## 3  3     Nixon     IL    78 Illinois

    zap_labels(df1)

    ##   id last_name stabbr score state
    ## 1  1   Jackson     VA    74    51
    ## 2  2  Harrison     KY    86    21
    ## 3  3     Nixon     IL    78    17

Combined example: `dplyr` chain
-------------------------------

The `renamefrom()` and `encodefrom()` functions can be combined in a
`dplyr` chain.

    df <- rbind(file_1 %>%
                tbl_df() %>%
                renamefrom(., crosswalk, file_1_raw, clean, label) %>%
                mutate(stabbr = encodefrom(., stabbr, stcrosswalk, stabbr, stfips, stname)),
                file_2 %>%
                tbl_df() %>%
                renamefrom(., crosswalk, file_2_raw, clean, label) %>%
                mutate(stabbr = encodefrom(., stabbr, stcrosswalk, stfips, stfips, stname)),
                file_3 %>%
                tbl_df() %>%
                renamefrom(., crosswalk, file_3_raw, clean, label) %>%
                mutate(stabbr = encodefrom(., stabbr, stcrosswalk, stname, stfips, stname)))

    df

    ## # A tibble: 9 x 4
    ##      id  last_name    stabbr score
    ##   <chr>      <chr> <chr+lbl> <chr>
    ## 1     1    Jackson        51    74
    ## 2     2   Harrison        21    86
    ## 3     3      Nixon        17    78
    ## 4     4 Washington         2    92
    ## 5     5  Roosevelt        11    82
    ## 6     6     Taylor        47    89
    ## 7     7      Tyler        38    91
    ## 8     8      Grant        46    82
    ## 9     9      Adams        17    89

    as_factor(df)

    ## # A tibble: 9 x 4
    ##      id  last_name               stabbr score
    ##   <chr>      <chr>               <fctr> <chr>
    ## 1     1    Jackson             Virginia    74
    ## 2     2   Harrison             Kentucky    86
    ## 3     3      Nixon             Illinois    78
    ## 4     4 Washington               Alaska    92
    ## 5     5  Roosevelt District of Columbia    82
    ## 6     6     Taylor            Tennessee    89
    ## 7     7      Tyler         North Dakota    91
    ## 8     8      Grant         South Dakota    82
    ## 9     9      Adams             Illinois    89
