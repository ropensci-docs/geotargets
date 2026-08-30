# Copy a raster within a window

Create a new SpatRaster object as specified by a window (area of
interest) over the original SpatRaster. This is a wrapper around
[`terra::window()`](https://rspatial.github.io/terra/reference/window.html)
which, rather than modifying the SpatRaster in place, returns a new
SpatRaster leaving the original unchanged.

## Usage

``` r
set_window(raster, window)
```

## Arguments

- raster:

  a SpatRaster object.

- window:

  a SpatExtent object defining the area of interest.

## Value

SpatRaster

## Note

While this may have general use, it was created primarily for use within
[`tar_terra_tiles()`](https://docs.ropensci.org/geotargets/reference/tar_terra_tiles.md).

## Author

Eric Scott

## Examples

``` r
f <- system.file("ex/elev.tif", package="terra")
r <- terra::rast(f)
e <- terra::ext(c(5.9, 6,49.95, 50))
r2 <- set_window(r, e)
terra::ext(r)
#> SpatExtent : 5.7416666666666663, 6.5333333333333332, 49.441666666666663, 50.191666666666663 (xmin, xmax, ymin, ymax)
terra::ext(r2)
#> SpatExtent : 5.8999999999999995, 5.9999999999999991, 49.950000000000003, 50 (xmin, xmax, ymin, ymax)
```
