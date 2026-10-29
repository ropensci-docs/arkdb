# arkdb: Archive and Unarchive Databases Using Flat Files

Flat text files provide a more robust, compressible, and portable way to
store tables. This package provides convenient functions for exporting
tables from relational database connections into compressed text files
and streaming those text files back into a database without requiring
the whole table to fit in working memory.

## Details

It has two functions:

- [`ark()`](https://docs.ropensci.org/arkdb/reference/ark.md): archive a
  database into flat files, chunk by chunk.

- [`unark()`](https://docs.ropensci.org/arkdb/reference/unark.md):
  Unarchive flat files back int a database connection.

arkdb will work with any `DBI` supported connection. This makes it a
convenient and robust way to migrate between different databases as
well.

## See also

Useful links:

- <https://github.com/ropensci/arkdb>

- <https://docs.ropensci.org/arkdb/>

- Report bugs at <https://github.com/ropensci/arkdb/issues>

## Author

**Maintainer**: Carl Boettiger <cboettig@gmail.com>
([ORCID](https://orcid.org/0000-0002-1642-628X)) \[copyright holder\]

Other contributors:

- Richard FitzJohn \[contributor\]

- Brandon Bertelsen <brandon@bertelsen.ca> \[contributor\]
