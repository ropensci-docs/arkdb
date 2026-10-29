# streamable tsv using base R functions

streamable tsv using base R functions

## Usage

``` r
streamable_base_tsv()
```

## Value

a `streamable_table` object (S3)

## Details

Follows the tab-separate-values standard using
[`utils::read.table()`](https://rdrr.io/r/utils/read.table.html), see
IANA specification at:
<https://www.iana.org/assignments/media-types/text/tab-separated-values>

## See also

[`utils::read.table()`](https://rdrr.io/r/utils/read.table.html),
[`utils::write.table()`](https://rdrr.io/r/utils/write.table.html)
