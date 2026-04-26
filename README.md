# Measurement Error Models in Action  
## The Latest Methods and Their Applications in Nutrition and Environmental Health

### 👨‍🏫 Welcome to the repository for the short course!


This repository contains the **demo code and datasets** used in the short course. The materials provide hands-on examples demonstrating how to implement modern measurement error correction methods in **R**.


## Course Instructors 

<img src="images/Raymond_Dusty_Rose.jpg" width="150">

**Raymond J. Carroll** is Distinguished Professor of Statistics, Nutrition, and Toxicology at Texas A&M University and one of the leading contributors to the statistical theory and practice of measurement error modeling. He has served as editor of *Biometrics* and *Journal of the American Statistical Association* and is the author of the widely used textbook *Measurement Error in Nonlinear Models*. His research spans measurement error methodology, semiparametric modeling, and applications in epidemiology and biomedical science.


![Donna Spiegelman](images/SC5-Donna-Spiegelman.png)

**Donna Spiegelman** is Professor of Biostatistics at the Yale School of Public Health and founding director of the Center for Methods in Implementation and Prevention Science. She previously served on the faculty at the Harvard T.H. Chan School of Public Health for nearly three decades. Her research focuses on statistical methods for epidemiology, including measurement error correction, causal inference, and methods for complex public health studies. 

![Molin Wang](images/SC5-Molin-Wang.png)

**Molin Wang** is Associate Professor of Epidemiology and Biostatistics at the Harvard T.H. Chan School of Public Health. Her research addresses statistical challenges arising in large epidemiologic cohort studies, with particular emphasis on measurement error methods and their applications in nutritional and environmental epidemiology. She serves as lead statistician for several major cohort studies including the Nurses’ Health Study II and the Health Professionals Follow-up Study. 

![Jingyu Cui](images/SC5-Jingyu-Cui.jpg)

**Jingyu Cui** is a Postdoctoral Associate in the Department of Biostatistics at the Yale School of Public Health. His research focuses on developing statistical methods for complex and high-dimensional data, including measurement error models, adaptive study designs for clinical trials, and statistical inference under imperfect data. 

---

## 📚 Agenda

### 1. Impact of Measurement Error and Overview of Correction Methods  [1:00–1:30 PM]  
- **Raymond J. Carroll** *[Slides](https://raw.githubusercontent.com/JingyuCui639/ENAR-2026-SC5-Measurement-Error-Models-in-Action/main/course_material/1-Intro/ENAR_2026_RJC.pdf)*  
- **Donna Spiegelman** *[Slides](https://raw.githubusercontent.com/JingyuCui639/ENAR-2026-SC5-Measurement-Error-Models-in-Action/main/course_material/1-Intro/intro%20donna.pptx)*  

### 2. Main Study/Validation Study Designs and Main Study/Reliability Study Designs  [ 1:30–2:00 PM]  
- **Donna Spiegelman** *[Slides](https://raw.githubusercontent.com/JingyuCui639/ENAR-2026-SC5-Measurement-Error-Models-in-Action/main/course_material/2-Study_design/study%20design%20module%20--%20ENAR%20half%20day%20workshop%202026.pptx)*  

### 3. Regression Calibration Methods for Adjusting Measurement Error Bias  [2:00–3:00 PM]  
- **Molin Wang** *[Slides](https://raw.githubusercontent.com/JingyuCui639/ENAR-2026-SC5-Measurement-Error-Models-in-Action/main/course_material/3-Regression_calibration/ENAR_short_course%20MW%20final.pdf)*  

### 4. Hands-on Lab: R Examples from Nutritional and Environmental Epidemiology  [ 3:00–3:45 PM]  
- **Jingyu Cui**  *[Download Code](https://raw.githubusercontent.com/JingyuCui639/ENAR-2026-SC5-Measurement-Error-Models-in-Action/main/course_material/4-R_lab.zip
)*;      *[Code running instructions](https://github.com/JingyuCui639/ENAR-2026-SC5-Measurement-Error-Models-in-Action/blob/main/README.md#%EF%B8%8F-setup-instructions-for-r-seccion)*

### Break  [**Time:** 3:45–4:00 PM]  

### 5. Machine Learning Methods for Measurement Error Correction  [4:00–4:30 PM]  
- **Molin Wang** *[Slides](https://raw.githubusercontent.com/JingyuCui639/ENAR-2026-SC5-Measurement-Error-Models-in-Action/main/course_material/5-Machine_learning/DML_ENAR%202026%20Molin%20final.pptx
)*
  - Variable selection in regression calibration, with discussion of alternative approaches based on our work with Wenze 
  - Robust confounder control in higher-dimensional settings   
   

### 6. Applications in Nutritional and Environmental Epidemiology  [4:30–5:00 PM]  
- **Donna Spiegelman** *[Slides](https://github.com/JingyuCui639/ENAR-2026-SC5-Measurement-Error-Models-in-Action/tree/main/course_material/6-Case_studies)*
---

## ⚙️ Setup Instructions for R Seccion

#### 1. Install R and RStudio
Please ensure that the following software is installed on your computer.

**R**
https://cran.r-project.org/

**RStudio**
https://posit.co/download/rstudio-desktop/

#### 2. Download R Code 
[Link](https://raw.githubusercontent.com/JingyuCui639/ENAR-2026-SC5-Measurement-Error-Models-in-Action/main/course_material/4-R_lab.zip
)

**⚠ Data Use Notice:**  
**The dataset provided here is for instructional use in this short course only and may not be copied, redistributed, or used for any other purpose.**

#### 3. Unzip the folder

After unzipping the file, the folder will contain the following files:

| File | Description |
|-----|-------------|
| `main_data_external.csv` | Main study dataset |
| `valid_data_external.csv` | External validation dataset |
| `regCalibCRS.R` | R function implementing regression calibration using the **imputation method** |
| `regCalibRSW.R` | R function implementing regression calibration using the **deattenuation factor method** |
| `testLinear.R` | R function testing the linearity of measurement error and outcome models |
| `Regression Calibration demo2.qmd` | Quarto document containing the demonstration code |

#### 4. Run `Regression Calibration demo2.qmd` in **RStudio**

---

## Solution of Demo Code

Please see the solution of the demo code: [solution](https://jingyucui639.github.io/ENAR-2026-SC5-Measurement-Error-Models-in-Action/).


# 📌 Notes

- [Course Slides](/course_material) are available in the folder `/course_material`.
- See the  documentation for [regCalibCRS](docs/regCalibCRS_help.md) to implement imputation-based regression calibration method.
- See the documentation for  [regCalibRSW](docs/regCalibRSW_help.md) to implement deattenuation factor method.
- See the documentation for [testLinear](docs/testLinear_help.md) to test linearity for measurement error and outcome models.




