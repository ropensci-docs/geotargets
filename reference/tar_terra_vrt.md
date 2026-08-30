# Create a GDAL Virtual Dataset (VRT) with terra

Provides a target format for
[terra::SpatRaster](https://rspatial.github.io/terra/reference/SpatRaster-class.html),
[terra::SpatRasterDataset](https://rspatial.github.io/terra/reference/SpatRaster-class.html),
and
[terra::SpatRasterCollection](https://rspatial.github.io/terra/reference/SpatRaster-class.html)
objects representing a [GDAL Virtual Dataset
(VRT)](https://gdal.org/en/stable/drivers/raster/vrt.html).

## Usage

``` r
tar_terra_vrt(
  name,
  command,
  pattern = NULL,
  ...,
  tidy_eval = targets::tar_option_get("tidy_eval"),
  packages = targets::tar_option_get("packages"),
  library = targets::tar_option_get("library"),
  repository = targets::tar_option_get("repository"),
  error = targets::tar_option_get("error"),
  memory = targets::tar_option_get("memory"),
  garbage_collection = targets::tar_option_get("garbage_collection"),
  deployment = targets::tar_option_get("deployment"),
  priority = targets::tar_option_get("priority"),
  resources = targets::tar_option_get("resources"),
  storage = targets::tar_option_get("storage"),
  retrieval = targets::tar_option_get("retrieval"),
  cue = targets::tar_option_get("cue"),
  description = targets::tar_option_get("description")
)
```

## Arguments

- name:

  Symbol, name of the target. In
  [`tar_target()`](https://docs.ropensci.org/targets/reference/tar_target.html),
  `name` is an unevaluated symbol, e.g. `tar_target(name = data)`. In
  [`tar_target_raw()`](https://docs.ropensci.org/targets/reference/tar_target.html),
  `name` is a character string, e.g. `tar_target_raw(name = "data")`.

  A target name must be a valid name for a symbol in R, and it must not
  start with a dot. Subsequent targets can refer to this name
  symbolically to induce a dependency relationship: e.g.
  `tar_target(downstream_target, f(upstream_target))` is a target named
  `downstream_target` which depends on a target `upstream_target` and a
  function `f()`.

  In most cases, The target name is the name of its local data file in
  storage. Some file systems are not case sensitive, which means
  converting a name to a different case may overwrite a different
  target. Please ensure all target names have unique names when
  converted to lower case.

  In addition, a target's name determines its random number generator
  seed. In this way, each target runs with a reproducible seed so
  someone else running the same pipeline should get the same results,
  and no two targets in the same pipeline share the same seed. (Even
  dynamic branches have different names and thus different seeds.) You
  can recover the seed of a completed target with
  `tar_meta(your_target, seed)` and run
  [`tar_seed_set()`](https://docs.ropensci.org/targets/reference/tar_seed_set.html)
  on the result to locally recreate the target's initial RNG state.

- command:

  R code to run the target. In
  [`tar_target()`](https://docs.ropensci.org/targets/reference/tar_target.html),
  `command` is an unevaluated expression, e.g.
  `tar_target(command = data)`. In
  [`tar_target_raw()`](https://docs.ropensci.org/targets/reference/tar_target.html),
  `command` is an evaluated expression, e.g.
  `tar_target_raw(command = quote(data))`.

- pattern:

  Code to define a dynamic branching branching for a target. In
  [`tar_target()`](https://docs.ropensci.org/targets/reference/tar_target.html),
  `pattern` is an unevaluated expression, e.g.
  `tar_target(pattern = map(data))`. In
  [`tar_target_raw()`](https://docs.ropensci.org/targets/reference/tar_target.html),
  `command` is an evaluated expression, e.g.
  `tar_target_raw(pattern = quote(map(data)))`.

  To demonstrate dynamic branching patterns, suppose we have a pipeline
  with numeric vector targets `x` and `y`. Then,
  `tar_target(z, x + y, pattern = map(x, y))` implicitly defines
  branches of `z` that each compute `x[1] + y[1]`, `x[2] + y[2]`, and so
  on. See the user manual for details.

- ...:

  Additional arguments passed to
  [`terra::vrt()`](https://rspatial.github.io/terra/reference/vrt.html)

- tidy_eval:

  Logical, whether to enable tidy evaluation when interpreting `command`
  and `pattern`. If `TRUE`, you can use the "bang-bang" operator `!!` to
  programmatically insert the values of global objects.

- packages:

  Character vector of packages to load right before the target runs or
  the output data is reloaded for downstream targets. Use
  [`tar_option_set()`](https://docs.ropensci.org/targets/reference/tar_option_set.html)
  to set packages globally for all subsequent targets you define.

- library:

  Character vector of library paths to try when loading `packages`.

- repository:

  Character of length 1, remote repository for target storage. Choices:

  - `"local"`: file system of the local machine.

  - `"aws"`: Amazon Web Services (AWS) S3 bucket. Can be configured with
    a non-AWS S3 bucket using the `endpoint` argument of
    [`tar_resources_aws()`](https://docs.ropensci.org/targets/reference/tar_resources_aws.html),
    but versioning capabilities may be lost in doing so. See the cloud
    storage section of <https://books.ropensci.org/targets/data.html>
    for details for instructions.

  - `"gcp"`: Google Cloud Platform storage bucket. See the cloud storage
    section of <https://books.ropensci.org/targets/data.html> for
    details for instructions.

  - A character string from
    [`tar_repository_cas()`](https://docs.ropensci.org/targets/reference/tar_repository_cas.html)
    for content-addressable storage.

  Note: if `repository` is not `"local"` and `format` is `"file"` then
  the target should create a single output file. That output file is
  uploaded to the cloud and tracked for changes where it exists in the
  cloud. As of `targets` version 1.11.0 and higher, the local file is no
  longer deleted after the target runs.

- error:

  Character of length 1, what to do if the target stops and throws an
  error. Options:

  - `"stop"`: the whole pipeline stops and throws an error.

  - `"continue"`: the whole pipeline keeps going.

  - `"null"`: The errored target continues and returns `NULL`. The data
    hash is deliberately wrong so the target is not up to date for the
    next run of the pipeline. In addition, as of `targets` version
    1.8.0.9011, a value of `NULL` is given to upstream dependencies with
    `error = "null"` if loading fails.

  - `"abridge"`: any currently running targets keep running, but no new
    targets launch after that.

  - `"trim"`: all currently running targets stay running. A queued
    target is allowed to start if:

    1.  It is not downstream of the error, and

    2.  It is not a sibling branch from the same
        [`tar_target()`](https://docs.ropensci.org/targets/reference/tar_target.html)
        call (if the error happened in a dynamic branch).

    The idea is to avoid starting any new work that the immediate error
    impacts. `error = "trim"` is just like `error = "abridge"`, but it
    allows potentially healthy regions of the dependency graph to begin
    running. (Visit <https://books.ropensci.org/targets/debugging.html>
    to learn how to debug targets using saved workspaces.)

- memory:

  Character of length 1, memory strategy. Possible values:

  - `"auto"` (default): equivalent to `memory = "transient"` in almost
    all cases. But to avoid superfluous reads from disk,
    `memory = "auto"` is equivalent to `memory = "persistent"` for for
    non-dynamically-branched targets that other targets dynamically
    branch over. For example: if your pipeline has
    `tar_target(name = y, command = x, pattern = map(x))`, then
    `tar_target(name = x, command = f(), memory = "auto")` will use
    persistent memory for `x` in order to avoid rereading all of `x` for
    every branch of `y`.

  - `"transient"`: the target gets unloaded after every new target
    completes. Either way, the target gets automatically loaded into
    memory whenever another target needs the value.

  - `"persistent"`: the target stays in memory until the end of the
    pipeline (unless `storage` is `"worker"`, in which case `targets`
    unloads the value from memory right after storing it in order to
    avoid sending copious data over a network).

  For cloud-based file targets (e.g. `format = "file"` with
  `repository = "aws"`), the `memory` option applies to the temporary
  local copy of the file: `"persistent"` means it remains until the end
  of the pipeline and is then deleted, and `"transient"` means it gets
  deleted as soon as possible. The former conserves bandwidth, and the
  latter conserves local storage.

- garbage_collection:

  Logical: `TRUE` to run [`base::gc()`](https://rdrr.io/r/base/gc.html)
  just before the target runs, in whatever R process it is about to run
  (which could be a parallel worker). `FALSE` to omit garbage
  collection. Numeric values get converted to `FALSE`. The
  `garbage_collection` option in
  [`tar_option_set()`](https://docs.ropensci.org/targets/reference/tar_option_set.html)
  is independent of the argument of the same name in
  [`tar_target()`](https://docs.ropensci.org/targets/reference/tar_target.html).

- deployment:

  Character of length 1. If `deployment` is `"main"`, then the target
  will run on the central controlling R process. Otherwise, if
  `deployment` is `"worker"` and you set up the pipeline with
  distributed/parallel computing, then the target runs on a parallel
  worker. For more on distributed/parallel computing in `targets`,
  please visit <https://books.ropensci.org/targets/crew.html>.

- priority:

  Deprecated on 2025-04-08 (`targets` version 1.10.1.9013). `targets`
  has moved to a more efficient scheduling algorithm
  (<https://github.com/ropensci/targets/issues/1458>) which cannot
  support priorities. The `priority` argument of
  [`tar_target()`](https://docs.ropensci.org/targets/reference/tar_target.html)
  no longer has a reliable effect on execution order.

- resources:

  Object returned by
  [`tar_resources()`](https://docs.ropensci.org/targets/reference/tar_resources.html)
  with optional settings for high-performance computing functionality,
  alternative data storage formats, and other optional capabilities of
  `targets`. See
  [`tar_resources()`](https://docs.ropensci.org/targets/reference/tar_resources.html)
  for details.

- storage:

  Character string to control when the output of the target is saved to
  storage. Only relevant when using `targets` with parallel workers
  (<https://books.ropensci.org/targets/crew.html>). Must be one of the
  following values:

  - `"worker"` (default): the worker saves/uploads the value.

  - `"main"`: the target's return value is sent back to the host machine
    and saved/uploaded locally.

  - `"none"`: `targets` makes no attempt to save the result of the
    target to storage in the location where `targets` expects it to be.
    Saving to storage is the responsibility of the user. Use with
    caution.

- retrieval:

  Character string to control when the current target loads its
  dependencies into memory before running. (Here, a "dependency" is
  another target upstream that the current one depends on.) Only
  relevant when using `targets` with parallel workers
  (<https://books.ropensci.org/targets/crew.html>). Must be one of the
  following values:

  - `"auto"` (default): equivalent to `retrieval = "worker"` in almost
    all cases. But to avoid unnecessary reads from disk,
    `retrieval = "auto"` is equivalent to `retrieval = "main"` for
    dynamic branches that branch over non-dynamic targets. For example:
    if your pipeline has `tar_target(x, command = f())`, then
    `tar_target(y, command = x, pattern = map(x), retrieval = "auto")`
    will use `"main"` retrieval in order to avoid rereading all of `x`
    for every branch of `y`.

  - `"worker"`: the worker loads the target's dependencies.

  - `"main"`: the target's dependencies are loaded on the host machine
    and sent to the worker before the target runs.

  - `"none"`: `targets` makes no attempt to load its dependencies. With
    `retrieval = "none"`, loading dependencies is the responsibility of
    the user. Use with caution.

- cue:

  An optional object from
  [`tar_cue()`](https://docs.ropensci.org/targets/reference/tar_cue.html)
  to customize the rules that decide whether the target is up to date.

- description:

  Character of length 1, a custom free-form human-readable text
  description of the target. Descriptions appear as target labels in
  functions like
  [`tar_manifest()`](https://docs.ropensci.org/targets/reference/tar_manifest.html)
  and
  [`tar_visnetwork()`](https://docs.ropensci.org/targets/reference/tar_visnetwork.html),
  and they let you select subsets of targets for the `names` argument of
  functions like
  [`tar_make()`](https://docs.ropensci.org/targets/reference/tar_make.html).
  For example,
  `tar_manifest(names = tar_described_as(starts_with("survival model")))`
  lists all the targets whose descriptions start with the character
  string `"survival model"`.

## Value

target class "tar_stem" for use in a target pipeline

## Details

`tar_terra_vrt()` accepts SpatRaster, SpatRasterDataset, or
SpatRasterCollection objects as input, and returns a SpatRaster
referencing a GDAL Virtual Dataset file (.vrt). The .vrt file format
uses XML and describes the layers and tiles that comprise a virtual
raster data source. To use a list of SpatRaster of varying extent, such
as output from
[`tar_terra_tiles()`](https://docs.ropensci.org/geotargets/reference/tar_terra_tiles.md),
or a character vector of paths, wrap the tile result in a call to
[`terra::sprc()`](https://rspatial.github.io/terra/reference/sprc.html)
to create a SpatRasterCollection.

## See also

[`tar_terra_tiles()`](https://docs.ropensci.org/geotargets/reference/tar_terra_tiles.md)

## Examples

``` r
if (Sys.getenv("TAR_LONG_EXAMPLES") == "true") {
 targets::tar_dir({ # tar_dir() runs code from a temporary directory.
   targets::tar_script({
     list(
       geotargets::tar_terra_vrt(
         terra_rast_example,
         terra::rast(system.file("ex/elev.tif", package = "terra"))
       )
     )
   })
   targets::tar_make()
   x <- targets::tar_read(terra_rast_example)
 })

targets::tar_dir({
    targets::tar_script({
        library(targets)
        library(geotargets)
        list(
            tar_terra_rast(r, terra::rast(
                system.file("ex", "elev.tif", package = "terra")
            )),
            tar_terra_rast(r2, r * 2),
            tar_terra_tiles(rt, c(r, r2), function(x)
                tile_grid(x, ncol = 2, nrow = 2)),
            tar_terra_vrt(r3, terra::sprc(rt))
        )
    })
})

}
```
