# Using \`terra\` with \`geotargets\`

``` r

library(targets)
#> Warning: package 'targets' was built under R version 4.6.1
library(geotargets)
```

The `geotargets` package extends `targets` to work with geospatial data
formats, such as rasters and vectors (e.g., shape files). In particular,
`geotargets` aims to support use of the `terra` package, which tend to
cause problems if used in targets created with
[`tar_target()`](https://docs.ropensci.org/targets/reference/tar_target.html)
. If you are new to `targets`, you should start by looking at the
[targets manual](https://books.ropensci.org/targets/) to get a handle on
the basics.

The design of `geotargets` is to specify target factories like so:
`tar_<pkg>_<type>`.

In this vignette we will demonstrate the use of the `terra` R package,
and we will demonstrate how to build raster (`rast`), vector (`vect`),
raster collection (`sprc`), and raster dataset (`sds`) targets with:

- [`tar_terra_rast()`](https://docs.ropensci.org/geotargets/reference/tar_terra_rast.md)
- [`tar_terra_vect()`](https://docs.ropensci.org/geotargets/reference/tar_terra_vect.md)
- [`tar_terra_sprc()`](https://docs.ropensci.org/geotargets/reference/tar_terra_sprc.md)
- [`tar_terra_sds()`](https://docs.ropensci.org/geotargets/reference/tar_terra_sds.md)

## How to run targets examples from vignettes

The example code in this vignette is designed for you to be able to just
copy and paste into the R console. However, this is not a typical way to
run a targets workflow.

The examples make use of
[`targets::tar_script()`](https://docs.ropensci.org/targets/reference/tar_script.html),
which creates or overwrites a `_targets.R` file (See for example, the
[`_targets.R` file in the demo-geotargets
repo](https://github.com/njtierney/demo-geotargets/blob/main/_targets.R)).
In these examples, everything inside of `tar_script({})` is what would
go inside a `_targets.R` file defining a workflow. When running the
`tar_script` code, it will ask you each time if you want to overwrite
the `_targets.R` file. This means if you are exploring these examples
and copying the entire examples, it is worthwhile doing this in a
separate directory/project/repository to avoid overwriting your own
`_targets.R` file.

So, when building your own targets workflow it is **not recommended** to
use
[`tar_script()`](https://docs.ropensci.org/targets/reference/tar_script.html),
but instead to create `_targets.R` and edit `_targets.R` directly.

### `tar_terra_rast()`: targets with `terra` rasters

``` r

targets::tar_script({
  library(targets)
  library(geotargets)
  tar_option_set(packages = "terra")
  geotargets_option_set(gdal_raster_driver = "COG")
  list(
    tar_target(
      tif_file,
      system.file("ex/elev.tif", package = "terra"),
      format = "file"
    ),
    tar_terra_rast(
      r,
      {
        rast <- rast(tif_file)
        units(rast) <- "m"
        rast
      }
    ),
    tar_terra_rast(
      r_agg,
      aggregate(r, 2)
    )
  )
})
```

Above is a basic example showing the use of
[`tar_terra_rast()`](https://docs.ropensci.org/geotargets/reference/tar_terra_rast.md)
in a targets pipeline. The command for
[`tar_terra_rast()`](https://docs.ropensci.org/geotargets/reference/tar_terra_rast.md)
can be any function that returns a `SpatRaster` object.

In this example, we’ve set the output to a cloud optimized geotiff
(“COG”), but any GDAL driver that works with
[`terra::writeRaster()`](https://rspatial.github.io/terra/reference/writeRaster.html)
should also work here. By default, we use “GTiff”. You can also set this
option on a target-by-target basis with the `filetype` argument to
[`tar_terra_rast()`](https://docs.ropensci.org/geotargets/reference/tar_terra_rast.md).

Running the pipeline:

``` r

tar_make()
#> + tif_file dispatched
#> ✔ tif_file completed [17ms, 7.99 kB]
#> + r dispatched
#> ✔ r completed [4ms, 10.47 kB]
#> + r_agg dispatched
#> ✔ r_agg completed [3ms, 6.30 kB]
#> ✔ ended pipeline [239ms, 3 completed, 0 skipped]
#> Warning message:
#> package ‘targets’ was built under R version 4.6.1
tar_read(r)
#> class       : SpatRaster
#> size        : 90, 95, 1  (nrow, ncol, nlyr)
#> resolution  : 0.008333333, 0.008333333  (x, y)
#> extent      : 5.741667, 6.533333, 49.44167, 50.19167  (xmin, xmax, ymin, ymax)
#> coord. ref. : lon/lat WGS 84 (EPSG:4326)
#> source      : r
#> name        : elevation
#> min value   :       141
#> max value   :       547
#> unit        : m
tar_read(r_agg)
#> class       : SpatRaster
#> size        : 45, 48, 1  (nrow, ncol, nlyr)
#> resolution  : 0.01666667, 0.01666667  (x, y)
#> extent      : 5.741667, 6.541667, 49.44167, 50.19167  (xmin, xmax, ymin, ymax)
#> coord. ref. : lon/lat WGS 84 (EPSG:4326)
#> source      : r_agg
#> name        : elevation
#> min value   :    166.75
#> max value   :     529.5
```

#### Raster metadata

You may have noticed the units for the `r` target above have gone
missing. This is due to limitations of `terra` and `targets`—`terra`
saves some metadata in “sidecar” aux.json files and `targets` enforces a
strict one file per target rule.

You can get around this by setting `preserve_metadata = "zip"` in
[`tar_terra_rast()`](https://docs.ropensci.org/geotargets/reference/tar_terra_rast.md)
to save the output files, including the metadata, as a minimally
compressed zip archive.

You can also set this for all raster targets with
`geotargets_option_set(terra_preserve_metadata = "zip")`.

Note: there are likely performance costs associated with this option.  
As an alternative, you can encode information in the layer names by
setting `names(r) <-` which are retained even with the default
`preserve_metadata = "drop"`.

``` r

targets::tar_script({
  # contents of _targets.R:
  library(targets)
  library(geotargets)
  tar_option_set(packages = "terra")
  geotargets_option_set(gdal_raster_driver = "COG")
  list(
    tar_target(
      tif_file,
      system.file("ex/elev.tif", package = "terra"),
      format = "file"
    ),
    tar_terra_rast(
      r,
      {
        rast <- rast(tif_file)
        units(rast) <- "m"
        rast
      },
      preserve_metadata = "zip"
    )
  )
})
```

``` r

tar_make()
#> + r dispatched
#> ✔ r completed [4ms, 10.23 kB]
#> ✔ ended pipeline [177ms, 1 completed, 1 skipped]
#> Warning message:
#> package ‘targets’ was built under R version 4.6.1
terra::units(tar_read(r))
#> [1] "m"
```

### `tar_terra_vect()`: targets with `terra` vectors

For `terra` `SpatVector` objects, use
[`tar_terra_vect()`](https://docs.ropensci.org/geotargets/reference/tar_terra_vect.md)
in the pipeline. You can set vector specific options with
[`geotargets_option_set()`](https://docs.ropensci.org/geotargets/reference/geotargets-options.md)
or with the `filetype` and `gdal` arguments to individual
[`tar_terra_vect()`](https://docs.ropensci.org/geotargets/reference/tar_terra_vect.md)
calls.

``` r

targets::tar_script({
  # contents of _targets.R:
  library(targets)
  library(geotargets)
  geotargets_option_set(gdal_vector_driver = "GeoJSON")
  list(
    tar_target(
      vect_file,
      system.file("ex", "lux.shp", package = "terra"),
      format = "file"
    ),
    tar_terra_vect(
      v,
      terra::vect(vect_file)
    ),
    tar_terra_vect(
      v_proj,
      terra::project(v, "EPSG:2196")
    )
  )
})
```

``` r

tar_make()
#> + vect_file dispatched
#> ✔ vect_file completed [1ms, 64.69 kB]
#> + v dispatched
#> ✔ v completed [8ms, 117.63 kB]
#> + v_proj dispatched
#> ✔ v_proj completed [17ms, 213.54 kB]
#> ✔ ended pipeline [258ms, 3 completed, 0 skipped]
#> Warning message:
#> package ‘targets’ was built under R version 4.6.1
tar_read(v)
#> class       : SpatVector
#> geometry    : polygons
#> dimensions  : 12, 6  (geometries, attributes)
#> extent      : 5.74414, 6.528252, 49.44781, 50.18162  (xmin, xmax, ymin, ymax)
#> source      : v
#> coord. ref. : lon/lat WGS 84 (EPSG:4326)
#> names       :  ID_1   NAME_1  ID_2   NAME_2  AREA   POP
#> type        : <num>    <chr> <num>    <chr> <num> <num>
#> values      :     1 Diekirch     1 Clervaux   312 18081
#>                   1 Diekirch     2 Diekirch   218 32543
#>                   1 Diekirch     3  Redange   259 18664
#>               ...
tar_read(v_proj)
#> class       : SpatVector
#> geometry    : polygons
#> dimensions  : 12, 6  (geometries, attributes)
#> extent      : -69990.51, -13879.85, 5484907, 5566555  (xmin, xmax, ymin, ymax)
#> source      : v_proj
#> coord. ref. : ETRS89 / Kp2000 Jutland (EPSG:2196)
#> names       :  ID_1   NAME_1  ID_2   NAME_2  AREA   POP
#> type        : <num>    <chr> <num>    <chr> <num> <num>
#> values      :     1 Diekirch     1 Clervaux   312 18081
#>                   1 Diekirch     2 Diekirch   218 32543
#>                   1 Diekirch     3  Redange   259 18664
#>               ...
```

### `tar_terra_sprc()`: targets with `terra` raster collections

Targets that produce a `SpatRasterCollection` can be created with
[`tar_terra_sprc()`](https://docs.ropensci.org/geotargets/reference/tar_terra_sprc.md).
The various rasters in the collection are saved as subdatasets to adhere
to `targets` one file per target rule.

``` r

targets::tar_script({
  # contents of _targets.R:
  library(targets)
  library(geotargets)
  elev_scale <- function(raster, z = 1, projection = "EPSG:4326") {
    terra::project(
      raster * z,
      projection
    )
  }
  tar_option_set(packages = "terra")
  geotargets_option_set(gdal_raster_driver = "GTiff")
  list(
    tar_target(
      elev_file,
      system.file("ex", "elev.tif", package = "terra"),
      format = "file"
    ),
    tar_terra_rast(
      r,
      rast(elev_file)
    ),
    tar_terra_sprc(
      raster_elevs,
      # two rasters, one unaltered, one scaled by factor of 2 and
      # reprojected to interrupted good homolosine
      terra::sprc(list(
        elev_scale(r, 1),
        elev_scale(r, 2, "+proj=igh")
      ))
    )
  )
})
```

``` r

tar_make()
#> + elev_file dispatched
#> ✔ elev_file completed [18ms, 7.99 kB]
#> + r dispatched
#> ✔ r completed [4ms, 8.52 kB]
#> + raster_elevs dispatched
#> ✔ raster_elevs completed [73ms, 37.90 kB]
#> ✔ ended pipeline [332ms, 3 completed, 0 skipped]
#> Warning message:
#> package ‘targets’ was built under R version 4.6.1
tar_read(raster_elevs)
#> class       : SpatRasterCollection
#> length      : 2
#> nrow        : 90, 115
#> ncol        : 95, 114
#> nlyr        :  1,   1
#> extent      : 5.741667, 1558890, 49.44167, 5556741  (xmin, xmax, ymin, ymax)
#> crs (first) : lon/lat WGS 84 (EPSG:4326)
#> names       : raster_elevs, raster_elevs
```

### `tar_terra_sds()`: targets with `terra` raster datasets

A `terra` `SpatRasterDataset` is very similar to a
`SpatRasterCollection` except that all sub-datasets must have the same
projection and extent

``` r

targets::tar_script({
  # contents of _targets.R:
  library(targets)
  library(geotargets)
  tar_option_set(packages = "terra")
  list(
    tar_target(
      logo_file,
      system.file("ex/logo.tif", package = "terra"),
      format = "file"
    ),
    tar_terra_sds(
      raster_dataset,
      {
        x <- sds(rast(logo_file), rast(logo_file) / 2)
        names(x) <- c("first", "second")
        x
      }
    )
  )
})
```

``` r

tar_make()
#> + logo_file dispatched
#> ✔ logo_file completed [17ms, 22.46 kB]
#> + raster_dataset dispatched
#> ✔ raster_dataset completed [42ms, 54.73 kB]
#> ✔ ended pipeline [229ms, 2 completed, 0 skipped]
#> Warning message:
#> package ‘targets’ was built under R version 4.6.1
tar_read(raster_dataset)
#> class       : SpatRasterDataset
#> subdatasets : 2
#> dimensions  : 77, 101 (nrow, ncol)
#> nlyr        : 3, 3
#> resolution  : 1, 1  (x, y)
#> extent      : 0, 101, 0, 77  (xmin, xmax, ymin, ymax)
#> coord. ref. : Cartesian (Meter)
#> source(s)   : raster_dataset
#> names       : raster_dataset, raster_dataset
```
