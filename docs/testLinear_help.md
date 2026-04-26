# `testLinear`

**Wenze Tang** and **Molin Wang**  
*Harvard T.H. Chan School of Public Health* 

## Description

`testLinear()` fits a restricted cubic spline (RCS) model to assess whether the association between a single exposure variable and an outcome can be adequately represented as linear. The function supports both linear models (`lm`) and generalized linear models (`glm`), allows adjustment for additional covariates, and returns formal likelihood-ratio-test-based evidence on nonlinearity together with a fitted curve plot.

For generalized linear models with `logit` or `log` link, the fitted curve is displayed on a ratio scale relative to a reference exposure value. For linear models, the fitted curve is displayed on the outcome scale.

---

## Usage

```r
testLinear(
  ds,
  var,
  outcome,
  adj = NULL,
  nknots = 5,
  knotsvalues = NULL,
  method = "lm",
  family,
  link,
  ref = NULL,
  densplot = FALSE
)
```

---

## Arguments

### `ds`
A data frame containing the outcome, the exposure of interest, and any adjustment covariates.

### `var`
A single character string giving the name of the exposure (independent variable) to be evaluated for linearity.

### `outcome`
A single character string giving the name of the outcome (dependent variable).

### `adj`
An optional character vector of covariate names to be adjusted for in the model. For plotting, numeric covariates are fixed at their mean values, while binary or categorical covariates are fixed at their mode.

### `nknots`
Number of knots for the restricted cubic spline. Default is `5`. Must be at least `3`.

### `knotsvalues`
Optional numeric vector specifying knot locations. If not supplied, knot locations are generated automatically using `Hmisc::rcspline.eval()`.

The code uses default knot placement rules based on the distribution of `var`:

- for 3 knots, outer quantiles are 0.10 and 0.90;
- for 4 to 6 knots, outer quantiles are 0.05 and 0.95;
- for more than 6 knots, outer quantiles are 0.025 and 0.975;
- with fewer than 100 non-missing values, outer knots are based on the 5th smallest and 5th largest observed values.

If the automatically generated knots are not unique, the function reduces the effective number of knots to the number of unique knot values and issues a warning.

### `method`
Model fitting method. Currently supported values are:

- `"lm"` for linear regression;
- `"glm"` for generalized linear regression.

### `family`
The GLM family function passed to `glm()`, such as `binomial` or `poisson`. Required when `method = "glm"`.

### `link`
Character string specifying the GLM link, such as `"logit"` or `"log"`. Required when `method = "glm"`.

### `ref`
Optional reference value for the exposure variable. For GLMs with `logit` or `log` link, fitted ratios are computed relative to this value. If omitted, the minimum observed exposure value is used.

### `densplot`
Logical value indicating whether to add a density plot of the exposure below the fitted spline curve. Default is `FALSE`.

---

## Details

The function first constructs restricted cubic spline basis terms for the exposure variable using `Hmisc::rcspline.eval()`. It then fits two or three nested models depending on the requested test and compares them with likelihood ratio tests using `lmtest::lrtest()`.

The following three tests are returned:

1. **Test of non-linear cubic spline terms**  
   Compares a model containing only the linear exposure term (plus adjustment covariates) against a model containing both the linear exposure term and the nonlinear spline terms. A small p-value suggests evidence against linearity.

2. **Test of overall curvature / overall association of spline terms**  
   Compares the full spline model against a model containing only the adjustment covariates (or intercept only if no covariates are provided). A small p-value suggests that the exposure is associated with the outcome through the spline-expanded terms.

3. **Test of linear term only**  
   Compares the model containing the linear exposure term (plus covariates) against the model with covariates only. A small p-value suggests evidence for a linear association.

For plotting:

- when `method = "lm"`, predictions and 95% confidence intervals are obtained directly from `predict(..., interval = "confidence")`;
- when `method = "glm"` with `link = "logit"` or `link = "log"`, the function computes fitted ratios relative to the reference value using the estimated coefficients and their covariance matrix;
- vertical dashed lines indicate the knot locations;
- if `densplot = TRUE`, a kernel density estimate of the exposure is shown underneath the fitted curve.

---

## Value

A named list with four components:

### `testOfLinearity`
A data frame summarizing the three likelihood ratio tests, including the chi-square statistic, degrees of freedom, and p-value.

### `fittedCBSplineCurve`
A `ggplot` object (or a combined figure if `densplot = TRUE`) showing the fitted restricted cubic spline curve and its pointwise 95% confidence band.

### `covariateFixedValues`
A one-row data frame containing the covariate values held fixed when generating the fitted curve. Numeric covariates are set to their mean and binary/categorical covariates to their mode.

### `knotValues`
The numeric vector of knot locations actually used in the spline construction.

---

## Interpretation

A common interpretation strategy is:

- inspect the **test of non-linear cubic spline terms** first to assess departure from linearity;
- inspect the fitted curve and confidence band to understand the shape of the exposure-outcome association;
- use the density plot, when requested, to check whether apparent nonlinear features occur in regions with limited data support.

A non-significant nonlinearity test does not prove exact linearity, but it suggests that the data do not provide strong evidence against a linear specification under the chosen model and knot configuration.

---

## Dependencies

The function loads the following packages internally:

- `dplyr`
- `lmtest`
- `Hmisc`
- `ggpubr`
- `data.table`

These packages must be installed before running the function.

---

## Notes

- The function is designed for assessing the linearity of **one exposure variable at a time**.
- For GLMs, the most fully supported plotting behavior in the code is for `logit` and `log` links.
- The plotted covariate pattern corresponds to a representative profile, not to every subject in the data.
- Automatic knot placement can become unstable when the exposure has many tied values or limited support; in such cases, user-specified `knotsvalues` may be preferable.

---

## Examples

### Example 1: Linear regression

```r
fit1 <- testLinear(
  ds = valid,
  var = "fqalc",
  outcome = "dralc",
  adj = c("fqcal", "fqtot", "agec"),
  nknots = 5,
  method = "lm",
  densplot = TRUE
)

fit1$testOfLinearity
fit1$fittedCBSplineCurve
fit1$covariateFixedValues
fit1$knotValues
```

### Example 2: Logistic regression with default reference

```r
fit2 <- testLinear(
  ds = main,
  var = "fqalc",
  outcome = "newcase",
  adj = c("fqtot", "fqcal", "agec"),
  nknots = 5,
  method = "glm",
  family = binomial,
  link = "logit",
  densplot = TRUE
)

fit2$testOfLinearity
fit2$fittedCBSplineCurve
```

### Example 3: Logistic regression with user-specified knots and reference value

```r
fit3 <- testLinear(
  ds = main,
  var = "fqalc",
  outcome = "newcase",
  adj = c("fqtot", "fqcal", "agec"),
  knotsvalues = quantile(main$fqalc, c(0.1, 0.3, 0.5, 0.7, 0.9)),
  method = "glm",
  family = binomial,
  link = "logit",
  ref = 5,
  densplot = FALSE
)
```

---

## Source

This documentation was prepared from the attached `testLinear` function code provided by the user. The function header, inline comments, argument descriptions, and implementation details were used to reconstruct this reference page.
