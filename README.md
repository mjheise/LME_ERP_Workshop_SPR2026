# LME_ERP_Workshop_SPR2026
## Linear Mixed Effects Models for Event-Related Potential Research: Advantages, Theory, and a Practical Walkthrough

### Overview

This repository contains the materials from a workshop presented at the Society for Psychophysiological Research (SPR) 2026 Annual Meeting (October 14, 2026; Halifax, Nova Scotia, Canada). The workshop covers why linear mixed effects models (LMEs) are useful for trial-level ERP data, the theory behind fixed and random effects, and a step-by-step walkthrough of fitting, checking, and reporting LMEs in R. It also includes simulation-based power analyses for trial-level LMEs.

All of the data in these tutorials are **simulated**, so every script runs on its own without any data files. Each tutorial comes in two versions: a complete, knitted walkthrough (`.Rmd` + `.html`) and a code-along version with blanks to fill in during the workshop.

For a written summary of the model-building process, see **[Nine steps for fitting LMEs to ERP data](NineStepsForFittingLMEs.md)**.

### Repository Structure

```
01_EmotionFaces_Nc/
  LME_ex1_emotionfaces.Rmd             # Full walkthrough: the 9 steps, interpretation, common errors
  LME_ex1_emotionfaces.html            # Knitted version (open in a browser)
  LME_ex1_emotionfaces_codealong.Rmd   # Code-along version used during the workshop
02_Oddball_MMN/
  LME_ex2_oddball.Rmd                  # Single-trial MMN difference waves across age
  LME_ex2_oddball.html
  LME_ex2_oddball_codealong.Rmd
  LME_ex2_oddball_codealong.html
03_PowerAnalysis/
  LME_tutorial3_power.Rmd              # Simulation-based power for two study designs
  LME_tutorial3_power.html
Slides/                                # Workshop slides (PDF)
NineStepsForFittingLMEs.md             # Summary of the 9-step model-building process
README.md
```

### Requirements

R (≥ 4.4.0) with the following packages. The tutorials were run with R 4.4.3; versions in comments are the ones used.

```r
install.packages(c(
  "tidyverse",    # data wrangling, ggplot2 (2.0.0)
  "lme4",         # linear mixed effects models (1.1-36)
  "lmerTest",     # p-values with Satterthwaite df (3.1-3)
  "emmeans",      # marginal means, contrasts, and slopes (1.11.0)
  "performance",  # check_model(), ICC, R², convergence and singularity checks (0.18.2)
  "see",          # plots for performance (0.14.2)
  "sjPlot",       # tab_model() tables and random-effect plots (2.8.17)
  "broom.mixed",  # tidy model output
  "kableExtra",   # formatted tables
  "faux"          # simulating crossed designs (1.2.4)
))

# mixedpower is not on CRAN:
# install.packages("remotes")
remotes::install_github("DejanDraschkow/mixedpower")   # simulation-based power (0.1.0)
```

### Tutorial Overview

#### 1. Face emotion and the Nc (`01_EmotionFaces_Nc`)
Simulated preschoolers (40 children aged 3–5) view happy and angry faces from 5 actors, 10 times each (100 trials per child), and we model single-trial Nc mean amplitude. The tutorial walks through all nine model-building steps: defining the research question, choosing fixed and random effects, coding and centering predictors, fitting the maximal model and simplifying it when it doesn't converge or is singular, checking assumptions with `performance::check_model()`, and interpreting, plotting, and reporting the final model. It also compares dummy and effects coding, estimated marginal means, and standardized betas as an effect size.

#### 2. Auditory oddball and the MMN (`02_Oddball_MMN`)
A simulated oddball study (40 children aged 2–4; a standard /ba/ and deviant /da/ recorded by 6 speakers) loosely based on Mon et al. (2024) and Coy et al. (2024). We build a single-trial MMN for every deviant (the deviant minus the standard right before it) and ask whether the MMN changes with age and habituates across the session, with random slopes for habituation by participant.

#### 3. Power analysis (`03_PowerAnalysis`)
Simulation-based power analyses with `mixedpower` and `faux` for two designs:
- **Design 1:** an N400 study of dyslexic and typical adults, with parameters taken from ERP CORE (Kappenman et al., 2021) and a meta-analysis (Basma et al., 2024). Covers smallest effects of interest, how individual differences (random slopes) cost power, and how many participants to recruit when trials are lost to artifacts.
- **Design 2:** a preschool face-emotion (Nc) study with high trial-to-trial noise, Nc habituation, and heavy data loss. Covers how participants, trials per condition, artifact rejection, and the number of stimuli trade off.

### Resources

**Textbooks**
- Raudenbush, S. W., & Bryk, A. S. (2002). *Hierarchical linear models: Applications and data analysis methods* (2nd ed.). Sage.
- Snijders, T. A. B., & Bosker, R. J. (2012). *Multilevel analysis: An introduction to basic and advanced multilevel modeling* (2nd ed.). Sage.

**Power and study planning**
- DeBruine, L. M., & Barr, D. J. (2021). Understanding mixed-effects models through data simulation. *Advances in Methods and Practices in Psychological Science, 4*(1).
- Kappenman, E. S., Farrens, J. L., Zhang, W., Stewart, A. X., & Luck, S. J. (2021). ERP CORE: An open resource for human event-related potential research. *NeuroImage, 225*, 117465.
- Kumle, L., Võ, M. L.-H., & Draschkow, D. (2021). Estimating power in (generalized) linear mixed models: An open introduction and tutorial in R. *Behavior Research Methods, 53*, 2528–2543.
- Westfall, J., Kenny, D. A., & Judd, C. M. (2014). Statistical power and optimal design in experiments in which samples of participants respond to samples of stimuli. *Journal of Experimental Psychology: General, 143*(5), 2020–2045.

**LME methods for ERP research**
- Barr, D. J., Levy, R., Scheepers, C., & Tily, H. J. (2013). Random effects structure for confirmatory hypothesis testing: Keep it maximal. *Journal of Memory and Language, 68*(3), 255–278.
- Ferron, J. M., Hogarty, K. Y., Dedrick, R. F., Hess, M. R., Niles, J. D., & Kromrey, J. D. (2008). Reporting results from multilevel analyses. In A. A. O'Connell & D. B. McCoach (Eds.), *Multilevel modeling of educational data* (pp. 391–426). Information Age.
- Heise, M. J., Mon, S. K., & Bowman, L. C. (2022). Utility of linear mixed effects models for event-related potential research with infants and children. *Developmental Cognitive Neuroscience, 54*, 101070. https://doi.org/10.1016/j.dcn.2022.101070
- Heise, M. J., Mon, S. K., & Bowman, L. C. (2025). Analysis of event-related potential difference waves can benefit from linear mixed effects modeling: Recommendations for analyses and general model fitting. *Developmental Cognitive Neuroscience, 76*, 101614. https://doi.org/10.1016/j.dcn.2025.101614
- Matuschek, H., Kliegl, R., Vasishth, S., Baayen, H., & Bates, D. (2017). Balancing Type I error and power in linear mixed models. *Journal of Memory and Language, 94*, 305–315.
- Mon, S. K., Manning, B. L., Wakschlag, L. S., & Norton, E. S. (2024). Leveraging mixed-effects location scale models to assess the ERP mismatch negativity's psychometric properties and trial-by-trial neural variability in toddler-mother dyads. *Developmental Cognitive Neuroscience, 70*, 101459.
- Schielzeth, H., Dingemanse, N. J., Nakagawa, S., Westneat, D. F., Allegue, H., Teplitsky, C., Réale, D., Dochtermann, N. A., Garamszegi, L. Z., & Araya-Ajoy, Y. G. (2020). Robustness of linear mixed-effects models to violations of distributional assumptions. *Methods in Ecology and Evolution, 11*(9), 1141–1152.
- Volpert-Esmond, H. I., Page-Gould, E., & Bartholow, B. D. (2021). Using multilevel models for the analysis of event-related potentials. *International Journal of Psychophysiology, 162*, 145–156.

**Code and materials**
- LME tutorial and simulation scripts from Heise et al. (2022): https://github.com/basclab/LME_MixedEffectsERPTutorial
- LME scripts for ERP difference waves from Heise et al. (2025): https://github.com/basclab/LME_MixedEffectsERPDifferenceWave
- `mixedpower` power notebooks (Kumle, Võ, & Draschkow): https://lkumle.github.io/power_notebooks/
- ERP CORE: https://erpinfo.org/erp-core
- Westfall, Kenny, & Judd (2014) power app: https://jakewestfall.shinyapps.io/crossedpower/

### Citation

Heise, M. J. (2026, October 14). *Linear mixed effects models for event-related potential research* [Workshop materials]. Society for Psychophysiological Research Annual Meeting, Halifax, Nova Scotia, Canada.

### Contact

MJ Heise, PhD — mj.heise@ucsf.edu
