# Data submission instructions

This page is intended to provide teams with all the information they need to
submit forecasts.
Data in this directory should be added to the repository through a pull request
so that automatic data validation checks are run.

These instructions provide detail about the 
[data format](#Data-formatting) as well as 
[validation](#Data-validation)
that you can do prior to this pull request. 
In addition, we describe 
[meta-data](#Meta-data) that each model should provide.

*Table of Contents*

- [ground truth data](#ground-truth-data)
- [data formatting](#Data-formatting)
- [data validation](#Data-validation)
- [metadata format](#Meta-data)


## Ground truth data

There are several different sources for death data. Currently, 
all forecasts will be compared to the 
[daily reports containing death data from the JHU CSSE group](https://github.com/CSSEGISandData/COVID-19/blob/master/csse_covid_19_data/csse_covid_19_time_series/time_series_covid19_deaths_US.csv) 
as the gold standard reference data for deaths in the US. 
Note that there are significant differences 
(especially in daily incident death data) 
between the JHU data and another commonly used source, from the New York Times. 
The team at UTexas-Austin is tracking this issue on [a separate GitHub repository](https://github.com/spencerwoody/covid19-data-comparsion).

We may add additional sources of ground-truth data at a future time.


## Data formatting

The automatic check validates both the filename and file contents to ensure the 
file can be used in the visualization and ensemble forecasting.


### Subdirectory

Each subdirectory within this data-processed/ directory has the format

    team-model
    
where 

- `team` is the teamname and 
- `model` is the name of your model. 


### Filenames

Each file within the subdirectory should have the following format

    YYYY-MM-DD-team-model.csv
    
where

- `YYYY` is the 4 digit year, 
- `MM` is the 2 digit month,
- `DD` is the 2 digit day,
- `team` is the teamname, and
- `model` is the name of your model. 

The date YYYY-MM-DD is the date the forecasts were made from your mdoel, 
i.e. the most recent data is YYYY-MM-DD.

The `team` and `model` in this file must match the `team` and `model` in the
directory this file is in.


### File format

The file must be a comma-separated value (csv) file with the following 
columns (in any order):

- `forecast_date`
- `target`
- `target_end_date`
- `location`
- `type`
- `quantile`
- `value`

Additional columns are allowed, but ignored. 

Each row in the file is either a point or quantile forecast for a location on a 
particular date for a particular target. 


#### `forecast_date`

Values in the `forecast_date` column must be a date in the format

    YYYY-MM-DD
    
This is the date on which the submitted forecast data was made available in `YYYY-MM-DD` format. In general, this will typically be the date on which the model finishes running and produces the standard formatted file. `forecast_date` should correspond and be redundant with the date in the filename, but is included here by request from some analysts. We will enforce that the forecast_date for a file must be either the date on which the file was submitted to the repository or the previous day. Exceptions will be made for legitimate extenuating circumstances.


#### `target`

Values in the `target` column must be a character (string) and be one of the 
following specific targets:

- "N day ahead cum death" where N is a number between 0 and 130
- "N day ahead inc death" where N is a number between 0 and 130
- "N wk ahead cum death"  where N is a number between 1 and  20
- "N wk ahead inc death"  where N is a number between 1 and  20
- "N day ahead inc hosp"  where N is a number between 0 and 130

For week-ahead forecasts, we will use the specification of epidemiological weeks (EWs) [defined by the US CDC](https://wwwn.cdc.gov/nndss/document/MMWR_Week_overview.pdf). 
There are standard software packages to convert from dates to epidemic weeks and vice versa. E.g. [MMWRweek](https://cran.r-project.org/web/packages/MMWRweek/) for R and [pymmwr](https://pypi.org/project/pymmwr/) and [epiweeks](https://pypi.org/project/epiweeks/) for python.

We have created [a csv file](../template/covid19-death-forecast-dates.csv) describing forecast collection dates and dates for which forecasts refer to can be found.

##### N day ahead cum death

This target is the cumulative number of deaths predicted by the model for 
N days after `forecast_date`. 

As an example, for day-ahead forecasts with a `forecast_date` of a Monday, a 1 day ahead cum death forecast corresponds to cumulative deaths by the end of Tuesday, 2 day ahead to Wednesday, etc.... 

##### N day ahead inc death

This target is the incident (daily) number of deaths predicted by the model
on day N after `forecast_date`. 

As an example, for day-ahead forecasts with a `forecast_date` of a Monday, a 1 day ahead cum death forecast corresponds to incident deaths on Tuesday, 2 day ahead to Wednesday, etc.... 


##### N wk ahead cum death

This target is the cumulative number of deaths predicted by the model up to 
and including N weeks after `forecast_date`. 

For week-ahead forecasts with `forecast_date` of Sunday or Monday of EW12, a 1 week ahead forecast corresponds to EW12 and should have `target_end_date` of the Saturday of EW12. For week-ahead forecasts with `forecast_date` of Tuesday through Saturday of EW12, a 1 week ahead forecast corresponds to EW13 and should have `target_end_date` of the Saturday of EW13. 

A week-ahead forecast should represent the cumulative number of deaths reported on the Saturday of a given epiweek.


##### N wk ahead inc death

This target is the incident (weekly) number of deaths predicted by the model 
during the week that is N weeks after `forecast_date`. 

For week-ahead forecasts with `forecast_date` of Sunday or Monday of EW12, a 1 week ahead forecast corresponds to EW12 and should have `target_end_date` of the Saturday of EW12. For week-ahead forecasts with `forecast_date` of Tuesday through Saturday of EW12, a 1 week ahead forecast corresponds to EW13 and should have `target_end_date` of the Saturday of EW13. 

A week-ahead forecast should represent the total number of incident deaths within a given epiweek (from Sunday through Saturday, inclusive).


##### N day ahead inc hosp

This target is the incident (weekly) number of deaths predicted by the model
on day # after `forecast_date`.

As an example, for day-ahead forecasts with a `forecast_date` of a Monday, a 1 day ahead inc hosp forecast corresponds to the number of incident hospitalizations on Tuesday, 2 day ahead to Wednesday, etc.... 


#### `target_end_date`

Values in the `target_end_date` column must be a date in the format

    YYYY-MM-DD
    
This is the date for the forecast `target`. 
For "# day" targets, `target_end_date` will be # days after `forecast_date`.
For "# wk" targets, `target_end_date` will be the Saturday at the end of the 
week time period.


#### `location`

Values in the `location` column must be

- "US" or
- a two-digit number representing the US state, territory, or district 
[fips numeric code](../template/state_fips_codes.csv). 

This location identifies the geographical location for the forecast.

A file with FIPS codes for states in the US is available through the `fips_code` dataset in the `tigris` R package, and saved as a [public CSV file](./template/state_fips_codes.csv). Please note that when reading in FIPS codes, they should be read in as characters to preserve any leading zeroes.


#### `type`

Values in the `type` column are either

- "point" or
- "quantile". 

This value indicates whether that row corresponds to a point forecast or a 
quantile forecast. 
Point forecasts are used in visualization while quantile forecasts are used in
visualization and in ensemble construction.


#### `quantile`

Values in the `quantile` column are either missing (if `type` is "point") or 
a quantile in the format

    0.###
    
For quantile forecasts, this value indicates the quantile for the `value`
in this row. 

Teams should provide the following quantiles:


```r
c(0.01, 0.025, seq(0.05, 0.95, by = 0.05), 0.975, 0.99)
```

```
##  [1] 0.010 0.025 0.050 0.100 0.150 0.200 0.250 0.300 0.350 0.400 0.450
## [12] 0.500 0.550 0.600 0.650 0.700 0.750 0.800 0.850 0.900 0.950 0.975
## [23] 0.990
```


#### `value`

Values in the `value` column are numeric indicating the "point" or "quantile"
prediction for this row. 
For a "point" prediction, `value` is simply the value of that point prediction
for the `target` and `location` associated with that row. 
For a "quantile" prediction, `value` is the inverse of the cumulative 
distribution function (CDF)
for the `target`, `location`, and `quantile` associated with that row.

An example inverse CDF is below.

![plot of chunk example_inverse_cdf](./example_inverse_cdf-1.png)



## Data validation

To ensure proper data formatting, 
pull requests for new data in data-processed/ will be automatically run.



### Pull request data validation

When a pull request is submitted, 
the data are validated through [Travis CI](https://travis-ci.org/) which runs
the tests in [test-formatting.py](../code/validation/test-formatting.py).
The intent for these tests are to validate the requirements above and 
specifically enumerated [on the wiki](https://github.com/reichlab/covid19-forecast-hub/wiki/Validation-Checks).
Please [let us know](https://github.com/reichlab/covid19-forecast-hub/issues) 
if the wiki is inaccurate.

If the pull request fails, please 
[follow these instructions](https://github.com/reichlab/covid19-forecast-hub/wiki/Troubleshooting-Pull-Requests)
for details on how to troubleshoot.

For those familiar with python, 
you can run the tests in 
[test-formatting.py](../code/validation/test-formatting.py) 
prior to submitting a pull request to ensure the data will validate. 


### R data validation

For those familiar with R (but not python),
there is a separate set of tests that may be useful to diagnose data 
formatting issues in
[functions_plausibility.R](../code/validation/functions_plausibility.R).
We intend to keep these tests in sync with the python checks 
automatically run during a pull request.
If you discover any discrepancies,
please [let us know](https://github.com/reichlab/covid19-forecast-hub/issues).

As an example of using these test from the base of the repository run


```r
source("code/validation/functions_plausibility.R")
```

To check a single file, run


```r
validate_file("data-processed/UMass-MechBayes/2020-04-26-UMass-MechBayes.csv")
```

To check a directory, run


```r
validate_directory("data-processed/UMass-MechBayes/")
```

Any "ERROR"s will result in a failed pull request. 
"Warning"s and "Message"s are informational, but may help prevent unwanted 
or incomplete forecasts from getting pushed to the repository.

In addition to purely technical sanity checks, three plausibility checks 
are performed:

- avoid quantile crossing: quantiles should be non-decreasing, e.g. it 
does not make sense to have a median of 500, but a 75% quantile of 400 in
the same forecast.
- avoid temporal inconsistencies: quantiles of *cumulative* forecasts should
be non-decreasing over time. It does not make sense to predict a median
of 500 one week ahead and a median of 400 two weeks ahead (for the same
cumulative target and if both forecasts are issued at the same time).
- avoid inconsistencies between incidence and cumulative forecasts:
quantiles of cumulative deaths should not be below those of incident 
deaths for the same forecast horizon (as incident deaths are a subset of 
cumulative deaths).

If inconsistencies are found here, the list returned by `validate_file`
contains a table pointing you to the respective parts of your file which
caused the problem.


## Meta-data

Participating teams must provide a metadata file (see [example](../data-processed/UMass-ExpertCrowd/metadata-UMass-ExpertCrowd.txt)), including methodological detail about their approach and a link to a file 
(or a file itself) describing the methods used. 