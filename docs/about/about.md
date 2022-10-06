---
layout: default
title: Learn More
nav_order: 2
description: "Spatio-temporal analysis of univariate or multivariate data, e.g., standardizing data for multiple species or stages"
permalink: /about
---

## Background

* This tool is designed to estimate spatial variation in density using spatially referenced data, with the goal of habitat associations (correlations among species and with habitat) and estimating total abundance for a target species in one or more years.
* The model builds upon spatio-temporal delta-generalized linear mixed modelling techniques (Thorson Shelton Ward Skaug 2015 ICESJMS), which separately models the proportion of tows that catch at least one individual ("encounter probability") and catch rates for tows with at least one individual ("positive catch rates").
* Submodels for encounter probability and positive catch rates by default incorporate variation in density among years (as a fixed effect), and can incorporate variation among sampling vessels (as a random effect, Thorson and Ward 2014) which may be correlated among categories (Thorson Fonner Haltuch Ono Winker In press).
* Spatial and spatiotemporal variation are approximated as Gaussian Markov random fields (Thorson Skaug Kristensen Shelton Ward Harms Banante 2014 Ecology), which imply that correlations in spatial variation decay as a function of distance.

## User resources for learning about VAST

**Need to update this**

There are eleven main resources for learning about VAST:

*  *Model structure*:  Please see the [User Manual](https://github.com/James-Thorson/VAST/blob/master/manual/VAST_model_structure.pdf) for a document listing model equations and relating them to the input/output used in R.
*  *Change documentation*:  Please see [NEWS](https://github.com/James-Thorson/VAST/blob/master/manual/NEWS.pdf) for a document listing major changes in each numbered release; this is useful to track development changes (interface improvements, bug fixes, issues regarding dependencies, etc.) over time. 
*  *Guidance for user decisions*:  Please see [Thorson-2019](https://www.sciencedirect.com/science/article/abs/pii/S0165783618302820) for guidance regarding the 15 major decisions needed in every VAST model
*  *High-level wrapper functions*:  I have recently added high-level wrapper functions, which provide a [gentle introduction](https://github.com/James-Thorson/VAST/wiki/Simple-example) to running `VAST`
*  *Examples*:  The wiki also includes example code to run VAST for many common [use-cases](https://github.com/James-Thorson-NOAA/VAST/wiki), as listed in the right-hand-side toolbar at that link.
*  *R-help documentation*:  Please see the standard R-help documentation, e.g., by typing `?fit_model` or `?make_data` in the R-terminal after installing the package and loading it using `library(VAST)`.
*  *Publications*:  Please see the [publications list](https://github.com/nwfsc-assess/geostatistical_delta-GLMM/wiki/Applications) to identify peer-reviewed publications regarding individual features.  These publications include statistical theory and model testing.
*  *List-serv*: Consider joining the [FishStats listserve](https://groups.google.com/forum/#!forum/fishstats-listserv) for 4-6 updates per year, including training classes.
*  *Slack channel*:  A [slack channel](https://join.slack.com/t/vastsupportgroup/shared_invite/zt-18zyad1zl-dLQKYE3kOPI~n~h7PM6coA) was developed by J. Morano and colleagues to allow real-time, casual discussions among new and longtime users.
*  *Talks available online*:  We post recorded talks and seminars [online](https://www.youtube.com/channel/UCNgFcss1X9Hgox3eWSTRI3Q/)
*  *Issue-tracker*:  Before posting new issues, users should explore the previous issues in the github issue tracker for [VAST](https://github.com/James-Thorson/VAST/issues), [SpatialDeltaGLMM](https://github.com/nwfsc-assess/geostatistical_delta-GLMM/issues), and [FishStatsUtils](https://github.com/james-thorson/FishStatsUtils/issues), including a search for old and closed issues.
*  *Wiki*:  Users should read and are encouraged to actively contribute to the wiki, which is housed at [the github for SpatialDeltaGLMM](https://github.com/nwfsc-assess/geostatistical_delta-GLMM/wiki)

If there are questions that arise after this, please look for a [VAST Point-of-Contact](https://docs.google.com/spreadsheets/d/1YfYeHHTLwHPxh_5jz4_-hhaRd4gbTq-Cvmii84vxWw0/edit) at your institution and consider contacting them prior to posting an issue.


## Database

Regions available in the [example script](https://github.com/james-thorson/VAST/blob/master/examples/Example--simple.R):
![alt text](https://github.com/nwfsc-assess/geostatistical_delta-GLMM/raw/master/examples/global_coverage.png "Global data coverage")
and see [FishViz.org](http://www.FishViz.org) for visualization of results for regions with a public API for their data.
