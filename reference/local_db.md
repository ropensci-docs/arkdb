# Connect to a local stand-alone database

This function will provide a connection to the best available database.
This function is a drop-in replacement for `[DBI::dbConnect]` with
behaviour that makes it more subtle for R packages that need a database
backend with minimal complexity, as described in details.

## Usage

``` r
local_db(
  dbdir = arkdb_dir(),
  driver = Sys.getenv("ARKDB_DRIVER", "duckdb"),
  readonly = FALSE,
  cache_connection = TRUE,
  memory_limit = getOption("duckdb_memory_limit", NA),
  ...
)
```

## Arguments

- dbdir:

  Path to the database.

- driver:

  Default driver, one of "duckdb", "MonetDBLite", "RSQLite". It will
  select the first one of those it finds available if a driver is not
  set. This fallback can be overwritten either by explicit argument or
  by setting the environmental variable `ARKDB_DRIVER`.

- readonly:

  Should the database be opened read-only? (duckdb only). This allows
  multiple concurrent connections (e.g. from different R sessions)

- cache_connection:

  should we preserve a cache of the connection? allows faster load times
  and prevents connection from being garbage-collected. However, keeping
  open a read-write connection to duckdb or MonetDBLite will block
  access of other R sessions to the database.

- memory_limit:

  Set a memory limit for duckdb, in GB. This can also be set for the
  session by using options, e.g. `options(duckdb_memory_limit=10)` for a
  limit of 10GB. On most systems duckdb will automatically set a limit
  to 80% of machine capacity if not set explicitly.

- ...:

  additional arguments (not used at this time)

## Value

Returns a `[DBIconnection]` connection to the default database

## Details

This function provides several abstractions to `[DBI::dbConnect]` to
provide a seamless backend for use inside other R packages.

First, this provides a generic method that allows the use of a
``` [RSQLite::SQLite]`` connection if nothing else is available, while being able to automatically select a much faster, more powerful backend from  ```[duckdb::duckdb](https://r.duckdb.org/reference/duckdb.html)\`
if available. An argument or environmental variable can be used to
override this to manually set a database endpoint for testing purposes.

Second, this function will cache the database connection in an R
environment and load that cache. That means you can call `local_db()` as
fast/frequently as you like without causing errors that would occur by
rapid calls to `[DBI::dbConnect]`

Third, this function defaults to persistent storage location set by
`[tools::R_user_dir]` and configurable by setting the environmental
variable `ARKDB_HOME`. This allows a package to provide persistent
storage out-of-the-box, and easily switch that storage to a temporary
directory (e.g. for testing purposes, or custom user configuration)
without having to edit database calls directly.

## Examples

``` r
# \donttest{
## OPTIONAL: you can first set an alternative home location,
## such as a temporary directory:
Sys.setenv(ARKDB_HOME = tempdir())

## Connect to the database:
db <- local_db()
#> duckdb keeps downloaded extensions and secrets in a temporary directory:
#> ℹ /tmp/RtmpIgkJzy/duckdb
#> This is removed when the R session ends.
#> • Extensions are re-downloaded each session.
#> • Secrets are lost.
#> ℹ Run duckdb(shared_home = TRUE) (or create ~/.duckdb) to keep them (suitable for most users).
#> ℹ Run duckdb(shared_home = FALSE) to accept the temporary directory (and silence this message).
#> ℹ See ?duckdb_storage for details and alternatives.
# }
```
