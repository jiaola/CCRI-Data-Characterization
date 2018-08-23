# Overview

The R code in this repository contain functions which generate summary reports 
for PCORI CDM tables.

# Prerequisites 

* r-base (>=3.4.2) or RStudio
* The following R packages:
    * dplyr, dbplyr **(version 1.2)**, glue, stringr, readr, tidyr, tidyselect, purrr for data wrangling
    * DT, htmltools, htmlwidgets for data visualization
    * rJava, RJDBC, and getPass for connecting to the database
* Appropriate SQL driver stored locally
    * for Oracle JDBC connections, ojdbc7.jar or ojdbc8.jar
    * for MSSQL JDBC connections, jtds-1.3.1.jar

# Usage

* 00-config-{oracle, mssql}.R set up the connection to your database.
* 01-functions.R contains all functions required to generate summary reports.
* 02-unit-tests-{oracle, mssql}.R run a prespecified list of data validation tests located in `inst/unit_tests.csv`. These tests are replications of selected required checks in the PCORnet data characterization SAS file.
* 03-execute-{oracle, mssql}.R loop over lists of tables/fields to run data characterization.

## 1 Set up connection information ##

Edit the config file 00-config-oracle.R or 00-config-mssql.R depending on your 
RDBMS. You must assign either 3.1 or 4.1 to the version object so the scripts will know
which CDM version to run against. **For Oracle systems, also please be sure to declare the schema name, otherwise the scripts will fail.**

## 2 Schema config ##

Provide your execute file with a list of tables to analyze:

```r
# Declare list of tables to characterize
table_list <- c("CONDITION", "DEATH", "DEATH_CAUSE", "DEMOGRAPHIC", "DIAGNOSIS", 
                "DISPENSING", "ENCOUNTER", "ENROLLMENT", "PCORNET_TRIAL",
                "PRESCRIBING", "PROCEDURES", "PRO_CM", "VITAL")
```

Running the data characterization scripts on a LAB_RESULT table could run into resource limitations. Therefore, it is advised to run data characterization on-demand for a set of lab results. To run an on-demand summary for a given LOINC code, add the following to the end of the execute file for your RDBMS:

```r
# Run an on-demand DC for a LOINC code in the LAB_RESULT_CM table.
generate_summary(conn, backend = [either "Oracle" or "MSSQL"], schema = [required if backend is Oracle], table = "LAB_RESULT_CM", filtered = TRUE, field = "LAB_LOINC", value = ["LOINC code of choice"])
```

## 3 Execute! ##

To run data validation tests, open an R session and issue `source('02-unit-tests-oracle.R')` if your RDBMS is Oracle or `source('02-unit-tests-mssql.R')` if your RDBMS is SQL server. Results are saved to the subdirectory `/unit_tests/` in a datestamped CSV file.

To generate the data characterization summary for each table, open an R session and issue `source('03-execute-oracle.R')` or `source('03-execute-mssql.R')` according to your RDBMS. You will be prompted for your db password. Your queries will start after successful authentication. Completed reports are saved to the subdirectories `summaries/{CSV, HTML}` and are created during execution. 

# Known Issues / Caveats

* Please submit an issue or email pmo14@pitt.edu with any bug reports.

