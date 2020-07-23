
# BEEPmbp

| Branch        | Test status   |
| ------------- | ------------- |
| master        | [![](https://github.com/ScottishCovidResponse/CoronaPMCMC/workflows/CI/badge.svg?branch=master)](https://github.com/ScottishCovidResponse/CoronaPMCMC/actions?query=workflow%3ACI) |
| dev           | [![](https://github.com/ScottishCovidResponse/CoronaPMCMC/workflows/CI/badge.svg?branch=dev)](https://github.com/ScottishCovidResponse/CoronaPMCMC/actions?query=workflow%3ACI) |
| chrispooley   | [![](https://github.com/ScottishCovidResponse/CoronaPMCMC/workflows/CI/badge.svg?branch=chrispooley)](https://github.com/ScottishCovidResponse/CoronaPMCMC/actions?query=workflow%3ACI) |

C. M. Pooley† [1] and Glenn Marion [1]

[1] Biomathematics and Statistics Scotland, James Clerk Maxwell Building, The King's Buildings, Peter Guthrie Tait Road, Edinburgh, EH9 3FD, UK 

† Corresponding author

Email: [chris.pooley@bioss.ac.uk](mailto:chris.pooley@bioss.ac.uk)

BEEPmbp (Bayesian Estimation of Epidemic Parameters using Model Based Proposals) is a code for analysing coronavirus using regional level data. This analysis is performed by dividing the area under study (e.g. Scotland or the UK) into small geographical groupings, e.g. at medium super output area (MSOA) level or output area (OA) level, and modelling the spread of disease. The model captures short range and long range disease transmission by making use of census flow data and previously published age mixing matrices. The data to be analysed is weekly hospitalisations and deaths for Covid-19 patients at a health board level along with national demographic data. The time-varying disease transmission rate and infection rate from abroad are estimated, along with the effects of covariates (e.g. age, sex, and population density) on disease progression. 

Parameter inference is performed using a multi-temperature model-based proposal MCMC (MBP-MCMC) approach. This runs MCMC chains at different "temperatures" spanning from the posterior to the prior. This enables the model evidence to be estimated allowing for reliable comparison between different potential models. 

## Performing analysis

Compilation: make

Simulation:  ./beepmbp inputfile="examples/sim.toml" 

Inference:   mpirun -n 2 ./beepmbp inputfile="examples/inf.toml" nchain=2
(nchain and -n must be the same; they are the number of chains run, and hence the number of processes used)

The input TOML file provides details of simulation or inference and contains all the information BEEPmbp needs to define the compartmental model and provide the filenames for the data. Examples of these files can be found in the "examples" directory, along with an simple test dataset.
 
# INPUTS:

Here is a description of the various commands used in the TOML files:

DETAILS

**mode** - Defines how the code operates:
		"sim" generates simulated data.
		"inf" performs inference using multi-temperature MBP-MCMC.

**timeformat** - The time format used (either dates or numerical times).

**start** and **end** - The time period for simulation / inference.

**seed** - Sets the random seed when performing inference (this is set to zero by default).

**nchain** - The total number of chains used when performing MCMC (should be a multiple of the number of cores).

**nsamp** - The number of samples used for inference (note, burnin is assumed to be a quarter this value).

**outputdir** - Gives the name of the output directory (optional).

THE MODEL

Note, for examples of how these commands are used, see 'inf.toml'.

**comps** - Defines the compartments in the model.

**trans** - Defines transitions between compartments.

**params** - Defines parameters in the model (simulation only).

**priors** - Defines priors in the model (inference only).

**indmax** - The maximum number of infected individuals (placed as a prior).

**betaspline** - Defines a linear spline used to capture time variation in transmission rate beta.

**phispline** - Defines the linear spline used to represent external force of infection phi.

**ages** - The age groups used in the analysis.

**democats** - Used to define other demographic categories.

**timep** - Used to define discrete time periods (e.g. before and after lockdown).

THE DATA 

**datadir** - The data directory.

**regions** - Filename for a table giving data regions.

**areas** - Filename for a table giving information about areas (e.g. MSOAs or OAs).

**genQ** - Sets whether spatial and age mixing matrices are used to derive the Q tensors for the model.

**geomix** - Filename for a table informing the matrix of contacts between different areas.
 
**agemix** - Filename for a matrix giving the contact rates between different age groups.

**genQoutput** - Filenames for the derived Q tensors.

**Q** - Assigns which Q tensor is applied to which compartment and time period (as defined in 'timep').

**threshold** - Used to define the threshold when data is truncated below this value (represented by '*' in the data files).

**transdata** - Transition data. Gives the observed numbers of transitions between different compartments. 

**popdata** - Population data. Gives the number of individuals in specified compartments at specified time points.

**margdata** - Allows for marginalised distributions to be added (e.g. the percentage of overall cases in different age categories).


Note, all the single variable quantities in the TOML file can be overridden using equivalent command line definitions.

For example: mpirun -n 1 ./beepmbp inputfile="inf.toml" nsamp=10000 

generate 10000 samples (irrespective of the definition given in "inf.toml").
	
# OUTPUTS:

Simulation - This creates the specified 'transdata', 'popdata' and/or 'margdata' files in the output directory.

Inference - The output directory contains posterior information (with means and 90% credible intervals) for:
1) Plots for the transitions corresponding to the 'transdata', 'popdata' and/or 'margdata' files.
2) "Posterior_R0.txt", which gives posterior plots time variation in R0.
3) "Posterior_parameter.txt", which gives information about parameters.
4) "trace.txt", which gives trace plots for different models.
5) "traceLi.txt", which gives trace plots for the likelihoods on different chains.
6) "MCMCdiagnostic.txt", which gives diagnostic information on the MCMC algorithm.

Diagnostic checks - Two types of checks can be performed to ensure that the results obtained are reliable:
1) Estimates for the effective sample size in "Posterior_parameter.txt". These should exceed 200 for all parameters if the number of samples is sufficiently large. If this is not the case it indicates that MCMC should be run with more samples (see the 'nsamp' option in the input TOML file).
2) Results from different runs can be combined to ensure that they all converge on the same posterior distribution (if the likelihood exhibits significant multimodality then under some circumstances different runs can converge on different solutions rendering the results questionable). This is achieved by running BEEPmbp in 'combinetrace' mode. For example, if two sets of inference results using different seeds have been placed into directories 'OutputA' and 'OutputB', the following command:

./beepmbp mode="combinetrace" dirs="OutputA,OutputB" output="parameter_combined.txt"

generates a file combining the two sets of samples along with Gelman–Rubin convergence diagnostic results that test for convergence across runs.

# Development

- [Code documentation](https://projectdata.scrc.uk/coronapmcmc/branches/master/doxygen/html/index.html) -- not yet automatically updated
  - [Call tree](https://projectdata.scrc.uk/coronapmcmc/branches/master/doxygen/html/analysis_8cc.html#a3c04138a5bfe5d72780bb7e82a18e627)
- Currently work on feature branches and then merge into master
- Continuous integration is implemented using Github Actions,
  controlled by a [workflow](.github/workflows/ci.yml) file. Whenever
  a branch is pushed to Github, the workflow is run on that
  branch. The workflow compiles and runs the code in both simulation
  and inference mode. The results of the CI run should be emailed to
  the committer.  They are also visible on the
  [Actions](https://github.com/ScottishCovidResponse/CoronaPMCMC/actions)
  page.
- Regression tests can be run with
  ```
  make test
  ```
  This will report whether the code gives the same results as when the reference data in `tests/*/refdata` was
  committed. The output of the tests will be stored in `regression_test_results`.  If the tests fail because you
  have made a change which *should* change the results, run
  ```
  make test-update
  ```
  which will store the new results from `regression_test_results` into `tests/*/refdata'.  If you ran the tests
  with an uncommitted version of the code, this will fail; you need to commit the code changes and rerun "make test"
  before storing the results. This ensures that the reference data corresponds to a committed version of the code.
  You can then commit the new results with
  ```
  git add tests/*/refdata
  git commit -m "Regenerate test reference data"
  ```
