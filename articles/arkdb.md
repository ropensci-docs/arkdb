# Introduction to arkdb

## arkdb

### Package rationale

Increasing data sizes create challenges for the fundamental tasks of
publishing, distributing, and preserving data. Despite (or perhaps
because of) the diverse and ever-expanding number of database and file
formats, the humble plain text file such as comma or
tab-separated-values (e.g. `.csv` or `.tsv` files) remains the gold
standard for data archiving and distribution. These files can read on
almost any platform or tool and can be efficiently compressed using
long-standing and widely available standard open source libraries like
`gzip` or `bzip2`. In contrast, database storage formats and dumps are
usually particular to the database platform used to generate them, and
will likely not be compatible between different database engines
(e.g. PostgreSQL -\> SQLite) or even between different versions of the
same engine. Researchers unfamiliar with these databases will have
difficulty accessing such data, and these dumps may also be in formats
that are less efficient to compress.

Working with tables that are too large for working memory on most
machines by using external relational database stores is now a common R
practice, thanks to ever-rising availability of data and increasing
support and popularity of packages such as `DBI`, `dplyr`, and `dbplyr`.
Working with plain text files becomes increasingly difficult in this
context. Many R users will not have sufficient RAM to simply read in a
10 GB `.tsv` file into R. Similarly, moving a 10 GB database out of a
relational data file and into a plain text file for archiving and
distribution is similarly challenging from R. While most relational
database back-ends implement some form of `COPY` or `IMPORT` that allows
them to read in and export out plain text files directly, these methods
are not consistent across database types and not part of the standard
SQL interface. Most importantly for our case, they also cannot be called
directly from R, but require a separate stand-alone installation of the
database client. `arkdb` provides a simple solution to these two tasks.

The goal of `arkdb` is to provide a convenient way to move data from
large compressed text files (e.g. `*.tsv.bz2`) into any DBI-compliant
database connection (see
[DBI](https://solutions.rstudio.com/db/r-packages/DBI/)), and move
tables out of such databases into text files. The key feature of `arkdb`
is that files are moved between databases and text files in chunks of a
fixed size, allowing the package functions to work with tables that
would be much to large to read into memory all at once. This will be
slower than reading the file into memory at one go, but can be scaled to
larger data and larger data with no additional memory requirement.

### Installation

You can install `arkdb` from GitHub with:

``` r

# install.packages("devtools")
devtools::install_github("cboettig/arkdb")
```

## Tutorial

``` r

library(arkdb)

# additional libraries just for this demo
library(dbplyr)
library(dplyr)
library(nycflights13)
library(fs)
```

### Creating an archive of an existing database

First, we’ll need an example database to work with. Conveniently, there
is a nice example using the NYC flights data built into the `dbplyr`
package.

``` r

tmp <- tempdir() # Or can be your working directory, "."
db <- dbplyr::nycflights13_sqlite(tmp)
#> Caching nycflights db at /tmp/Rtmpae8pom/nycflights13.sqlite
#> Creating table: airlines
#> Creating table: airports
#> Creating table: flights
#> Creating table: planes
#> Creating table: weather
```

To create an archive, we just give `ark` the connection to the database
and tell it where we want the `*.tsv.bz2` files to be archived. We can
also set the chunk size as the number of `lines` read in a single chunk.
More lines per chunk usually means faster run time at the cost of higher
memory requirements.

``` r

dir <- fs::dir_create(fs::path(tmp, "nycflights"))
ark(db, dir, lines = 50000)
#> Exporting airlines in 50000 line chunks:
#>  ...Done! (in 0.002319813 secs)
#> Exporting airports in 50000 line chunks:
#>  ...Done! (in 0.006029844 secs)
#> Exporting flights in 50000 line chunks:
#>  ...Done! (in 3.07664 secs)
#> Exporting planes in 50000 line chunks:
#>  ...Done! (in 0.007812738 secs)
#> Exporting weather in 50000 line chunks:
#>  ...Done! (in 0.2146513 secs)
```

We can take a look and confirm the files have been written. Note that we
can use [`fs::dir_info`](https://fs.r-lib.org/reference/dir_ls.html) to
get a nice snapshot of the file sizes. Compare the compressed sizes to
the original database:

``` r

fs::dir_info(dir) %>% 
  select(path, size) %>%
  mutate(path = fs::path_file(path))
#> # A tibble: 5 × 2
#>   path                    size
#>   <chr>            <fs::bytes>
#> 1 airlines.tsv.bz2         260
#> 2 airports.tsv.bz2      28.13K
#> 3 flights.tsv.bz2        4.85M
#> 4 planes.tsv.bz2        11.96K
#> 5 weather.tsv.bz2      278.84K

fs::file_info(fs::path(tmp,"nycflights13.sqlite")) %>% 
  pull(size)
#> 44.9M
```

### Unarchive

Now that we’ve gotten all the database into (compressed) plain text
files, let’s get them back out. We simply need to pass `unark` a list of
these compressed files and a connection to the database. Here we create
a new local SQLite database. Note that this design means that it is also
easy to use `arkdb` to move data between databases.

``` r

files <- fs::dir_ls(dir, glob = "*.tsv.bz2")
new_db <- DBI::dbConnect(RSQLite::SQLite(), fs::path(tmp, "local.sqlite"))
```

As with `ark`, we can set the chunk size to control the memory footprint
required:

``` r

unark(files, new_db, lines = 50000)  
#> Importing /tmp/Rtmpae8pom/nycflights/airlines.tsv.bz2 in 50000 line chunks:
#>  ...Done! (in 0.003970623 secs)
#> Importing /tmp/Rtmpae8pom/nycflights/airports.tsv.bz2 in 50000 line chunks:
#>  ...Done! (in 0.007636547 secs)
#> Importing /tmp/Rtmpae8pom/nycflights/flights.tsv.bz2 in 50000 line chunks:
#>  ...Done! (in 2.626231 secs)
#> Importing /tmp/Rtmpae8pom/nycflights/planes.tsv.bz2 in 50000 line chunks:
#>  ...Done! (in 0.01145291 secs)
#> Importing /tmp/Rtmpae8pom/nycflights/weather.tsv.bz2 in 50000 line chunks:
#>  ...Done! (in 0.09117508 secs)
```

`unark` returns a `dplyr` database connection that we can use in the
usual way:

``` r

tbl(new_db, "flights")
#> # A query:  ?? x 19
#> # Database: sqlite 3.53.3 [/tmp/Rtmpae8pom/local.sqlite]
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
```

``` r

# Remove example files we created.
DBI::dbDisconnect(new_db)
unlink(dir, TRUE)
unlink(fs::path(tmp, "local.sqlite"))
```

### Pluggable text formats

By default, `arkdb` uses `tsv` format, implemented in base tools, as the
text-based serialization. The `tsv` standard is particularly attractive
because it side-steps some of the ambiguities present in the CSV format
due to string quoting. The [IANA Standard for
TSV](https://www.iana.org/assignments/media-types/text/tab-separated-values)
neatly avoids this for tab-separated values by insisting that a tab can
only ever be a separator.

`arkdb` provides a pluggable mechanism for changing the back end utility
used to write text files. For instance, if we need to read in or export
in `.csv` format, we can simply swap in a `csv` based reader in both
[`ark()`](https://docs.ropensci.org/arkdb/reference/ark.md) and
[`unark()`](https://docs.ropensci.org/arkdb/reference/unark.md) methods,
as illustrated here:

``` r

dir <- fs::dir_create(fs::path(tmp, "nycflights"))

ark(db, dir, 
    streamable_table = streamable_base_csv())
#> Exporting airlines in 50000 line chunks:
#>  ...Done! (in 0.0009803772 secs)
#> Exporting airports in 50000 line chunks:
#>  ...Done! (in 0.005463123 secs)
#> Exporting flights in 50000 line chunks:
#>  ...Done! (in 3.198302 secs)
#> Exporting planes in 50000 line chunks:
#>  ...Done! (in 0.008078575 secs)
#> Exporting weather in 50000 line chunks:
#>  ...Done! (in 0.2149503 secs)
```

``` r

files <- fs::dir_ls(dir, glob = "*.csv.bz2")
new_db <- DBI::dbConnect(RSQLite::SQLite(), fs::path(tmp, "local.sqlite"))

unark(files, new_db,
      streamable_table = streamable_base_csv())
#> Importing /tmp/Rtmpae8pom/nycflights/airlines.csv.bz2 in 50000 line chunks:
#>  ...Done! (in 0.004074335 secs)
#> Importing /tmp/Rtmpae8pom/nycflights/airports.csv.bz2 in 50000 line chunks:
#>  ...Done! (in 0.00784421 secs)
#> Importing /tmp/Rtmpae8pom/nycflights/flights.csv.bz2 in 50000 line chunks:
#>  ...Done! (in 2.610356 secs)
#> Importing /tmp/Rtmpae8pom/nycflights/planes.csv.bz2 in 50000 line chunks:
#>  ...Done! (in 0.01102924 secs)
#> Importing /tmp/Rtmpae8pom/nycflights/weather.csv.bz2 in 50000 line chunks:
#>  ...Done! (in 0.1389174 secs)
```

`arkdb` also provides the function
[`streamable_table()`](https://docs.ropensci.org/arkdb/reference/streamable_table.md)
to facilitate users creating their own streaming table interfaces. For
instance, if you would prefer to use `readr` methods to read and write
`tsv` files, we could construct the table as follows
([`streamable_readr_tsv()`](https://docs.ropensci.org/arkdb/reference/streamable_readr_tsv.md)
and
[`streamable_readr_csv()`](https://docs.ropensci.org/arkdb/reference/streamable_readr_csv.md)
are also shipped inside `arkdb` for convenience):

``` r

stream <- 
   streamable_table(
     function(file, ...) readr::read_tsv(file, ...),
     function(x, file, omit_header)
       readr::write_tsv(x = x, file = file, append = omit_header),
     "tsv")
```

and we can then pass such a streamable table directly to
[`ark()`](https://docs.ropensci.org/arkdb/reference/ark.md) and
[`unark()`](https://docs.ropensci.org/arkdb/reference/unark.md), like
so:

``` r

ark(db, dir, 
    streamable_table = stream)
#> Exporting airlines in 50000 line chunks:
#>  ...Done! (in 0.05984855 secs)
#> Exporting airports in 50000 line chunks:
#>  ...Done! (in 0.01993918 secs)
#> Exporting flights in 50000 line chunks:
#>  ...Done! (in 1.946624 secs)
#> Exporting planes in 50000 line chunks:
#>  ...Done! (in 0.03437257 secs)
#> Exporting weather in 50000 line chunks:
#>  ...Done! (in 0.1309748 secs)
```

Note several constraints on this design. The write method must be able
to take a generic R `connection` object (which will allow it to handle
the compression methods used, if any), and the read method must be able
to take a `textConnection` object. `readr` functions handle these cases
out of the box, so the above method is easy to write. Also note that the
write method must be able to `append`, i.e. it should use a header if
`append=TRUE`, but omit when it is `FALSE`. See the built-in methods for
more examples.

### A note on compression

`unark` can read from a variety of compression formats recognized by
base R: `bzip2`, `gzip`, `zip`, and `xz`, and `ark` can choose any of
these as the compression algorithm. Note that there is some trade-off
between speed of compression and efficiency (i.e. the final file size).
`ark` uses the `bz2` compression algorithm by default, supported in base
R, to compress `tsv` files. The `bz2` offers excellent compression
levels, but is considerably slower to compress than `gzip` or `zip`. It
is comparably fast to uncompress. For faster archiving when maximum file
size reduction is not critical, `gzip` will give nearly as effective
compression in significantly less time. Compression can also be turned
off, e.g. by using
[`ark()`](https://docs.ropensci.org/arkdb/reference/ark.md) with
`compress="none"` and
[`unark()`](https://docs.ropensci.org/arkdb/reference/unark.md) with
files that have no compression suffix (e.g. `*.tsv` instead of
`*.tsv.gz`).

### Distributing data

Once you have archived your database files with `ark`, consider sharing
them privately or publicly as part of your project GitHub repo using the
[`piggyback` R package](https://github.com/ropensci/piggyback). For more
permanent, versioned, and citable data archiving, upload your
`*.tsv.bz2` files to a data repository like
[Zenodo.org](https://zenodo.org).
