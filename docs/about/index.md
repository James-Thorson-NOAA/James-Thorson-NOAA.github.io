---
layout: default
title: About
nav_order: 1
description: "Spatio-temporal analysis of univariate or multivariate data, e.g., standardizing data for multiple species or stages"
has_children: true
permalink: /about
---


# VAST
{: .fs-9 }

Spatio-temporal analysis of univariate or multivariate data, e.g., standardizing data for multiple species or stages
{: .fs-6 .fw-300 }

[Installation](#installation){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 } [View it on GitHub](https://github.com/James-Thorson-NOAA/VAST){: .btn .fs-5 .mb-4 .mb-md-0 }

---

{: .new }
> **Latest version `3.9.1` released July 22, 2022**
> See all [RELEASES](https://github.com/James-Thorson-NOAA/VAST/releases) for a detailed breakdown.


## Description

### VAST

* Is an R package for implementing a spatial delta-generalized linear mixed model (delta-GLMM) for multiple categories (species, size, or age classes) when standardizing survey or fishery-dependent data.
* Builds upon a previous R package `SpatialDeltaGLMM` (public available [here](https://github.com/nwfsc-assess/geostatistical_delta-GLMM)), and has unit-testing to automatically confirm that `VAST` and `SpatialDeltaGLMM` give identical results (to the 3rd decimal place for parameter estimates) for several varied real-world case-study examples
* Has built in diagnostic functions and model-comparison tools
* Is intended to improve analysis speed, replicability, peer-review, and interpretation of index standardization methods

### Background

* This tool is designed to estimate spatial variation in density using spatially referenced data, with the goal of habitat associations (correlations among species and with habitat) and estimating total abundance for a target species in one or more years.
* The model builds upon spatio-temporal delta-generalized linear mixed modelling techniques (Thorson Shelton Ward Skaug 2015 ICESJMS), which separately models the proportion of tows that catch at least one individual ("encounter probability") and catch rates for tows with at least one individual ("positive catch rates").
* Submodels for encounter probability and positive catch rates by default incorporate variation in density among years (as a fixed effect), and can incorporate variation among sampling vessels (as a random effect, Thorson and Ward 2014) which may be correlated among categories (Thorson Fonner Haltuch Ono Winker In press).
* Spatial and spatiotemporal variation are approximated as Gaussian Markov random fields (Thorson Skaug Kristensen Shelton Ward Harms Banante 2014 Ecology), which imply that correlations in spatial variation decay as a function of distance.

### Citation

Core functionality,
* Thorson, J.T., Barnett, L.A.K., 2017. Comparing estimates of abundance trends and distribution shifts using single- and multispecies models of fishes and biogenic habitat. ICES J. Mar. Sci. 74, 1311–1321. https://doi.org/10.1093/icesjms/fsw193
* Thorson, J.T., 2019. Guidance for decisions using the Vector Autoregressive Spatio-Temporal (VAST) package in stock, ecosystem, habitat and climate assessments. Fish. Res. 210, 143–161. https://doi.org/10.1016/j.fishres.2018.10.013

See also [References](/References).


### User resources for learning about VAST

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


### Database

Regions available in the [example script](https://github.com/james-thorson/VAST/blob/master/examples/Example--simple.R):
![alt text](https://github.com/nwfsc-assess/geostatistical_delta-GLMM/raw/master/examples/global_coverage.png "Global data coverage")
and see [FishViz.org](http://www.FishViz.org) for visualization of results for regions with a public API for their data.

### Steering Committee

Updates to VAST will be planned with input from a Steering Committee. The Steering Committee will include Jim Thorson and three rotating members, where each rotating member will serve as long as they have time and interest, and new members will be identified as needed by existing committee members. Member duties will be:

* some occasional discussion (<1 per month) regarding any externally suggested modifications (i.e., through the issue tracker), to prioritize which are important vs. could be postponed or ignored.
* availability and effort to help brainstorm changes, perhaps 1-3 times per year.

Current members are:

* Jim Thorson
* Christine Stawitz
* Cole Monnahan
* Eric Ward


### Funding and support for the tool

* Ongoing:  Support from Fisheries Resource Analysis and Monitoring Division (FRAM), Northwest Fisheries Science Center, National Marine Fisheries Service, NOAA
* Ongoing:  Support from Danish Technical University (in particular Kasper Kristensen) for  development of Template Model Builder software, URL: https://www.jstatsoft.org/article/view/v070i05
* Generous support from people knowledgeable about each region and survey! Specific contributions are listed [here](https://github.com/nwfsc-assess/geostatistical_delta-GLMM/blob/master/shiny/Acknowledgements_for_regional_inputs.csv).
* Hoff, G., Thorson, J., and Punt, A.  2018. Spatio-temporal dynamics of groundfish availability to EBS bottom trawl surveys and abundance estimate uncertainties.  North Pacific Research Board (NPRB) 2018 RFP.   
* Rooper, C., Thorson, J., and Boldt, J.  2017. Detecting changes in life history traits and distribution shifts in eastern Bering Sea fishes in response to climate change.  Habitat Information for Stock Assessments 2016 RFP.  
* Ianelli, J., Thorson, J., Kotwicki, S. 2017. Combining acoustic and bottom-trawl data in a spatio-temporal index standardization model for Eastern Bering Sea pollock.  Improve a Stock Assessment 2017 RFP. 
* Thorson, J., Ianelli, J., and O’Brien, L.  2015.  Distribution and application of a new geostatistical index standardization and habitat modeling tool for stock assessments and essential fish habitat designation in Alaska and Northwest Atlantic regions.  Habitat Assessment Improvement Plan 2014 RFP.  URL: https://www.st.nmfs.noaa.gov/ecosystems/habitat/funding/projects/project15-027


### Disclaimer

“The United States Department of Commerce (DOC) GitHub project code is provided on an ‘as is’ basis and the user assumes responsibility for its use. DOC has relinquished control of the information and no longer has responsibility to protect the integrity, confidentiality, or availability of the information. Any claims against the Department of Commerce stemming from the use of its GitHub project will be governed by all applicable Federal law. Any reference to specific commercial products, processes, or services by service mark, trademark, manufacturer, or otherwise, does not constitute or imply their endorsement, recommendation or favoring by the Department of Commerce. The Department of Commerce seal and logo, or the seal and logo of a DOC bureau, shall not be used in any manner to imply endorsement of any commercial product or activity by DOC or the United States Government.”
