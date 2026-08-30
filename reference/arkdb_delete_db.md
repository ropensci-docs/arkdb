# delete the local arkdb database

delete the local arkdb database

## Usage

``` r
arkdb_delete_db(db_dir = arkdb_dir(), ask = interactive())
```

## Arguments

- db_dir:

  neon database location

- ask:

  Ask for confirmation first?

## Details

Just a helper function that deletes the database files. Usually
unnecessary but can be helpful in resetting a corrupt database.

## Examples

``` r

# Create a db
dir <- tempfile()
db <- local_db(dir)
#> duckdb keeps downloaded extensions and secrets in a temporary directory:
#> ℹ /tmp/RtmpIgkJzy/duckdb
#> This is removed when the R session ends.
#> • Extensions are re-downloaded each session.
#> • Secrets are lost.
#> ℹ Run duckdb(shared_home = TRUE) (or create ~/.duckdb) to keep them (suitable for most users).
#> ℹ Run duckdb(shared_home = FALSE) to accept the temporary directory (and silence this message).
#> ℹ See ?duckdb_storage for details and alternatives.

# Delete it
arkdb_delete_db(dir, ask = FALSE)
```
