# Add separation constraints

Add constraints to a conservation planning problem to ensure that
selected planning units are separated from each other. If the problem
has multiple zones, then these constraints can be used to ensure that
planning units allocated to different zones are separated from each
other.

## Usage

``` r
# S4 method for class 'ConservationProblem,ANY,ANY'
add_separation_constraints(x, zones, data)

# S4 method for class 'ConservationProblem,ANY,data.frame'
add_separation_constraints(x, zones, data)

# S4 method for class 'ConservationProblem,ANY,matrix'
add_separation_constraints(x, zones, data)

# S4 method for class 'ConservationProblem,ANY,array'
add_separation_constraints(x, zones, data)
```

## Arguments

- x:

  [`problem()`](https://prioritizr.net/reference/problem.md) object.

- zones:

  `matrix` or `Matrix` object describing the separation scheme for
  different zones. Each row and column corresponds to a different zone
  in `x`, and cell values must contain binary `numeric` values (i.e.,
  one or zero) that indicate if neighboring planning units (per `data`)
  must not be allocated to the corresponding combination of zones. The
  cell values along the diagonal of the matrix indicate if neighboring
  planning units must not be allocated to the same zone. Defaults to an
  identity matrix (i.e., a matrix with ones along the matrix diagonal
  and zeros elsewhere), so that neighboring planning units cannot both
  be allocated to the same zone.

- data:

  `NULL`, `matrix`, `Matrix`, `data.frame`, or `array` object showing
  which planning units should be separated from each other. For example,
  [`adjacency_matrix()`](https://prioritizr.net/reference/adjacency_matrix.md)
  can be used to separate planning units from their immediate neighbors.
  Alternatively,
  [`proximity_matrix()`](https://prioritizr.net/reference/proximity_matrix.md)
  can be used to separate planning units based on a distance threshold.
  Defaults to `NULL` such that planning units are automatically
  separated based on the
  [`adjacency_matrix()`](https://prioritizr.net/reference/adjacency_matrix.md)
  function. See the Data format section for more information.

## Value

An updated [`problem()`](https://prioritizr.net/reference/problem.md)
object with the constraints added to it.

## Data format

The following formats can be used to specify `data`.

- `data` as a `NULL` value:

  Here the neighborhood data are calculated automatically using the
  [`adjacency_matrix()`](https://prioritizr.net/reference/adjacency_matrix.md)
  function. This is the default for `data`. Note that the neighborhood
  data must be manually defined using one of the other formats below if
  the planning unit data in `x` is not spatially referenced (e.g.,
  `data.frame` or `numeric` format).

- `data` as a `matrix`/`Matrix` object:

  Here rows and columns correspond to different planning units and cell
  values indicate if two planning units are neighbors or not. Cells must
  have binary `numeric` values (i.e., one or zero). Note that cells
  along the matrix diagonal have no effect on the solution because each
  planning unit cannot be a neighbor with itself.

- `data` as a `data.frame` object:

  Here rows correspond to a pair of planning units and columns provide
  information about each pair of planning units. In particular, `data`
  must have the columns: `"id1"`, `"id2"`, and `"boundary"`. The `"id1"`
  and `"id2"` columns contain identifiers (indices) for a pair of
  planning units, and the `"boundary"` column contains binary `numeric`
  values that indicate if the two planning units specified in the
  `"id1"` and `"id2"` columns should be treated as neighbors or not.
  These data can be used to describe symmetric or asymmetric
  relationships between planning units. By default, input data is
  assumed to be symmetric unless asymmetric data is specified (e.g., if
  data is present for planning units 2 and 3, then the same amount of
  connectivity is expected for planning units 3 and 2, unless
  connectivity data is also provided for planning units 3 and 2). If `x`
  has multiple zones, then the `"zone1"` and `"zone2"` columns can
  optionally be provided to manually specify that the neighborhood data
  pertain to specific zones. The `"zone1"` and `"zone2"` columns should
  contain the `character` names of the zones. Note that if the columns
  `"zone1"` and `"zone2"` are present, then `zones` must be `NULL`.

- `data` as an `array` object:

  Here a four-dimension array containing binary `numeric` values is used
  to specify if planning unit should be treated as neighbors with every
  other planning unit when they are allocated to every combination of
  management zone. The first two dimensions (i.e., rows and columns)
  correspond to the planning units, and second two dimensions correspond
  to the management zones. For example, if `data` had a value of 1 at
  the index `data[1, 2, 3, 4]`, this would indicate that planning unit 1
  and planning unit 2 should be treated as neighbors when they are
  allocated to zones 3 and 4 (respectively).

## See also

Other functions for adding constraints:
[`add_contiguity_constraints()`](https://prioritizr.net/reference/add_contiguity_constraints.md),
[`add_cost_constraints()`](https://prioritizr.net/reference/add_cost_constraints.md),
[`add_feature_contiguity_constraints()`](https://prioritizr.net/reference/add_feature_contiguity_constraints.md),
[`add_linear_constraints()`](https://prioritizr.net/reference/add_linear_constraints.md),
[`add_locked_in_constraints()`](https://prioritizr.net/reference/add_locked_in_constraints.md),
[`add_locked_out_constraints()`](https://prioritizr.net/reference/add_locked_out_constraints.md),
[`add_mandatory_allocation_constraints()`](https://prioritizr.net/reference/add_mandatory_allocation_constraints.md),
[`add_manual_bounded_constraints()`](https://prioritizr.net/reference/add_manual_bounded_constraints.md),
[`add_manual_locked_constraints()`](https://prioritizr.net/reference/add_manual_locked_constraints.md),
[`add_neighbor_constraints()`](https://prioritizr.net/reference/add_neighbor_constraints.md),
[`add_xor_constraints()`](https://prioritizr.net/reference/add_xor_constraints.md)

## Examples

``` r
# load data
sim_pu_raster <- get_sim_pu_raster()
sim_features <- get_sim_features()

# create minimal problem
p1 <-
  problem(sim_pu_raster, sim_features) %>%
  add_min_set_objective() %>%
  add_relative_targets(0.1) %>%
  add_binary_decisions() %>%
  add_default_solver(verbose = FALSE)

# create problem with constraints so that none of the selected
# planning units are neighbors with each other
p2 <- p1 %>% add_separation_constraints()

# solve problems
s1 <- c(solve(p1), solve(p2))
names(s1) <- c("basic solution", "separation constraints")

# plot solutions
plot(s1, axes = FALSE)


# create problem so that each feature is represented in at least three
# spatially separate protected areas, using a zone for each replicate

# prepare feature data for zones,
# here we define three zones called r1, r2, and r3, and each zone has
# a copy of the same feature data
z <- zones(
  "r1" = sim_features, "r2" = sim_features, "r3" = sim_features,
  feature_names = names(sim_features)
)

# prepare zones,
# here we create a matrix that a value of 1 on non-diagonal elements and
# zones on diagonal elements of the matrix to specify that the separation
# constraints should be applied to planning units when they are allocated to
# different zones
zones_matrix <- matrix(1, nrow = 3, ncol = 3)
diag(zones_matrix) <- 0
print(zones_matrix)
#>      [,1] [,2] [,3]
#> [1,]    0    1    1
#> [2,]    1    0    1
#> [3,]    1    1    0

# create problem,
# here we specify that each zone has same cost values, targets so that each
# feature should be represented at least once in each and every zone, and
# separation constraints to prevent planning units that are allocated to
# different zones from being neighbors
p3 <-
  problem(c(sim_pu_raster, sim_pu_raster, sim_pu_raster), z) %>%
  add_min_set_objective() %>%
  add_absolute_targets(
    matrix(1, nrow = terra::nlyr(sim_features), ncol = 3)
  ) %>%
  add_separation_constraints(zones = zones_matrix) %>%
  add_binary_decisions() %>%
  add_default_solver(verbose = FALSE)

# solve problem
s3 <- category_layer(solve(p3))

# plot solution
plot(s3, main = "replicates", axes = FALSE)
```
