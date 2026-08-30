# Unarchive a list of compressed tsv files into a database

Unarchive a list of compressed tsv files into a database

## Usage

``` r
unark(
  files,
  db_con,
  streamable_table = NULL,
  lines = 50000L,
  overwrite = "ask",
  encoding = Sys.getenv("encoding", "UTF-8"),
  tablenames = NULL,
  try_native = TRUE,
  ...
)
```

## Arguments

- files:

  vector of filenames to be read in. Must be `tsv` format, optionally
  compressed using `bzip2`, `gzip`, `zip`, or `xz` format at present.

- db_con:

  a database src (`src_dbi` object from `dplyr`)

- streamable_table:

  interface for serializing/deserializing in chunks

- lines:

  number of lines to read in a chunk.

- overwrite:

  should any existing text files of the same name be overwritten?
  default is "ask", which will ask for confirmation in an interactive
  session, and overwrite in a non-interactive script. TRUE will always
  overwrite, FALSE will always skip such tables.

- encoding:

  encoding to be assumed for input files.

- tablenames:

  vector of tablenames to be used for corresponding files. By default,
  tables will be named using lowercase names from file basename with
  special characters replaced with underscores (for SQL compatibility).

- try_native:

  logical, default TRUE. Should we try to use a native bulk import
  method for the database connection? This can substantially speed up
  read times and will fall back on the DBI method for any table that
  fails to import. Currently only MonetDBLite connections support this.

- ...:

  additional arguments to `streamable_table$read` method.

## Value

the database connection (invisibly)

## Details

`unark` will read in a files in chunks and write them into a database.
This is essential for processing large compressed tables which may be
too large to read into memory before writing into a database. In
general, increasing the `lines` parameter will result in a faster total
transfer but require more free memory for working with these larger
chunks.

If using `readr`-based streamable-table, you can suppress the progress
bar by using `options(readr.show_progress = FALSE)` when reading in
large files.

## Examples

``` r
# \donttest{
## Setup: create an archive.
library(dplyr)
dir <- tempdir()
db <- dbplyr::nycflights13_sqlite(tempdir())

## database -> .tsv.bz2
ark(db, dir)
#> Warning: overwriting airlines.tsv.bz2
#> Exporting airlines in 50000 line chunks:
#>  ...Done! (in 0.001210928 secs)
#> Warning: overwriting airports.tsv.bz2
#> Exporting airports in 50000 line chunks:
#>  ...Done! (in 0.005736589 secs)
#> Warning: overwriting flights.tsv.bz2
#> Exporting flights in 50000 line chunks:
#>  ...Done! (in 3.045362 secs)
#> Warning: overwriting planes.tsv.bz2
#> Exporting planes in 50000 line chunks:
#>  ...Done! (in 0.007821798 secs)
#> Warning: overwriting weather.tsv.bz2
#> Exporting weather in 50000 line chunks:
#>  ...Done! (in 0.2110779 secs)

## list all files in archive (full paths)
files <- list.files(dir, "bz2$", full.names = TRUE)

## Read archived files into a new database (another sqlite in this case)
new_db <- DBI::dbConnect(RSQLite::SQLite())
unark(files, new_db)
#> Importing /tmp/RtmpIgkJzy/airlines.tsv.bz2 in 50000 line chunks:
#>  ...Done! (in 0.003885269 secs)
#> Importing /tmp/RtmpIgkJzy/airports.tsv.bz2 in 50000 line chunks:
#>  ...Done! (in 0.007760048 secs)
#> Importing /tmp/RtmpIgkJzy/flights.tsv.bz2 in 50000 line chunks:
#>  ...Done! (in 2.66075 secs)
#> Importing /tmp/RtmpIgkJzy/planes.tsv.bz2 in 50000 line chunks:
#>  ...Done! (in 0.01149559 secs)
#> Importing /tmp/RtmpIgkJzy/weather.tsv.bz2 in 50000 line chunks:
#>  ...Done! (in 0.08750725 secs)

## Prove table is returned successfully.
tbl(new_db, "flights")
#> # A query:  ?? x 19
#> # Database: sqlite 3.53.3 []
#>     year month   day dep_time sched_dep_time dep_delay arr_time sched_arr_time
#>    <int> <int> <int>    <int>          <int>     <int>    <int>          <int>
#>  1  2013     1     1      517            515         2      830            819
#>  2  2013     1     1      533            529         4      850            830
#>  3  2013     1     1      542            540         2      923            850
#>  4  2013     1     1      544            545        -1     1004           1022
#>  5  2013     1     1      554            600        -6      812            837
#>  6  2013     1     1      554            558        -4      740            728
#>  7  2013     1     1      555            600        -5      913            854
#>  8  2013     1     1      557            600        -3      709            723
#>  9  2013     1     1      557            600        -3      838            846
#> 10  2013     1     1      558            600        -2      753            745
#> # ℹ more rows
#> # ℹ 11 more variables: arr_delay <int>, carrier <chr>, flight <int>,
#> #   tailnum <chr>, origin <chr>, dest <chr>, air_time <int>, distance <int>,
#> #   hour <int>, minute <int>, time_hour <dbl>
# }
```
