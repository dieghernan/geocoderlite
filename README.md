
<!-- README.md is generated from README.Rmd. Please edit that file -->

# nominatimlite <a href='https://dieghernan.github.io/nominatimlite/'><img src='man/figures/logo.png' align="right" height="139" /></a>

<!-- badges: start -->

[![CRAN
status](https://www.r-pkg.org/badges/version/nominatimlite)](https://CRAN.R-project.org/package=nominatimlite)
[![CRAN
results](https://badges.cranchecks.info/worst/nominatimlite.svg)](https://cran.r-project.org/web/checks/check_results_nominatimlite.html)
[![Downloads](https://cranlogs.r-pkg.org/badges/nominatimlite)](https://CRAN.R-project.org/package=nominatimlite)
[![R-CMD-check](https://github.com/dieghernan/nominatimlite/actions/workflows/check-full.yaml/badge.svg)](https://github.com/dieghernan/nominatimlite/actions/workflows/check-full.yaml)
[![codecov](https://codecov.io/gh/dieghernan/nominatimlite/branch/main/graph/badge.svg?token=jSZ4RIsj91)](https://app.codecov.io/gh/dieghernan/nominatimlite)
[![r-universe](https://dieghernan.r-universe.dev/badges/nominatimlite)](https://dieghernan.r-universe.dev/)
[![Project Status: Active – The project has reached a stable, usable
state and is being actively
developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)
[![DOI](https://img.shields.io/badge/DOI-10.5281/zenodo.5113195-blue)](https://doi.org/10.5281/zenodo.5113195)
[![status](https://tinyverse.netlify.com/badge/nominatimlite)](https://CRAN.R-project.org/package=nominatimlite)

<!-- badges: end -->

The goal of `nominatimlite` is to provide a light interface for
geocoding addresses, based on the [Nominatim
API](https://nominatim.org/release-docs/latest/). **Nominatim** is a
tool to search [OpenStreetMap](https://www.openstreetmap.org/) data by
name and address
([geocoding](https://wiki.openstreetmap.org/wiki/Geocoding "Geocoding"))
and to generate synthetic addresses of OSM points (reverse geocoding).

It also allows to load spatial objects using the `sf` package.

Full site with examples and vignettes on
<https://dieghernan.github.io/nominatimlite/>

## Why `nominatimlite`?

The main goal of `nominatimlite` is to access the Nominatim API avoiding
the dependency on `curl`. In some situations, `curl` may not be
available or accessible, so `nominatimlite` uses base functions to
overcome this limitation.

## Recommended packages

There are other packages much more complete and mature than
`nominatimlite`, that presents similar features:

- [`tidygeocoder`](https://jessecambon.github.io/tidygeocoder/) by Jesse
  Cambon. Allows to interface with Nominatim, Google, TomTom, Mapbox,
  etc. for geocoding and reverse geocoding.
- [`osmdata`](https://docs.ropensci.org/osmdata/) by Mark Padgham. Great
  for downloading spatial data from OpenStreetMap, via the [Overpass
  API](https://wiki.openstreetmap.org/wiki/Overpass_API).

## Installation

Install `nominatimlite` from
[**CRAN**](https://CRAN.R-project.org/package=nominatimlite):

``` r
install.packages("nominatimlite")
```

You can install the developing version of `nominatimlite` with:

``` r
devtools::install_github("dieghernan/nominatimlite")
```

Alternatively, you can install `nominatimlite` using the
[r-universe](http://dieghernan.r-universe.dev/ui/):

``` r
# Enable this universe
options(repos = c(
  dieghernan = "https://dieghernan.r-universe.dev",
  CRAN = "https://cloud.r-project.org"
))


install.packages("nominatimlite")
```

## Usage

### `sf` objects

With `nominatimlite` you can extract spatial objects easily:

``` r
library(nominatimlite)

# Extract some points - Pizza Hut in California

CA <- geo_lite_sf("California", points_only = FALSE)

pizzahut <- geo_lite_sf("Pizza Hut, California",
  limit = 50,
  custom_query = list(countrycodes = "us")
)

library(ggplot2)

ggplot(CA) +
  geom_sf() +
  geom_sf(data = pizzahut, col = "red")
```

<img src="man/figures/README-pizzahut-1.png" width="100%" />

You can also extract polygon and line objects (as provided by the
Nominatim API) using the option `points_only = FALSE`:

``` r
sol_poly <- geo_lite_sf("Statue of Liberty, NY, USA", points_only = FALSE) # a building - a polygon

ggplot(sol_poly) +
  geom_sf()
```

<img src="man/figures/README-statue_liberty-1.png" width="100%" />

``` r
dayton <- geo_lite_sf("Dayton, OH") # default - a point
ohio_state <- geo_lite_sf("Ohio, USA", points_only = FALSE) # a US state - a polygon
ohio_river <- geo_lite_sf("Ohio river", points_only = FALSE) # a river - a line

ggplot() +
  geom_sf(data = ohio_state) +
  geom_sf(data = dayton, color = "red", pch = 4) +
  geom_sf(data = ohio_river, color = "blue")
```

<img src="man/figures/README-line-object-1.png" width="100%" />

### Geocoding and reverse geocoding

*Note: examples adapted from `tidygeocoder` package*

In this first example we will geocode a few addresses using the
`geo_lite()` function:

``` r
library(tibble)

# create a dataframe with addresses
some_addresses <- tribble(
  ~name,                  ~addr,
  "White House",          "1600 Pennsylvania Ave NW, Washington, DC",
  "Transamerica Pyramid", "600 Montgomery St, San Francisco, CA 94111",
  "Willis Tower",         "233 S Wacker Dr, Chicago, IL 60606"
)

# geocode the addresses
lat_longs <- geo_lite(some_addresses$addr, lat = "latitude", long = "longitude")
```

Only latitude and longitude are returned from the geocoder service in
this example, but `full_results = TRUE` can be used to return all of the
data from the geocoder service.

| query                                      | latitude |  longitude | address                                                                                                           |
|:-------------------------------------------|---------:|-----------:|:------------------------------------------------------------------------------------------------------------------|
| 1600 Pennsylvania Ave NW, Washington, DC   | 38.89770 |  -77.03655 | White House, 1600, Pennsylvania Avenue Northwest, Washington, District of Columbia, 20500, United States          |
| 600 Montgomery St, San Francisco, CA 94111 | 37.79520 | -122.40279 | Transamerica Pyramid, 600, Montgomery Street, Financial District, San Francisco, California, 94111, United States |
| 233 S Wacker Dr, Chicago, IL 60606         | 41.87887 |  -87.63591 | Willis Tower, 233, South Wacker Drive, Printer’s Row, Loop, Chicago, Cook County, Illinois, 60606, United States  |

To perform reverse geocoding (obtaining addresses from geographic
coordinates), we can use the `reverse_geo_lite()` function. The
arguments are similar to the `geo_lite()` function, but now we specify
the input data columns with the `lat` and `long` arguments. The dataset
used here is from the geocoder query above. The single line address is
returned in a column named by the `address`.

``` r
reverse <- reverse_geo_lite(
  lat = lat_longs$latitude, long = lat_longs$longitude,
  address = "address_found"
)
```

| address_found                                                                                                     |      lat |        lon |
|:------------------------------------------------------------------------------------------------------------------|---------:|-----------:|
| White House, 1600, Pennsylvania Avenue Northwest, Washington, District of Columbia, 20500, United States          | 38.89770 |  -77.03655 |
| Transamerica Pyramid, 600, Montgomery Street, Financial District, San Francisco, California, 94111, United States | 37.79520 | -122.40279 |
| Willis Tower, West Adams Street, Printer’s Row, Loop, Chicago, Cook County, Illinois, 60606, United States        | 41.87874 |  -87.63602 |

For more advance users, see [Nominatim
docs](https://nominatim.org/release-docs/latest/api/Search/) to check
the parameters available.

## Citation

To cite the ‘nominatimlite’ package in publications use:

Hernangomez D (2023). nominatimlite: Interface with Nominatim API
Service. <https://doi.org/10.5281/zenodo.5113195>,
<https://dieghernan.github.io/nominatimlite/>

A BibTeX entry for LaTeX users is

    @Manual{R-nominatimlite,
      title = {nominatimlite: Interface with 'Nominatim' API Service},
      author = {Diego Hernangómez},
      year = {2023},
      version = {0.1.6.9000},
      doi = {10.5281/zenodo.5113195},
      url = {https://dieghernan.github.io/nominatimlite/},
      abstract = {Lite interface for getting data from OSM service Nominatim <https://nominatim.org/release-docs/latest/>. Extract coordinates from addresses, find places near a set of coordinates, search for amenities and return spatial objects on sf format.},
    }
