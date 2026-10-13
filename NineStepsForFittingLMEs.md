# Nine steps for fitting LMEs to ERP data

*This is a written version of the model-building walkthrough from the SPR 2026 workshop "No trial left behind: A practical guide to linear mixed effects models for event-related potential data". The full R code for the running example is in [`01_EmotionFaces_Nc`](01_EmotionFaces_Nc/), and there's a second example in [`02_Oddball_MMN`](02_Oddball_MMN/).*

Linear mixed effects models (LMEs) let you analyze ERP mean amplitude on every trial instead of averaging trials first. That means you keep participants who only have a few usable trials, you can model trial-level effects like habituation, and your estimates are less biased when data are systematically missing (Heise et al., 2022).

LMEs are flexible, and that's one of their biggest strengths but also part of what makes them hard to implement: there aren't many universal rules about which variables to include, which random effects to fit, how to center predictors, whether to use dummy or effects coding...(the list goes on!). Instead of rules for model fitting, we'll apply a 9-step process to help guide us toward making the right decisions for our ERP data, study design, and research question.

1. Define the research question
2. List all relevant variables
3. Draw the study design schematic
4. Categorize variables by type and level
5. Assign fixed and random effects
6. Consider data transformations and center predictors (optional)
7. Fit the maximal model and simplify until it converges
8. Check model assumptions
9. Report and visualize results

**The example I use throughout:** 40 children aged 3–5 see faces from 5 actors showing happy and angry expressions. Each actor × emotion is shown 10 times, so each child gets 100 trials, and every child sees every actor show every emotion. The outcome is Nc mean amplitude on every trial.

---

## Step 1: Define the research question

Start with your outcome. In an LME, trial-level mean amplitude is the outcome and everything else (condition, presentation number, age) predicts mean amplitude. LMEs have particular utility for datasets where you want to model within-participant variability or trial-level effects like habituation, if participants have different numbers of trials, or if data are likely to be systematically missing.

One thing LMEs can't do is use the ERP as a *predictor*. For example, you can't use trial-level N400 amplitude at 2 years to predict the child's vocabulary score at 4 years. There are some potential workarounds, but overall LMEs perform best and have the most utility for research where the outcome is single-trial mean amplitude (or single-trial difference waves!).

Our questions: (1) Is the Nc different for happy vs. angry faces? (2) Does that difference change with age?

## Step 2: List all relevant variables

List anything that predicts mean amplitude, or that predicts both mean amplitude and whether a trial is missing. If you can, this step is ideal to do when planning new studies before any data are collected.

## Step 3: Draw the study design schematic

Sketch how participants, stimuli, conditions, and trials fit together: what's crossed (e.g., every participant sees every stimulus) and what's nested (e.g.,  (each actor only appears in one emotion condition, so actors are nested within emotion). This will later help with identifying what random effects structure your design can support.

## Step 4: Categorize variables by type and level

**Type:** Is it *continuous* (e.g., age, vocabulary) or *categorical* (e.g., emotion, condition, electrode, race/ethnicity, education)?

**Level:** Is it *participant-level*, with one value per participant (e.g., age, group, questionnaire scores)? Or *trial-level*, changing from trial to trial (e.g., condition, electrode, reaction time, stimulus features, presentation number)?

## Step 5: Assign fixed and random effects

**Fixed effects** are any predictors you care about interpreting from your model, plus nuisance variables with only a few categories (e.g., 3 electrodes). In our example, the fixed effects are emotion, presentation number, age, and any interactions between these variables.

**Random effects** are grouping variables with lots of categories (more than about 5; Volpert-Esmond et al., 2021) that you want your results to generalize beyond. If you ran the study again with 50 new kids, or with a different face set, you'd want the same answer.

- A **random intercept** gives each participant (or actor) their own baseline amplitude.
- A **random slope** lets an effect differ across participants (or actors). For example, `(1 + PresentNum | ActorID)` lets amplitude elicited by some actors/faces habituate faster than others.

You can only fit a random slope for a predictor that varies *within* that grouping variable. Emotion and presentation number vary within both children and actors, so either can get slopes. Age can't have a slope by participant, because each child only has one age, but it can have one by actor, because each actor is seen by kids of many ages.

## Step 6: Consider transformations and center predictors (optional)

**Presentation number.** You can count presentations of each unique stimulus (as in Heise et al., 2022) or count every trial in the session (i.e., as an index of time spent on task). Either way, we number trials based on trials that were *presented*, not trials that survived artifact rejection. In general, rejected trials were still seen by participants (we assume, unless we've coded attention in some way). Breaks can cause recovery, and habituation may level off, so plot your data and compare linear, log, and quadratic terms.

**Centering** subtracts a constant from a predictor. It changes where zero is (and therefore what the intercept means) but not the units. Always center on your final dataset.

Two options for centering are:
- *Grand-mean centering* often used on participant-level predictors, so coefficients describe the average participant.
- *Center within cluster* often used for trial-level predictors, so coefficients describe trials that are higher or lower than that participant's usual. For something like reaction time, which varies both within and between people, you can include both versions to separate the two effects (Volpert-Esmond et al., 2021).

**Scaling** (z-scoring) also changes the units to SDs. If you scale the outcome and the continuous predictors, you get standardized betas, which work as an effect size (Ferron et al., 2008; Snijders & Bosker, 2012).

**Coding categorical predictors.** R's default is dummy coding: the intercept is the reference group and β is the difference from it. With effects coding (±0.5), the intercept is the grand mean and β is still the full difference between groups. With ±1, β is *half* the difference. If your model has interactions, coding also changes what the interactions mean, so set your contrasts before you fit the final model. If you'd rather not deal with any of this, `emmeans` gives you the same marginal means no matter how you coded.

## Step 7: Fit the maximal model and simplify until it converges

Fit the maximal model for your design. Switching the optimizer and allowing more iterations helps it converge:

```r
model <- lmer(MeanAmp ~ Emotion * Age_gmc + PresentNum +
                (1 + Emotion * PresentNum | SubjectID) + (1 + Emotion * PresentNum | ActorID),
              data = dat,
              control = lmerControl(optimizer = "bobyqa", optCtrl = list(maxfun = 20000)))
```

Two warnings tell you the model is more complex than your data can support:
- **Non-convergence** (`Model failed to converge with max|grad| = ...`): the optimizer couldn't confirm it found the best estimates. To first troubleshoot, you can try rescaling predictors, using a different optimizer, or `allFit()` to compare optimizers.
- **Singular fit** (`boundary (singular) fit`): part of the random-effects structure hit an impossible value, a variance of 0 or a correlation of ±1.

These usually happen with small samples, few trials per participant or condition, lots of random slopes, or when participants (or stimuli) don't differ much compared with the trial-to-trial noise. To reach an interpretable model, we will simplify the random effects one step at a time:

1. Remove interaction slopes first. They're often the least theoretically plausible and need the most data to converge.
2. Remove main-effect slopes that aren't part of your question next (here, presentation number before emotion).
3. Leave one out at a time. Remove the actor slope and refit. If it's still singular, put it back and remove the participant slope. If neither works, remove both.
4. With only a few stimuli (5 actors vs. 40 kids), try removing the stimulus slopes before the participant slopes.
5. Remove the slopes for your effect of interest last. A by-participant emotion slope keeps the test of emotion from being too liberal (Barr et al., 2013).

The bare minimum random effects structure is a random intercept for participants. Without it, you're not modeling the nested structure of your data anymore.

## Step 8: Check model assumptions

| Assumption | What to look at | Things to try if it's violated |
|---|---|---|
| Linearity | Residuals vs. fitted values and vs. each continuous predictor | Add nonlinear terms (e.g., log or quadratic trial number) |
| Equal variance | Spread of residuals vs. fitted values, and by condition or group | Check for outliers; model the variance (e.g., a location scale model) |
| Normal residuals | Q–Q plot | Fixed effects are fairly robust (Schielzeth et al., 2020); check outlying trials; transform skewed outcomes |
| Normal random effects | Q–Q plot for each grouping factor | Look for outlying participants or stimuli |
| Independence | Residuals vs. trial order within participant | Add trial number or block; revisit the random effects |
| Influential points | Cook's distance; participants with extreme random effects | Refit without them and report both |
| Collinearity | Variance inflation factors (VIF < 5) | Center predictors, especially for interactions; drop redundant predictors |

The easiest way to do all of this is `check_model()` from the **performance** package (Lüdecke et al., 2021). You also need **see** installed to get the plots. Each check also has its own function (`check_convergence()`, `check_singularity()`, `check_collinearity()`, `check_outliers()`, `icc()`, ...). With thousands of trials (>4,000 - 5,000), formal tests like Shapiro-Wilk will be statistically significant even for small deviations.

```r
library(performance)
library(see)
check_model(fit_final)
```

## Step 9: Report and visualize results

**Plot** I prefer to plot the model's estimates with raw data behind them, e.g., estimated marginal means ± 1 SE with each child's single trials or averages in the background. Grand-average ERPs are familiar and easy to read, but they hide the trial-to-trial variability your model is actually fit to. Showing single trials from a representative participant is nice to pair with averaged waveforms in a figure (or in supplemental materials).

**Report:**
- A table of fixed effects (b in µV, SE, 95% CI, t, df, p, and standardized β if you're using it as an effect size), the random-effect variances, and the residual variance
- How you calculated df, and any correction for multiple comparisons
- Your maximal model and each simplification step that got you to the final model
- Your assumption checks
- Your exclusion criteria and how many trials and participants you included

**Describe the model** in words, in R (Wilkinson) notation, or with Raudenbush and Bryk level 1/level 2 equations. 

**Report what the next study needs.** Think about what you'd want if you were planning a power analysis from someone else's paper: every fixed effect including nuisance terms (like the habituation slope), every random-effect SD and correlation, the residual SD, how many trials were presented and kept (by participant, condition, and group), rejection rates and what predicted missingness, design counts (participants, stimuli, repetitions), data quality, and how you got from the maximal model to the final one.

---

### References

- Barr, D. J., Levy, R., Scheepers, C., & Tily, H. J. (2013). Random effects structure for confirmatory hypothesis testing: Keep it maximal. *Journal of Memory and Language, 68*(3), 255–278.
- Bates, D., Kliegl, R., Vasishth, S., & Baayen, H. (2015). Parsimonious mixed models. *arXiv:1506.04967*.
- Ferron, J. M., Hogarty, K. Y., Dedrick, R. F., Hess, M. R., Niles, J. D., & Kromrey, J. D. (2008). Reporting results from multilevel analyses. In A. A. O'Connell & D. B. McCoach (Eds.), *Multilevel modeling of educational data* (pp. 391–426). Information Age.
- Heise, M. J., Mon, S. K., & Bowman, L. C. (2022). Utility of linear mixed effects models for event-related potential research with infants and children. *Developmental Cognitive Neuroscience, 54*, 101070.
- Lüdecke, D., Ben-Shachar, M. S., Patil, I., Waggoner, P., & Makowski, D. (2021). performance: An R package for assessment, comparison and testing of statistical models. *Journal of Open Source Software, 6*(60), 3139.
- Matuschek, H., Kliegl, R., Vasishth, S., Baayen, H., & Bates, D. (2017). Balancing Type I error and power in linear mixed models. *Journal of Memory and Language, 94*, 305–315.
- Schielzeth, H., Dingemanse, N. J., Nakagawa, S., Westneat, D. F., Allegue, H., Teplitsky, C., Réale, D., Dochtermann, N. A., Garamszegi, L. Z., & Araya-Ajoy, Y. G. (2020). Robustness of linear mixed-effects models to violations of distributional assumptions. *Methods in Ecology and Evolution, 11*(9), 1141–1152.
- Snijders, T. A. B., & Bosker, R. J. (2012). *Multilevel analysis: An introduction to basic and advanced multilevel modeling* (2nd ed.). Sage.
- Volpert-Esmond, H. I., Page-Gould, E., & Bartholow, B. D. (2021). Using multilevel models for the analysis of event-related potentials. *International Journal of Psychophysiology, 162*, 145–156.
