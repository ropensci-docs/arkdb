# process a table in chunks

process a table in chunks

## Usage

``` r
process_chunks(
  file,
  process_fn,
  streamable_table = NULL,
  lines = 50000L,
  encoding = Sys.getenv("encoding", "UTF-8"),
  ...
)
```

## Arguments

- file:

  path to a file

- process_fn:

  a function of a `chunk`

- streamable_table:

  interface for serializing/deserializing in chunks

- lines:

  number of lines to read in a chunk.

- encoding:

  encoding to be assumed for input files.

- ...:

  additional arguments to `streamable_table$read` method.

## Examples

``` r
con <- system.file("extdata/mtcars.tsv.gz", package = "arkdb")
dummy <- function(x) message(paste(dim(x), collapse = " x "))
process_chunks(con, dummy, lines = 8)
#> Importing /github/home/R/x86_64-pc-linux-gnu-library/4.6/arkdb/extdata/mtcars.tsv.gz in 8 line chunks:
#> 8 x 11
#> 8 x 11
#> 8 x 11
#> 8 x 11
#>  ...Done! (in 0.001754284 secs)
```
