# Changelog

## geotargets (development version)

## geotargets 0.3.1 (15 May 2025)

CRAN release: 2025-05-15

- Throws an error if `preserve_metadata = "gdalraster_sozip"` in
  function
  [`tar_terra_rast()`](https://docs.ropensci.org/geotargets/reference/tar_terra_rast.md)
  and if GDAL is less than 3.7. Skips testing this feature when GDAL \<
  3.7 also. This fixes a bug picked up by the CRAN team.
- Tests also don’t report progress bars, as mentioned by CRAN team.

## geotargets 0.3.0 (16 April 2025)

CRAN release: 2025-05-07

- Bugfix by [@brownag](https://github.com/brownag) that fixes use of
  [`file.rename()`](https://rdrr.io/r/base/files.html) in
  `tar_terra_rast(..., preserve_metadata = "zip")`, which does not work
  when the temporary directory is on a different partition.
  ([\#121](https://github.com/ropensci/geotargets/issues/121), PR
  [\#122](https://github.com/ropensci/geotargets/issues/122)).
- Fixed examples for
  [`tar_terra_tiles()`](https://docs.ropensci.org/geotargets/reference/tar_terra_tiles.md),
  [`tile_grid()`](https://docs.ropensci.org/geotargets/reference/tile_helpers.md),
  [`tar_terra_sds()`](https://docs.ropensci.org/geotargets/reference/tar_terra_sds.md),
  and
  [`tar_terra_sprc()`](https://docs.ropensci.org/geotargets/reference/tar_terra_sprc.md)
  as reported by [@amart90](https://github.com/amart90) as part of
  [rOpenSci
  review](https://github.com/ropensci/software-review/issues/675)
- Added details to the documentation for
  [`tar_terra_tiles()`](https://docs.ropensci.org/geotargets/reference/tar_terra_tiles.md)
  (suggested by [@amart90](https://github.com/amart90) as part of
  [rOpenSci
  review](https://github.com/ropensci/software-review/issues/675))
- Completed ropensci review and transferred ownership to ropensci
- [`tar_terra_rast()`](https://docs.ropensci.org/geotargets/reference/tar_terra_rast.md)
  gains a `datatype` argument and
  [`tar_stars()`](https://docs.ropensci.org/geotargets/reference/tar_stars.md)
  gains a `type` argument. Both default to the geotargets option
  `"gdal.raster.data.type"` (when set).
- Additional arguments `...` are now passed to the target “write”
  method:
  [`terra::writeRaster()`](https://rspatial.github.io/terra/reference/writeRaster.html)
  for
  [`tar_terra_rast()`](https://docs.ropensci.org/geotargets/reference/tar_terra_rast.md),
  [`terra::writeVector()`](https://rspatial.github.io/terra/reference/writeVector.html)
  for
  [`tar_terra_vect()`](https://docs.ropensci.org/geotargets/reference/tar_terra_vect.md)
  and
  [`stars::write_stars()`](https://r-spatial.github.io/stars/reference/write_stars.html)
  for
  [`tar_stars()`](https://docs.ropensci.org/geotargets/reference/tar_stars.md)
  (Thanks to [@brownag](https://github.com/brownag) in
  [\#137](https://github.com/ropensci/geotargets/issues/137), resolves
  [\#132](https://github.com/ropensci/geotargets/issues/132) and
  [\#127](https://github.com/ropensci/geotargets/issues/127))
- Added
  [`tar_terra_vrt()`](https://docs.ropensci.org/geotargets/reference/tar_terra_vrt.md)
  for `SpatRaster` object targets that reference multiple data sources
  (e.g. tiles created with
  [`tar_terra_tiles()`](https://docs.ropensci.org/geotargets/reference/tar_terra_tiles.md))
  using a GDAL Virtual Dataset (VRT) XML file (Thanks to
  [@brownag](https://github.com/brownag) in
  [\#138](https://github.com/ropensci/geotargets/issues/138))
- The default driver for
  [`tar_terra_vect()`](https://docs.ropensci.org/geotargets/reference/tar_terra_vect.md)
  has been changed to `"GPKG"` in order to preserve CRS information
  ([\#166](https://github.com/ropensci/geotargets/issues/166)).
- Added `preserve_metadata = "gdalraster_sozip"` option to use
  [`gdalraster::addFilesInZip()`](https://firelab.github.io/gdalraster/reference/addFilesInZip.html)
  to write multi-file Seek-Optimized ZIP (SOZip) file targets, and
  `/vsizip/` GDAL Virtual File System paths for reading without
  extraction
  ([\#167](https://github.com/ropensci/geotargets/issues/167))

## geotargets 0.2.0 (29 November 2024)

- Created
  [`tar_stars()`](https://docs.ropensci.org/geotargets/reference/tar_stars.md)
  and
  [`tar_stars_proxy()`](https://docs.ropensci.org/geotargets/reference/tar_stars.md)
  that create `stars` and `stars_proxy` objects, respectively.
- Created
  [`tar_terra_tiles()`](https://docs.ropensci.org/geotargets/reference/tar_terra_tiles.md),
  a “target factory” for splitting a raster into multiple tiles with
  dynamic branching
  ([\#69](https://github.com/ropensci/geotargets/issues/69)).
- Created two helper functions for use in
  [`tar_terra_tiles()`](https://docs.ropensci.org/geotargets/reference/tar_terra_tiles.md):
  [`tile_grid()`](https://docs.ropensci.org/geotargets/reference/tile_helpers.md),
  [`tile_n()`](https://docs.ropensci.org/geotargets/reference/tile_helpers.md),
  and
  [`tile_blocksize()`](https://docs.ropensci.org/geotargets/reference/tile_helpers.md)
  ([\#69](https://github.com/ropensci/geotargets/issues/69),
  [\#86](https://github.com/ropensci/geotargets/issues/86),
  [\#87](https://github.com/ropensci/geotargets/issues/87),
  [\#89](https://github.com/ropensci/geotargets/issues/89)).
- Created utility function
  [`set_window()`](https://docs.ropensci.org/geotargets/reference/set_window.md)
  mostly for internal use within
  [`tar_terra_tiles()`](https://docs.ropensci.org/geotargets/reference/tar_terra_tiles.md).
- Removes the `iteration` argument from all `tar_*()` functions.
  `iteration` now hard-coded as `"list"` since it is the only option
  that works (for now at least).
- Added the `description` argument to all `tar_*()` functions which is
  passed to
  [`tar_target()`](https://docs.ropensci.org/targets/reference/tar_target.html).
- Suppressed the warning “\[rast\] skipped sub-datasets” from
  [`tar_terra_sprc()`](https://docs.ropensci.org/geotargets/reference/tar_terra_sprc.md),
  which is misleading in this context
  ([\#92](https://github.com/ropensci/geotargets/issues/92),
  [\#104](https://github.com/ropensci/geotargets/issues/104)).
- Requires GDAL 3.1 or greater to use “ESRI Shapefile” driver in
  [`tar_terra_vect()`](https://docs.ropensci.org/geotargets/reference/tar_terra_vect.md)
  ([\#71](https://github.com/ropensci/geotargets/issues/71),
  [\#97](https://github.com/ropensci/geotargets/issues/97))
- `geotargets` now requires `targets` version 1.8.0 or higher
- [`tar_terra_rast()`](https://docs.ropensci.org/geotargets/reference/tar_terra_rast.md)
  gains a `preserve_metadata` option that when set to `"zip"`
  reads/writes targets as zip archives that include aux.json “sidecar”
  files sometimes written by `terra`
  ([\#58](https://github.com/ropensci/geotargets/issues/58))
- `terra` (\>= 1.7.71), `withr` (\>= 3.0.0), and `zip` are now required
  dependencies of `geotargets` (moved from `Suggests` to `Imports`)

## geotargets 0.1.0 (14 May 2024)

- Created
  [`tar_terra_rast()`](https://docs.ropensci.org/geotargets/reference/tar_terra_rast.md)
  and
  [`tar_terra_vect()`](https://docs.ropensci.org/geotargets/reference/tar_terra_vect.md)
  for targets that create `SpatRaster` and `SpatVector` objects,
  respectively
- Created
  [`tar_terra_sprc()`](https://docs.ropensci.org/geotargets/reference/tar_terra_sprc.md)
  that creates a `SpatRasterCollection` object.
- `geotargets_options_get()` and `geotargets_options_set()` can be used
  to set and get options specific to `geotargets`.
- `geotargets` now requires `targets` version 1.7.0 or higher
- fixed a bug where `resources` supplied to `tar_terra_*()` were being
  ignored ([\#66](https://github.com/ropensci/geotargets/issues/66))
