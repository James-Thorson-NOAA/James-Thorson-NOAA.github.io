---
layout: default
title: Installation
nav_order: 2
---

# Installation Instructions
{: .no_toc }

[![Build Status](https://travis-ci.org/James-Thorson/VAST.svg?branch=master)](https://travis-ci.org/James-Thorson/VAST)
[![DOI](https://zenodo.org/badge/55002718.svg)](https://zenodo.org/badge/55002718.

---

This function depends on R version >=3.1.1 and a variety of other tools.

First, install the "devtools" package from CRAN

    # Install and load devtools package
    install.packages("devtools")
    library("devtools")

Next, please install the VAST package from this GitHub repository using a function in the "devtools" package.  This may require using the `INSTALL_opts` option depending upon your version of R:

    # Install package
    install_github("james-thorson/VAST@main", INSTALL_opts="--no-staged-install")
    # Load package
    library(VAST)

If you are having problems with installation, please consider installing dependencies individually, e.g. using:

    # Install TMB from CRAN
    install.packages("TMB")
    # Install INLA using currently recommended method
    install.packages("INLA", repos=c(getOption("repos"), INLA="https://inla.r-inla-download.org/R/stable"), dep=TRUE)
    # Install FishStatsUtils from CRAN
    install_github("james-thorson/FishStatsUtils@main", INSTALL_opts="--no-staged-install")

Finally, please confirm that VAST is installed by running a model, e.g., following the simple example [here](https://github.com/James-Thorson-NOAA/VAST/wiki/Index-standardization).


Known installation/usage issues
=============

1.  If using a NOAA laptop, sometimes the PATH for Rtools is not correctly specified during installation. In those cases, please follow instructions [here](https://github.com/nwfsc-assess/geostatistical_delta-GLMM/issues/50)

2.  Some versions of R are having problems downloading dependencies from GitHub, see details [here](https://github.com/James-Thorson-NOAA/FishStatsUtils/issues/21)

3.  People using R version 3.6.0 or MRAN 3.5.3 are having a problem with changing standards for package namespaces, see details [here](https://github.com/James-Thorson-NOAA/VAST/issues/189), which appears to be particularly a problem with loading INLA due to install issues with that package.

4.  MacOS users have specific install issues and a discussion of potential fixes is [here](https://github.com/James-Thorson-NOAA/VAST/issues/218#issuecomment-587105809)

5.  MacOS users should be aware that significant speed-ups in model fitting can be accomplished by switching the library used for Basic Linear Algebra Subprograms (BLAS) from the default. There are a few BLAS alternatives available, though, the simplest seems to be using the vecLib library, part of Apple's Accelerate Framework and included in most recent R binaries. To switch the BLAS library, run the following lines in the terminal and then confirm the switch with a call to `sessionInfo()` in R.

```
    # Terminal commands to switch R BLAS library to increase speed
    cd /Library/Frameworks/R.framework/Resources/lib
    ln -sf /System/Library/Frameworks/Accelerate.framework/Frameworks/vecLib.framework/Versions/Current/libBLAS.dylib libRblas.dylib
```
6.  Windows has a speed-limit on the rate that users can access the GitHub API. You can get around this by installing each package locally from a ZIP file.  You'll need to first download a ZIP file for GitHub repositories `TMBhelper` ([here](https://github.com/kaskr/TMB_contrib_R/tree/master/TMBhelper)), then `ThorsonUtilities` ([here](https://github.com/James-Thorson/utilities)), then `FishStatsUtils` ([here](https://github.com/James-Thorson-NOAA/FishStatsUtils/releases)), then `VAST` ([here](https://github.com/James-Thorson-NOAA/VAST/releases)) to your harddrive in a local directory while recording the directory name (which I will reference as `download_dir`), and then install these packages from each ZIP file in the same order. To install each package, please click "clone or download" -> "Download ZIP" -> `devtools::install_local(path=download_dir, dependencies=FALSE)`
