# rio: A Swiss-army knife for data I/O #

The aim of **rio** is to make data file I/O in R as easy as possible by implementing three simple functions in Swiss-army knife style:

 - `export()` and `import()` provide a painless data I/O experience by automatically choosing the appropriate data read or write function based on file extension
 - `convert()` wraps `import()` and `export()` to allow the user to easily convert between file formats (thus providing a FOSS replacement for programs like [Stat/Transfer](https://www.stattransfer.com/) or [Sledgehammer](http://www.openmetadata.org/site/?page_id=1089)). [Luca Braglia](https://lbraglia.github.io/) has created a Shiny app called [rioweb](https://github.com/lbraglia/rioweb) that provides access to the file conversion features of rio.

## Supported file formats ##

**rio** supports a variety of different file formats for import and export.

| Format | Import | Export |
| ------ | ------ | ------ |
| Tab-separated data (.tsv) | Yes | Yes |
| Comma-separated data (.csv) | Yes | Yes |
| CSVY (CSV + YAML metadata header) (.csvy) | Yes | Yes |
| Pipe-separated data (.psv) | Yes | Yes |
| Fixed-width format data (.fwf) | Yes | Yes |
| Serialized R objects (.rds) | Yes | Yes |
| Saved R objects (.RData) | Yes | Yes |
| JSON (.json) | Yes | Yes |
| Stata (.dta) | Yes | Yes |
| SPSS and SPSS portable | Yes (.sav and .por) | Yes (.sav only) |
| "XBASE" database files (.dbf) | Yes | Yes |
| Excel (.xls) | Yes |  |
| Excel (.xlsx) | Yes | Yes |
| Weka Attribute-Relation File Format (.arff) | Yes | Yes |
| R syntax (.R) | Yes | Yes |
| Shallow XML documents (.xml) | Yes | Yes |
| SAS and SAS XPORT | Yes (.sas7bdat and .xpt) |  |
| Minitab (.mtp) | Yes |  |
| Epiinfo (.rec) | Yes |  |
| Systat (.syd) | Yes |  |
| Data Interchange Format (.dif) | Yes |  |
| OpenDocument Spreadsheet  (.ods) | Yes |  |
| Fortran data (no recognized extension) | Yes |  |
| [Google Sheets](https://www.google.com/sheets/about/) | Yes |  |
| Clipboard (default is tsv) | Yes (Mac and Windows) | Yes (Mac and Windows) |

Additionally, any format that is not supported by **rio** but that has a known R implementation will produce an informative error message pointing to a package and import or export function. Unrecognized formats will yield a simple "Unrecognized file format" error.

## Examples ##

Because **rio** is meant to streamline data I/O, the package is extremely easy to use. Here are some examples of reading, writing, and converting data files.

### Export ###

Exporting data is handled with one function, `export()`:


```r
library("rio")

export(mtcars, "mtcars.csv") # comma-separated values
export(mtcars, "mtcars.rds") # R serialized
export(mtcars, "mtcars.sav") # SPSS
```

### Import ###

Importing data is handled with one function, `import()`:


```r
x <- import("mtcars.csv")
```

```
## [1] "mtcars.csv"
## attr(,"class")
## [1] "rio_csv"
```

```r
y <- import("mtcars.rds")
z <- import("mtcars.sav")

# confirm data match
all.equal(x, y, check.attributes = FALSE)
```

```
## [1] TRUE
```

```r
all.equal(x, z, check.attributes = FALSE)
```

```
## [1] TRUE
```

Note: Because of inconsistencies across underlying packages, the data.frame returned by `import` might vary slightly (in variable classes and attributes) depending on file type.

### Convert ###

The `convert()` function links `import()` and `export()` by constructing a dataframe from the imported file and immediately writing it back to disk. `convert()` invisibly returns the file name of the exported file, so that it can be used to programmatically access the new file.


```r
convert("mtcars.sav", "mtcars.dta")
```

It is also possible to use **rio** on the command-line by calling `Rscript` with the `-e` (expression) argument. For example, to convert a file from Stata (.dta) to comma-separated values (.csv), simply do the following:

```
Rscript -e "rio::convert('iris.dta', 'iris.csv')"
```



## Package Philosophy ##

The core advantage of **rio** is that it makes assumptions that the user is probably willing to make. Seven of these are important:

 1. **rio** uses the file extension of a file name to determine what kind of file it is. This is the same logic used by Windows OS, for example, in determining what application is associated with a given file type. By taking away the need to manually match a file type (which a beginner may not recognize) to a particular import or export function, **rio** allows almost all common data formats to be read with the same function. Other packages do this as well, but rio aims to be more complete and more consistent than each:
 
   - [**reader**](http://cran.r-project.org/web/packages/reader/index.html) handles certain text formats and R binary files
   - [**io**](http://cran.r-project.org/web/packages/io/index.html) offers a set of custom formats
   - [**SchemaOnRead**](https://cran.r-project.org/web/packages/SchemaOnRead/index.html) iterates through a large number of possible import methods until one works successfully
   
 2. **rio** uses uses `data.table::fread()` for text-delimited files to automatically determine the file format regardless of the extension. So, a CSV that is actually tab-separated will still be correctly imported.
 
 3. **rio**, wherever possible, does not import character strings as factors.
 
 4. **rio** supports web-based imports from SSL (HTTPS) URLs, from shortened URLs, and from URLs that lack proper extensions.
 
 5. **rio** imports from from single-file .zip and .tar archives automatically, without the need to explicitly decompress them.
 
 6. **rio** imports and exports files based on an internal S3 class infrastructure. This means that other packages can contain extensions to **rio** by importing `import()` and/or `export()` and then writing internal S3 methods. These methods should take the form `.import.rio_X()` and `.export.rio_X()`, where `X` is the file extension of a file type.
 
 7. **rio** wraps a variety of faster, more stream-lined I/O packages than those provided by base R or the **foreign** package. It uses [**haven**](https://github.com/hadley/haven) for reading and writing SAS, Stata, and SPSS files, [the `fread` function from **data.table**](https://github.com/Rdatatable/data.table) for delimited formats, smarter and faster fixed-width file import and export routines, and [**readxl**](https://github.com/hadley/readxl) for reading from Excel workbooks.

## Package Installation ##

The package is available on [CRAN](http://cran.r-project.org/web/packages/rio/) and can be installed directly in R using:

```R
install.packages("rio")
```

The latest development version on GitHub can be installed using **devtools**:

```R
if(!require("ghit")){
    install.packages("ghit")
}
ghit::install_github("leeper/rio")
```

[![CRAN Version](http://www.r-pkg.org/badges/version/rio)](http://cran.r-project.org/package=rio)
![Downloads](http://cranlogs.r-pkg.org/badges/rio)
[![Travis-CI Build Status](https://travis-ci.org/leeper/rio.png?branch=master)](https://travis-ci.org/leeper/rio)
[![Appveyor Build status](https://ci.appveyor.com/api/projects/status/40ua5l06jw0gjyjb?svg=true)](https://ci.appveyor.com/project/leeper/rio)
[![codecov.io](http://codecov.io/github/leeper/rio/coverage.svg?branch=master)](http://codecov.io/github/leeper/rio?branch=master)

