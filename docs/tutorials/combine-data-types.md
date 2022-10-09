---
layout: default
title: Combine data types
parent: Tutorials
nav_order: 7
---

# Combine data types
{: .no_toc }

---

# Joint models fitting encounter, count, and biomass-sampling data

It is possible to use `VAST` to fit models jointly to different modes (a.k.a. types) of data, including encounter/non-encounter, count, and biomass-sampling data.  Combining data from different sampling designs can be useful to expand the spatial footprint beyond that covered by any one sampling program.

As a simple example, I show code for combining three data types for red snapper in the Gulf of Mexico:
* encounter/non-encounter data from the NMFS Red Snapper/Shark Bottom Longline Survey (“BLL”) dataset;
* count data from the National Marine Fisheries Service (NMFS) Pelagic Acoustic Trawl Survey (“PELACTR”) dataset;
* biomass data from the SEAMAP Groundfish Trawl Survey (“TRAWL”) dataset;
This example is fully documented in Grüss and Thorson (2019). 

```R
# Load packages
library(VAST)

# load data set
# see `?load_example` for list of stocks with example data
# that are installed automatically with `FishStatsUtils`.
example = load_example( data_set="multimodal_red_snapper" )

# Make settings
settings = make_settings( n_x = 100,
  Region = example$Region,
  purpose = "index2",
  strata.limits = example$strata.limits )

# Change `ObsModel` to indicate type of data for level of `e_i`
settings$ObsModel = cbind( c(13,14,2), 1 )

# Add a design matrix representing differences in catchability relative to a reference (biomass-sampling) gear
catchability_data = example$sampling_data[,'Data_type',drop=FALSE]
Q1_formula = ~ factor(Data_type)

# Run model
fit = fit_model( settings = settings,
  Lat_i = example$sampling_data[,'Lat'],
  Lon_i = example$sampling_data[,'Lon'],
  t_i = example$sampling_data[,'Year'],
  c_i = rep(0,nrow(example$sampling_data)),
  b_i = example$sampling_data[,'Response_variable'],
  a_i = example$sampling_data[,'AreaSwept_km2'],
  e_i = as.numeric(example$sampling_data[,'Data_type'])-1,
  Q1_formula = Q1_formula,
  catchability_data = catchability_data )

# Plot results
plot( fit ) #, col = c("blue","red","green")[catchability_data$Data_type] )

# Plot SEs
plot( fit, 
      plot_value = sd,
      check_residuals = FALSE ) #, col = c("blue","red","green")[catchability_data$Data_type] )
```

Inspecting the distribution of data (encounter: blue;  count: red;  biomass: green) shows, e.g.:
* that biomass data are generally lacking in the eastern Gulf of Mexico in 2006-2008, such that other data sources are needed to inform densities and 
* that count data are typically hte primary source of information for offshore densities prior to 2014;

![Expanded length compositions](/assets/images/combined-data/Data_by_year.png)
 
Given these multiple data sets, we can obtain precise estimates of total biomass and maps of distribution:

![Expanded length compositions](/assets/images/combined-data/Index.png)
![Expanded length compositions](/assets/images/combined-data/ln_density-predicted.png)

However, a plot of the standard deviation for log-biomass density shows that estimates are extremely imprecise in offshore portions of the modeled domain.  

![Expanded length compositions](/assets/images/combined-data/ln_density-transformed.png)

## Works cited

Grüss, A., and Thorson, J. T. 2019. Developing spatio-temporal models using multiple data types for evaluating population trends and habitat usage. ICES Journal of Marine Science, 76: 1748–1761.



