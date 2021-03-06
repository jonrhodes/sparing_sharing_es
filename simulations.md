Simple Sparing Sharing Ecosystem Service Models
================

## Introduction

In this document we develop simple models of the effect of land sparing
and land sharing on ecosystem service provision.

At the local scale we assume that the relationships between land use
intensity and supply and demand follow a power function and declines
with land use intensity for supply, but can decline, increase, or not
change with land use intensity for demand.

We then assume that land use intensities at locations have frequency
distributions following a beta distribution with the precision parameter
controlling the level of sharing (high precision) and sparing (low
precision).

## Setting up Supply and Demand Functions

First we set up the functions that define the relationship between
supply and demand and land use intensity

``` r
supply_demand <- function(Intensity, Direction, Gamma) {
    # Intensity specific the land-use intensity (values between 0 and 1)
    # Direction = -1 for negative association
    # Direction = 0 no association
    # Direction = 1 for positive association
    # Gamma specifies the shape
    ifelse(Direction == 0, return(1), ifelse(Direction == -1, return((1 - Intensity) ^ Gamma),return(Intensity ^ Gamma)))
}
```

Then we set up the function that defines the association between
connectivity weight (flow) and the differences between intensity values.
When the relationship between the weight and differences in intensity
values is \> 0 then this reflects a clustered landscape. When the
relationship between the weight and differences in intensity values is
\< 0 then this reflects a dispersed (fragmented) landscape. When the
relationship between the weight and differences in intensity values is 0
then this reflects a random landscape.

``` r
connectivity <- function(Diff, R) {
    # Diff is the absolute difference in intensity values
    # R specifies direction and size of the relationship between connectivity weight and the difference in intensity values
    return(exp(R * (Diff - 0.5)) / (1 + exp(R * (Diff - 0.5))))
}
```

Then we set up the function for calculating the landscape level
provision based on the distribution of land use
intensities

``` r
f <- function(x, Mean, VarLand, SDirection, SGamma, DDirection, DGamma, R, VarCon) {
    # x is a 3 x 1 vector of parameters we are integrating over (i.e., intensities for supply, intensities for Demand
    # and differences in intensity values)
    # Mean is the mean Intensity
    # VarLand is variance of land use intensities, so where you are on the sparing/sharing gradient (low = sharing, high = sparing)
    # SDirection is the direction of the supply/intensity relationship (-1,0,1)
    # SGamma is the shape of the supply/intensity relationship
    # DDirection is the direction of the demand/intensity relationship (-1,0,1)
    # DGamma is the shape of the demand/intensity relationship
    # R specifies the relationship between connectivity weight and the difference in intensity values
    # VarCon is the variance of the connectivity weight values given and expected value   

    # Calculate alpha and beta for distribution of sparing/sharing
    AlphaL <- (((1 - Mean) * Mean ^ 2) - Mean * VarLand) / VarLand
    BetaL <- ((1 - Mean) * (Mean - (Mean ^ 2) - VarLand)) / VarLand

    # Calculate alpha and beta for distribution of the expected connectivity weights
    AlphaC <- ((1 - connectivity(abs(x[1] - x[2]), R)) * connectivity(abs(x[1] - x[2]), R) ^ 2 - connectivity(abs(x[1] - x[2]), R) * VarCon) / VarCon
    BetaC <- ((1 - connectivity(abs(x[1] - x[2]), R)) * (connectivity(abs(x[1] - x[2]), R) - connectivity(abs(x[1] - x[2]), R) ^ 2 - VarCon)) / VarCon

    #get supply, demand, and connectivity weight
    Supply <- supply_demand(x[1], SDirection, SGamma)
    Demand <- supply_demand(x[2], DDirection, DGamma)
    Conn <- connectivity(x[3], R)

    return(Supply * Conn * Demand * dbeta(x[1], AlphaL, BetaL) * dbeta(x[2], AlphaL, BetaL) * dbeta(Conn, AlphaC, BetaC))
}
```

Now we integrate numerically over the distribution of intensities for
supply and demand, and the distribution of connectivity values to get
the expected provision (here we just show an example for
now)

``` r
    Test <- adaptIntegrate(f, lowerLimit = c(0, 0, 0), upperLimit = c(1, 1, 1), Mean = 0.5, VarLand = 0.05 , SDirection = -1, SGamma = 1,
                                            DDirection = 0, DGamma = 1, R = 2, VarCon = 0.05)
```
