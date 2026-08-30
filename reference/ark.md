# Archive tables from a database as flat files

Archive tables from a database as flat files

## Usage

``` r
ark(
  db_con,
  dir,
  streamable_table = streamable_base_tsv(),
  lines = 50000L,
  compress = c("bzip2", "gzip", "xz", "none"),
  tables = list_tables(db_con),
  method = c("keep-open", "window", "window-parallel", "sql-window"),
  overwrite = "ask",
  filter_statement = NULL,
  filenames = NULL,
  callback = NULL
)
```

## Arguments

- db_con:

  a database connection

- dir:

  a directory where we will write the compressed text files output

- streamable_table:

  interface for serializing/deserializing in chunks

- lines:

  the number of lines to use in each single chunk

- compress:

  file compression algorithm. Should be one of "bzip2" (default), "gzip"
  (faster write times, a bit less compression), "xz", or "none", for no
  compression.

- tables:

  a list of tables from the database that should be archived. By
  default, will archive all tables. Table list should specify schema if
  appropriate, see examples.

- method:

  method to use to query the database, see details.

- overwrite:

  should any existing text files of the same name be overwritten?
  default is "ask", which will ask for confirmation in an interactive
  session, and overwrite in a non-interactive script. TRUE will always
  overwrite, FALSE will always skip such tables.

- filter_statement:

  Typically an SQL "WHERE" clause, specific to your dataset. (e.g.,
  `WHERE year = 2013`)

- filenames:

  An optional vector of names that will be used to name the files
  instead of using the tablename from the `tables` parameter.

- callback:

  An optional function that acts on the data.frame before it is written
  to disk by `streamable_table`. It is recommended to use this on a
  single table at a time. Callback functions must return a data.frame.

## Value

the path to `dir` where output files are created (invisibly), for
piping.

## Details

`ark` will archive tables from a database as (compressed) tsv files. Or
other formats that have a `streamtable_table method`, like parquet.
`ark` does this by reading only chunks at a time into memory, allowing
it to process tables that would be too large to read into memory all at
once (which is probably why you are using a database in the first
place!) Compressed text files will likely take up much less space,
making them easier to store and transfer over networks. Compressed
plain-text files are also more archival friendly, as they rely on widely
available and long-established open source compression algorithms and
plain text, making them less vulnerable to loss by changes in database
technology and formats.

In almost all cases, the default method should be the best choice. If
the
[`DBI::dbSendQuery()`](https://dbi.r-dbi.org/reference/dbSendQuery.html)
implementation for your database platform returns the full results to
the client immediately rather than supporting chunking with `n`
parameter, you may want to use "window" method, which is the most
generic. The "sql-window" method provides a faster alternative for
databases like PostgreSQL that support windowing natively (i.e.
`BETWEEN` queries). Note that "window-parallel" only works with
`streamable_parquet`.

## Examples

``` r
# \donttest{
# setup
library(dplyr)
#> 
#> Attaching package: ‘dplyr’
#> The following objects are masked from ‘package:stats’:
#> 
#>     filter, lag
#> The following objects are masked from ‘package:base’:
#> 
#>     intersect, setdiff, setequal, union
dir <- tempdir()
db <- dbplyr::nycflights13_sqlite(tempdir())
#> Caching nycflights db at /tmp/RtmpIgkJzy/nycflights13.sqlite
#> Creating table: airlines
#> Creating table: airports
#> Creating table: flights
#> Creating table: planes
#> Creating table: weather

## And here we go:
ark(db, dir)
#> Exporting airlines in 50000 line chunks:
#>  ...Done! (in 0.001873255 secs)
#> Exporting airports in 50000 line chunks:
#>  ...Done! (in 0.005831242 secs)
#> Exporting flights in 50000 line chunks:
#>  ...Done! (in 3.057337 secs)
#> Exporting planes in 50000 line chunks:
#>  ...Done! (in 0.007766724 secs)
#> Exporting weather in 50000 line chunks:
#>  ...Done! (in 0.2138212 secs)
# }
if (FALSE) { # \dontrun{

## For a Postgres DB with schema, we can append schema names first
## to each of the table names, like so:
schema_tables <- dbGetQuery(db, sqlInterpolate(db,
  "SELECT table_name FROM information_schema.tables
WHERE table_schema = ?schema",
  schema = "schema_name"
))

ark(db, dir, tables = paste0("schema_name", ".", schema_tables$table_name))
} # }
```
