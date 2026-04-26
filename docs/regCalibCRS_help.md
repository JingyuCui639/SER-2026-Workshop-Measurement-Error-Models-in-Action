# `regCalibCRS`


**Wenze Tang** and **Molin Wang**  
*Harvard T.H. Chan School of Public Health*  

Correct regression coefficients for continuous surrogates using regression calibration

## Description

`regCalibCRS()` implements a regression calibration approach for correcting measurement error in continuous exposures and, optionally, additional mismeasured continuous covariates. The function fits a calibration model in a validation sample and then replaces the error-prone variables in the main study with calibrated values before fitting the outcome model. It returns corrected coefficient estimates, standard errors, Wald statistics, p-values, confidence intervals, and the estimated variance-covariance matrix.

The function is designed for main-study/external-validation settings and also contains code paths for internal validation settings controlled by `external = FALSE`. The supplementary slides describe both options and the required inputs.

## Usage

```r
regCalibCRS(
  ms,
  vs,
  sur,
  exp,
  vsIndicator,
  covCalib = NULL,
  covOutcome = NULL,
  outcome = NA,
  method = "lm",
  family = NA,
  link = NA,
  external = TRUE
)
```

## Arguments

### `ms`
Main study dataset as a data frame. It should minimally contain the variables named in `sur`, `outcome`, and `covOutcome` when applicable. For internal validation, it must also contain `exp` and `vsIndicator`.

### `vs`
Validation dataset as a data frame. Under external validation, it should minimally contain the variables in `exp`, `sur`, and `covCalib` when applicable. Under internal validation, the function constructs the validation subset from `ms`, but `vs` still appears in the function signature.

### `sur`
Character vector giving the names of the mismeasured exposure variables and/or covariates in the main study. These are the surrogate variables to be calibrated. `sur` must have the same length as `exp`.

### `exp`
Character vector giving the names of the correctly measured counterparts in the validation data. Each element must correspond one-to-one with the variable in the same position of `sur`.

### `vsIndicator`
Character string naming the indicator in the main study for whether a subject has a validation record. This is only needed when `external = FALSE`. Subjects with validation records are coded as `1`.

### `covCalib`
Optional character vector of correctly measured covariates to include in the calibration model. Nonlinear terms, interactions, and spline terms should be created in advance and supplied as ordinary columns rather than added through a formula.

### `covOutcome`
Optional character vector of correctly measured covariates to include in the outcome model together with the calibrated exposures. Nonlinear terms should also be pre-computed and passed as columns.

### `outcome`
Character string naming the outcome variable in the main study. This argument is required.

### `method`
Outcome-model fitting method. Currently supported values are `"lm"` and `"glm"`. The code uses `lm()` when `method = "lm"` and `glm()` when `method = "glm"`.

### `family`
Family object passed to `glm()` when `method = "glm"`, for example `binomial(link = "logit")` or `gaussian()`. Required for generalized linear outcome models. See ['family'](https://www.rdocumentation.org/packages/stats/versions/3.6.2/topics/family) for details of family functions.

### `link`
Character link name documented in the source and slides, such as `"logit"` or `"log"`. The current code uses `link` when constructing the asymptotic variance calculations, but the actual `glm()` call is driven by `family`, not by `link` directly. In practice, the safest choice is to specify the desired link inside `family` and keep `link` consistent with it.

### `external`
Logical indicator for the validation-study design. Use `TRUE` for an external validation dataset and `FALSE` for an internal validation design. When `external = FALSE`, the main study must contain the outcome and validation indicator so the function can separate validation and non-validation records.

## Details

The function proceeds in two main stages.

First, it fits a linear calibration model in the validation data, regressing the correctly measured variables `exp` on the surrogate variables `sur` and any covariates in `covCalib`. The fitted calibration model is then used to predict calibrated values in the main study.

Second, it replaces each target variable in `exp` within the main study by its calibrated prediction and fits the user-specified outcome model using `lm()` or `glm()` with the corrected variables and any additional covariates in `covOutcome`.

For inference, the function computes a large-sample sandwich-style variance estimator using block matrices built from the calibration and outcome models, and then reports Wald-based standard errors, z-statistics, p-values, and 95% confidence intervals. The output also includes the corrected coefficient variance-covariance matrix.

Rows with missing values in the variables required for a given stage are removed using complete-case selection before fitting the calibration and outcome models.

## Value

A named list with two components:

- `correctedCoefTable`: a matrix-like table containing corrected coefficient estimates, standard errors, z-values, p-values, and lower and upper 95% confidence limits.
- `correctedVCOV`: the estimated variance-covariance matrix for the corrected outcome-model coefficients.

## Required input structure

At a minimum, the following variables should be available.

### External validation (`external = TRUE`)

- `ms`: `sur`, `outcome`, and optionally `covOutcome`
- `vs`: `sur`, `exp`, and optionally `covCalib`

### Internal validation (`external = FALSE`)

- `ms`: `sur`, `exp`, `outcome`, `vsIndicator`, and any variables named in `covCalib` and `covOutcome`
- subjects with `vsIndicator == 1` are treated as having validation information

## Notes

1. The calibration model is linear in the supplied columns. If you want spline bases, interactions, or other nonlinear terms, create them in the dataset beforehand and include them through `covCalib` or `covOutcome`. This is also illustrated in the supplementary slides.

2. The function currently targets continuous error-prone variables under regression calibration. It is not written as a general-purpose interface for arbitrary measurement-error structures. The header comments describe it as handling measurement error in exposures or exposures plus covariates under linear calibration and linear/GLM outcome models.

3. The source code includes a revision note added by Jingyu Cui on Feb 20, 2026 to fix dimension-handling issues in the variance calculation and in extracting the outcome-model covariance block. That means the current file is slightly more recent than the original header version stamp.

4. Although the argument list contains `link`, the fitted generalized linear model is called as `glm(..., family = family)`. So `family` is the argument that actually determines the model family and link during fitting.

## Examples

### Example 1: logistic regression with an external validation study

```r
rcim <- regCalibCRS(
  ms = main,
  vs = valid,
  sur = c("fqtfatinc", "fqcalinc", "fqalcinc"),
  exp = c("drtfatinc", "drcalinc", "dralcinc"),
  covCalib = c("agec"),
  covOutcome = c("agec"),
  outcome = "case",
  method = "glm",
  family = binomial(link = "logit"),
  link = "logit",
  external = TRUE
)
```

This mirrors the example shown in the slides, except that the `family` argument is written in the standard `glm()` style with the link embedded in the family object.

### Example 2: linear regression with multiple calibrated variables

```r
fit <- regCalibCRS(
  ms = MS,
  vs = EVS,
  sur = c("Z1", "Z2"),
  exp = c("X1", "X2"),
  covCalib = c("V2", "V3", "V4"),
  covOutcome = c("V3", "V4"),
  outcome = "Y",
  method = "lm",
  external = TRUE
)
```

### Example 3: external validation with pre-computed nonlinear terms

```r
rcim_nl <- regCalibCRS(
  ms = main1,
  vs = valid1,
  sur = c("fqtfatinc", "fqcalinc", "fqalcinc"),
  exp = c("drtfatinc", "drcalinc", "dralcinc"),
  covCalib = c("agec", "rcsfqalc.1", "rcsfqalc.2", "rcsfqalc.3"),
  covOutcome = c("agec"),
  outcome = "case",
  method = "glm",
  family = binomial(link = "logit"),
  link = "logit",
  external = TRUE
)
```

This example follows the slide deck’s illustration of including nonlinear spline terms as already-created columns in the input data.

## Returned-object example

```r
fit <- regCalibCRS(...)

fit$correctedCoefTable
fit$correctedVCOV
```

## References

Carroll, R. J., Ruppert, D., Stefanski, L. A., and Crainiceanu, C. M. *Measurement Error in Nonlinear Models*. Chapman & Hall/CRC. The source file cites this book as the underlying reference for the method.
