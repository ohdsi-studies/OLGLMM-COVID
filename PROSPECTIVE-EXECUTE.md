Instructions To Run Study For Any Dates
===================
- This study requires running steps to 1) pick wave dates, 2) download data, 3) run analysis and 4) submitting results to https://pda-ota.pdamethods.org/ 
- After step 4 is finished, results will be generated and each site will be sent their own site ranking.  A manuscript will be written.


## Step 0a - Register (skip this if you already have an account)
- register an account at https://pda-ota.pdamethods.org/

## Step 0b - Sign Up for the wave studies
- Get the `<project name>` and `inviation code` per wave from the study lead.  This will then enable you to download the control json for the wave of interest. 

- Create a directory where you want to save the results to.  We will refer to this directory as `outputFolder`.  Create a folder named `<project name>` in the `outputFolder`.

- For each wave, please place the corresponding control.json file into the `outputFolder/<project name>`.  For example, `outputFolder/omicron_1`, `outputFolder/omicron_2` and `outputFolder/omicron_3`.


## Step 1 - Download the data for each wave of interest

Update the stdy package and pda
```r
remotes::install_github('penncil/pda')
remotes::install_github('ohdsi-studies/OLGLMM-COVID')
```

Now run the following code to extract summary data for each wave (but use your dates):

```r

library(olglmmCovid)
# USER INPUTS
#=======================

siteId <- 'an id given to you by the study lead'

# The folder where the study intermediate and result files will be written:
outputFolder <- "your dirctory to save results" # e.g., "C:/OLGLMMResults"

# Details for connecting to the server:
dbms <- "you dbms"
user <- 'your username'
pw <- 'your password'
server <- 'your server'
port <- 'your port'

connectionDetails <- DatabaseConnector::createConnectionDetails(dbms = dbms,
                                                                server = server,
                                                                user = user,
                                                                password = pw,
                                                                port = port)

# Add the database containing the OMOP CDM data
cdmDatabaseSchema <- 'cdm database schema'

# Add a database with read/write access as this is where the cohorts will be generated
cohortDatabaseSchema <- 'work database schema'

tempEmulationSchema <- NULL

# table name where the cohorts will be generated
cohortTable <- 'olglmm_covid'

olglmmCovid:::executeProspective(
  databaseDetails = databaseDetails,
  siteId = siteId,
  outputFolder = outputFolder,
  cohortTable = cohortTable,
  createCohorts = T,
  createData = T,
  createPlot = F,
  run = F,
  verbosity = "INFO",
  dates = list(
    omicron_1 = list(startDate = '20211101', endDate = '20220228'),
    omicron_2 = list(startDate = '20220301', endDate = '20220630'),
    omicron_3 = list(startDate = '20220701', endDate = '20221101')
  )
    )
```

After running the code you will see a file called `data.csv` in the directories `outputFolder/<project name>` where in this example we have three project names, omicron_1/omicron_2/omicron_3.  Therefore there will be data.csv files in `outputFolder/omicron_1`, `outputFolder/omicron_2` and `outputFolder/omicron_3`.


## Step 3 - Run Analysis
- Make sure each `<project name>` folder contains the data.csv and the control.json.

- run the following code to create the json summary for each wave.  After running the code you will see a new file in `outputFolder/omicron_1`, `outputFolder/omicron_2` and `outputFolder/omicron_3` folders called `<siteId>_initialize.json`.

```r
library(olglmmCovid)
# USER INPUTS
#=======================

siteId <- 'an id given to you by the study lead'

# The folder where the study intermediate and result files will be written:
outputFolder <- "your dirctory to save results" # e.g., "C:/OLGLMMResults"

# Details for connecting to the server:
dbms <- "you dbms"
user <- 'your username'
pw <- 'your password'
server <- 'your server'
port <- 'your port'

connectionDetails <- DatabaseConnector::createConnectionDetails(dbms = dbms,
                                                                server = server,
                                                                user = user,
                                                                password = pw,
                                                                port = port)

# Add the database containing the OMOP CDM data
cdmDatabaseSchema <- 'cdm database schema'

# Add a database with read/write access as this is where the cohorts will be generated
cohortDatabaseSchema <- 'work database schema'

tempEmulationSchema <- NULL

# table name where the cohorts will be generated
cohortTable <- 'olglmm_covid'

databaseDetails <- PatientLevelPrediction::createDatabaseDetails(
  connectionDetails = connectionDetails, 
  cdmDatabaseSchema = cdmDatabaseSchema, 
  cdmDatabaseName = cdmDatabaseSchema,
  tempEmulationSchema = tempEmulationSchema,
  cohortDatabaseSchema = cohortDatabaseSchema,
  cohortTable = cohortTable,
  outcomeDatabaseSchema = cohortDatabaseSchema,
  outcomeTable = cohortTable,
  cdmVersion = 5
)

arrange_all <- dplyr::arrange_all
as_tibble <- dplyr::as_tibble
summarise <- dplyr::summarise
group_by_at <- dplyr::group_by_at
n <- dplyr::n
left_join <- dplyr::left_join
row.match <- prodlim::row.match

olglmmCovid:::executeProspective(
  databaseDetails = databaseDetails,
  siteId = siteId,
  outputFolder = outputFolder,
  createCohorts = F, 
  cohortTable = cohortTable,
  createData = F, 
  createPlot = F, 
  run = T,
  verbosity = "INFO"
        )
```
## Step 4 - Submit Results
- You now need to inspect the `<siteId>_initialize.json` and if happy log into https://pda-ota.pdamethods.org/  and upload the file to the correct study (omicron_1/omicron_2/omicron_3).
- Your part is now done.  Sit back and wait for the study lead to send you the results.
