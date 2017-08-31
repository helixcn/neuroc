




All code for this document is located at [here](https://raw.githubusercontent.com/muschellij2/neuroc/master/install_oslerinhealth/index.R).

# Installing OSLERinHealth Packages 

[Install and start the latest release version of R](#installing-and-starting-r).  Although the installer will try to download and install `devtools`, there may be some system requirements for `devtools` that you may need before going forward.  Please visit [installing devtools](../installing_devtools/index.html) before going forward if you do not have `devtools` currently installed. 

Then, you can install a package using the following command:

```r
## try http:// if https:// URLs are supported
source("http://oslerinhealth.org/oslerLite.R")
osler_install("PACKAGE")
```
where `PACKAGE` is the name of the package you'd like to install, such as `fslr`.  For example, if we want to install `neurohcp` and `fslr` we can run:
```r
source("http://oslerinhealth.org/oslerLite.R")
osler_install(c("fslr", "neurohcp"))
```
### `oslerLite`: an alias for `osler_install`

As Bioconductor uses the `biocLite` function to install packages, we have created a duplicate of `osler_install`, called `oslerLite`, for ease of use for those users accustomed to Bioconductor.  The same command could have been executed as follows:
```r
source("http://oslerinhealth.org/oslerLite.R")
oslerLite(c("fslr", "neurohcp"))
```

### Installing the `oslerInstall` package

The `oslerInstall` package contains the `oslerLite`/`osler_install` functions, as well as others relevant for OSLERinHealth.  You can install the package as follows:

```r
source("http://oslerinhealth.org/oslerLite.R")
osler_install("oslerInstall")
```

After installation, you can use `` oslerInstall::oslerLite() `` to install packages without source-ing the URL above.

## Installing OSLERinHealth Packages without upgrading dependencies

The `oslerLite`/`osler_install` functions depend on `devtools::install_github`, which will upgrade dependencies by default, which is recommended.  If you would like to install a package, but not upgrade the dependencies (missing dependencies will still be installed), you can set the `upgrade_dependencies` argument to `FALSE`:

```r
oslerLite(c("fslr", "neurohcp"), upgrade_dependencies = FALSE)
```

# Installing and starting R 

1.  Download the most recent version of R from [https://cran.r-project.org/](https://cran.r-project.org/). There are detailed instructions on the R website as well as the specific R installation for the platform you are using, typically Linux, OSX, and Windows.

2.  Start R; we recommend using R through [RStudio](https://www.rstudio.com/).  You can start R using RStudio (Windows, OSX, Linux), typing "R" at in a terminal (Linux or OSX), or using the R application either by double-clicking on the R application (Windows and OSX).

3.  For learning R, there are many resources such as [Try-R at codeschool](http://tryr.codeschool.com/) and [DataCamp](https://www.datacamp.com/getting-started?step=2&track=r).


# Packages not available on OSLERinHealth

If a package is not in the OSLERinHealth [list of packages ](http://oslerinhealth.org/list-packages/all), then it is not located on the [OSLERinHealth Github](https://github.com/oslerinhealth?tab=repositories).  Therefore, when installing, you'll get the following error:

```r
Error in neuro_install(...) : 
  Package(s) PACKAGE_TRIED_TO_INSTALL are not in oslerinhealth
```

Once a package is located on the list of packages, then it will be available to install. 


# Session Info


```r
devtools::session_info()
```

```
## Session info -------------------------------------------------------------
```

```
##  setting  value                       
##  version  R version 3.4.1 (2017-06-30)
##  system   x86_64, darwin15.6.0        
##  ui       X11                         
##  language (EN)                        
##  collate  en_US.UTF-8                 
##  tz       America/New_York            
##  date     2017-08-31
```

```
## Packages -----------------------------------------------------------------
```

```
##  package   * version date       source                            
##  backports   1.1.0   2017-05-22 CRAN (R 3.4.0)                    
##  base      * 3.4.1   2017-07-07 local                             
##  colorout  * 1.1-0   2015-04-20 Github (jalvesaq/colorout@1539f1f)
##  compiler    3.4.1   2017-07-07 local                             
##  datasets  * 3.4.1   2017-07-07 local                             
##  devtools    1.13.3  2017-08-02 CRAN (R 3.4.1)                    
##  digest      0.6.12  2017-01-27 CRAN (R 3.4.0)                    
##  evaluate    0.10.1  2017-06-24 cran (@0.10.1)                    
##  graphics  * 3.4.1   2017-07-07 local                             
##  grDevices * 3.4.1   2017-07-07 local                             
##  htmltools   0.3.6   2017-04-28 CRAN (R 3.4.0)                    
##  knitr       1.17    2017-08-10 cran (@1.17)                      
##  magrittr    1.5     2014-11-22 CRAN (R 3.4.0)                    
##  memoise     1.1.0   2017-04-21 CRAN (R 3.4.0)                    
##  methods     3.4.1   2017-07-07 local                             
##  Rcpp        0.12.12 2017-07-15 cran (@0.12.12)                   
##  rmarkdown   1.6     2017-06-15 cran (@1.6)                       
##  rprojroot   1.2     2017-01-16 CRAN (R 3.4.0)                    
##  stats     * 3.4.1   2017-07-07 local                             
##  stringi     1.1.5   2017-04-07 CRAN (R 3.4.0)                    
##  stringr     1.2.0   2017-02-18 CRAN (R 3.4.0)                    
##  tools       3.4.1   2017-07-07 local                             
##  utils     * 3.4.1   2017-07-07 local                             
##  withr       2.0.0   2017-07-28 cran (@2.0.0)                     
##  yaml        2.1.14  2016-11-12 CRAN (R 3.4.0)
```

# References