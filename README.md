
<!-- README.md is generated from README.Rmd. Please edit that file -->
RSQLServer
==========

[![CRAN](http://www.r-pkg.org/badges/version-ago/RSQLServer)](https://cran.r-project.org/package=RSQLServer) [![Travis-CI build status](https://travis-ci.org/imanuelcostigan/RSQLServer.svg?branch=master)](https://travis-ci.org/imanuelcostigan/RSQLServer) [![Appveyor build status](https://ci.appveyor.com/api/projects/status/muw348v007ja7dqf?svg=true)](https://ci.appveyor.com/project/imanuelcostigan/rsqlserver) [![Coverage status](https://codecov.io/gh/imanuelcostigan/RSQLServer/branch/master/graph/badge.svg)](https://codecov.io/gh/imanuelcostigan/RSQLServer)

An R package that provides a SQL Server R Database Interface ([DBI](https://github.com/rstats-db/DBI)), based on the cross-platform [jTDS JDBC driver](http://jtds.sourceforge.net/index.html).

Installation
------------

You can install the development version from GitHub:

``` r
# install.packages('devtools')
devtools::install_github('imanuelcostigan/RSQLServer')
```

And when the package is back on CRAN, you can install it the usual way:

``` r
install.packages("RSQLServer")
```

Config file
-----------

We recommend that you store server details and credentials in `~/sql.yaml`. This is partly so that you do not need to specify a username and password in calls to `dbConnect()`. But it is also because in testing, we've found that the jTDS single sign-on (SSO) library is a bit flaky. The contents of this file should look something like this:

``` yaml
SQL_PROD:
    server: 11.1.111.11
    type: &type sqlserver
    port: &port 1433
    domain: &domain companyname
    user: &user winusername
    password: &pass winpassword
    useNTLMv2: &ntlm true
SQL_DEV:
    server: 11.1.111.15
    type: *type
    port: *port
    domain: *domain
    user: *user
    password: *pass
    useNTLMv2: *ntlm
```

Usage
-----

Ensure that your `~/sql.yaml` file contains a valid SQL Server entry named `TEST`. In the following, the `TEST` server, generously provided by Microsoft for the purposes of this package's development, has a database containing band data sets.

### DBI usage

The following illustrates how you can make use of the DBI interface. Note that we **do not** attach the `RSQLServer` package.

``` r
library(DBI)
con <- dbConnect(RSQLServer::SQLServer(), server = "TEST", database = 'DBItest')
dbWriteTable(con, "band_members", dplyr::band_members)
dbWriteTable(con, "band_instruments", dplyr::band_instruments)
# RSQLServer only returns tables with type TABLE and VIEW.
dbListTables(con)
#> [1] "band_instruments" "band_members"
dbReadTable(con, 'band_members')
#>   name    band
#> 1 Mick  Stones
#> 2 John Beatles
#> 3 Paul Beatles
dbListFields(con, 'band_instruments')
#> [1] "name"  "plays"

# Fetch all results
res <- dbSendQuery(con, "SELECT * FROM band_members WHERE band = 'Beatles'")
dbFetch(res)
#>   name    band
#> 1 John Beatles
#> 2 Paul Beatles
dbClearResult(res)
#> [1] TRUE
```

### dplyr usage

The following illustrates how you can make use of the dplyr interface. Again, we **do not** attach the `RSQLServer` package.

``` r
library(dplyr, warn.conflicts = FALSE)
members <- tbl(con, "band_members")
instruments <- tbl(con, "band_instruments")
members %>% 
  left_join(instruments) %>% 
  filter(band == "Beatles")
#> Joining, by = "name"
#> # Source:   lazy query [?? x 3]
#> # Database: SQLServerConnection
#>    name    band  plays
#>   <chr>   <chr>  <chr>
#> 1  John Beatles guitar
#> 2  Paul Beatles   bass
collect(members)
#> # A tibble: 3 x 2
#>    name    band
#> * <chr>   <chr>
#> 1  Mick  Stones
#> 2  John Beatles
#> 3  Paul Beatles
```

Clean up

``` r
dbRemoveTable(con, "band_instruments")
#> [1] TRUE
dbRemoveTable(con, "band_members")
#> [1] TRUE
dbDisconnect(con)
#> [1] TRUE
```
