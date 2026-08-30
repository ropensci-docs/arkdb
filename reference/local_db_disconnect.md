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
# }
```
