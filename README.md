# The Prevention Impact and Efficiency (PIE) Model

## Step-By-Step Instructions

The Prevention Impact and Efficiency (PIE) Model is a simple microsimulation modeling method and implementation, programmed in Stata, for analyzing the impact and efficiency of an intervention designed to prevent disease from occurring in a population.  **Impact** is expressed as the relative % reduction in disease events that occur (usually over a defined time period), and **Efficiency** is expressed as the number needed to treat (NNT) with the intervention to prevent each disease event from occuring (lower NNT is more efficient).  The basic method entails:
* Finding of a dataset that is representative of your population of interest, and that has all the measurements you need
* Estimating baseline disease risk for each individual in your dataset using measurements in the dataset and some externally derived risk prediction algorithm
* Estimating the reduction in risk for each individual in your dataset that might be achieved from some specific intervention
* Applying the reduction in risk to participants who meet some targeting criterion
* Estimating overall population average risk before and after treatment to estimate relative % reduction in disease (impact)
* Estimating the NNT by calculating average absolute risk reduction for each patient and taking the inverse

Here we provide a set of template files for implementing this method in Stata (see PIEModelTemplate_v1.0.zip), and step-by-step instructions for how to use the template.  The template files implement a simple analysis of the National Health and Nutrition Examination Survey (NHANES) to estimate prevention impact and efficiency of statin prescribing for prevention of atherosclerotic cardiovascular disease (ASCVD) events, comparing risk-based targeting to age-based targeting to illustrate use of the model.  As noted in the model description, these templates are easily adapted for use with a different dataset (other than NHANES), different outcome of interest (other than atherosclerotic cardiovascular disease), different intervention (other than statins), and different targeting methods (other than use of risk-based or age-based targeting thresholds).

Steps 0, 1 and 2 will all require substantial customization tailored to your research question.  These steps will create a standard dataset, however, analysis of which is then quite simple.  The code provided for Steps 3-5 and for the standard Tables and Figures should require only simple configuration to produce useful output - this is the added value of the PIE Model.

## Step 0: Choose your dataset and gather your variables
Choose a dataset that is representative of your population of interest, and includes the measurements you need to estimate baseline risk, target your intervention, and estimate post-intervention risk.  We recommend using NHANES, which is representative of the US population and includes many useful measurements, but any such dataset could easily be used.  Gather the variables you need from the dataset and merge them together into a single file with one observation/row per participant.  Step0.do provides an example using NHANES and gathers variables required for a simple statin analysis.

**0a: Housekeeping**

Step0.do includes some housekeeping commands that you can modify if you like.  You’ll at least need to modify or delete the cd command.

**0b: Gather variables**

Gather whatever variables you will require for risk estimation, intervention targeting, and risk reduction calculations into a single Stata .dta dataset.  You may want to include variables that can help with accuracy of multiple imputation (see Step 1, below), even if you don’t directly use these variables for required calculations.  For datasets with multiple observation rows per participant (e.g., prescription medications), you’ll need to reduce to a single row such that the resulting dataset has only 1 observation per participant.  

The provided Template gathers medication names (many:1 relationship with participant) and reduces these to indicator variables for statin use and blood pressure medication use (1:1 with participant), self-reported medical conditions and habits (e.g., smoking), blood test (e.g., LDL) and examination (e.g., blood pressure) results, and demographics.

**0c: Include sample design variables and rename**

Find and include any sampling design variables provided with your dataset.  The Template includes the ID variable (SEQN), the PSU variable (SDMVPSU) and the Stratum variable (SDMVSTRA) from NHANES.  Rename these variables (sampleid), (samplepsu) and (samplestrata) for use in bootstrap resampling analyses.

**0d: Decide which weight variable to use**

Find and include a sampling weight variable.  For NHANES, this will depend on the actual measurements you are using: you may want to use the Interview Weight, Exam Weight, or a Subsample Weight like the fasting laboratory subsample.  See NHANES documentation.  For the Statin analysis in our Template, we used the Fasting Subsample 2 Year MEC Weight (WTSAF2YR).  Rename this variable (samplewt) and include this variable in the dataset.  

Note: if your variable does not include weights, you can create a variable called samplewt and set it equal to 1 (constant).  Be careful, however, that your dataset is truly representative of your population of interest; lack of a weight variable implies a simple random sample of that population.

**0e: Name the dataset**

Name your dataset “Dataset0_NHANESVariables.dta”, and save it in your working directory.  You can erase the temporary datasets created along the way, as we do in the Template.

## Step 1: Clean and rename variables, and impute missing values
Step 1.do cleans, renames and then imputes missing values of key variables using multiple imputation.  The multiple imputation commands get complex and will require modification for your analysis, but they are optional.  Also note that multiple imputation may not be appropriate for all missing data problems.  Use this code at your own risk…

**1a: Housekeeping**

Step1.do includes some housekeeping commands that you can modify if you like.  You’ll at least need to modify or delete the cd command.

Open Dataset0_NHANES.dta, which we created in Step 0.

**1b: Drop if the weight variable is missing**

In NHANES, if the weight variable is missing (as you’ve defined it in Step 0d, above), then the participant is not relevant to your analysis.

**1c: Rename and label variables**

Rename and label variables so they are easier to recognize and work with (if you want).  We have done some additional categorization and simplification of the smoking variables in NHANES. 

**1d. Limit to your target population**

Decide on your target population, and drop the observations for participants not included in that target population.  For our simple statin analysis, we were interested only in adults.

**1e. Recode categorical variables**

Look carefully at each categorical variable.  Recode all categorical variables so that missing values are literally missing in Stata (NHANES often uses 7’s and 9’s to indicate unknown values and refusals) so that the imputation algorithm will know to fill them in.   Recode dichotomous variables to 1/0 so the logit part of the imputation algorithm works.  Don’t do any calculations, combine variables or categories, or manipulate the data in any other way at this stage.  Note: even if you are not doing multiple imputation, these data cleaning steps are necessary, so they are included here before Step 1e below.

**1f. Decide if you are going to do multiple imputation**

Multiple imputation is complex, takes additional work, and is not completely explained or implemented in the step-by-step instructions below.  If you do NOT want to use multiple imputation, simply create two simple new variables called original (= 1 for all observations) and index (=_n), save the dataset (Dataset1_NHANESVariablesFinal.dta), and close the log.  You may need to make additional modifications of Template files, particularly for Step 4.  If you DO want to proceed with multiple imputation, continue below.

**1g. Transform non-normally distributed variables that require imputation**

Check continuous variables and transform them if necessary so they are normally distributed.  For our statin analysis, we log-transformed triglycerides, BMI and fasting blood glucose.  Note that the transformed versions of these variables will be imputed directly, and then the original versions (registered as “passive” variables for multiple imputation, see below), will be imputed indirectly by back-transforming the imputed transformed versions.

**1h. Create spline variables for potential key predictors**

Multiple imputation uses the other variables in the dataset to estimate and fill in missing values by iterative multivariable modeling.  These models can be optimized to make imputation more accurate.  While exhaustive model optimization is probably not required, some key continuous variables may be very important for imputation models.  If the associations between these key continuous variables and the imputed variables are non-linear, imputations will be non-optimal.  Making spline variables for key continuous predictors may therefore be useful.  We have chosen to make age into a spline variable for our statin analysis.

**1i. Categorize every variable in the dataset**

We have defined 12 different categories (Types 1-12) for variables.  Once you assign variables to these categories, we can create some basic imputation code automatically (though it may require trouble-shooting).  The categories are:

*REGULAR* variables (Types 1-5) should have no missing values and don’t need to be imputed

* Type 1 – Not useful for prediction (e.g., sampleid)
* Type 2 – Original variable, useful for prediction, continuous
* Type 3 – Original variable, useful for prediction, categorical (dichotomous or 2+ categories)
* Type 4 – Derived variable, useful for prediction, continuous
* Type 5 – Derived variable, useful for prediction, categorical (dichotomous or 2+ categories)

*PASSIVE* variables (Type 6) will be imputed indirectly (via a derived variable, see step 1f)

* Type 6 – Original variable, any type

*IMPUTED* variables (Types 7-12) have missing values that will be imputed directly

* Type 7 – Original, continuous
* Type 8 – Original, dichotomous (2 categories)
* Type 9 – Original, categorical (3+ categories)
* Type 10 – Derived, continuous
* Type 11 – Derived, dichotomous (2 categories)
* Type 12 – Derived, categorical (3+ categories)

For our statin analysis, note that we’ve decided to use age spline variables for prediction (Type 4), such that the original age variable is not useful for prediction (Type 1).

The code provided runs through a series of checks that will catch some categorization errors.

**1j. Register variables for multiple imputation**

If you categorize your variables correctly in the previous step, this one should not need modification.  Check the Stata output to make sure you have no unregistered variables.

**1k. Do multiple imputation**

The code provided should implement standard multiple imputation if you’ve categorized your variables above correctly.  However, you may need to adjust things if one or more of the models don’t fit or assumptions are violated.  Please take these instructions and sample code as simply an example of how this might be performed.  In our statin analysis, standard imputation using regress for hdl and ldl produced some negative values, so we needed to use an alternate method (predictive mean matching) for these variables.  See code example.

**1l. Back-transform and backfill imputed continuous variables**

For the continuous variables that you transformed in step 1f, backfill the original variables with a back-transformed version for values that were missing.  See code example.

**1m. Create index and describe new, imputed dataset**

The provided code need not be altered if you did multiple imputation.  It creates a new index for the imputed dataset, describes the dataset structure, and saves Dataset1_NHANESVariablesFinal.dta.  

## Step 2: Create standard variables for the model
Step2.do uses the imputed dataset created in Step 1, and creates standard variables that we will use in the model.

**2a: Housekeeping**

Step2.do includes some housekeeping commands that you can modify if you like.  You’ll at least need to modify or delete the cd command.

Open Dataset1_NHANESVariablesFinal.dta, which you created in Step 1.

**2c: Estimate baseline risk**

Use your Baseline Risk Algorithm to generate a risk estimate for each person in your dataset.  Name this variable (baselinerisk).  

For the Statin analysis, we use the 10-year Atherosclerotic Cardiovascular Disease (ASCVD) Risk to calculate 10-year risk for persons without pre-existing CVD (from the 2013 ACC/AHA Guideline on the Assessment of Cardiovascular Risk).  For persons with pre-existing CVD, we use the Framingham-based 2-year risk of recurrent coronary heart disease calculation, and recalculate this every 2 years to estimate 10-year risk (see Methods).  An alternate approach would have been to limit the target population to persons without pre-existing CVD, but this approach would not capture the effects of secondary prevention from statins.  For our Statin analysis, this step was complicated by the fact that some persons were already apparently taking a statin.   For these participants, we “de-treated” statin users assuming either “standard” or “high dose” statin effects (depending on which statin was reported) on Total and HDL cholesterol, used the de-treated values to estimate de-treated risk, and re-applied the effect of standard dose statins (as we did for Step 6).  An alternative approach would have been to limit the target population to non-users of statins, but then our impact estimates (relative % reduction in events) would not reflect the US population.

**2d: Estimate post-intervention risk**

Use your Intervention Risk Reduction Algorithm(s) to generate a risk estimate for each person in your dataset after treatment with each proposed intervention, assuming that the treatment is actually applied in that person.  You can program as many different types of interventions as you like.  For “x” number of interventions, name these variables (risk1, risk2…riskx).  Add a “Theoretical 100% efficacy” intervention and name it risk0.  

While any intervention could be targeted using any threshold, some interventions are such that they would not apply to all persons, or might apply to all persons regardless of whether they cross the designated threshold.  For example, standard dose statin therapy will “always” apply to someone with a history of CVD or LDL>190 according to current guidelines, but “never” apply to someone already taking a statin (because they are already on a statin).  For each intervention, therefore, create a new indicator variable for each intervention (apply_i0, apply_i1, apply_i2...apply_ix), set it equal to “if over threshold” as the default, and then modify it to “always” or “never” for particular types of patients, as needed.

It is useful to keep an index of your interventions, and I have added that here, in comment text.

For our simple statin analysis, our interventions were: 1) Standard dose statins, and 2) High dose statins.  We used relative risk reduction estimates per mmol/L reduction in LDL from the Mihaylova CTT meta-analysis (different for primary vs. secondary prevention), and applied average LDL reduction for standard vs. high dose statins of 40% and 50% for persons not currently taking a statin.

**2e: Set treatment thresholds for intervention targeting**

Program your Treatment Threshold(s).  You can program a continuous threshold variable, or an ordinal or dichotomous variable.  Note that the programming is designed to test impact and efficiency of treatment OVER a given threshold (not under), so program your variables accordingly (or tweak the code).  For “x” number of targeting algorithms, name these variables (thresh1, thresh2…threshx).  

It is useful to keep an index of your thresholds, and I have added that here, in comment text.

For our simple statin analysis, we used baseline risk, age, and presence of cardiovascular disease (dichotomous, coded 1/0) as the three thresholds for intervention targeting.

**2f. Round weights and save a version of the dataset with all variables**

Round the weights because Stata can’t deal with unrounded weights, and then save “Dataset2a_StandardVariables.dta”.  This will be useful for Table 1 and other descriptions of the dataset and processes you used.

**2g: Drop observations from original dataset and keep only standard variables**

Drop observations from the original dataset where any of the standard variables are missing.  You should not need to drop any observations from the imputed datasets (where original==0).  Check how many observations were dropped at each stage and make sure you know why.  In our analysis, 337 individuals from the original dataset because they were missing risk factor measurements required for the baseline risk estimation.

Keep only the standard variables (named in this Appendix in bold), and name the dataset “Dataset2b_StandardVariablesOnly.dta”.  You should be able to run the code here without modification.  

## Step 3: Generate results for each intervention-threshold combination

You’ve made it to the “standard” part of the PIE Model, where you’ll reap the benefits of standardizing all the variables in Steps 0-2.  This should be a relief for you! (It is for me).  For subsequent files, our templates include a single easily-modified customization step, and then a large chunk of code that should not require much, if any, modification.  

The purpose of Step3.do is to generate point estimate results for impact and efficiency for each intervention applied at each level of each threshold.

**3a: Housekeeping**

Step3.do includes some housekeeping commands that you can modify if you like.  You’ll at least need to modify or delete the cd command.

**3b. Customize your parameters**

For the program to run, you have to tell it how many interventions you have programmed, and how many thresholds you have programmed, and for what LEVELS of each threshold you want results generated.  Make sure that the minimum value you specify is the minimum value in the dataset.  This will ensure that, when the minimum value is specified as the treatment threshold, this treatment strategy corresponds to “Treat All”.  The .do file will walk you through this.

**3c. Run the do file**

After customizing your parameters, who should not need to modify the do file any further and can simply run it.  Note that it may take a while to run, especially if you want results for a very wide range of scenarios (see additional notes in the .do file). 

## Step 4: Do Monte Carlo simulation to get 95% confidence intervals
The purpose of Step4.do is to generate 95% confidence intervals for impact and efficiency estimates.  This step will run each intervention-threshold combination that you specify 10,000 (1000 times for each of the 10 imputed datasets, using the cluster sample design variables), so it takes a very long time to run.  Be parsimonious about exactly which estimates you want to get 95% confidence intervals for.  You can run and save multiple versions of this file and date them so that you don’t have to re-run estimates that you’ve already made when you want to add more for any reason.

For our template files, we have to run the file twice to get 2 different sets of results.  We’ll combine and analyze these results later.

If you have to run multiple Monte Carlo simulations as I did, there is a danger of overwriting your results (that may take many hours to generate).  We’ve included an imperfect safeguard against this, which will automatically assign the date to the file name.  However, if you run the second simulation on the same day as the first, it will be overwritten.  So you should immediately rename your Dataset4 file once you’ve completed your run, and number it, e.g. Dataset4_MonteCarloResults_1.  You should have a matching Step4.do file, e.g., Step4_1.do.  Follow this file naming convention so that Step 5 will work.

**4a: Housekeeping**

Step4.do includes some housekeeping commands that you can modify if you like.  You’ll at least need to modify or delete the cd command.

**4b. Customize your parameters**

For the program to run, you have to tell it exactly which interventions and thresholds you want to run.

**4c. Run the do file**

After customizing your parameters, who should not need to modify the do file any further and can simply run it.

**4d. Rename your Dataset4 filename**

As described above, rename your Dataset4 filename and put the run number at the end (e.g., “..._2”) so it won’t be overwritten if you need to run another Monte Carlo simulation.

## Step 5: Consolidate Monte Carlo simulation results
The only purpose of Step5.do is to consolidate the Monte Carlo simulation results you obtained in Step 4.  It is very simple as long as you followed the naming convention in Step 4.

**5a: Housekeeping**

Step5.do includes some housekeeping commands that you can modify if you like.  You’ll at least need to modify or delete the cd command.

**5b. Customize your parameters**

Just tell it how many Monte Carlo simulation runs you did (and named).

**5c. Run the do file**

After customizing your parameters, who should not need to modify the do file any further and can simply run it.

**5d. Check and clean up**

You may realize that you’ve actually run a particular result (for a given intervention-threshold pair at a given threshold value) more than once.  We’ve provided a check and sample code for you to fix that problem here.  Run the do file again with your clean up code and make sure there are 10,000 results for each cell in the table of the results check.

## Making Tables and Figures
We have provided templates for a number of Tables and Figures that may be useful.  As with Steps 3, 4 and 5 above, the code is somewhat complex, but these files are all easily configured for your analysis once you’ve completed Steps 0, 1 and 2.  As examples, we provide the files used to create the “Tables and Figures for Simple Statin Analysis.doc” file provided with these Instructions.  We provide notes on each of these templates below.  Each of the do files includes instructions that should be relatively self-explanatory.  

**Table1.do** provides a template for making a Table that describes the target population for your analysis, stratified by a categorical variable that defines columns of the table.  It uses Dataset2a_AllVariables.dta.  Results are saved in a log file.

**Table2.do** provides a template for making a Table that presents point estimates and 95% confidence intervals for a set of results (treatment with an intervention at a given treatment threshold value) that you can define.  It uses Dataset3 and Dataset5.  Results are saved in a log file.

**Figure1.do** provides a template for making a figure that describes the cumulative distribution of any of your targeting threshold variables (e.g., baseline risk and age for the statin analysis).

**Figure2.do** provides a template for making a figure that describes prevention impact (relative % reduction in events) as a function of your threshold for targeting your intervention(s).

**Figure3.do** provides a template for making a figure that describes prevention efficiency (NNT) as a function of your threshold for targeting your intervention(s).

**Figure4.do** provides a template for making a figure that describes and compares Prevention Impact and Efficiency Curves that illustrate the relationship between prevention impact and efficiency for a series of intervention-targeting threshold pairs.

