# Helper functions to create tiles

Wrappers around
[`terra::getTileExtents()`](https://rspatial.github.io/terra/reference/makeTiles.html)
that return a list of named numeric vectors describing the extents of
tiles rather than `SpatExtent` objects. While these may have general
use, they are intended primarily for supplying to the `tile_fun`
argument of
[`tar_terra_tiles()`](https://docs.ropensci.org/geotargets/reference/tar_terra_tiles.md).

## Usage

``` r
tile_grid(raster, ncol, nrow)

tile_blocksize(raster, n_blocks_row = 1, n_blocks_col = 1)

tile_n(raster, n)
```

## Arguments

- raster:

  a SpatRaster object.

- ncol:

  integer; number of columns to split the SpatRaster into.

- nrow:

  integer; number of rows to split the SpatRaster into.

- n_blocks_row:

  integer; multiple of blocksize to include in each tile vertically.

- n_blocks_col:

  integer; multiple of blocksize to include in each tile horizontally.

- n:

  integer; total number of tiles to split the SpatRaster into.

## Value

list of named numeric vectors with xmin, xmax, ymin, and ymax values
that can be coerced to SpatExtent objects with
[`terra::ext()`](https://rspatial.github.io/terra/reference/ext.html).

## Details

`tile_blocksize()` creates extents using the raster's native block size
(see
[`terra::fileBlocksize()`](https://rspatial.github.io/terra/reference/readwrite.html)),
which should be more memory efficient. Create tiles with multiples of
the raster's blocksize with `n_blocks_row` and `n_blocks_col`. We
strongly suggest the user explore how many tiles are created by
`tile_blocksize()` before creating a dynamically branched target using
this helper. Note that block size is a property of *files* and does not
apply to in-memory `SpatRaster`s. Therefore, if you want to use this
helper in
[`tar_terra_tiles()`](https://docs.ropensci.org/geotargets/reference/tar_terra_tiles.md)
you may need to ensure the upstream target provided to the `raster`
argument is not in memory by setting `memory = "transient"`.

`tile_grid()` allows specification of a number of rows and columns to
split the raster into. E.g. nrow = 2 and ncol = 2 would create 4 tiles
(because it specifies a 2x2 matrix, which has 4 elements).

`tile_n()` creates (about) `n` tiles and prints the number of rows,
columns, and total tiles created.

## Author

Eric Scott

## Examples

``` r
f <- system.file("ex/elev.tif", package="terra")
r <- terra::rast(f)
tile_grid(r, ncol = 2, nrow = 2)
#> [[1]]
#>      xmin      xmax      ymin      ymax 
#>  5.741667  6.141667 49.816667 50.191667 
#> 
#> [[2]]
#>      xmin      xmax      ymin      ymax 
#>  6.141667  6.533333 49.816667 50.191667 
#> 
#> [[3]]
#>      xmin      xmax      ymin      ymax 
#>  5.741667  6.141667 49.441667 49.816667 
#> 
#> [[4]]
#>      xmin      xmax      ymin      ymax 
#>  6.141667  6.533333 49.441667 49.816667 
#> 
tile_blocksize(r)
#> [[1]]
#>      xmin      xmax      ymin      ymax 
#>  5.741667  6.533333 49.833333 50.191667 
#> 
#> [[2]]
#>      xmin      xmax      ymin      ymax 
#>  5.741667  6.533333 49.475000 49.833333 
#> 
#> [[3]]
#>      xmin      xmax      ymin      ymax 
#>  5.741667  6.533333 49.441667 49.475000 
#> 
tile_n(r, 8)
#> creating 2 * 4 = 8 tile extents
#> [[1]]
#>      xmin      xmax      ymin      ymax 
#>  5.741667  5.941667 49.816667 50.191667 
#> 
#> [[2]]
#>      xmin      xmax      ymin      ymax 
#>  5.941667  6.133333 49.816667 50.191667 
#> 
#> [[3]]
#>      xmin      xmax      ymin      ymax 
#>  6.133333  6.333333 49.816667 50.191667 
#> 
#> [[4]]
#>      xmin      xmax      ymin      ymax 
#>  6.333333  6.533333 49.816667 50.191667 
#> 
#> [[5]]
#>      xmin      xmax      ymin      ymax 
#>  5.741667  5.941667 49.441667 49.816667 
#> 
#> [[6]]
#>      xmin      xmax      ymin      ymax 
#>  5.941667  6.133333 49.441667 49.816667 
#> 
#> [[7]]
#>      xmin      xmax      ymin      ymax 
#>  6.133333  6.333333 49.441667 49.816667 
#> 
#> [[8]]
#>      xmin      xmax      ymin      ymax 
#>  6.333333  6.533333 49.441667 49.816667 
#> 

# \donttest{
#Example usage with tar_terra_tiles
list(
    tar_terra_rast(
        my_map,
        terra::rast(system.file("ex/logo.tif", package = "terra"))
    ),
    tar_terra_tiles(
        name = rast_split,
        raster = my_map,
        tile_fun = tile_blocksize,
        description = "Each tile is 1 block"
    ),
    tar_terra_tiles(
        name = rast_split_2blocks,
        raster = my_map,
        tile_fun = \(x) tile_blocksize(
          x,
          n_blocks_row = 2,
          n_blocks_col = 1
          ),
        description = "Each tile is 2 blocks tall, 1 block wide"
    ),
    tar_terra_tiles(
        name = rast_split_grid,
        raster = my_map,
        tile_fun = \(x) tile_grid(x, ncol = 2, nrow = 2),
        description = "Split into 4 tiles in a 2x2 grid"
    ),
    tar_terra_tiles(
        name = rast_split_n,
        raster = my_map,
        tile_fun = \(x) tile_n(x, n = 6),
        description = "Split into 6 tiles"
    )
)
#> [[1]]
#> <tar_stem> 
#>   name: my_map 
#>   description:  
#>   command:
#>     terra::rast(system.file("ex/logo.tif", package = "terra")) 
#>   format: format_custom&read=dGVycmE6OnJhc3QocGF0aCk&write=ewogICAgZG8uY2FsbCh0ZXJyYTo6d3JpdGVSYXN0ZXIsIGMobGlzdChvYmplY3QsIGZpbGVuYW1lID0gcGF0aCwgCiAgICAgICAgZmlsZXR5cGUgPSAiR1RpZmYiLCBvdmVyd3JpdGUgPSBUUlVFLCBnZGFsID0gTlVMTCksIGxpc3QoKSkpCn0&marshal=dGVycmE6OndyYXAob2JqZWN0KQ&unmarshal=dGVycmE6OnVud3JhcChvYmplY3Qp&convert=&copy=&repository= 
#>   repository: local 
#>   iteration method: list 
#>   error mode: stop 
#>   memory mode: auto 
#>   storage mode: worker 
#>   retrieval mode: auto 
#>   deployment mode: worker 
#>   priority: 0 
#>   resources:
#>     list() 
#>   cue:
#>     seed: TRUE
#>     file: TRUE
#>     iteration: TRUE
#>     repository: TRUE
#>     format: TRUE
#>     depend: TRUE
#>     command: TRUE
#>     mode: thorough 
#>   packages:
#>     geotargets
#>     stats
#>     graphics
#>     grDevices
#>     utils
#>     datasets
#>     methods
#>     base 
#>   library:
#>     NULL
#> [[2]]
#> [[2]][[1]]
#> <tar_stem> 
#>   name: rast_split_exts 
#>   description: Each tile is 1 block 
#>   command:
#>     tile_blocksize(my_map) 
#>   format: rds 
#>   repository: local 
#>   iteration method: list 
#>   error mode: stop 
#>   memory mode: auto 
#>   storage mode: worker 
#>   retrieval mode: auto 
#>   deployment mode: worker 
#>   priority: 0 
#>   resources:
#>     list() 
#>   cue:
#>     seed: TRUE
#>     file: TRUE
#>     iteration: TRUE
#>     repository: TRUE
#>     format: TRUE
#>     depend: TRUE
#>     command: TRUE
#>     mode: thorough 
#>   packages:
#>     geotargets
#>     stats
#>     graphics
#>     grDevices
#>     utils
#>     datasets
#>     methods
#>     base 
#>   library:
#>     NULL
#> [[2]][[2]]
#> <tar_pattern> 
#>   name: rast_split 
#>   description: Each tile is 1 block 
#>   command:
#>     set_window(my_map, terra::ext(rast_split_exts)) 
#>   pattern:
#>     map(rast_split_exts) 
#>   format: format_custom&read=dGVycmE6OnJhc3QocGF0aCk&write=ewogICAgdGVycmE6OndyaXRlUmFzdGVyKG9iamVjdCwgcGF0aCwgZmlsZXR5cGUgPSAiR1RpZmYiLCBvdmVyd3JpdGUgPSBUUlVFLCAKICAgICAgICBnZGFsID0gY2hhcmFjdGVyKDApKQp9&marshal=dGVycmE6OndyYXAob2JqZWN0KQ&unmarshal=dGVycmE6OnVud3JhcChvYmplY3Qp&convert=&copy=&repository= 
#>   repository: local 
#>   iteration method: list 
#>   error mode: stop 
#>   memory mode: auto 
#>   storage mode: worker 
#>   retrieval mode: auto 
#>   deployment mode: worker 
#>   priority: 0 
#>   resources:
#>     list() 
#>   cue:
#>     seed: TRUE
#>     file: TRUE
#>     iteration: TRUE
#>     repository: TRUE
#>     format: TRUE
#>     depend: TRUE
#>     command: TRUE
#>     mode: thorough 
#>   packages:
#>     geotargets
#>     stats
#>     graphics
#>     grDevices
#>     utils
#>     datasets
#>     methods
#>     base 
#>   library:
#>     NULL
#> 
#> [[3]]
#> [[3]][[1]]
#> <tar_stem> 
#>   name: rast_split_2blocks_exts 
#>   description: Each tile is 2 blocks tall, 1 block wide 
#>   command:
#>     (function(x) tile_blocksize(x, n_blocks_row = 2, n_blocks_col = 1))(my_map) 
#>   format: rds 
#>   repository: local 
#>   iteration method: list 
#>   error mode: stop 
#>   memory mode: auto 
#>   storage mode: worker 
#>   retrieval mode: auto 
#>   deployment mode: worker 
#>   priority: 0 
#>   resources:
#>     list() 
#>   cue:
#>     seed: TRUE
#>     file: TRUE
#>     iteration: TRUE
#>     repository: TRUE
#>     format: TRUE
#>     depend: TRUE
#>     command: TRUE
#>     mode: thorough 
#>   packages:
#>     geotargets
#>     stats
#>     graphics
#>     grDevices
#>     utils
#>     datasets
#>     methods
#>     base 
#>   library:
#>     NULL
#> [[3]][[2]]
#> <tar_pattern> 
#>   name: rast_split_2blocks 
#>   description: Each tile is 2 blocks tall, 1 block wide 
#>   command:
#>     set_window(my_map, terra::ext(rast_split_2blocks_exts)) 
#>   pattern:
#>     map(rast_split_2blocks_exts) 
#>   format: format_custom&read=dGVycmE6OnJhc3QocGF0aCk&write=ewogICAgdGVycmE6OndyaXRlUmFzdGVyKG9iamVjdCwgcGF0aCwgZmlsZXR5cGUgPSAiR1RpZmYiLCBvdmVyd3JpdGUgPSBUUlVFLCAKICAgICAgICBnZGFsID0gY2hhcmFjdGVyKDApKQp9&marshal=dGVycmE6OndyYXAob2JqZWN0KQ&unmarshal=dGVycmE6OnVud3JhcChvYmplY3Qp&convert=&copy=&repository= 
#>   repository: local 
#>   iteration method: list 
#>   error mode: stop 
#>   memory mode: auto 
#>   storage mode: worker 
#>   retrieval mode: auto 
#>   deployment mode: worker 
#>   priority: 0 
#>   resources:
#>     list() 
#>   cue:
#>     seed: TRUE
#>     file: TRUE
#>     iteration: TRUE
#>     repository: TRUE
#>     format: TRUE
#>     depend: TRUE
#>     command: TRUE
#>     mode: thorough 
#>   packages:
#>     geotargets
#>     stats
#>     graphics
#>     grDevices
#>     utils
#>     datasets
#>     methods
#>     base 
#>   library:
#>     NULL
#> 
#> [[4]]
#> [[4]][[1]]
#> <tar_stem> 
#>   name: rast_split_grid_exts 
#>   description: Split into 4 tiles in a 2x2 grid 
#>   command:
#>     (function(x) tile_grid(x, ncol = 2, nrow = 2))(my_map) 
#>   format: rds 
#>   repository: local 
#>   iteration method: list 
#>   error mode: stop 
#>   memory mode: auto 
#>   storage mode: worker 
#>   retrieval mode: auto 
#>   deployment mode: worker 
#>   priority: 0 
#>   resources:
#>     list() 
#>   cue:
#>     seed: TRUE
#>     file: TRUE
#>     iteration: TRUE
#>     repository: TRUE
#>     format: TRUE
#>     depend: TRUE
#>     command: TRUE
#>     mode: thorough 
#>   packages:
#>     geotargets
#>     stats
#>     graphics
#>     grDevices
#>     utils
#>     datasets
#>     methods
#>     base 
#>   library:
#>     NULL
#> [[4]][[2]]
#> <tar_pattern> 
#>   name: rast_split_grid 
#>   description: Split into 4 tiles in a 2x2 grid 
#>   command:
#>     set_window(my_map, terra::ext(rast_split_grid_exts)) 
#>   pattern:
#>     map(rast_split_grid_exts) 
#>   format: format_custom&read=dGVycmE6OnJhc3QocGF0aCk&write=ewogICAgdGVycmE6OndyaXRlUmFzdGVyKG9iamVjdCwgcGF0aCwgZmlsZXR5cGUgPSAiR1RpZmYiLCBvdmVyd3JpdGUgPSBUUlVFLCAKICAgICAgICBnZGFsID0gY2hhcmFjdGVyKDApKQp9&marshal=dGVycmE6OndyYXAob2JqZWN0KQ&unmarshal=dGVycmE6OnVud3JhcChvYmplY3Qp&convert=&copy=&repository= 
#>   repository: local 
#>   iteration method: list 
#>   error mode: stop 
#>   memory mode: auto 
#>   storage mode: worker 
#>   retrieval mode: auto 
#>   deployment mode: worker 
#>   priority: 0 
#>   resources:
#>     list() 
#>   cue:
#>     seed: TRUE
#>     file: TRUE
#>     iteration: TRUE
#>     repository: TRUE
#>     format: TRUE
#>     depend: TRUE
#>     command: TRUE
#>     mode: thorough 
#>   packages:
#>     geotargets
#>     stats
#>     graphics
#>     grDevices
#>     utils
#>     datasets
#>     methods
#>     base 
#>   library:
#>     NULL
#> 
#> [[5]]
#> [[5]][[1]]
#> <tar_stem> 
#>   name: rast_split_n_exts 
#>   description: Split into 6 tiles 
#>   command:
#>     (function(x) tile_n(x, n = 6))(my_map) 
#>   format: rds 
#>   repository: local 
#>   iteration method: list 
#>   error mode: stop 
#>   memory mode: auto 
#>   storage mode: worker 
#>   retrieval mode: auto 
#>   deployment mode: worker 
#>   priority: 0 
#>   resources:
#>     list() 
#>   cue:
#>     seed: TRUE
#>     file: TRUE
#>     iteration: TRUE
#>     repository: TRUE
#>     format: TRUE
#>     depend: TRUE
#>     command: TRUE
#>     mode: thorough 
#>   packages:
#>     geotargets
#>     stats
#>     graphics
#>     grDevices
#>     utils
#>     datasets
#>     methods
#>     base 
#>   library:
#>     NULL
#> [[5]][[2]]
#> <tar_pattern> 
#>   name: rast_split_n 
#>   description: Split into 6 tiles 
#>   command:
#>     set_window(my_map, terra::ext(rast_split_n_exts)) 
#>   pattern:
#>     map(rast_split_n_exts) 
#>   format: format_custom&read=dGVycmE6OnJhc3QocGF0aCk&write=ewogICAgdGVycmE6OndyaXRlUmFzdGVyKG9iamVjdCwgcGF0aCwgZmlsZXR5cGUgPSAiR1RpZmYiLCBvdmVyd3JpdGUgPSBUUlVFLCAKICAgICAgICBnZGFsID0gY2hhcmFjdGVyKDApKQp9&marshal=dGVycmE6OndyYXAob2JqZWN0KQ&unmarshal=dGVycmE6OnVud3JhcChvYmplY3Qp&convert=&copy=&repository= 
#>   repository: local 
#>   iteration method: list 
#>   error mode: stop 
#>   memory mode: auto 
#>   storage mode: worker 
#>   retrieval mode: auto 
#>   deployment mode: worker 
#>   priority: 0 
#>   resources:
#>     list() 
#>   cue:
#>     seed: TRUE
#>     file: TRUE
#>     iteration: TRUE
#>     repository: TRUE
#>     format: TRUE
#>     depend: TRUE
#>     command: TRUE
#>     mode: thorough 
#>   packages:
#>     geotargets
#>     stats
#>     graphics
#>     grDevices
#>     utils
#>     datasets
#>     methods
#>     base 
#>   library:
#>     NULL
#> 
# }
```
