# Disconnect from the arkdb database.

Disconnect from the arkdb database.

## Usage

``` r
local_db_disconnect(db = local_db(), env = arkdb_cache)
```

## Arguments

- db:

  a DBI connection. By default, will call
  [local_db](https://docs.ropensci.org/arkdb/reference/local_db.md) for
  the default connection.

- env:

  The environment where the function looks for a connection.

## Details

This function manually closes a connection to the `arkdb` database.

## Examples

``` r
# \donttest{

## Disconnect from the database:
local_db_disconnect()
#> duckdb keeps downloaded extensions and secrets in a temporary directory:
#> ℹ /tmp/RtmpfR5yUs/duckdb
#> This is removed when the R session ends.
#> • Extensions are re-downloaded each session.
#> • Secrets are lost.
#> ℹ Run duckdb(shared_home = TRUE) (or create ~/.duckdb) to keep them (suitable for most users).
#> ℹ Run duckdb(shared_home = FALSE) to accept the temporary directory (and silence this message).
#> ℹ See ?duckdb_storage for details and alternatives.
#> Error in dbConnect(dbdir, read_only, bigint, config): IO Error: Could not read from file "/tmp/RtmpfR5yUs/duckdb": Is a directory
#> ℹ Context: rapi_startup
#> ℹ Error type: IO
# }
```
