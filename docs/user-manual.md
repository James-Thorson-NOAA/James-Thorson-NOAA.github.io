---
layout: default
title: User Manual
nav_order: 7
permalink: /user-manual
---

# VAST model structure and user interface

**James Thorson**

## Purpose of document

R package VAST includes many different forms of documentation, which are
documented on the [package GitHub
page](https://github.com/James-Thorson-NOAA/VAST#user-resources-for-learning-about-vast).
This "VAST model structure and user interface" document is intended to
complement these other resources by documenting and describing the model
structure (all model equations and notation). Please see reference
documentation for explanation of the user interface, and GitHub wiki for
examples.

## Package architecture

VAST is developed as an R package available on GitHub. It depends upon helper functions that are bundled in package FishStatsUtils, and these helper functions are installed separately because they are also used by other spatio-temporal packages (e.g., EOFR). VAST and FishStatsUtils use S3 objects to ease interpretation of objects that are commonly saved to terminal (see Table 1 for list). VAST can be run using two primarylevels of abstraction:

1.  *High-level wrapper functions*: New users are recommended to explore
    using `FishStatsUtils::make_settings` and
    `FishStatsUtils::fit_model` to run VAST, and to explore results
    using `plot` and `summary`.

2.  *Mid-level utilities*: Experienced users often run lower-level
    functions to accomplish basic tasks in spatial analysis, using
    `FishStatsUtils::make_extrapolation_info`,
    `FishStatsUtils::make_spatial_info`, `VAST::make_data`, and
    `VAST::make_model` individually.

Updates to VAST are released using semantic-version numbering (e.g., version 3.2.0) and a battery of integrated tests (comparing results using updated code to saved results from earlier versions) are run prior to numbered releases to ensure that results are backwards compatible.

## Model description

In the following, I use mathematical notation similar to the C++ code used to define the model in TMB: Notation is close to common recommendations, e.g., Edwards and Auger‐Méthé (2019), although I use parentheses to indicate indices of vectors, matrices, and arrays, and reserve subscripts for naming (see Table 2 for summary of notation that may be slightly out-of-date). Feel free to change notation when describing the model to suit your purposes in reports or publications. For further details regarding terminology, motivation, and statistical properties, please read the papers listed on the GitHub main page.

### Model Overview

VAST predicts variation in density across multiple locations $s$, time intervals $t$, for multiple categories $c$. Categories could include either multiple species, multiple size/age/sex classes for each individual species, and/or a mix of biological, physical, and fishery variables describing an ecosystem. VAST approximates the covariance between these multiple categories and years using a factor-model decomposition (Thorson et al. 2015b, 2016a), i.e., by summing across the contribution of multiple random effects (termed factors). If there is only a single category, the model reduces to a standard univariate spatio-temporal model.

After estimating variation in density across space, time, and among
categories, VAST then predicts variables at extrapolation-grid cells
distributed within across a user-specified spatial domain. This allows
derived quantities to be calculated by summing across extrapolation-grid
cells (as an approximation to the integral across this spatial domain);
this is analogous to an "area-weighting" approach to index
standardization, and the resulting prediction of total abundance can be
used an index of abundance.

In addition to spatial and spatio-temporal covariance among multiple
categories, VAST allows users to specify either density or catchability
covariates. Both explain variation in observed catch-rate data, but VAST
predicts density (for use in calculating the abundance index) using
density covariates but not catchability covariates. Therefore, VAST
"controls for" catchability covariates when calculating an index (i.e.,
removes their estimated effect) while "conditioning on" density
covariates when calculating an index (i.e., uses them to improve
interpolated/extrapolated predictions of density).

VAST estimates the value of spatial variables at $n_{x}$ knots, as well
as additional boundary vertices such that the total number of spatial
"vertices" is $n_{s}$. VAST specifically uses a k-means algorithm to
identify the location of $n_{x}$ knots to minimize the total distance
between the location of knots and either data or extrapolation-grid
cells. This distributes knots as a function of the spatial intensity of
sampling data.

### Linear predictors

The model potentially includes two linear predictors (because it is
designed to support delta-models, which include two components). The
first linear predictor $p_{1}(i)$ represents encounter probability in a
delta-model, or zero-inflation in a count-data model:

$$p_{1}(i) = \underset{Temporal\ variation}{\overset{\beta_{1}(c_i,t_i)}{︸}} + \underset{Spatial\ variation}{\overset{\omega_1^*( s_i,c_i )}{︸}} + \underset{Spatio - temporal\ variation}{\overset{\varepsilon_{1}^*(s_i,c_i,t_i)}{︸}} + \underset{Vessel\ effects}{\overset{\eta_{1}(v_i,c_i)}{︸}} + \underset{Habitat\ covariates}{\overset{\nu_{1}( c_i,t_i )}{︸}} + \underset{Catchability\ covariate}{\overset{\zeta_{1}(i)}{︸}} - \underset{Fishing\ impacts}{\overset{\iota(c_i,t_i)}{︸}}$$

where $p_{1}(i)$ is the predictor for observation $i$, arising for
category $c_i$ at location $s_i$ and time $t_i$. Similarly, the
second linear predictor $p_{2}(i)$ represents positive catch rates in a
delta-model, or the count-data intensity function in a count-data model,
where all variables and parameters are defined similarly except using
different subscripts (Thorson and Barnett 2017; Thorson 2019). Model
components are specified hierarchically to efficiently compute
correlated variation among categories and years as explained next.

### Temporal variation

Regarding intercepts representing temporal variation:

$$\beta_{1}( c_i,t_i ) = \mu_{\beta_1}( c_i ) + \sum_{f = 1}^{n_{\beta_1}}{L_{\beta_1}( c_i,f )\beta_{1}( t_i,f )}$$

where $\beta_{1}( t_i,f )$ represents temporal variation for time $t_i$ for factor $f$ (of $n_{\beta_1}$ factors representing temporal variation), $L_{\beta_1}(c_i,f)$ is the loadings matrix that generates temporal covariation among categories for this linear predictor, and $\mu_{\beta_1}( c_i )$ represents the time-average for each category $c_i$. The number of factors $n_{\beta_1}$ can range from zero to the number of categories $n_{c}$, $0 \leq n_{\beta_1} \leq n_{c}$, where $n_{\beta_1} = 0$ is equivalent to eliminating all temporal terms from the model. By default, $n_{\beta_1} = n_{c}$, $\beta_{1}( t_i,f  )$ is treated as a fixed effect for each year $t$ and factor $f$, $\mu_{\beta_1}( c_i ) = 0$, and $L_{\beta_1}$ is an identity matrix; this formulation is equivalent to estimating a separate intercept $\beta_{1}( t_i,c ) = \beta_{1}( t_i,f )$ as fixed effect for each category and year.

Intercepts can instead be treated as a random effect using the
factor-model formulation, which allows for sharing information among
years and categories. When treated as random,
$\beta_{1}( t_i,f )$ is assigned a normal distribution with
unit variance, such that $L_{\beta_1}^{T}L_{\beta_1}$
is the covariance among categories for a given process (Thorson et al.
2015b). When treating intercepts as random, and when there is only one
category and using one factor ($n_{\beta_1} = 1$), then
$L_{\beta_1}$ is a 1x1 matrix (i.e. a scalar) such
$L_{\beta_1}^{2}$ is the variance and the absolute value,
$abs(L_{\beta_1})$ is the standard deviation for temporal
variation.

By default the model specifies that each intercept $\beta_{1}(c,t)$ and
$\beta_{2}(c,t)$ is a fixed effect. However, other settings specify the
following autocorrelation structure:

$$\beta_{1}(t,f)\sim\left\{ \begin{matrix}
Normal(0,1) & if\ t = t_{\min} \\
Normal( \rho_{\beta_1}\beta_{1}(t - 1,f),1 ) & if\ t > t_{\min} \\
\end{matrix} \right.\ $$

Where $t_{\min}$ is the index for the first modelled year and
$\rho_{\beta_1}$ and $\rho_{\beta_2}$ are the estimated degree of
first-order autocorrelation in temporal variation (note that random
effects have a variance of one given that they are subsequently
multiplied by loadings matrices that represent the temporal covariance
among factors). Options treating intercepts as a random effect include:

1.  *Independent among years* --specifies $\rho_{\beta_1} = 0$

2.  *Random walk* --specifies $\rho_{\beta_1} = 1$

3.  *Constant intercept* --specifies $\rho_{\beta_1} = 0$ and
    $\sigma_{\beta_1}^{2} = 0$ (i.e., $\beta_{1}(t)$ is constant for all
    $t$)

4.  *Autoregressive* --estimates $\rho_{\beta_1}$ as a fixed effect

and settings are defined identically for specifying $\rho_{\beta_2}$.

### Spatial variation

Regarding spatial variation:

$$\omega_1^*(s,c) = \sum_{f = 1}^{n_{\omega 1}}{L_{\omega 1}(c_i,f)\omega_1^*( s_i,f  )}$$

where $\omega_1^*( s_i,f  )$ represents predicted
spatial variation in the first linear predictor occurring at the
location $s_i$ of sample $i$ for factor $f$ (of $n_{\omega 1}$ factors
representing spatial variation), and $L_{\omega 1}(c_i,\ f)$ is the
loadings matrix that generates spatial covariation among categories for
this linear predictor.

VAST specifies internally that the spatial and spatio-temporal Gaussian
random fields (GMRFs) have a variance of 1.0. By default VAST estimates
their values at each of $n_{s}$ vertices as follows:

$$\omega_1(f)\sim MVN( \mathbf{0},\mathbf{R}_{1} )$$

where $\omega_1(f)$ is the vector of length $n_{s}$ formed
when subsetting $\omega_1(s,f)$ for a given $f$. Specifying a variance
of 1.0 ensures that the covariance among categories is defined by the
loadings matrix for that term. These GMRFs are then projected to
calculate their value at every location $s_i$ using matrix
$\mathbf{A}$ with $n_i$ rows and $n_{s}$ columns. Specifically, values
are projected as:

$$\omega_1^*(f) = \mathbf{A}_i\omega_1(f)$$

where $\omega_1^* (f)$ is the vector of length $n_i$,
containing the predicted value $\omega_1^* ( s_i,f  )$
for spatial variation in the first linear predictor at every location
$s_i$, and other spatial variables are predicted similarly using
matrix $\mathbf{A}$.

### Spatio-temporal variation

Regarding spatio-temporal the model by default specifies that each
vector of spatio-temporal random effects,
$\mathbf{\varepsilon}_{1}( f_{1},f_{2} )$ and
$\mathbf{\varepsilon}_{2}( f_{1},f_{2} )$ composed of
$\varepsilon_{1}( s,f_{1},f_{2} )$ and
$\varepsilon_{2}( s,f_{1},f_{2} )$ across locations $s$, is
independent for each factor representing covariation among categories
($f_{1}$) and among years ($f_{2}$). We describe the process for the
1^st^ linear predictor, and an identical process is used for the 2^nd^
linear predictor (using different subscripts):

$$\mathbf{\varepsilon}_{1}( f_{1},f_{2} )\sim MVN( \mathbf{0},\mathbf{R}_{1} )$$

Values are then projected as:

$$\mathbf{\varepsilon}_{1}^*( f_{1},f_{2} ) = \mathbf{A}_i\mathbf{\varepsilon}_{1}( f_{1},f_{2} )$$

This is then projected across years and categories using loadings
matrices $L_{\varepsilon_{t}1}$ and
$\L_{\varepsilon_{c}2}$:

$$\varepsilon_{1}^{'}(s,c,t) = \sum_{f_{1} = 1}^{n_{\varepsilon_{c}1}}{\sum_{f_{2} = 1}^{n_{\varepsilon_{t}1}}{L_{\varepsilon_{c}1}(c,f_{1})L_{\varepsilon_{t}1}(f_{2},t)\varepsilon_{1}( s,f_{1},f_{2} )}}$$

Using a factor-decomposition to approximate covariation among years is a
generalization of empirical orthogonal function (EOF) analysis (Thorson
et al. 2020). The user can also specify a vector-autoregressive
structure:

$$\varepsilon_{1}( s,c_{1},t ) = \left\{ \begin{matrix}
\varepsilon_{1}^{'}( s,c_{1},t ) & if\ t = t_{\min} \\
\sum_{c_{2} = 1}^{n_{c}}{b( c_{1},c_{2} )\varepsilon_{1}^{'}( s,c_{2},t - 1 )} & if\ t > t_{\min} \\
\end{matrix} \right.\ $$

Where $b( c_{1},c_{2} )$ is the estimated impact of
spatio-temporal variation in category $c_{2}$ on spatio-temporal changes
in category $c_{1}$:

$$b( c_{1},c_{2} ) = \left\{ \begin{matrix}
\sum_{f = 1}^{n_{b}}{\chi(c_{1},f)\psi(f,c_{2})} + \rho_{\varepsilon 1}(c_{1}) & if\ c_{1} = c_{2} \\
\sum_{f = 1}^{n_{b}}{\chi(c_{1},f)\psi(f,c_{2})} & if\ c_{1} \neq c_{2} \\
\end{matrix} \right.\ $$

Where $\chi(c_{1},f)$ and $\psi(f,c_{2})$ represent elements of matrices
$\mathbf{Χ}$ and $\mathbf{\Psi}$, where the product $\mathbf{Χ\Psi}$ is
the typical interaction matrix in a cointegration model (Engle and
Granger 1987), where $\mathbf{\Psi}$ projects dynamics to a
low-dimensional subspace and $\mathbf{Χ}$ represents responses within
that subspace. By default $n_{b} = 0$ corresponding to
$\mathbf{Χ\Psi = 0}$, and these terms drop out of the model; however,
they allow a parsimonious representation of species interactions
(Thorson et al. 2017, 2019). Meanwhile $\rho_{\varepsilon 1}(c)$ is the
estimated degree of first-order autocorrelation in temporal variation:

1.  *Random walk* -- specifies $\rho_{\varepsilon 1}(c) = 1$

2.  *Autoregressive* -- estimates $\rho_{\varepsilon 1}$ as a single
    fixed effect with the same value for all categories

3.  *Individual autoregressive* \-- estimates a separate value of
    $\rho_{\varepsilon 1}(c)$ as a single fixed effect for each category

and settings are defined identically for specifying
$\rho_{\varepsilon 2}$.

### Overdisperison

Regarding overdispersion:

$$\eta_{1}( v_i,c_i ) = \sum_{f = 1}^{n_{\eta 1}}{L_{1}(c_i,f)\eta_{1}( v_i,f )}$$

where $\eta_{1}( v_i,f )$ represents random variation in
catchability among a grouping variable (tows or vessels) for each factor
$f$ (of $n_{\eta 1}$ factors representing overdispersion), and
$L_{1}(c_i,f)$ is a loadings matrix that generates covariation in
catchability among categories for this predictor. All loadings matrices
are specified similarly to $\L_{\beta_1}$, i.e., where factors
have a variance of one such that $\mathbf{L}^{T}\mathbf{L}$ represents
the covariance among categories. The main difference is that spatial,
spatio-temporal, and overdispersion factors can only be specified as
random effects, while the intercepts can be specified as either random
or fixed (where specifying as fixed "turns off" all factor-modelling for
that intercept).

### Density covariates

Regarding covariates affecting densities ("density" or "habitat"
covariates):

$$\nu_{1}( c_i,t_i ) = \sum_{p = 1}^{n_{p}}{( \gamma_{1}( c_i,p ) + \sigma_{\xi 1}(c_i,p)\xi_{1}^*( s_i,c_i,p ) )X( i,t_i,p )}$$

where $X( i,t_i,p )$ is an three-dimensional array of
$n_{p}$ measured density covariates that explain variation in density
for time *t* and the location $s_i$ where sampling occurred for sample
$i$. VAST can include a separate, spatially-varying effect of each
habitat covariate $p$ for each category $c$. The spatially varying slope
is
$\gamma_{1}( c_i,t_i,p ) + \sigma_{\xi 1}(c,p)\xi_{n}(s,c,p)$,
where $\gamma_{1}( c_i,\ t_i,p )$ is the average effect
of density covariate $X( i,t_i,p )$ for category $c$,
$\xi_{n}( s_i,c_i,p )$ represents spatial variation in
that effect (which has a mean of zero and standard deviation of one),
and $\sigma_{\xi 1}(c,p)$ represents the estimated standard deviation of
spatial variation of covariate $p$ for category $c$. By default VAST
estimates spatially-varying slope terms values at each vertex as
follows:

$$\mathbf{\xi}_{1}(c,p)\sim MVN( \mathbf{0},\mathbf{R}_{1} )$$

Values are then predicted as e.g.:

$$\mathbf{\xi}_{1}^{\mathbf{*}}(c,p) = \mathbf{A}_i\mathbf{\xi}_{1}(c,p)$$

### Catchability covariates

Finally, regarding covariates affecting the process of obtaining
measurements ("catchability" or "detectability" covariates):

$$\zeta_{1}(i) = \sum_{k = 1}^{n_{k}}( \lambda_{1}(k) + \sigma_{\varphi 1}(k)\varphi_{1}^*( s_i,k ) )q_{1}(i,k)$$

Where $q_{1}(i,k)$ is an element of matrix $\mathbf{Q}_{1}$ composed of
$n_{k}$ measured catchability covariates that explain variation in
catchability, $\lambda_{1}(k)$ is the estimated impact of catchability
covariates for this linear predictor,
$\varphi_{1}^*( s_i,k )$ is unit-variance spatial
variation in that slope term such that
$\sigma_{\varphi 1}(k)\varphi_{1}^*( s_i,k )$ has
standard deviation $\sigma_{\varphi 1}(k)$, where spatial variation in
detectability is specified as follows:

$$\mathbf{\varphi}_{1}(k)\sim MVN( \mathbf{0},\mathbf{R}_{1} )$$

Values are then predicted as e.g.:

$$\mathbf{\varphi}_{1}^{\mathbf{*}}(c,p) = \mathbf{A}_i\mathbf{\varphi}_{1}(k)$$

### Fishing impacts

Fishing impacts are included to represent the effect of known human
impacts on variables. They are not yet documented in detail here, but
see Thorson et al. (2019) for details. By default this term is excluded
(i.e., $\iota( c_i,t_i ) = 0$) and it is only applicable
within MICE or single-species production models following
vector-autoregressive dynamics (i.e., Gompertz density dependence). Feel
free to contact the package author if desiring more documentation.

### Link functions and observation error distributions

There are currently four options for the link function. For the latest
set of options see the R help documentation by typing into the R
terminal \`?VAST::Data_Fn\`.

1.  ObsModel\[2\]=0 applies a logit-link for the first linear predictor:

$$r_{1}(i) = {logit}^{- 1}( p_{1}(i) )$$

> where $r_{1}(i)$ is the predictor encounter probability in a
> delta-model, or zero-inflation in a count-data model, and
> ${logit}^{- 1}(p_{1}(i))$ is the inverse-logit (a.k.a. logistic)
> function of $p_{1}(i)$, and:

$$r_{2}(i) = a_i \times \log^{- 1}( p_{2}(i) )$$

> where $r_{2}(i)$ is the predicted biomass density for positive catch
> rates in a delta-model or mean-intensity function for a count-data
> model, $\log^{- 1}(p_{2}(i))$ is the exponential function of
> $p_{2}(i)$, and $a_i$ is the area-swept for observation $i$, which
> enters as a linear offset for expected biomass given an encounter.

2.  ObsModel\[2\]=1 corresponds to a "Poisson-link" delta-model that
    approximates a Tweedie distribution:

$$r_{1}(i) = 1 - \exp( - a_i \times \exp( p_{1}(i) ) )\ $$

> where $r_{1}(i)$ is the predictor encounter probability and
> $1 - \exp( - a_i \times \exp( p_{1}(i) ) )$ is
> a complementary log-log link of $p_{1}(i) + \log( a_i )$,
> and:

$$r_{2}(i) = \frac{a_i \times \exp( p_{1}(i) )}{r_{1}(i)} \times \exp( p_{2}(i) )$$

> where $r_{2}(i)$ is the predicted biomass given that the species is
> encountered. In this "Poisson-process" link function,
> $\exp( p_{1}(i) )$ is interpreted as the density in number
> of individuals per area such that
> $a_i \times \exp( p_{1}(i) )$ is the predicted number of
> individuals encountered, and $\exp( p_{2}(i) )$ is
> interpreted as the average weight per individual. Area-swept $a_i$
> therefore enters as a linear offset for the expected number of
> individuals encountered (Thorson 2018). This Poisson-link function
> should only be used for delta-models, and not for count-data models,
> but can also be used to combine encounter, count, and biomass-sampling
> data (see section below for details).

## Observation models

There are different user-controlled options for observation models for available sampling data. I distinguish between observation models for continuous-valued data (e.g., biomass, or numbers standardized to a fixed area), and observation models for count data (e.g., numbers treating area-swept as an offset). However, both are parameterized such that the expectation for sampling data $\mathbb{E}( B_i ) = r_{1}(i) \times r_{2}(i)$.

*Continuous-valued data (e.g., biomass)*

If using an observation model with continuous support (e.g., a normal, lognormal, gamma, or Tweedie models), then data $b_i$ can be any non-negative real number, $b_i\mathcal{\in R}$ and $b_i \geq 0$. VAST calculates the probability of these data as:

$$\Pr ( b_i = B ) = \left\{ \begin{matrix}1 - r_{1}(i) & if\ B = 0 \\r_{1}(i) \times g\{\left. \ B|r_{2}(i),\sigma_{m}^{2}(c)\} \right\  & if\ B > 0 \\\end{matrix} \right.\$$

where `ObsModel[1]` controls the probability density function $g{\left. \ B|r_{2}(i),\sigma_{m}^{2}(c)\} \right.\$ used for positive catch rates (see `?Data_Fn` for a list of options), where each options is defined to have with expectation $r_{2}(i)$ and dispersion $\sigma_{m}^{2}(c)$, where dispersion parameter $\sigma_{m}^{2}(c)$ varies among categories by default.

*Discrete-valued data (e.g., abundance)*

If using an observation model with discrete support (e.g., a Poisson, negative-binomial, Conway-Maxwell Poisson, or lognormal-Poisson models), then data $b_i$ can be any whole number, $b_i \in \{ 0,1,2,\ldots\}$. VAST calculates the probability of these data as:

$$\Pr( B = b_i ) = \left\{ \begin{matrix} ( 1 - r_{1}(i) ) + g\{ B = \left. \ 0|r_{2}(i),\ldots\} \right.\  & if\ B = 0 \\ r_{1}(i) \times g\{\left. \ B = b_i|r_{2}(i),\ldots\} \right.\  & if\ B > 0 \\ \end{matrix} \right.\ $$

where `ObsModel[1]` controls the probability mass function $g\{\left. \ B|r_{2}(i),\ldots\} \right.\ $ used (again, see `?Data_Fn` for a list of options), where I use ... to signify that these probability mass functions generally can have one or more parameter governing dispersion, and the precise number and interpretation varies among observation models (i.e., the value of `ObsModel[1]`). For these count-data models, $( 1 - r_{1}(i) )$ is the "zero-inflation probability" (i.e., the proportion of habitat in the immediate vicinity of location $s_i$ and time $t_i$ that is never occupied), while $r_{2}(i)$ is the expected value for probability mass function $g\{\left. \ B = b_i|r_{2}(i),\ldots\} \right.\ $ (i.e., the number of individuals that are in the vicinity of sampling in habitat that is occupied), and $g\{ B = \left. \ 0|r_{2}(i),\ldots\} \right.\ $ is the probability of not encountering category *c* given that sampling occurs in occupied habitat (Martin et al. 2005).

## Settings regarding spatial smoothers

VAST then uses a stochastic partial differential equation (SPDE)
approximation to the probability density function for spatial and
spatio-temporal variation (Lindgren et al. 2011). This SPDE
approximation involves generating a triangulated mesh that has a vertex
of a triangle at each knot, and VAST generates this triangulated mesh
using package *R-INLA* (Lindgren 2012). This mesh includes all $n_{x}$
user-specified "interior vertices," as well as additional "boundary
vertices" such that the total number of interior and boundary vertices
is $n_{s}$. Outputs from this triangulated mesh can then be used to
calculate the precision (inverse-covariance) matrix for a multivariate
normal probability density function for the value of a spatial variable
at all $n_{s}$ verticies. Specifically, the correlation
$\mathbf{R}_{1}( s_{1},s_{2} )$ between location $s$ and
location $s_{2}$ for spatial and spatio-temporal terms included in the
first linear predictor is approximated as following a Matern function:

$$\mathbf{R}_{1}( s_{1},s_{2} ) = \frac{1}{2^{\nu - 1}\Gamma(\nu)} \times ( \kappa_{1}\left| (s_{1}\mathbf{-}s_{2})\mathbf{H} \right| )^{\nu} \times K_{\nu}( \kappa_{1}\left| (s_{1}\mathbf{-}s_{2})\mathbf{H} \right| )$$

where $\mathbf{H}$ is a two-dimensional linear transformation
representing geometric anisotropy (with a determinant of 1.0), $\nu$ is
the Matern smoothness (fixed at 1.0), and $\kappa_{1}$ governs the
decorrelation distance for that first linear predictor ($\kappa_{2}$ is
also separately estimated for the second linear predictor). The linear
transformation representing geometric anisotropy includes two degrees of
freedom (included in vector $\mathbf{h}$). By default these are
estimated as fixed effects and then transformed:

$$\mathbf{H} = \begin{bmatrix}
\exp( h(1) ) & h(2) \\
h(2) & \frac{1 + h^{2}(2)}{\exp( h(1) )} \\
\end{bmatrix}$$

However, the user can specify isotropy (i.e., $\mathbf{h} = \mathbf{0}$
such that $\mathbf{H = I}$) as alternative.

There are also other options:

1.  *barrier effects*: avoiding correlations traveling across land;

2.  *spherical projections*: calculating distance based on spherical
    coordinates, to avoid sensitivity to chosen projection;

3.  *stream-network distance*: calculating distance based on river
    distances in a stream network or other graphical spatial dependency
    (Hocking et al. 2018).

## Interpolating spatial variation from knots to the location of samples

Starting with VAST release 3.0.0, users can choose between two options
for smoothing spatial variation.

1.  *Piecewise constant*: Following the conventional for releases of
    VAST prior to 3.0.0, users can specify fine_scale=FALSE. Given this
    specification, spatial variables at location $s$ are fixed equal to
    their value at the nearest "knot." This involves specifying matrix
    $\mathbf{A}_i$ such that row $i$ has value zero except for one
    cell containing a value of one for the knot closest to sample $i$.

2.  *Bilinear interpolation*: Following standard practices using the
    software R-INLA (Lindgren 2012; Lindgren and Rue 2015), users can
    specify fine_scale=TRUE. Given this specification, spatial variables
    at location $s$ are interpolated using the triangulated mesh that is
    also used to approximate spatial variation. Specifically, matrix
    $\mathbf{A}_i$ has row $i$ with value zero except for three cells,
    representing the vertices of the triangle containing location
    $s_i$.

## Structure on parameters among years

There are different user-controlled options for specifying structure for
intercepts or spatio-temporal variation across time.

## Parameter estimation

Parameters are estimated using maximum likelihood, where the maximum
likelihood of fixed effects is obtained by integrating a joint
likelihood function with respect to random effects (Searle et al. 1992;
Gelman and Hill 2007; Thorson and Minto 2015). This integral is
approximated using the Laplace approximation (Skaug and Fournier 2006),
as implemented in Template Model Builder (Kristensen et al. 2016). The
likelihood is then optimized in the R statistical environment (R Core
Team 2017), and standard errors are obtained using a generalization of
the delta method (Kass and Steffey 1989). Derived quantities calculated
via a nonlinear transformation of random effects can be bias-corrected
using the epsilon-method (Tierney et al. 1989; Thorson and Kristensen
2016). Depending upon user-specified options, different parameters will
be either fixed (estimated via maximizing the log-likelihood) or random
(integrated across when calculating the log-likelihood). Please use R
function \`ThorsonUtilities::list_parameters( Obj )\` to see a list of
estimated parameters (where \`Obj\` is the compiled VAST object),
including which are fixed or random.

## Identifiability constraints

The model as described requires several identifiability constraints to
ensure that the resulting Hessian is positive definite (and hence allow
calculation of asymptotic standard errors):

1.  All loadings matrices are defined to be lower-triangular (i.e.,
    elements above the diagonal are fixed at 0);

2.  When estimating spatial random fields $\omega_1^*(s,c)$ and
    estimating a loadings matrix across years for spatio-temporal
    variation, it is helpful to impose a sum-to-zero constraint on
    factors of the loadings matrix $L_{\varepsilon_{t}1}(f_{2},t)$. This
    ensures that spatial terms represent the distribution in an
    "average" year, defined as year $t$ when
    $L_{\varepsilon_{t}1}( f_{2},t ) = 0$ for all columns;

3.  When estimating loadings across species
    $L_{\varepsilon_{c}1}(c,f_{1})$ and across years
    $L_{\varepsilon_{t}1}(f_{2},t)$, the magnitude (determinant) of
    these two matrices is confounded. The solution adopted here is to
    impose the constraint that
    $\sum_{f = 1}^{n_{f}}{\sum_{t = 1}^{n_{t}}{L_{\varepsilon_{t}}(f,t)}} = 1$
    for both linear predictors, such that the magnitude of
    $L_{\varepsilon_{c}}(c,f)$ can be interpreted similarly to other
    loadings matrices.

4.  When estimating a spatially varying response to intercepts
    $\beta_{1} + \beta_{2}$, it is helpful to center these prior to
    using them as a covariate (NOTE: this feature is still in
    development, and recommended constraints may change).

The model also has issues arising from "label switching," i.e., where
any column of any loadings matrix could be multiplied by negative-one
(and similarly for the associated factor) without any change in the
model predicts and likelihood. This implies that the negative
log-likelihood has a series of local minima that all have the same
properties. We do not address "label switching" because it does not have
any practical effect on maximum-likelihood estimation or resulting
predictions, but we note that it gives rise to numerical complexities
when tuning or interpreting mixing for conventional samplers within a
Bayesian estimation paradigm.

## Combining multiple data types

VAST can be used to combine encounter/non-encounter, count, and
biomass-sampling data. This involves specifying a Poisson-link delta
model which predicts each data type from numbers density
$\exp( p_{1}(i) )$ and biomass-per-individual
$\exp( p_{2}(i) )$, see Grüss and Thorson (2019) for details.
This approach is specified by associating each observation with a given
error distribution using input e_i where e.g. e_i\[1\] is the
error-distribution for the 1^st^ observation. The user then specifies
multiple observation errors via input ObsModel_ez:

\# Control observation error

ObsModel_ez = cbind( \"PosDist\"=c(13,14,2), \"Link\"=c(1,1,1) )

In this specification, e_i\[1\]==1 indicates that the first observation
follows a Bernoulli distribution for encounter/non-encounter data,
e_i\[1\]==2 indicates that this observation follows a lognormal-Poisson
distribution for count data, and e_i\[1\]==3 indicates that it follows a
gamma distribution for biomass-sampling data. This specification can be
modified to include different combinations of these same data types.

## Relationship to other named models

VAST can be configured to be identical to (or closely mimic) many models
that have previously been published in ecology and fisheries:

1.  *Spatial Gompertz model*: If intercepts are constant across years,
    spatio-temporal variation follows an autoregressive process, and
    only one category is modelled, then VAST is identical to a
    spatio-temporal Gompertz model (Thorson et al. 2014).

2.  *Spatial factor analysis*: If only one year is analysed and multiple
    categories are modelled, VAST is similar to spatial factor analysis
    (Thorson et al. 2015b), although it permits the use of a delta-model
    (i.e., separate analysis of encounters and positive catch rates).

3.  *Spatial dynamic factor analysis*: If intercepts are constant among
    years, spatio-temporal variation follows an autoregressive process,
    and multiple categories are modelled, then VAST is similar to
    spatial dynamic factor analysis (Thorson et al. 2016a), although
    VAST allows separate estimates of spatial vs. spatio-temporal
    covariation and also the use of a delta-model.

4.  *Empirical orthogonal function analysis*: VAST can be configured to
    replicates empirical orthogonal function analysis, e.g., as commonly
    used by physical oceanographers to summarize physical conditions to
    produce an annual index and spatial map associated with a positive
    phase of the resulting index. However, I will wait to document this
    until the associated paper is published.

## Predicting variables across the spatial domain and calculating derived quantities

After a nonlinear minimizer has identified the value of fixed effects
that maximizes the Laplace approximation to the marginal likelihood,
Template Model Builder predicts the value of random effects that
maximizes the joint likelihood conditional on these fixed effects. It
then uses the predicted values of random effects to predict each spatial
variable at each of $n_{g}$ "extrapolation-grid cells" that are used to
summarize the spatial domain of sampling (Shelton et al. 2014; Thorson
et al. 2015a). Predicting random effects at extrapolation-grid cell $g$
at location $s_{g}$ is accomplished using matrix $\mathbf{A}_{g}$ with
$n_{g}$ rows and $n_{s}$ columns. Values are predicted as e.g.:

$$\omega_1^*(f ) = \mathbf{A}_{g}\omega_1(f)$$

where $\omega_1^*(f )$ is the vector of length $n_i$,
containing the predicted value $\omega_1^*( s_{g},f  )$
for spatial variation in the first linear predictor at every location
$s_{g}$, and other spatial variables are predicted similarly using
matrix $\mathbf{A}_{g}$. Predicted values for random effects are then
plugged into the linear predictor, e.g.:

$$p_{1}(g,c,t) = \underset{Temporal\ variation}{\overset{\beta_{1}^*(c) + \sum_{f = 1}^{n_{\beta_1}}{L_{\beta_1}(c,f)\beta_{1}(t,f)}}{︸}} + \underset{Spatial\ variation}{\overset{\sum_{f = 1}^{n_{\omega 1}}{L_{\omega 1}(x,f)\omega_1^*(g,f )}}{︸}} + \underset{Spatio - temporal\ variation}{\overset{\sum_{f = 1}^{n_{\varepsilon 1}}{L_{\varepsilon 1}(c,f)\varepsilon_{1}^*(g,f,t)}}{︸}} + \underset{Habitat\ covariates}{\overset{\sum_{p = 1}^{n_{p}}{( \gamma_{1}(c,t,p) + \sigma_{\xi 1}(c,p)\xi_{1}^*(g,c,p) )X(g,t,p)}}{︸}}$$

where $p_{2}(g,c,t)$ is predicted similar, and these linear predictors
are used in turn to predict $r_{1}(g,c,t)$ and $r_{2}(g,c,t)$, where
their product is predicted biomass-density $d(g,c,t)$ at every
extrapolation-grid cell $g$, category $c$, and time $t$.

By default, density is used to predict total abundance for the entire
domain (or a subset of the domain) for a given species:

$$I(c,t,l) = \sum_{x = 1}^{n_{x}}( a(g,l) \times d(g,c,t) )$$

where $a(g,l)$ is the area associated with extrapolation-grid cell $g$
for index $l$; and. The user can also specify additional post-hoc
calculations via the Options vector:

Options = c(\"SD_site_density\"=0, \"SD_site_logdensity\"=0,
\"Calculate_Range\"=0, \"Calculate_evenness\"=0,
\"Calculate_effective_area\"=0, \"Calculate_Cov_SE\"=0,
\'Calculate_Synchrony\'=0, \'Calculate_Coherence\'=0)

1.  *Distribution shift* -- RhoConfig\[3\]=1 turns on calculation of the
    centroid of the population's distribution:

$$Z(c,t,m) = \sum_{x = 1}^{n_{x}}\frac{( z(g,m) \times a(g,1) \times d(g,c,t) )}{I(c,t,1)}$$

where $z(g,m)$ is a matrix representing location for each
extrapolation-grid cell (by default $z(g,m)$ is the location in Eastings
and Northings of each knot), representing movement North-South and
East-West). This model-based approach to estimating distribution shift
can account for differences in the spatial distribution of sampling,
unlike conventional sample-based estimators (Thorson et al. 2016b).

2.  *Range expansion* -- RhoConfig\[5\]=1 turns on calculation of
    effective area occupied. This involves calculating biomass-weighted
    average density:

$$D(c,t,l) = \sum_{x = 1}^{n_{x}}{\frac{a(x,l) \times d(x,c,t)}{I(c,t,l)}d(x,c,t)}$$

Effective area occupied is then calculated as the area required to
contain the population at this average density:

$$A(c,t,l) = \frac{I(c,t,l)}{D(c,t,l)}$$

This effective-area occupied estimator can then be used to monitor range
expansion or contraction or density-dependent range expansion (Thorson
et al. 2016c).

The calculation of these and other derived quantities can be turned on
and off using input Options to function make_data (see reference
documentation for details regarding user interface).

## List of features

I next provide a list of "features" organized as decisions that can be
made by the analyst. Although this is somewhat redundant with the
explanations provided above, this list might be useful for some readers
to provide a high-level overview of different options that are
available. This "feature set" is also provided as a high-level summary
of what VAST is designed to be capable of doing; any software replacing
VAST would ideally include this same set of features.

*Basic features in a generalized linear model (GLM)*

1.  Specifying one of several possible distributions for data, including
    for:

    a.  Count data using a Poisson, negative-binomial,
        Conway-Maxwell-Poisson, or Poisson-lognormal distribution,
        including zero-inflated versions of each;

    b.  Continuous-valued data that include zeros using a delta-model
        with a lognormal or gamma distribution for positive values.

2.  Specifying one of several possible link functions for predicting
    data given linear predictors including:

    a.  A conventional delta-model;

    b.  A Poisson-link delta model.

3.  Including dynamic habitat covariates or not;

4.  Including catchability covariates or not;

*Basic features in a spatio-temporal generalized linear mixed model
(GLMM)*

5.  Specify an "extrapolation grid" using input
    FishStatsUtils::make_extrapolation_info(\..., Region), which is used
    to calculate the area associated with each knot $a_{x}$. This can be
    a user-specified extrapolation grid if
    FishStatsUtils::make_extrapolation_info(\..., Region="User",
    input_grid=Input), where Input is a data frame supplied by the user.

6.  Specifying a method for defining "knots";

7.  Specifying the number of "knots";

8.  Spatial variation being estimated ("turned on") or ignored ("turned
    off") for either linear predictor #1 or #2;

9.  Spatio-temporal variation being estimated ("turned on") or ignored
    ("turned off") for either linear predictor #1 or #2;

10. Specifying that habitat covariates can affect linear predictors
    different ways including as:

    a.  a linear effect;

    b.  a spatially-varying effect; or

    c.  both linear and spatially-varying effects simultaneously.

*Multivariate analysis*

11. Including a "multivariate" structure with multiple responses that
    covary due to a specified number of "factors" for spatial and
    spatio-temporal terms;

12. Rotate results prior to interpretation, using either:

    a.  principle components rotation; or

    b.  varimax rotation.

*Decisions regarding temporal structure*

13. Annual intercepts being structured over time, including:

    a.  estimated as fixed effects in every year;

    b.  fixed as fixed effect with the same value for all years;

    c.  estimated as a random effect with independent deviations in each
        year;

    d.  estimated as a random effect with first-order autoregressive
        structure; or

    e.  estimated as a random effect with a random-walk structure.

14. Spatio-temporal variation being structured over time, including:

    a.  estimated as independent deviations in each year;

    b.  estimated as following a first-order autoregressive structure
        over time;

    c.  estimated as following a random-walk structure over time; or

    d.  estimated as following a vector-autoregressive structure
        involving a matrix of 1^st^ order autoregressive interactions.

*Derived quantities*

15. Specifying spatial strata for use when calculating derived
    quantities;

16. Calculating one of many possible "univariate derived quantities",
    including:

    a.  abundance indices;

    b.  range shift;

    c.  effective area occupied

    d.  covariance among categories within a multivariate model; or

    e.  synchrony among categories.

17. Calculating "multivariate derived quantities" that are derived from
    estimates for multiple categories in a multivariate model, e.g.,
    where one category represents a standardized diet sample (e.g., prey
    biomass per predator biomass in a stomach-content sample) and
    another category represents a biomass-density sample (e.g., predator
    biomass in a bottom-trawl sample) such that their product represents
    predator-expanded consumption.

*Unusual circumstances and special cases*

18. Specifying separate distributions for different data sets (e.g.,
    when multiple surveys providing different data types are available);

19. Specifying that some data are predicted based on summing linear
    predictors across multiple variables (e.g., when modelling density
    for different size classes, and specifying that some data are
    aggregated measurements of multiple sizes-classes);

20. Specifying multiple "seasons" (e.g., when modelling data with both
    annual and monthly spatio-temporal variation).

## Common problems

There are two basic problems that are often encountered during
spatio-temporal delta-GLMMs:

1.  *Encounter rates*: Some combination of categories and year has 0% or
    100% encounter rate. If there is 100% encounter rate for category
    $c$ in year $t$, then $\beta_{1}(c,t) \rightarrow \infty$ and/or
    $\varepsilon_{1}(s,c,t) \rightarrow \infty$ for that year. If there
    is 0% encounter rate in year $t$, then
    $\beta_{1}(c,t) \rightarrow - \infty$ and/or
    $\varepsilon_{1}(s,c,t) \rightarrow - \infty$ and there is no
    information to estimate $\beta_{2}(c,t)$ or $\varepsilon_{2}(s,c,t)$
    for that category $c$ and year $t$;

2.  *Bounds*: Some parameter(s) hits a bound;

These problems can be solved by:

1.  *Encounter rates*: constraining terms that vary among years (e.g.,
    intercept $\beta$ and spatio-temporal variation
    $\varepsilon(s,t,p)$). This can be done in many different ways that
    are each idiosyncratic and require some special justification. The
    easiest options are:

    a.  If there is a small number of years with 100% encounter rate,
        try ObsModel\[2\]=3. This indicates that VAST should check for
        species-years combinations with 100% encounter rates and fix
        corresponding intercepts for encounter probability to an
        extremely high value.

    b.  If there is a small number of years with either 100% of 0%
        encounter rate, add temporal structure to intercepts and
        spatio-temporal terms using RhoConfig options.

    c.  Four other options are listed on the
        [wiki](https://github.com/nwfsc-assess/geostatistical_delta-GLMM/wiki/What-to-do-with-a-species-with-0%25-or-100%25-encounters-in-any-year).

2.  *Bounds*: Please try running the model without estimating standard
    errors or a final newton step:

> \# Specify derived quantities to calculate

TMBhelper::fit_tmb( \..., getsd=FALSE, newtonsteps=0 )

Then check what parameters are being estimated near an upper or lower
boundary.

## How to implement basic model changes

There are a few basic model types that users often want to fit using
VAST. I briefly describe how these can be done here.

1.  *Fitting encounter/non-encounter data*: If the user wishes to use
    only the first component of a delta-model, i.e., to fit a binomial
    model to simply predict encounter probabilities, then, the ObsModel
    vector should be set to c(\"PosDist\"=\[Make Choice\], \"Link\"=0),
    where \[Make Choice\] can be any option for continuous data (i.e.,
    0, 1, or 2). The user should then turn off the last two elements of
    the FieldConfig vector (i.e., FieldConfig\[3\]=0 and
    FieldConfig\[4\]=0) such that there is no spatial or spatio-temporal
    variability in positive catch rates, and also turn off annual
    variation in the intercept for positive catch rates (i.e.,
    RhoConfig\[2\]=3). Finally, the user should "jitter" their presence
    observations by a very small amount (i.e., add a random normal
    deviation with a very small standard deviation,
    rnorm(n=1,mean=0,sd=0.001), to each observation for which b_i=1).
    This will result in VAST estimating a logistic regression model for
    encounter/non-encounter data, except with one additional parameter
    estimated ($\sigma_{M}$), plus one additional parameter per category
    ($\beta_{2}(c)$), where these additional parameters have no impact
    on other parameters, are not meant to be interpreted statistically
    or biologically, and are an artefact of using VAST (which is
    designed to fit a delta-model) to encounter/non-encounter data. This
    feature has been used to estimate species distributions for use in
    ecosystem models (Grüss et al. 2017, 2018).

## Acknowledgements

I thank K. Kristensen, H. Skaug, and the developers of Template Model
Builder, without which this research and resulting R package VAST would
not be possible. I also thank the many collaborators who have
contributed to developing features (see
<https://github.com/nwfsc-assess/geostatistical_delta-GLMM/wiki/Applications>),
as well as the funding sources that have supported development (see
<https://github.com/James-Thorson-NOAA/VAST#funding-and-support-for-the-tool>).
In particular, I think C. Monnahan and M. Rudd for contributing
substantially to coding new features, and A. Gruss for identifying
indexing errors in several (little used) features. I also thank the many
volunteers and NOAA scientists who have served on sampling vessels that
provided data to test these methods. Finally, I think A. Grüss and S.
Hoyle for providing edits to this document.

## Works cited

Edwards, A.M., and Auger‐Méthé, M. 2019. Some guidance on using
mathematical notation in ecology. Methods Ecol. Evol. **10**(1): 92--99.
doi:10.1111/2041-210X.13105.

Engle, R.F., and Granger, C.W. 1987. Co-integration and error
correction: representation, estimation, and testing. Econom. J. Econom.
Soc.: 251--276.

Gelman, A., and Hill, J. 2007. Data analysis using regression and
multilevel/hierarchical models. Cambridge University Press, Cambridge,
UK.

Godefroid, M., Boldt, J.L., Thorson, J.T., Forrest, R., Gauthier, S.,
Flostrand, L., Ian Perry, R., Ross, A.R.S., and Galbraith, M. 2019.
Spatio-temporal models provide new insights on the biotic and abiotic
drivers shaping Pacific Herring (Clupea pallasi) distribution. Prog.
Oceanogr. **178**: 102198. doi:10.1016/j.pocean.2019.102198.

Grüss, A., and Thorson, J.T. 2019. Developing spatio-temporal models
using multiple data types for evaluating population trends and habitat
usage. ICES J. Mar. Sci. **76**(6): 1748--1761.
doi:10.1093/icesjms/fsz075.

Grüss, A., Thorson, J.T., Babcock, E.A., and Tarnecki, J.H. 2018.
Producing distribution maps for informing ecosystem-based fisheries
management using a comprehensive survey database and spatio-temporal
models. ICES J. Mar. Sci. **75**(1): 158--177.
doi:10.1093/icesjms/fsx120.

Grüss, A., Thorson, J.T., Sagarese, S.R., Babcock, E.A., Karnauskas, M.,
Walter, J.F., and Drexler, M. 2017. Ontogenetic spatial distributions of
red grouper (Epinephelus morio) and gag grouper (Mycteroperca
microlepis) in the U.S. Gulf of Mexico. Fish. Res. **193**(Supplement
C): 129--142. doi:10.1016/j.fishres.2017.04.006.

Hocking, D.J., Thorson, J.T., O'Neil, K., and Letcher, B.H. 2018. A
geostatistical state-space model of animal densities for stream
networks. Ecol. Appl. **28**(7): 1782--1796. doi:10.1002/eap.1767.

Kass, R.E., and Steffey, D. 1989. Approximate Bayesian inference in
conditionally independent hierarchical models (parametric empirical
bayes models). J. Am. Stat. Assoc. **84**(407): 717--726.
doi:10.2307/2289653.

Kristensen, K., Nielsen, A., Berg, C.W., Skaug, H., and Bell, B.M. 2016.
TMB: Automatic differentiation and Laplace approximation. J. Stat.
Softw. **70**(5): 1--21. doi:10.18637/jss.v070.i05.

Lindgren. 2012. Continuous domain spatial models in R-INLA. ISBA Bull.
**19**(4): 14--20.

Lindgren, F., and Rue, H. 2015. Bayesian spatial modelling with r-inla.
J. Stat. Softw. **63**(19): 1--25. doi:10.18637/jss.v063.i19.

Lindgren, Rue, H., and Lindström, J. 2011. An explicit link between
Gaussian fields and Gaussian Markov random fields: the stochastic
partial differential equation approach. J. R. Stat. Soc. Ser. B Stat.
Methodol. **73**(4): 423--498. doi:10.1111/j.1467-9868.2011.00777.x.

Martin, T.G., Wintle, B.A., Rhodes, J.R., Kuhnert, P.M., Field, S.A.,
Low-Choy, S.J., Tyre, A.J., and Possingham, H.P. 2005. Zero tolerance
ecology: improving ecological inference by modelling the source of zero
observations. Ecol. Lett. **8**(11): 1235--1246.

R Core Team. 2017. R: A Language and Environment for Statistical
Computing. R Foundation for Statistical Computing, Vienna, Austria.
Available from https://www.R-project.org/.

Searle, S.R., Casella, G., and McCulloch, C.E. 1992. Variance
components. John Wiley & Sons, Hoboken, New Jersey.

Shelton, A.O., Thorson, J.T., Ward, E.J., and Feist, B.E. 2014. Spatial
semiparametric models improve estimates of species abundance and
distribution. Can. J. Fish. Aquat. Sci. **71**(11): 1655--1666.
doi:10.1139/cjfas-2013-0508.

Skaug, H., and Fournier, D. 2006. Automatic approximation of the
marginal likelihood in non-Gaussian hierarchical models. Comput. Stat.
Data Anal. **51**(2): 699--709.

Thorson, J.T. 2018. Three problems with the conventional delta-model for
biomass sampling data, and a computationally efficient alternative. Can.
J. Fish. Aquat. Sci. **75**(9): 1369--1382. doi:10.1139/cjfas-2017-0266.

Thorson, J.T. 2019. Guidance for decisions using the Vector
Autoregressive Spatio-Temporal (VAST) package in stock, ecosystem,
habitat and climate assessments. Fish. Res. **210**: 143--161.
doi:10.1016/j.fishres.2018.10.013.

Thorson, J.T., Adams, G., and Holsman, K. 2019. Spatio-temporal models
of intermediate complexity for ecosystem assessments: A new tool for
spatial fisheries management. Fish Fish. **20**(6): 1083--1099.
doi:10.1111/faf.12398.

Thorson, J.T., and Barnett, L.A.K. 2017. Comparing estimates of
abundance trends and distribution shifts using single- and multispecies
models of fishes and biogenic habitat. ICES J. Mar. Sci. **74**(5):
1311--1321. doi:10.1093/icesjms/fsw193.

Thorson, J.T., Ciannelli, L., and Litzow, M.A. 2020. Defining indices of
ecosystem variability using biological samples of fish communities: A
generalization of empirical orthogonal functions. Prog. Oceanogr.
**181**: 102244. doi:10.1016/j.pocean.2019.102244.

Thorson, J.T., and Haltuch, M.A. 2018. Spatiotemporal analysis of
compositional data: increased precision and improved workflow using
model-based inputs to stock assessment. Can. J. Fish. Aquat. Sci.
**76**(3): 401--414. doi:10.1139/cjfas-2018-0015.

Thorson, J.T., Ianelli, J.N., Larsen, E.A., Ries, L., Scheuerell, M.D.,
Szuwalski, C., and Zipkin, E.F. 2016a. Joint dynamic species
distribution models: a tool for community ordination and spatio-temporal
monitoring. Glob. Ecol. Biogeogr. **25**(9): 1144--1158.
doi:10.1111/geb.12464.

Thorson, J.T., and Kristensen, K. 2016. Implementing a generic method
for bias correction in statistical models using random effects, with
spatial and population dynamics examples. Fish. Res. **175**: 66--74.
doi:10.1016/j.fishres.2015.11.016.

Thorson, J.T., and Minto, C. 2015. Mixed effects: a unifying framework
for statistical modelling in fisheries biology. ICES J. Mar. Sci. J.
Cons. **72**(5): 1245--1256. doi:10.1093/icesjms/fsu213.

Thorson, J.T., Munch, S.B., and Swain, D.P. 2017. Estimating partial
regulation in spatiotemporal models of community dynamics. Ecology
**98**(5): 1277--1289. doi:10.1002/ecy.1760.

Thorson, J.T., Pinsky, M.L., and Ward, E.J. 2016b. Model-based inference
for estimating shifts in species distribution, area occupied and centre
of gravity. Methods Ecol. Evol. **7**(8): 990--1002.
doi:10.1111/2041-210X.12567.

Thorson, J.T., Rindorf, A., Gao, J., Hanselman, D.H., and Winker, H.
2016c. Density-dependent changes in effective area occupied for
sea-bottom-associated marine fishes. Proc R Soc B **283**(1840):
20161853. doi:10.1098/rspb.2016.1853.

Thorson, J.T., Scheuerell, M.D., Olden, J.D., and Schindler, D.E. 2018.
Spatial heterogeneity contributes more to portfolio effects than species
variability in bottom-associated marine fishes. Proc R Soc B
**285**(1888): 20180915. doi:10.1098/rspb.2018.0915.

Thorson, J.T., Shelton, A.O., Ward, E.J., and Skaug, H.J. 2015a.
Geostatistical delta-generalized linear mixed models improve precision
for estimated abundance indices for West Coast groundfishes. ICES J.
Mar. Sci. J. Cons. **72**(5): 1297--1310. doi:10.1093/icesjms/fsu243.

Thorson, J.T., Skaug, H.J., Kristensen, K., Shelton, A.O., Ward, E.J.,
Harms, J.H., and Benante, J.A. 2014. The importance of spatial models
for estimating the strength of density dependence. Ecology **96**(5):
1202--1212. doi:10.1890/14-0739.1.

Thorson, Scheuerell, M.D., Shelton, A.O., See, K.E., Skaug, H.J., and
Kristensen, K. 2015b. Spatial factor analysis: a new tool for estimating
joint species distributions and correlations in species range. Methods
Ecol. Evol. **6**(6): 627--637. doi:10.1111/2041-210X.12359.

Tierney, L., Kass, R.E., and Kadane, J.B. 1989. Fully exponential
Laplace approximations to expectations and variances of nonpositive
functions. J. Am. Stat. Assoc. **84**(407): 710--716.

**\
**

Table 1 -- List of S3 objects defined in package VAST (or its primary
dependency FishStatsUtils), listing S3 methods defined for each class as
well as the intended purpose of each method.

  ---------------------------------------------------------------------------------------------
  **S3 object**                             **S3        **Purpose**
                                            methods**   
  ----------------------------------------- ----------- ---------------------------------------
  VAST::make_data                           print       De-clutter terminal output

  VAST::make_model                          print       De-clutter terminal output

  FishStatsUtils::make_extrapolation_info   print       De-clutter terminal output

                                            plot        Simple organization for plotting
                                                        options

  FishStatsUtils::make_spatial_info         print       De-clutter terminal output

                                            print       Simple organization for plotting
                                                        options

  FishStatsUtils::fit_model                 print       De-clutter terminal output

                                            plot        Simple organization for plotting
                                                        options

                                            summary     Interface to access derived quantities
                                                        that users may want
  ---------------------------------------------------------------------------------------------

Table 2 -- Definition of mathematical notation, including the symbol
used, its type (Index, Data, fixed effects "FE", random effects "RE",
intermediate quantity computed internally "IQ", and derived quantities
that are outputted for users "DQ"), and its dimension

Table 2A -- Indices

  -------------------------------------------------------------------------
  Index name                                                    Symbol
  ------------------------------------------------------------- -----------
  Observation number                                            $$i$$

  Extrapolation-grid cell                                       $$g$$

  Knot number                                                   $$x$$

  Vertex number (including internal knots and boundary          $$s$$
  vertices)                                                     

  Time interval number                                          $$t$$

  Category number                                               $$c$$

  Factor number                                                 $$f$$

  Habitat covariate number for 1^st^ linear predictor           $$p_{1}$$

  Habitat covariate number for 2^nd^ linear predictor           $$p_{2}$$

  Catchability covariate number for 1^st^ linear predictor      $$k_{1}$$

  Catchability covariate number for 2^nd^ linear predictor      $$k_{2}$$

  Stratum number                                                $$l$$

  Index number for measures of center-of-gravity                $$m$$

  Index number for other book-keeping                           $$z$$
  -------------------------------------------------------------------------

Table 2B -- Data

  -----------------------------------------------------------------------------------------------------------
  Index name                                    Symbol                 Dimensions
  --------------------------------------------- ---------------------- --------------------------------------
  Sample response                               $$b_i$$              $$n_i$$

  Time interval for each sample                 $$t_i$$              $$n_i$$

  Category for each sample                      $$c_i$$              $$n_i$$

  Overdispersion level for each sample          $$v_i$$              $$n_i$$

  Area covered by each sample                   $$a_i$$              $$n_i$$

  Bilinear interpolation from vertices to       $$\mathbf{A}_{is}$$    $$n_i \times 3$$
  samples                                                              

  Bilinear interpolation from vertices to       $$\mathbf{A}_{gs}$$    $$n_{g} \times 3$$
  extrapolation-grid cells                                             

  Distance between two vertices                 $$d(s,s^{'})$$         $$n_{s} \times n_{s}$$

  Habitat covariates affecting 1^st^ linear     $$X_{1}(i,t,p_{1})$$   $$n_i \times n_{t} \times n_{p1}$$
  predictor for each sampling location, time,                          
  and variable                                                         

  Habitat covariates affecting 2^nd^ linear     $$X_{2}(i,t,p_{2})$$   $$n_i \times n_{t} \times n_{p2}$$
  predictor for each sampling location, time,                          
  and variable                                                         

  Habitat covariates affecting 1^st^ linear     $$X_{1}(g,t,p_{1})$$   $$n_{g} \times n_{t} \times n_{p1}$$
  predictor for each extrapolation-grid cell,                          
  time, and variable                                                   

  Habitat covariates affecting 2^nd^ linear     $$X_{2}(g,t,p_{2})$$   $$n_{g} \times n_{t} \times n_{p2}$$
  predictor for each extrapolation-grid cell,                          
  time, and variable                                                   

  Catchability covariates affecting 1^st^       $$Q_{1}(i,k_{1})$$     $$n_i \times n_{k1}$$
  linear predictor for each sample and variable                        

  Catchability covariates affecting 2^nd^       $$Q_{2}(i,k_{2})$$     $$n_i \times n_{k2}$$
  linear predictor for each sample and variable                        

  Area associated with extrapolation-grid cell  $$a(g,l)$$             $$n_{g} \times n_{l}$$
  in each stratum                                                      

  Statistic for each location used to calculate $$z(s,m)$$             $$n_{s} \times n_{m}$$
  center of gravity and range edges                                    
  -----------------------------------------------------------------------------------------------------------

Table 2C -- Coefficients, indicating whether they are fixed effects
(FE), random effects (RE) or whether their treatment depends upon user
settings (FE/RE)

  -----------------------------------------------------------------------------------------------------------------------------------------------------------
  Coefficient name                           Symbol                              Type    Dimensions
  ------------------------------------------ ----------------------------------- ------- --------------------------------------------------------------------
  Factor values for intercept for 1^st^      $$\beta_{1}(f,t)$$                  FE/RE   $$n_{\beta_1} \times n_{t}$$
  linear predictor                                                                       

  Factor values for intercept for 2^st^      $$\beta_{2}(f,t)$$                  FE/RE   $$n_{\beta_2} \times n_{t}$$
  linear predictor                                                                       

  Loadings matrix for intercepts for 1^st^   $$L_{\beta_1}(c,f)$$                FE      $$n_{c} \times n_{\beta_1}$$
  linear predictor                                                                       

  Loadings matrix for intercepts for 2^nd^   $$L_{\beta_2}(c,f)$$                FE      $$n_{c} \times n_{\beta_2}$$
  linear predictor                                                                       

  Loadings matrix for spatial covariation    $$L_{\omega 1}(c,f)$$               FE      $$n_{c} \times n_{\omega 1}$$
  for 1^st^ linear predictor                                                             

  Loadings matrix for spatial covariation    $$L_{\omega 2}(c,f)$$               FE      $$n_{c} \times n_{\omega 2}$$
  for 2^nd^ linear predictor                                                             

  Loadings matrix for spatio-temporal        $$L_{\varepsilon 1}(c,f)$$          FE      $$n_{c} \times n_{\varepsilon 1}$$
  covariation across categories for 1^st^                                                
  linear predictor                                                                       

  Loadings matrix for spatio-temporal        $$L_{\varepsilon 2}(c,f)$$          FE      $$n_{c} \times n_{\varepsilon 2}$$
  covariation across categories for 2^nd^                                                
  linear predictor                                                                       

  Loadings matrix for spatio-temporal        $$L_{\varepsilon 1}^{time}(t,f)$$   FE      $$n_{t} \times n_{\varepsilon 1}^{time}$$
  covariation across time for 1^st^ linear                                               
  predictor                                                                              

  Loadings matrix for spatio-temporal        $$L_{\varepsilon 2}^{time}(t,f)$$   FE      $$n_{t} \times n_{\varepsilon 2}^{time}$$
  covariation across time for 2^nd^ linear                                               
  predictor                                                                              

  Loadings matrix for overdispersion         $$L_{1}(c,f)$$                      FE      $$n_{c} \times n_{\eta 1}$$
  covariation for 1^st^ linear predictor                                                 

  Loadings matrix for overdispersion         $$L_{2}(c,f)$$                      FE      $$n_{c} \times n_{\eta 2}$$
  covariation for 2^nd^ linear predictor                                                 

  Impact of habitat covariates on 1^st^      $$\gamma_{1}(c,t,p)$$               FE      $$n_{c} \times n_{t} \times n_{p}$$
  linear predictor                                                                       

  Impact of habitat covariates on 2^nd^      $$\gamma_{2}(c,t,p)$$               FE      $$n_{c} \times n_{t} \times n_{p}$$
  linear predictor                                                                       

  Impact of catchability covariates on 1^st^ $$\lambda_{1}(k)$$                  FE      $$n_{k}$$
  linear predictor                                                                       

  Impact of catchability covariates on 2^nd^ $$\lambda_{2}(k)$$                  FE      $$n_{k}$$
  linear predictor                                                                       

  Parameters governing residual variation    $$\sigma_{m}^{2}(c,z)$$             FE      $$n_{c} \times 2$$

  Decorrelation rate for 1^st^ linear        $$\kappa_{1}$$                      FE      1
  predictor                                                                              

  Decorrelation rate for 2^nd^ linear        $$\kappa_{2}$$                      FE      1
  predictor                                                                              

  Autocorrelation for intercepts of 1^st^    $$\rho_{\beta_1}$$                  FE      1
  linear predictor                                                                       

  Autocorrelation for intercepts of 2^nd^    $$\rho_{\beta_2}$$                  FE      1
  linear predictor                                                                       

  Conditional variance for intercepts of     $$\sigma_{\beta_1}^{2}$$            FE      1
  1^st^ linear predictor                                                                 

  Conditional variance for intercepts of     $$\sigma_{\beta_2}^{2}$$            FE      1
  2^nd^linear predictor                                                                  

  Autocorrelation for spatio-temporal        $$\rho_{\varepsilon 1}$$            FE      1
  covariation of 1^st^ linear predictor                                                  

  Autocorrelation for spatio-temporal        $$\rho_{\varepsilon 2}$$            FE      1
  covariation of 2^nd^ linear predictor                                                  

  Parameters governing geometric anisotropy  $$h(z)$$                            FE      2

  Spatial factors for 1^st^ linear predictor $$\omega_1(s,f)$$                 RE      $$n_{s} \times n_{\omega 1}$$

  Spatial factors for 2^nd^ linear predictor $$\omega_2(s,f)$$                 RE      $$n_{s} \times n_{\omega 2}$$

  Spatio-temporal factors for 1^st^ linear   $$\varepsilon_{1}(s,f,f)$$          RE      $$n_{s} \times n_{\varepsilon 1} \times n_{\varepsilon 1}^{time}$$
  predictor                                                                              

  Spatio-temporal factors for 2^nd^ linear   $$\varepsilon_{2}(s,f,f)$$          RE      $$n_{s} \times n_{\varepsilon 1} \times n_{\varepsilon 1}^{time}$$
  predictor                                                                              

  Overdispersion factors for 1^st^ linear    $$\eta_{1}(v,f)$$                   RE      $$n_{v} \times n_{\eta 1}$$
  predictor                                                                              

  Overdispersion factors for 2^nd^ linear    $$\eta_{2}(v,f)$$                   RE      $$n_{v} \times n_{\eta 2}$$
  predictor                                                                              
  -----------------------------------------------------------------------------------------------------------------------------------------------------------

Table 2D -- Variable calculated internally

  -------------------------------------------------------------------------------------------------------------------
  Coefficient name                                 Symbol                       Dimensions
  ------------------------------------------------ ---------------------------- -------------------------------------
  1^st^ linear predictor                           $$p_{1}(i)$$                 $$n_i$$

  2^nd^ linear predictor                           $$p_{2}(i)$$                 $$n_i$$

  1^st^ link-transformed predictor                 $$r_{1}(i)$$                 $$n_i$$

  2^nd^ link-transformed predictor                 $$r_{2}(i)$$                 $$n_i$$

  Spatio-temporal variation for 1^st^ linear       $$\varepsilon_{1}(g,c,t)$$   $$n_{g} \times n_{c} \times n_{t}$$
  predictor at each extrapolation-grid cell                                     

  Spatio-temporal variationfor 2^nd^ linear        $$\varepsilon_{2}(g,c,t)$$   $$n_{g} \times n_{c} \times n_{t}$$
  predictor at each extrapolation-grid cell                                     

  Spatio-temporal variation for 1^st^ linear       $$\varepsilon_{1}(i,c,t)$$   $$n_i \times n_{c} \times n_{t}$$
  predictor at each sample                                                      

  Spatio-temporal variationfor 2^nd^ linear        $$\varepsilon_{2}(i,c,t)$$   $$n_i \times n_{c} \times n_{t}$$
  predictor at each sample                                                      

  Spatial variation for 1^st^ linear predictor at  $$\omega_1(g,c)$$          $$n_{g} \times n_{c}$$
  each extrapolation-grid cell                                                  

  Spatial variation for 2^nd^ linear predictor at  $$\omega_2(g,c)$$          $$n_{g} \times n_{c}$$
  each extrapolation-grid cell                                                  

  Spatial variation for 1^st^ linear predictor at  $$\omega_1(i,c)$$          $$n_i \times n_{c}$$
  each sample                                                                   

  Spatial variation for 2^nd^ linear predictor at  $$\omega_2(i,c)$$          $$n_i \times n_{c}$$
  each sample                                                                   

  Intercept for 1^st^ linear predictor             $$\beta_{1}(c,t)$$           $$n_{c} \times n_{t}$$

  Intercept for 2^st^ linear predictor             $$\beta_{2}(c,t)$$           $$n_{c} \times n_{t}$$

  Spatial correlation matrix among vertices for    $$\mathbf{R}_{1}$$           $$n_{s} \times n_{s}$$
  1^st^ linear predictor                                                        

  Spatial correlation matrix among vertices for    $$\mathbf{R}_{2}$$           $$n_{s} \times n_{s}$$
  2^nd^ linear predictor                                                        

  Anisotropy matrix                                $$\mathbf{H}$$               $$2 \times 2$$
  -------------------------------------------------------------------------------------------------------------------

Table 2E -- Derived quantities

  ----------------------------------------------------------------------------------------------------------------------------------
  Coefficient name                                Symbol                           Dimensions
  ----------------------------------------------- -------------------------------- -------------------------------------------------
  Predicted density for each sample               $$d^*(i,c,t)$$                 $$n_i \times n_{c} \times n_{t}$$

  Predicted density for each extrapolation-grid   $$d^*(g,c,t)$$                 $$n_{g} \times n_{c} \times n_{t}$$
  cell                                                                             

  Index of abundance                              $$I(c,t,l)$$                     $$n_{c} \times n_{t} \times n_{l}$$

  Center of gravity                               $$Z(c,t,m)$$                     $$n_{c} \times n_{t} \times n_{m}$$

  Average density                                 $$D(c,t,l)$$                     $$n_{c} \times n_{t} \times n_{l}$$

  Effective area occupied                         $$A(c,t,l)$$                     $$n_{c} \times n_{t} \times n_{l}$$

  Rotation matrix for spatial covariation for     $$\mathbf{B}_{\omega 1}$$        $$n_{c} \times n_{c}$$
  1^st^ linear predictor                                                           

  Rotation matrix for spatial covariation for     $$\mathbf{B}_{\omega 2}$$        $$n_{c} \times n_{c}$$
  2^nd^linear predictor                                                            

  Rotation matrix for spatio-temporal covariation $$\mathbf{B}_{\varepsilon 1}$$   $$n_{c} \times n_{c}$$
  for 1^st^ linear predictor                                                       

  Rotation matrix for spatio-temporal covariation $$\mathbf{B}_{\varepsilon 2}$$   $$n_{c} \times n_{c}$$
  for 2^nd^ linear predictor                                                       

  Rotation matrix for overdispersion covariation  $$\mathbf{B}_{1}$$               $$n_{c} \times n_{c}$$
  for 1^st^ linear predictor                                                       

  Rotation matrix for overdispersion covariation  $$\mathbf{B}_{2}$$               $$n_{c} \times n_{c}$$
  for 2^nd^linear predictor                                                        

  Rotated loadings matrix for spatial covariation $$L_{\omega 1}^*(c,f)$$        $$n_{c} \times n_{\omega 1}$$
  for 1^st^ linear predictor                                                       

  Rotated loadings for spatial covariation for    $$L_{\omega 2}^*(c,f)$$        $$n_{c} \times n_{\omega 2}$$
  2^nd^ linear predictor                                                           

  Rotated loadings for spatio-temporal            $$L_{\varepsilon 1}^*(c,f)$$   $$n_{c} \times n_{\varepsilon 1}$$
  covariation for 1^st^ linear predictor                                           

  Rotated loadings for spatio-temporal            $$L_{\varepsilon 2}^*(c,f)$$   $$n_{c} \times n_{\varepsilon 2}$$
  covariation for 2^nd^ linear predictor                                           

  Rotated loadings for overdispersion covariation $$L_{1}^*(c,f)$$               $$n_{c} \times n_{\eta 1}$$
  for 1^st^ linear predictor                                                       

  Rotated loadings for overdispersion covariation $$L_{2}^*(c,f)$$               $$n_{c} \times n_{\eta 2}$$
  for 2^nd^ linear predictor                                                       

  Rotated spatial factors for 1^st^ linear        $$\omega_1^*(s,f)$$          $$n_{s} \times n_{\omega 1}$$
  predictor                                                                        

  Rotated spatial factors for 2^nd^ linear        $$\omega_2^*(s,f)$$          $$n_{s} \times n_{\omega 2}$$
  predictor                                                                        

  Rotated spatio-temporal factors for 1^st^       $$\varepsilon_{1}^*(s,f,t)$$   $$n_{s} \times n_{\varepsilon 1} \times n_{t}$$
  linear predictor                                                                 

  Rotated spatio-temporal factors for 2^nd^       $$\varepsilon_{2}^*(s,f,t)$$   $$n_{s} \times n_{\varepsilon 1} \times n_{t}$$
  linear predictor                                                                 

  Rotated overdispersion factors for 1^st^ linear $$\eta_{1}^*(v,f)$$            $$n_{v} \times n_{\eta 1}$$
  predictor                                                                        

  Rotated overdispersion factors for 2^nd^ linear $$\eta_{2}^*(v,f)$$            $$n_{v} \times n_{\eta 2}$$
  predictor                                                                        
  ----------------------------------------------------------------------------------------------------------------------------------
