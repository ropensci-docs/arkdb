# streamable chunked parquet using `arrow`

streamable chunked parquet using `arrow`

## Usage

``` r
streamable_parquet()
```

## Value

a `streamable_table` object (S3)

## Details

Parquet files are streamed to disk by breaking them into chunks that are
equal to the `nlines` parameter in the initial call to `ark`. For each
`tablename`, a folder is created and the chunks are placed in the folder
in the form `part-000000.parquet`. The software looks at the folder, and
increments the name appropriately for the next chunk. This is done
intentionally so that users can take advantage of
[`arrow::open_dataset`](https://arrow.apache.org/docs/r/reference/open_dataset.html)
in the future, when coming back to review or perform analysis of these
data.

## See also

[`arrow::read_parquet()`](https://arrow.apache.org/docs/r/reference/read_parquet.html),
[`arrow::write_parquet()`](https://arrow.apache.org/docs/r/reference/write_parquet.html)
