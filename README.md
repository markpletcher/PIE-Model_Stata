# PIE-Model_Stata
The Prevention Impact and Efficiency (PIE) Model is a simple microsimulation method and implementation using Stata for analyzing the impact and efficiency of an intervention designed to prevent disease from occurring in a population.  Impact is measured in the relative % reduction in disease events that occur (usually over a defined time period), and efficiency is measured in the number needed to treat (NNT) with the intervention (lower NNT is more efficient) to prevent each disease event from occuring.  The basic method entails:
* Finding of a dataset that is representative of your population of interest, and that has all the measurements you need
* Estimating baseline disease risk for each individual in your dataset using measurements in the dataset and some externally derived risk prediction algorithm
* Estimating the reduction in risk for each individual in your dataset that might be achieved from some intervention
* Applying the reduction in risk to participants who meet some targeting criterion
* Estimating overall population average risk before and after treatment to estimate relative % reduction in disease (impact)
* Estimating the absolute risk reduction (average and minimum) and inverting it to get the NNT (average and maximum)

Here we provide a set of template files for implementing this method in Stata, and step-by-step instructions 


describe a step-by-step approach to modeling prevention impact and efficiency, as well as “template” Stata do files that you can download and modify for your own purposes.  The template files implement a simple analysis of the National Health and Nutrition Examination Survey (NHANES) to estimate prevention impact and efficiency of statin prescribing for primary prevention of cardiovascular disease, comparing risk-based targeting to age-based targeting to illustrate use of the model.  As noted in the model description, these templates are easily adapted for use with a different dataset (other than NHANES), different outcome of interest (other than atherosclerotic cardiovascular disease), different intervention (other than statins), and different targeting methods (other than use of risk-based or age-based targeting thresholds).
Steps 0, 1 and 2 will all require substantial customization tailored to your research question.  These steps will create a standard dataset, however, analysis of which is then quite simple.  Though the code provided for Steps 3 and 4 and the Table and Figure files is complex, the files use the standard dataset and require only simple configuration to produce output analyses that are generally useful - this is the added value of the PIE Model.

# Step 0: Choose your dataset and gather your variables
Choose a dataset that is representative of your population of interest, and includes the measurements you need to estimate baseline risk, target your intervention, and estimate post-intervention risk.  We recommend using NHANES, which is representative of the US population and includes many useful measurements, but any such dataset could easily be used.  Gather the variables you need from the dataset and merge them together into a single file with one observation/row per participant.  Step0.do provides an example using NHANES and gathers variables required for a simple statin analysis.  
0a: Housekeeping
Step0.do includes some housekeeping commands that you can modify if you like.  You’ll at least need to modify or delete the cd command.
