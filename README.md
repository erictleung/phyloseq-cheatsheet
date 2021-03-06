# phyloseq-cheatsheet

Minimal cheatsheet for functions in the phyloseq R package.

**Contents**

- [Workflow](#workflow)
- [Class accessors](#class-accessors)
- [Code snippets](#code-snippets)
- [Insights](#insights)
- [Useful Resources](#useful-resources)


## Workflow

![Analysis workflow](./img/analysis_workflow.png)

> Attribution: McMurdie, Holmes (2013)
> https://doi.org/10.1371/journal.pone.0061217.g002


## Class accessors

![phyloseq class](./img/phyloseq_class.png)

> Attribution: McMurdie, Holmes (2013)
> https://doi.org/10.1371/journal.pone.0061217.g003


## Code snippets

The table here is meant to be a quick look up on how to use common functions.

The table below has four columns:

1. **Type** = Input, Data, Wrangle, Plot
2. **Function** = Function as called in script
3. **Description** = Terse description of function
4. **Example** = Minimal code example to demonstrate function; will often make
   use of built-in data from `phyloseq`

| Type    | Function         | Description          | Example                |
|---------|------------------|----------------------|------------------------|
| Data    | `enterotype`     |                      | `data(enterotype)`     |
| Data    | `GlobalPatterns` |                      | `data(GlobalPatterns)` |
| Data    | `esophagus`      | Subset of esophageal | `data(esophagus)`      |
| Wrangle | `psmelt`         | Change to data frame | `psmelt(esophagus)`    |


## Insights

### `prune_*` versus `subset_*`

There is a subtle difference between the `prune_*` and `subset_*` functions.

The `prune_` functions (e.g., `prune_samples()`) keep the set of observations based
on the data in the phyloseq object itself, such as conditioning on one of the
columns.

The `subset_` functions (e.g., `subset_samples()`) keep the set of observations based
auxillary data and/or evaluated expressions. In other words, it is a wrapper function
around the base R `subset()` function.

Here are some examples:

```r
# Source: https://joey711.github.io/phyloseq/preprocess.html
GP.chl <- subset_taxa(GlobalPatterns, Phylum == "Chlamydiae")
GP.chl <- prune_samples(sample_sums(GP.chl) >= 20, GP.chl)

# Subset using sample names themselves
GP.subset <- prune_samples(c("CL3", "CC1", "SV1"), GlobalPatters)
```

The `subset_taxa()` function uses the already present `Phylum` column in the taxonomy
table to subset the data. 

Meanwhile, the `prune_samples()` function uses

- an expression that evaluates to `TRUE/FALSE`
- a character vector of sample names

in order to subset the samples. This sort of expression
could not be evaluated using the `subset_samples()` function.

See https://joey711.github.io/phyloseq/preprocess.html#preprocessing for more.


### Using `subset_*` functions

The `subset_*` functions (for example, `subset_samples()`) uses base R's `subset()`
function. The documentation of this function notes:

> This is a convenience function intended for use interactively. For programming it
> is better to use the standard subsetting functions like [, and in particular the
> non-standard evaluation of argument subset can have unanticipated consequences.

In other words, the `subset()` function and `phyloseq::subset_*` function by
association don't work well within functions.

To get around this, I have been using `phyloseq::prune_samples()` and having code
prior to it to get the sample names I want based on a boolean check.

**Note**: doesn't run reproducibly (yet), but the sentiment remains.

```r
# Get sample names
keep_samples <- sample_data(GlobalPatterns)$sample_name %in% c("CL3", "CC1", "SV1")

# Subset using sample names themselves
GP.subset <- prune_samples(keep_samples, GlobalPatters)
```

Source: https://www.rdocumentation.org/packages/base/versions/3.6.2/topics/subset

### Extracting data frames into tibbles

When extracting data frames from `phyloseq` using accessors like `sample_data()` can
end up with unintended consequences, as noted here.

In this example, the code shows various ways to access the sample data and converting
it to a non-`phyloseq` data frame and looking at the dimensions of the object.

Notice the cases when the dimensions go from 26 x 7 to 8 x 8.

``` r
library(phyloseq)
library(magrittr)
library(tibble)

data("GlobalPatterns")

# Natural use but with phyloseq object
GlobalPatterns %>%
    sample_data %>%
    dim()
#> [1] 26  7

# Using base R
GlobalPatterns %>%
    sample_data %>%
    data.frame() %>%
    dim()
#> [1] 26  7

# Using base R "as" convention
GlobalPatterns %>%
    sample_data %>%
    as.data.frame() %>%
    dim()
#> [1] 26  7

# Use tibble convention
GlobalPatterns %>%
    sample_data() %>%
    as_tibble() %>%
    dim()
#> Warning in class(x) <- c(subclass, tibble_class): Setting class(x) to
#> multiple strings ("tbl_df", "tbl", ...); result will no longer be an S4
#> object
#> [1] 26  7

# Use tibble function to keep row names
GlobalPatterns %>%
    sample_data() %>%
    rownames_to_column() %>%
    dim()
#> [1] 8 8

# Using base R and row names to column
GlobalPatterns %>%
    sample_data %>%
    data.frame() %>%
    rownames_to_column() %>%
    dim()
#> [1] 26  8

# Using base R "as" convention and row names to column
GlobalPatterns %>%
    sample_data %>%
    as.data.frame() %>%
    rownames_to_column() %>%
    dim()
#> [1] 8 8

# Use tibble convention and row names to column
GlobalPatterns %>%
    sample_data() %>%
    as_tibble() %>%
    rownames_to_column() %>%
    dim()
#> Warning in class(x) <- c(subclass, tibble_class): Setting class(x) to
#> multiple strings ("tbl_df", "tbl", ...); result will no longer be an S4
#> object
#> [1] 26  8
```

<sup>Created on 2019-06-17 by the [reprex package](https://reprex.tidyverse.org) (v0.2.1)</sup>


### Creating new variables in data

A common task is to quickly create new variables in the data (e.g., sample data).

Instead of extracting the entire data and the reassigning it, you can simply access the variable itself directly using the `$` accessor.

``` r
# Load phyloseq and data
library(phyloseq)
data("enterotype")
enterotype
#> phyloseq-class experiment-level object
#> otu_table()   OTU Table:         [ 553 taxa and 280 samples ]
#> sample_data() Sample Data:       [ 280 samples by 9 sample variables ]
#> tax_table()   Taxonomy Table:    [ 553 taxa by 1 taxonomic ranks ]

# See variables we can change
head(sample_data(enterotype))
#>           Enterotype Sample_ID SeqTech  SampleID     Project Nationality Gender
#> AM.AD.1         <NA>   AM.AD.1  Sanger   AM.AD.1      gill06    american      F
#> AM.AD.2         <NA>   AM.AD.2  Sanger   AM.AD.2      gill06    american      M
#> AM.F10.T1       <NA> AM.F10.T1  Sanger AM.F10.T1 turnbaugh09    american      F
#> AM.F10.T2          3 AM.F10.T2  Sanger AM.F10.T2 turnbaugh09    american      F
#> DA.AD.1            2   DA.AD.1  Sanger   DA.AD.1     MetaHIT      danish      F
#> DA.AD.1T        <NA>  DA.AD.1T  Sanger      <NA>        <NA>        <NA>   <NA>
#>           Age ClinicalStatus
#> AM.AD.1    28        healthy
#> AM.AD.2    37        healthy
#> AM.F10.T1  NA          obese
#> AM.F10.T2  NA          obese
#> DA.AD.1    59        healthy
#> DA.AD.1T   NA           <NA>

# Change variable
sample_data(enterotype)$Over30 <- sample_data(enterotype)$Age > 30

# See changes
head(sample_data(enterotype)[, c("Age", "Over30")])
#>           Age Over30
#> AM.AD.1    28  FALSE
#> AM.AD.2    37   TRUE
#> AM.F10.T1  NA     NA
#> AM.F10.T2  NA     NA
#> DA.AD.1    59   TRUE
#> DA.AD.1T   NA     NA
```

<sup>Created on 2020-09-10 by the [reprex package](https://reprex.tidyverse.org) (v0.3.0)</sup>


## Useful Resources

- [`phyloseq` Official Website](https://joey711.github.io/phyloseq/index.html)
- [DIY: Public Restroom Bacteria](http://joey711.github.io/phyloseq-demo/Restroom-Biogeography)
- [Vignette for phyloseq: Analysis of high-throughput microbiome census data](https://www.bioconductor.org/packages/devel/bioc/vignettes/phyloseq/inst/doc/phyloseq-analysis.html)


## License

[![CC0](http://mirrors.creativecommons.org/presskit/buttons/88x31/svg/cc-zero.svg)](https://creativecommons.org/publicdomain/zero/1.0/)

