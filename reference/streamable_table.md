# streamable table

streamable table

## Usage

``` r
streamable_table(read, write, extension)
```

## Arguments

- read:

  read function. Arguments should be "`file`" (must be able to take a
  [`connection()`](https://rdrr.io/r/base/connections.html) object) and
  "`...`" (for) additional arguments.

- write:

  write function. Arguments should be "`data`" (a data.frame), `file`
  (must be able to take a
  [`connection()`](https://rdrr.io/r/base/connections.html) object), and
  "`omit_header`" logical, include header (initial write) or not (for
  appending subsequent chunks)

- extension:

  file extension to use (e.g. "tsv", "csv")

## Value

a `streamable_table` object (S3)

## Details

Note several constraints on this design. The write method must be able
to take a generic R `connection` object (which will allow it to handle
the compression methods used, if any), and the read method must be able
to take a `textConnection` object. `readr` functions handle these cases
out of the box, so the above method is easy to write. Also note that the
write method must be able to `omit_header`. See the built-in methods for
more examples.

## Examples

``` r

streamable_readr_tsv <- function() {
  streamable_table(
    function(file, ...) readr::read_tsv(file, ...),
    function(x, file, omit_header) {
      readr::write_tsv(x = x, file = file, append = omit_header)
    },
    "tsv"
  )
}
```
