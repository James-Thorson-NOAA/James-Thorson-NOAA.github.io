---
layout: default
title: Compositional data
parent: Tutorials
nav_order: 4
---

# Expand age and length composition
{: .no_toc }

---

# Expanding age/length composition data for use in stock assessments

It is possible to use `VAST` to expand subsamples of age/length composition obtained during fishery or survey sampling.  To do so, we first perform first-stage expansion prior to fitting the model, i.e., expand from animals that were subsampled for length in a given tow to the predicted numbers-at-length in that entire tow.  We then fit a spatio-temporal model to those expanded samples.

In this example, we use data for the lingcod survey off the US West Coast for Oregon and Washington.  For this simplified demonstrate, we fit to 32 size bins over four years.

```R
# Load packages
library(VAST)

# load data set
example = load_example( data_set="Lingcod_comp_expansion" )

# Make settings
settings = make_settings( n_x = 50,
  Region = example$Region,
  purpose = "index2",
  strata.limits = example$strata.limits,
  use_anisotropy = FALSE,
  fine_scale = FALSE )

# Run model
fit = fit_model( settings = settings,
  Lat_i = example$sampling_data[,'Lat'],
  Lon_i = example$sampling_data[,'Lon'],
  t_i = example$sampling_data[,'Year'],
  c_i = as.numeric(example$sampling_data[,'Length_bin'])-1,
  b_i = example$sampling_data[,'First_stage_expanded_numbers'],
  a_i = example$sampling_data[,'AreaSwept_km2'],
  Npool = 40,
  newtonsteps = 1,
  test_fit = FALSE )

# Plot results
results = plot( fit,
  check_residuals = FALSE,
  cluster_results = FALSE )
```

This automatically yields a plot of compositions for each size-bin and year, where results could then be included in a size-structured stock assessment model

![Expanded length compositions](/assets/images/compositions/Proportion.png)

## Works cited

Thorson, J. T., and Haltuch, M. A. 2018. Spatiotemporal analysis of compositional data: increased precision and improved workflow using model-based inputs to stock assessment. Canadian Journal of Fisheries and Aquatic Sciences, 76: 401–414.



