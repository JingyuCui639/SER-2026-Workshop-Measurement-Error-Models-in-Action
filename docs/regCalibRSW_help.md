# regCalibRSW

**Wenze Tang** and **Molin Wang**  
*Harvard T.H. Chan School of Public Health*  


## Description

`regCalibRSW()` implements a regression-calibration deattenuation-factor approach (RC-DF) for correcting bias from measurement error in continuous exposures and/or covariates. The function supports both main study/external validation study (MS/EVS) and main study/internal validation study (MS/IVS) settings. It returns corrected coefficient estimates together with standard errors, Wald p-values, confidence intervals, and variance-covariance matrices. The function can either fit the uncorrected outcome model internally or, in the external-validation setting, accept user-supplied uncorrected estimates and their variance-covariance matrix.

## Usage

```r
regCalibRSW(
  supplyEstimates = FALSE,
  ms,
  vs,
  sur,
  exp,
  covCalib = NULL,
  covOutcome = NULL,
  outcome = NA,
  event,
  time,
  method = "lm",
  family = NA,
  link = NA,
  external = TRUE,
  pointEstimates = NA,
  vcovEstimates = NA
)
```

## Arguments

### `supplyEstimates`
Logical. Indicates whether uncorrected coefficient estimates and their variance-covariance matrix are supplied by the user. If `TRUE`, `ms` becomes optional and ordinary regression output from the uncorrected model is not returned. This option is only available when `external = TRUE`.

### `ms`
Data frame for the main study. When `supplyEstimates = FALSE`, this data set should minimally contain the variables listed in `sur`, `covCalib`, and `covOutcome` (if any), as well as the outcome variables needed by the chosen model.

### `vs`
Data frame for the validation study, either internal or external. It should minimally contain the variables listed in `exp`, `sur`, and `covCalib`; when `external = FALSE`, it must also contain the outcome variable.

### `sur`
Character vector naming the mismeasured exposure and/or covariates (surrogates) in the main study.

### `exp`
Character vector naming the corresponding correctly measured exposure and/or covariates in the validation study. Must have the same length as `sur` and correspond one-to-one to the surrogate variables.

### `covCalib`
Optional character vector of correctly measured covariates to adjust for in both the calibration model and the outcome model. These variables must not overlap with `covOutcome`.

### `covOutcome`
Optional character vector of correctly measured risk factors to include in the outcome model but not treated as exposure surrogates. These should not overlap with `covCalib`. The function warns if such variables appear strongly associated with a surrogate, suggesting they may belong in `covCalib` instead.

### `outcome`
Character string naming the outcome variable. Required when `method` is `"lm"` or `"glm"`.

### `event`
Character string naming the event indicator for Cox models. Required when `method = "cox"`. The code comments describe the usual coding as `0 = alive/censored`, `1 = dead/event`; interval-censored coding is also described in the source.

### `time`
Character string naming the follow-up time for Cox models. Required when `method = "cox"`.

### `method`
Character string specifying the outcome-modeling method. Supported values are `"lm"`, `"glm"`, and `"cox"`. Cox support is only available when `external = TRUE`.

### `family`
Family function passed to `glm()`. Required when `method = "glm"`. This should be a family function, not a character string, for example `binomial` or `gaussian`.

### `link`
Character string for the GLM link. The source and supplementary slides document this as a separate argument for GLM use. In practice, the function calls `glm(..., family = family(link))`, so valid usage should be consistent with that implementation.

### `external`
Logical. Indicates whether `vs` is an external validation study. If `FALSE`, the validation study is treated as internal and must contain the outcome variable. In that case, user-supplied uncorrected estimates are not allowed.

### `pointEstimates`
Numeric vector of uncorrected regression coefficient estimates from the standard outcome model, excluding the intercept. Must be named. Required when `supplyEstimates = TRUE`.

### `vcovEstimates`
Variance-covariance matrix for the uncorrected regression coefficients, excluding the intercept. Must be square and have column names matching the supplied coefficients. Required when `supplyEstimates = TRUE`.

## Details

The function first validates the inputs, checks variable availability in the supplied data frames, and ensures that `sur` and `exp` have the same length. It then constructs:

1. an uncorrected outcome model using the main study, unless `supplyEstimates = TRUE`;
2. a calibration model in the validation data linking the correctly measured variables in `exp` to the surrogate variables in `sur` and any calibration covariates in `covCalib`;
3. a corrected coefficient vector obtained by transforming the uncorrected coefficient estimates using the estimated calibration matrix;
4. a corrected variance-covariance matrix using matrix-based delta-method calculations.

When `external = FALSE`, the function additionally fits an outcome model using the correctly measured exposure variables in the internal validation data and combines that information with the corrected main-study estimator through inverse-covariance weighting.

The implementation uses complete cases for the variables required by the chosen outcome model and the calibration model.

## Returned Value

A named list. The exact components depend on `supplyEstimates`.

When `supplyEstimates = FALSE`, the output contains:

- `correctedCoefTable`: matrix/data frame with corrected estimates, standard errors, z-values, p-values, and 95% confidence intervals;
- `correctedVCOV`: variance-covariance matrix of corrected estimates;
- `standardCoefTable`: table from the uncorrected fitted outcome model;
- `standardVCOV`: variance-covariance matrix from the uncorrected fitted outcome model;
- `calibrationModelCoefTable`: estimated calibration-model coefficient matrix;
- `calibrationModelVCOV`: estimated covariance matrix for the calibration model.

When `supplyEstimates = TRUE`, the output contains:

- `correctedCoefTable`;
- `correctedVCOV`;
- `calibrationModelCoefTable`;
- `calibrationModelVCOV`.

## Warnings and Constraints

- `covCalib` and `covOutcome` must not overlap.
- In the internal-validation setting (`external = FALSE`), the validation data must contain the outcome.
- In the internal-validation setting, `supplyEstimates = TRUE` is not allowed.
- If factor levels for categorical variables are inconsistent between the main study and validation study, the function may stop because the required design-matrix dimensions do not match.
- The function source documents support for GLM and Cox models, but the deattenuation-factor slides emphasize the RC-DF implementation for general linear model correction and note the distinction from the sandwich-based implementation used in `regCalibCRS()`.

## Examples

### Example 1: external validation with built-in GLM

```r
rcdf <- regCalibRSW(
  supplyEstimates = FALSE,
  ms = main,
  vs = valid,
  sur = c("fqtfatinc", "fqcalinc", "fqalcinc"),
  exp = c("drtfatinc", "drcalinc", "dralcinc"),
  covCalib = c("agec"),
  covOutcome = NULL,
  outcome = "case",
  method = "glm",
  family = binomial,
  link = "logit",
  external = TRUE,
  pointEstimates = NA,
  vcovEstimates = NA
)
```

### Example 2: external validation with user-supplied uncorrected estimates

```r
rcdf <- regCalibRSW(
  supplyEstimates = TRUE,
  vs = EVS,
  sur = c("Z1", "Z2"),
  exp = c("X1", "X2"),
  covCalib = c("V1", "V2", "V3", "V4"),
  covOutcome = NA,
  outcome = "Y",
  external = TRUE,
  pointEstimates = pointEst,
  vcovEstimates = vcovEst
)
```

### Example 3: internal validation with linear regression

```r
rcdf <- regCalibRSW(
  supplyEstimates = FALSE,
  ms = MS,
  vs = IVS,
  sur = c("Z1", "Z2"),
  exp = c("X1", "X2"),
  covCalib = c("V1", "V2", "V3", "V4"),
  covOutcome = c("R"),
  outcome = "Y",
  method = "lm",
  external = FALSE
)
```

## Notes

The source comments state that this function corrects for measurement error in exposure or in both exposure and covariates, and that a validation study is required to characterize the calibration model empirically. The cited methodological background includes Rosner et al. (1989, 1990), Spiegelman et al. (1997), and Spiegelman, Carroll, and Kipnis (2001).

A practical implementation note is that the function documentation treats `family` and `link` as separate GLM arguments, while the code uses `glm(..., family = family(link))`. So, for successful use, the supplied `family` object must be compatible with that call pattern.

## References

Rosner B, Willett WC, Spiegelman D. Correction of logistic relative risk estimates and confidence intervals for systematic within-person measurement error. *Statistics in Medicine*. 1989;8:1051-1069.

Rosner B, Spiegelman D, Willett WC. Correction of logistic regression relative risk estimates and confidence intervals for measurement error: the case of multiple covariates measured with error. *American Journal of Epidemiology*. 1990;132:734-735.

Spiegelman D, McDermott A, Rosner B. The many uses of the regression calibration method for measurement error bias correction in nutritional epidemiology. *American Journal of Clinical Nutrition*. 1997;65:1179S-1186S.

Spiegelman D, Carroll RJ, Kipnis V. Efficient regression calibration for logistic regression in main study/internal validation study designs with an imperfect reference instrument. *Statistics in Medicine*. 2001;20:139-160.
