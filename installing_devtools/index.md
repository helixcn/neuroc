# Installing devtools
John Muschelli  
`r Sys.Date()`  

All code for this document is located at [here](https://raw.githubusercontent.com/muschellij2/neuroc/master/installing_devtools/index.R).

# First Pass

Overall, RStudio provides a fantastic tutorial and discussion on [installing devtools](https://www.rstudio.com/products/rpackages/devtools/).  Please consult this before the rest of the document.  If you have errors, please see below.



As Neuroconductor is GitHub-based, we will need a way for R to install packages directly from GitHub.  The `devtools` package provides this functionality.  In this tutorial, we will go through the steps of installing `devtools`, and some common problems.  You must have `devtools` installed to install from GitHub in subsequent tutorials on installing Neuroconductor packages.

There are other packages that will do this and are more lightweight (see `remotes` and `ghit`), but we will focus on `devtools`. 


# Mac OSX

You need to install [Command Line Tools](https://developer.apple.com/library/content/technotes/tn2339/_index.html), aka the command line tools for Xcode, if you have not already.  [http://osxdaily.com/2014/02/12/install-command-line-tools-mac-os-x/](http://osxdaily.com/2014/02/12/install-command-line-tools-mac-os-x/) is a great tutorial how.

# Installing devtools

If you already have `devtools` installed great! (Why are you in this section?)  You can always reinstall the most up to date version from the steps below.


```r
packages = installed.packages()
packages = packages[, "Package"]
if (!"devtools" %in% packages) {
  install.packages("devtools")
}
```

# The `remotes` and `ghit` packages
If you want a lighter-weight package that has the `install_github` functionality that `devtools` provides, but not all the "development" parts of `devtools`, the `remotes` package exists just for that:


```r
packages = installed.packages()
packages = packages[, "Package"]
if (!"remotes" %in% packages) {
  install.packages("remotes")
}
```

The `ghit` package is the lightest-weight package I have seen which has a `install_github` function, but may have some limited functionality compared to `remotes` in the functionality of installing package with dependencies in other systems, such as BitBucket.

In any subsequent tutorial, when you see `devtools::install_github`, just insert `remotes::install_github` and it should work just the same.


# Updating a package

In the `install_github` function, there are additional options to pass to the `install` function from `devtools`.  One of those arguments is `upgrade_dependencies`, which default is set to `TRUE`.  So if you want to install a package from GitHub, but not update any of the dependencies, then you can use `install_github(..., upgrade_dependencies = FALSE)`.  

# Troubleshooting errors 

## git2r dependency in devtools

If you cannot install `devtools`, many times it is due to `git2r`.  You should look at the installation logs and if you see something like:

```
   The OpenSSL library that is required to
   build git2r was not found.

   Please install:
libssl-dev    (package on e.g. Debian and Ubuntu)
openssl-devel (package on e.g. Fedora, CentOS and RHEL)
openssl       (Homebrew package on OS X)
```

Then run `sudo apt-get libssl-dev` or `sudo yum install openssl-devel` on your respective Linux machine.  Try to re-install `devtools`.

### Mac OSX

For Mac, you have to [install Homebrew](http://www.howtogeek.com/211541/homebrew-for-os-x-easily-installs-desktop-apps-and-terminal-utilities/) which the tutorial is located in the link.  After Homebrew is installed you should be able to type in the Terminal:
```
brew update
brew install openssl
```
Then try to re-install `devtools`.

# Session Info


```r
devtools::session_info()
```

```
## Session info -------------------------------------------------------------
```

```
##  setting  value                       
##  version  R version 3.3.2 (2016-10-31)
##  system   x86_64, darwin13.4.0        
##  ui       X11                         
##  language (EN)                        
##  collate  en_US.UTF-8                 
##  tz       America/New_York            
##  date     2017-02-16
```

```
## Packages -----------------------------------------------------------------
```

```
##  package   * version     date       source                            
##  backports   1.0.5       2017-01-18 cran (@1.0.5)                     
##  colorout  * 1.1-0       2015-04-20 Github (jalvesaq/colorout@1539f1f)
##  devtools    1.12.0.9000 2017-01-23 Github (hadley/devtools@1ce84b0)  
##  digest      0.6.12      2017-01-27 cran (@0.6.12)                    
##  evaluate    0.10        2016-10-11 CRAN (R 3.3.0)                    
##  htmltools   0.3.6       2016-12-08 Github (rstudio/htmltools@4fbf990)
##  knitr       1.15.1      2016-11-22 cran (@1.15.1)                    
##  magrittr    1.5         2014-11-22 CRAN (R 3.2.0)                    
##  memoise     1.0.0       2016-01-29 CRAN (R 3.2.3)                    
##  pkgbuild    0.0.0.9000  2016-12-08 Github (r-pkgs/pkgbuild@65eace0)  
##  pkgload     0.0.0.9000  2016-12-08 Github (r-pkgs/pkgload@def2b10)   
##  Rcpp        0.12.8.2    2016-12-08 Github (RcppCore/Rcpp@8c7246e)    
##  rmarkdown   1.3         2017-01-03 Github (rstudio/rmarkdown@3276760)
##  rprojroot   1.2         2017-01-16 cran (@1.2)                       
##  stringi     1.1.2       2016-10-01 CRAN (R 3.3.0)                    
##  stringr     1.1.0       2016-08-19 cran (@1.1.0)                     
##  withr       1.0.2       2016-06-20 CRAN (R 3.3.0)                    
##  yaml        2.1.14      2016-11-12 CRAN (R 3.3.2)
```

