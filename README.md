# Tiny guide for writing R packages

## Writing R package

If you search online about writing R package, then it is easy to get impression that writing R package is a complex process, that requires multiple other packages to be installed and learned.

Writing R packages is actually astounding straightforward and doesn't require any extra packages.
All you need for a minimal package is:

1. `DESCRIPTION` file describing your package
2. `NAMESPACE` file specifying imported packages/functions and exported functions
3. `R/` directory storing R functions
4. `man/` directory storing manuals of functions

Alternatively you can get package template structure populated by base R function `package.skeleton`. I usually find it easier to take an existing package, copy and edit `DESCRIPTION`, `NAMESPACE`, and a single `Rd` file from `man` directory. I don't have to remember exact fields/functions names.

### `DESCRIPTION` file

- Use `Imports:` field to specify other packages that your package is using. It is a good practice to use as few dependencies as necessary. More on the subject can be found at [tinyverse.org](https://www.tinyverse.org).

- If some functionality of your package is not essential, and requires other packages, then those packages can be listed in `Suggests:` field. This means that those packages don't have to be installed together with your package, but attempt of using a functionality without suggested package will raise an error. Note that code using suggested packages must be escaped, for example like this
```R
if (requireNamespace("other.pkg", quietly=TRUE)) {
  other.pkg::other.fun()
} else stop("this function requires 'other.pkg' package, install and retry")
```

### `NAMESPACE` file

File defines interfaces to other packages and its users.

#### import

Defining an import brings functions from external packages to scope of your package.
You can either bring all functions from a package:
```
import(data.table)
```
Or bring only selected functions:
```
importFrom(data.table, fread, fwrite)
```

#### export

Exporting defines functions of your package that are meant to be used by its users, or other packages.
```
export(hello, anError)
```

If you want your function to act as a method, then you need export it using `S3method` rather than `export`
```
S3method("print", myclass1)
S3method("as.data.frame", myclass1)
```

It is good practice to, whenever possible, avoid breaking changes in exported functions.

### `R/`

This directory contains R scripts. In most cases you keep only definition of R functions there.

```R
hello = function() "world"
anError = function() stop("here it is")
```

### `man/`

Each exported function should have manual. Internal (non-exported) functions may have manual as well, but those are not required by a package check.
```Rd
\name{hello}
\alias{hello}
\title{ Prints 'world' string }
\description{
  Description on hello function.
}
\usage{
  hello()
}
\value{
  Scalar character \code{"world"}.
}
\seealso{\code{\link{anError}}}
\examples{
hello()
}
\keyword{ data }
```

It is easiest to re-use existing manual file as a template.

## Building, checking and installing package

Our package is ready.

We can build package tarballs (`tar.gz` file) which is the standard format to distribute R packages.

```sh
R CMD build .
```

To check the package, we use tarballs rather than a path to directory.

```sh
R CMD check mypkg_1.0.0.tar.gz
```

To install package

```sh
R CMD INSTALL mypkg_1.0.0.tar.gz
```

or from R
```R
install.packages("mypkg_1.0.0.tar.gz")
```

Note that this won't automatically install dependencies required by your package. On the machine where you are developing the package those dependencies are most likely already installed. If they aren't, and if there are many dependencies, or dependencies have their own dependencies, then an automatic way to install them may be needed. In base R this can be solved in a way described in ["Install a local R package with dependencies from CRAN mirror"](https://stackoverflow.com/a/74006901/2490497), otherwise third party packages can be used to achieve the same.

## Extras

### `tests/` for R test scripts

Package may include tests, those are executed during `R CMD check`. Tests should be defined as R scripts and placed into `tests/` directory. They should not raise an error, as this is the condition that signals that a test failed.

```R
stopifnot(
  identical(hello(), "world")
)
```

If you want to tests that a function returned an error, you should catch it.

```R
stopifnot(
  inherits(e<-try(anError(), silent=TRUE), "try-error"),
  identical(attr(e, "condition")$message, "here it is")
)
```

### `src/` for storing compiled code

If you want to use compiled code like C/C++/Fortran then the code of those should go to `src` directory. In such case you also need to use `useDynLib` call in `NAMESPACE` file and an `src/init.c` file.

## More

For more see official R manual [Writing R Extensions](https://cran.r-project.org/doc/manuals/r-release/R-exts.html).
