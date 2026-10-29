# Get or Set geotargets Options

Get or set behaviour for geospatial data target stores using
geotargets-specific global options.

## Usage

``` r
geotargets_option_set(
  gdal_raster_driver = NULL,
  gdal_raster_creation_options = NULL,
  gdal_raster_data_type = NULL,
  gdal_vector_driver = NULL,
  gdal_vector_creation_options = NULL,
  terra_preserve_metadata = NULL
)

geotargets_option_get(name)
```

## Arguments

- gdal_raster_driver:

  character, length 1; set the driver used for raster data in target
  store (default: `"GTiff"`). Options for driver names can be found
  here: <https://gdal.org/en/stable/drivers/raster/index.html>.

- gdal_raster_creation_options:

  character; set the GDAL creation options used when writing raster
  files to target store (default: `""`). You may specify multiple values
  e.g. `c("COMPRESS=DEFLATE", "TFW=YES")`. Each GDAL driver supports a
  unique set of creation options. For example, with the default
  `"GTiff"` driver:
  <https://gdal.org/en/stable/drivers/raster/gtiff.html#creation-options>.

- gdal_raster_data_type:

  character; Data type for writing raster file. One of: `"INT1U"`,
  `"INT2U"`, `"INT4U"`, `"INT8U"`, `"INT2S"`, `"INT4S"`, `"INT8S"`,
  `"FLT4S"`, `"FLT8S"` (for terra), or `"Byte"`, `"UInt16"`, `"UInt32"`,
  `"UInt64"`, `"Int16"`, `"Int32"`, `"Int64"`, `"Float32"`, `"Float64"`
  (for stars).

- gdal_vector_driver:

  character, length 1; set the file type used for vector data in target
  store (default: `"GPKG"`).

- gdal_vector_creation_options:

  character; set the GDAL layer creation options used when writing
  vector files to target store (default: `"ENCODING=UTF-8"`). You may
  specify multiple values e.g.
  `c("WRITE_BBOX=YES", "COORDINATE_PRECISION=10")`. Each GDAL driver
  supports a unique set of creation options. For example, with the
  default `"GPKG"` driver:
  <https://gdal.org/en/stable/drivers/vector/gpkg.html#layer-creation-options>

- terra_preserve_metadata:

  character. When `"drop"` (default), any auxiliary files that would be
  written by
  [`terra::writeRaster()`](https://rspatial.github.io/terra/reference/writeRaster.html)
  containing raster metadata such as units and datetimes are lost (note
  that this does not include layer names set with `names() <-`). When
  `"zip"`, these metadata are retained by archiving all written files as
  a zip file upon writing and unzipping them upon reading. This adds
  extra overhead and will slow pipelines. Also note metadata may be
  impacted by different versions of GDAL and different drivers. Note
  that you can specify this option for individual targets, e.g., inside
  [`tar_terra_rast()`](https://docs.ropensci.org/geotargets/reference/tar_terra_rast.md)
  there is the option, `preserve_metadata`.

- name:

  character; option name to get.

## Value

Specific options, such as "gdal.raster.driver". See "Details" for more
information.

## Details

These options can also be set using
[`options()`](https://rdrr.io/r/base/options.html). For example,
`geotargets_options_set(gdal_raster_driver = "GTiff")` is equivalent to
`options("geotargets.gdal.raster.driver" = "GTiff")`.

## Potential issues retaining metadata

If you have an issue with retaining metadata (such as units, time, etc),
this could be due to the versions of GDAL and terra on your machine. We
recommend exploring if this issue persists outside of geotargets. That
is, try saving the file out and reading it back in using regular R code.
If you find that this is an issue with geotargets, please file an issues
at <https://github.com/ropensci/geotargets/issues/> and we will try and
get this working for you.

## Examples

``` r
# For CRAN. Ensures these examples run under certain conditions.
# To run this locally, run the code inside this if statement
if (Sys.getenv("TAR_LONG_EXAMPLES") == "true") {
# tar_dir() runs code from a temporary directory.
  targets::tar_dir({
    library(geotargets)
    op <- getOption("geotargets.gdal.raster.driver")
    withr::defer(options("geotargets.gdal.raster.driver" = op))
    geotargets_option_set(
      gdal_raster_driver = "COG",
      terra_preserve_metadata = "zip"
    )
    targets::tar_script({
      list(
        geotargets::tar_terra_rast(
          terra_rast_example,
          {
            new_rast <- system.file("ex/elev.tif", package = "terra") |>
              terra::rast()
            terra::units(new_rast) <- "m"
            new_rast
          }
        )
      )
    })
    targets::tar_make()
    x <- targets::tar_read(terra_rast_example)
    x
    terra::units(x)
  })
}

geotargets_option_get("gdal.raster.driver")
#> NULL
geotargets_option_get("gdal.raster.creation.options")
#> NULL
```
