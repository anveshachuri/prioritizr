# Add exclusive disjunction (xor) constraints

Add constraints to a conservation planning problem to specify that
particular pairs of planning units should not both be selected (in other
words, an exclusive or logical operation). If the problem has multiple
zones, then these constraints can be used to ensure that a pair of
planning units cannot both be allocated to the same zone, or that one
planning unit cannot be allocated to particular zone if another planning
unit is also allocated to a particular zone.

## Usage

``` r
# S4 method for class 'ConservationProblem,ANY,ANY'
add_xor_constraints(x, zones, data)

# S4 method for class 'ConservationProblem,ANY,data.frame'
add_xor_constraints(x, zones, data)

# S4 method for class 'ConservationProblem,ANY,matrix'
add_xor_constraints(x, zones, data)

# S4 method for class 'ConservationProblem,ANY,array'
add_xor_constraints(x, zones, data)
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
  cell values along the diagonal of the matrix indicate if pairs of
  planning units must not be allocated to the same zone. Defaults to an
  identity matrix (i.e., a matrix with ones along the matrix diagonal
  and zeros elsewhere), so that pairs of planning units cannot both be
  allocated to the same zone.

- data:

  `NULL`, `matrix`, `Matrix`, `data.frame`, or `array` object indicating
  which pairs of planning units cannot both be selected. For example, if
  `data` specifies that planning unit A and B planning unit cannot both
  be selected, then this means that a solution may select (i) only
  planning unit A, (ii) only planning unit B, or (iii) neither of the
  two planning units. See the Data format section for more information.

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
[`add_separation_constraints()`](https://prioritizr.net/reference/add_separation_constraints.md)

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
  add_default_solver(gap = 0, verbose = FALSE)

# define matrix for xor constraints so that a solution cannot select
# (i) both of the 85th and 86th raster pixels, and
# (ii) both the 97th and 98th raster pixels
xor_matrix <- Matrix::sparseMatrix(
  i = c(85, 97), j = c(86, 98), x = 1,
  dims = rep(terra::ncell(sim_pu_raster), 2),
  symmetric = TRUE
)

# create problem with constraints so that the solution cannot
# select these pairs of planning units
p2 <- p1 %>% add_xor_constraints(data = xor_matrix)

# solve problems
s1 <- c(solve(p1), solve(p2))
names(s1) <- c("basic solution", "xor constraints")

# prepare raster with labels to show pixel numbers
i <- sim_pu_raster * 0
i[] <- seq_len(terra::ncell(i))

# plot solutions
plot(s1, axes = FALSE, fun = function() terra::text(i))
```
