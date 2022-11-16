---
layout: default
title: Index Standardization
parent: Tutorials
nav_order: 1
---

# Index standardization
{: .no_toc }

---

This example will provide an abundance index for Alaska pollock in the eastern Bering Sea.

{: .important}
> It’s useful for reproducibility to record and use a specific VAST release number, while noting your system specifications. This example is run under VAST 3.9.1.
>
> Retrieve system and package version numbers with `sessionInfo()`


# Set up

Install the latest version of VAST
```R
devtools::install_github("James-Thorson-NOAA/VAST")
```

Set local working directory (change for your machine)
```R
setwd( "Enter directory location here" )
```

Load VAST and the example data set
```R
library(VAST)

# examples installed with VAST
example = load_example( data_set="EBS_pollock" )
```

{: .highlight }
Additional datasets are available in `load_example`. See `?load_example` for list of stocks with example data. To use a new data set, input new data in the same format as example data. See `?make_data` for more details.

# Configure model settings

Make settings, with defaults defined by the specified `purpose` argument.  Here, we use `purpose="index2"` which reflects current "good practices" for standardizing an abundance index for use in a subsequent stock-assessment model.  
```R
settings = make_settings( n_x = 100, 
  Region = example$Region, 
  purpose = "index2" )
```

# Run the model

Run model, where default `settings` can be overridden by passing additional arguments to `fit_model`
```R
fit = fit_model( settings = settings, 
  Lat_i = example$sampling_data[,'Lat'], 
  Lon_i = example$sampling_data[,'Lon'], 
  t_i = example$sampling_data[,'Year'], 
  b_i = example$sampling_data[,'Catch_KG'], 
  a_i = example$sampling_data[,'AreaSwept_km2'] )
```

# See the model results

Plot results. Add some more info here.
```R
plot( fit )
```

![Predicted density of Alaska pollock in the eastern Bering Sea for each year](/assets/images/index-standardization/ln_density-predicted.png)

## Works cited

Thorson, J. T., Shelton, A. O., Ward, E. J., and Skaug, H. J. 2015. Geostatistical delta-generalized linear mixed models improve precision for estimated abundance indices for West Coast groundfishes. ICES Journal of Marine Science: Journal du Conseil, 72: 1297–1310.


