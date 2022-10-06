---
layout: default
title: About
nav_order: 1
description: "Spatio-temporal analysis of univariate or multivariate data, e.g., standardizing data for multiple species or stages"
permalink: /
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

* Is an R package for implementing a spatial delta-generalized linear mixed model (delta-GLMM) for multiple categories (species, size, or age classes) when standardizing survey or fishery-dependent data.
* Builds upon a previous R package `SpatialDeltaGLMM` (public available [here](https://github.com/nwfsc-assess/geostatistical_delta-GLMM)), and has unit-testing to automatically confirm that `VAST` and `SpatialDeltaGLMM` give identical results (to the 3rd decimal place for parameter estimates) for several varied real-world case-study examples
* Has built in diagnostic functions and model-comparison tools
* Is intended to improve analysis speed, replicability, peer-review, and interpretation of index standardization methods

## Background

* This tool is designed to estimate spatial variation in density using spatially referenced data, with the goal of habitat associations (correlations among species and with habitat) and estimating total abundance for a target species in one or more years.
* The model builds upon spatio-temporal delta-generalized linear mixed modelling techniques (Thorson Shelton Ward Skaug 2015 ICESJMS), which separately models the proportion of tows that catch at least one individual ("encounter probability") and catch rates for tows with at least one individual ("positive catch rates").
* Submodels for encounter probability and positive catch rates by default incorporate variation in density among years (as a fixed effect), and can incorporate variation among sampling vessels (as a random effect, Thorson and Ward 2014) which may be correlated among categories (Thorson Fonner Haltuch Ono Winker In press).
* Spatial and spatiotemporal variation are approximated as Gaussian Markov random fields (Thorson Skaug Kristensen Shelton Ward Harms Banante 2014 Ecology), which imply that correlations in spatial variation decay as a function of distance.

## Citation

Core functionality,
* Thorson, J.T., Barnett, L.A.K., 2017. Comparing estimates of abundance trends and distribution shifts using single- and multispecies models of fishes and biogenic habitat. ICES J. Mar. Sci. 74, 1311–1321. https://doi.org/10.1093/icesjms/fsw193
* Thorson, J.T., 2019. Guidance for decisions using the Vector Autoregressive Spatio-Temporal (VAST) package in stock, ecosystem, habitat and climate assessments. Fish. Res. 210, 143–161. https://doi.org/10.1016/j.fishres.2018.10.013

See also [References](/References).


## Steering Committee

Updates to VAST will be planned with input from a Steering Committee. The Steering Committee will include Jim Thorson and three rotating members, where each rotating member will serve as long as they have time and interest, and new members will be identified as needed by existing committee members. Member duties will be:

* some occasional discussion (<1 per month) regarding any externally suggested modifications (i.e., through the issue tracker), to prioritize which are important vs. could be postponed or ignored.
* availability and effort to help brainstorm changes, perhaps 1-3 times per year.

Current members are:

* Jim Thorson
* Christine Stawitz
* Cole Monnahan
* Eric Ward


## Funding and support for the tool

* Ongoing:  Support from Fisheries Resource Analysis and Monitoring Division (FRAM), Northwest Fisheries Science Center, National Marine Fisheries Service, NOAA
* Ongoing:  Support from Danish Technical University (in particular Kasper Kristensen) for  development of Template Model Builder software, URL: https://www.jstatsoft.org/article/view/v070i05
* Generous support from people knowledgeable about each region and survey! Specific contributions are listed [here](https://github.com/nwfsc-assess/geostatistical_delta-GLMM/blob/master/shiny/Acknowledgements_for_regional_inputs.csv).
* Hoff, G., Thorson, J., and Punt, A.  2018. Spatio-temporal dynamics of groundfish availability to EBS bottom trawl surveys and abundance estimate uncertainties.  North Pacific Research Board (NPRB) 2018 RFP.   
* Rooper, C., Thorson, J., and Boldt, J.  2017. Detecting changes in life history traits and distribution shifts in eastern Bering Sea fishes in response to climate change.  Habitat Information for Stock Assessments 2016 RFP.  
* Ianelli, J., Thorson, J., Kotwicki, S. 2017. Combining acoustic and bottom-trawl data in a spatio-temporal index standardization model for Eastern Bering Sea pollock.  Improve a Stock Assessment 2017 RFP. 
* Thorson, J., Ianelli, J., and O’Brien, L.  2015.  Distribution and application of a new geostatistical index standardization and habitat modeling tool for stock assessments and essential fish habitat designation in Alaska and Northwest Atlantic regions.  Habitat Assessment Improvement Plan 2014 RFP.  URL: https://www.st.nmfs.noaa.gov/ecosystems/habitat/funding/projects/project15-027


## Disclaimer

“The United States Department of Commerce (DOC) GitHub project code is provided on an ‘as is’ basis and the user assumes responsibility for its use. DOC has relinquished control of the information and no longer has responsibility to protect the integrity, confidentiality, or availability of the information. Any claims against the Department of Commerce stemming from the use of its GitHub project will be governed by all applicable Federal law. Any reference to specific commercial products, processes, or services by service mark, trademark, manufacturer, or otherwise, does not constitute or imply their endorsement, recommendation or favoring by the Department of Commerce. The Department of Commerce seal and logo, or the seal and logo of a DOC bureau, shall not be used in any manner to imply endorsement of any commercial product or activity by DOC or the United States Government.”
