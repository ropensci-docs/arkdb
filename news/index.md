# Changelog

## arkdb 0.0.19

CRAN release: 2026-02-09

- patch tsv read

## arkdb 0.0.18

CRAN release: 2024-01-15

- patch test infrastructure for handling soft dependency on arrow

## arkdb 0.0.17

CRAN release: 2024-01-08

- patch test infrastructure for Windows

## arkdb 0.0.16

CRAN release: 2022-09-09

- patch for
  [`local_db()`](https://docs.ropensci.org/arkdb/reference/local_db.md)
  by defaulting path to subdir.
- update roxygen

## arkdb 0.0.15

CRAN release: 2022-02-15

- Added window-parallel option for ark’ing large tables in parallel
- More conditional testing on M1/arm Mac

## arkdb 0.0.14

CRAN release: 2021-10-18

- Patch for test suite for Solaris. `arrow` package installs on Solaris,
  but functions do not actually run correctly since the C++ libraries
  have not been set up properly on Solaris.

## arkdb 0.0.13

CRAN release: 2021-10-15

- Added ability to name output files directly.
- Add warning when users specify compression for parquet files.
- Added callback functionality to the `ark` function. Allowing users to
  perform transformations or recodes before chunked data.frames are
  saved to disk.
- Added ability to filter databases by allowing users to specify a
  “WHERE” clause.
- Added parquet as an streamable_table format, allowing users to `ark`
  to parquet instead of a text format.

## arkdb 0.0.12

CRAN release: 2021-04-05

- Bugfix for arkdb

## arkdb 0.0.11

CRAN release: 2021-03-13

- Make cached connection opt-out instead of applying only to read_only.
  This allows cache to work on read-write connections by default. This
  also avoids the condition of a connection being garbage-collected when
  functions call local_db internally.

## arkdb 0.0.10

CRAN release: 2021-03-05

- Better handling of read_only vs read_write connections. Only caches
  read_only connections.  
- Includes optional support for MonetDBLite

## arkdb 0.0.8

CRAN release: 2020-11-04

- Bugfix for dplyr 2.0.0 release

## arkdb 0.0.7

CRAN release: 2020-10-15

- Bugfix for upcoming dplyr 2.0.0 release

## arkdb 0.0.6

CRAN release: 2020-09-18

- Support vroom as an opt-in streamable table
- Export `process_chunks`
- Add mechanism to attempt a bulk importer, when available
  ([\#27](https://github.com/ropensci/arkdb/issues/27))
- Bugfix for case when text contains `#` characters in base parser
  ([\#28](https://github.com/ropensci/arkdb/issues/28))
- Lighten core dependencies. Fully recursive dependencies include only 4
  non-base packages now, as `progress` is now optional.
- Use “magic numbers” instead of extensions to guess compression type.
  (NOTE: requires that file is local and not a URL)
- Now that `duckdb` is on CRAN and `MonetDBLite` isn’t, drop built-in
  support for `MonetDBLite` in favor of `duckdb` alone.

## arkdb 0.0.5 2018-10-31

CRAN release: 2018-10-31

- [`ark()`](https://docs.ropensci.org/arkdb/reference/ark.md)’s default
  `keep-open` method would cut off header names for Postgres connections
  (due to variation in the behavior of SQL queries with `LIMIT 0`.) The
  issue is now resolved by accessing the header in a more robust,
  general way.

## arkdb 0.0.4 2018-09-27

CRAN release: 2018-09-27

- [`unark()`](https://docs.ropensci.org/arkdb/reference/unark.md) will
  strip out non-compliant characters in table names by default.
- [`unark()`](https://docs.ropensci.org/arkdb/reference/unark.md) gains
  the optional argument `tablenames`, allowing the user to specify the
  corresponding table names manually, rather than enforcing they
  correspond with the incoming file names.
  [\#18](https://github.com/ropensci/arkdb/issues/18)
- [`unark()`](https://docs.ropensci.org/arkdb/reference/unark.md) gains
  the argument `encoding`, allowing users to directly set the encoding
  of incoming files. Previously this could only be set by setting
  `options(encoding)`, which will still work as well. See `FAO.R`
  example in `examples` for an illustration.  
- [`unark()`](https://docs.ropensci.org/arkdb/reference/unark.md) will
  now attempt to guess which streaming parser to use (e.g `csv` or
  `tsv`) based on the file extension pattern, rather than defaulting to
  a `tsv` parser.
  ([`ark()`](https://docs.ropensci.org/arkdb/reference/ark.md) still
  defaults to exporting in the more portable `tsv` format).

## arkdb 0.0.3 2018-09-11

CRAN release: 2018-09-11

- Remove dependency on utils::askYesNo for backward compatibility,
  [\#17](https://github.com/ropensci/arkdb/issues/17)

## arkdb 0.0.2 2018-08-20 (First release to CRAN)

CRAN release: 2018-08-20

- Ensure the suggested dependency MonetDBLite is available before
  running unit test using it.

## arkdb 0.0.1 2018-08-20

CRAN release: 2018-08-20

- Overwrite existing tables of same name (with warning and interactive
  proceed) in both DB and text-files to avoid appending.

## arkdb 0.0.0.9000 2018-08-11

- Added a `NEWS.md` file to track changes to the package.
- Log messages improved as suggested by
  [@richfitz](https://github.com/richfitz)
- Improved mechanism for windowing in most DBs, from
  [@krlmlr](https://github.com/krlmlr)
  [\#8](https://github.com/ropensci/arkdb/pull/8)
- Support pluggable I/O, based on
  [@richfitz](https://github.com/richfitz) suggestions
  [\#3](https://github.com/ropensci/arkdb/issues/3),
  [\#10](https://github.com/ropensci/arkdb/pull/10)
