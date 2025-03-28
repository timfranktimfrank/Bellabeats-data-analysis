# Bellabeat Data Analysis Mar - May 2016

##### By Tim Frank 3.19.2025  

## Background 
Bellabeat is a high-tech manufacturer of health-focused products catered to women.  They have the opportunity to become a larger player in the smart device market.  The Bellabeat team have requested analysis of smart device fitness data which could help unlock new growth opportunities for the company, focusing how consumers are using their smart devices, and will help guide the companie's marketing strategy.

## Scenario
### - Characters
- **Urska Srsen**: cofounder and Chief Creative Officer
- **Sando Mur**: Mathematicians and Bellabeat's cofounder; key member of the Bellabeat executive team
- **Marketing Analytics team**: my team reviewing this data
### - Products
- **Bellabeat app**: app provides health data based on activity, sleep, stress, menstrual cycle, and mindfulness habits.
- **Leaf**: bracelet, necklace, clip-on wellness tracker (activity, sleep, stress)
- **Time**: wellness watch that tracks user activiey (activity, sleep, stress)
- **Spring**: water bottle that tracks daily water intake
- **Bellabeat** Membership: personalized guidance on nutrition, activity, sleep, health and beauty, and mindfullness.

## Business Task
#### Generally, we want to consider the following three questions throughout the analysis process:
- What health and fitness trends can we glean from smart device usage?  
- How can we apply those trends to Bellabeat customers?  
- How can these trends help influence BellaBeats Marketing Strategy?  

## Data Preparation 
#### General Info
For this analysis I will be using a publicly available Kaggle dataset "FitBit Fitness Tracker Data", which was generated by respondents to a distributed survey via Amazon Mechanical Turk from March 12 - May 12, 2016. This survey gathered 29 csvs in long format across 33 participants.  
Datasets: [https://www.kaggle.com/datasets/arashnic/fitbit ](url)
Fitbit Data Dictionary: [https://www.fitabase.com/media/1930/fitabasedatadictionary102320.pdf](url)

#### Focus
Upon review of the 29 files included in the kaggle dataset (https://www.kaggle.com/datasets/arashnic/fitbit/data), I will focus my first analysis on the **daily activity**, **MinuteMETs**, and **MinutesSleep** datasets:
Fitabase Data 3.12.16-4.11.16:
- dailyActivity_merged.csv
- minuteMETsNarrow_merged.csv
- minuteSleep_merged.csv
Fitabase Data 4.12.16-5.12.16
- dailyActivity_merged.csv
- minuteMETsNarrow_merged.csv
- minuteSleep_merged.csv

## How Can This Help BellaBeats?
### Fitness and sleep tracking can be analyzed to identify trends that can be scaled to Bellabeats in-house data for ongoing analaysis to create:
    - Tailored marketing creative for new and existing customers
    - New campaign strategies for new and existing customers
    - Robust audience segments available in data marketplaces for marketers

#### Limitations
- Small sample size of 33 users. Regular participation was optional.
- March - May is a relative short period of time for meaningful insights.
- Separated months may result in duplicate entries.
- There are 29 csvs in long format.
- Data is almost 9 years old. 


## Step 3) Process
### Tools
**Processing and Analysis**: BigQuery and Google Sheets
**Visualization**: Tableau Public
**Documentation**: Github 

### Filename organization
Removing duplicate names: 
Fitabase Data 3.12.16-4.11.16:
- dailyActivity_merged.csv --> **dailyactivity_MARAPR.csv**
- minuteMETsNarrow_merged.csv --> **minuteMETsNarrow_MARAPR.csv**
- minuteSleep_merged.csv _-> **minuteSleep_MARAPR.csv**
Fitabase Data 4.12.16-5.12.16
- dailyActivity_merged.csv --> **dailyActivity_APRMAY.csv**
- minuteMETsNarrow_merged.csv --> **minuteMETsNarrow_APRMAY.csv**
- minuteSleep_merged.csv --> **minuteSleep_APRMAY.csv**

### Checking The Data
**Reviewing Daily Activity**: I first checked dailyactivity_MARAPR.csv and dailyActivity_APRMAY.csv and confirmed uniformity in # of columns, column naming, and upper and lower bounds of each column. Nothing out of the ordinary was found. The minutesMETs and minuteSleep files are too large to open in Sheets. 

**Uploading to BigQuery**
I uploaded all 6 files to BigQuery, my initial findings included:

`For minuteMETs and minuteSleep, date needed to be converted to DATETIME (and much later on, Unix TimeStamp)`

`##for minuteMETsAPRMAY, converting string ActivityMinute to DateTime ActivityMinut_datetime ## 
CREATE OR REPLACE TABLE bellabeats-452721.Fitabase_MARAPRMAY.minuteMETsAPRMAY AS
SELECT 
    *, 
    PARSE_DATETIME('%m/%d/%Y %I:%M:%S %p', ActivityMinute) AS ActivityMinute_datetime
FROM bellabeats-452721.Fitabase_MARAPRMAY.minuteMETsAPRMAY;`

`##repeating step above for minuteMETsMARAPR ##
CREATE OR REPLACE TABLE bellabeats-452721.Fitabase_MARAPRMAY.minuteMETsMARAPR AS
SELECT 
    *, 
    PARSE_DATETIME('%m/%d/%Y %I:%M:%S %p', ActivityMinute) AS ActivityMinute_datetime
FROM bellabeats-452721.Fitabase_MARAPRMAY.minuteMETsMARAPR;`

`##repeating step above for bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_MarAPR ##
CREATE OR REPLACE TABLE bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_MarAPR AS
SELECT 
    *, 
    PARSE_DATETIME('%m/%d/%Y %I:%M:%S %p', date) AS date_datetime
FROM bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_MarAPR;`

`##repeating step above for bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_APRMAY ##
CREATE OR REPLACE TABLE bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_APRMAY AS
SELECT 
    *, 
    PARSE_DATETIME('%m/%d/%Y %I:%M:%S %p', date) AS date_datetime
FROM bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_APRMAY;`
  
**Renaming METS and sleep columns**
As the end goal is to merge files, a few column names needed to be adjusted. 
`CREATE TABLE `bellabeats-452721.Fitabase_MARAPRMAY.minuteMETs_MARAPRMAYdateready` AS
SELECT 
    ID AS METsID,
    ActivityMinute_datetime AS METsdatetime,
    date_only AS METsdate_only,
    time_only AS METstime_only
FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteMETs_MARAPRMAYdate`;`

`CREATE TABLE `bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_MARAPRMAYdateready` AS
SELECT 
    ID AS SleepID,
    value as SleepValue,
    logid as Sleeplogid
FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_MARAPRMAYdate`;`

**Deduping METs and Sleep**
`SELECT *, COUNT(*) AS duplicate_count
FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_MarAPR` 
GROUP BY ID, date, value, logid, date_datetime`
`HAVING COUNT(*)> 1;`
`## creating a deduped table minuteSleep_MarAPRdedupe ## 
CREATE OR REPLACE TABLE `bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_MarAPRdedupe` AS
SELECT DISTINCT * FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_MarAPR`;``

`## removing old table ##
DROP TABLE bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_MarAPR;`

`## repeating dedupe for minuteSleep_MARAPR ##
CREATE OR REPLACE TABLE `bellabeats-452721.Fitabase_MARAPRMAY.minuteMETsMARAPRdedupe` AS
SELECT DISTINCT * FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteMETsMARAPR`;`

`## repeating dedupe for minuteMETsAPRMAY ##
CREATE OR REPLACE TABLE `bellabeats-452721.Fitabase_MARAPRMAY.minuteMETsAPRMAYdedupe` AS
SELECT DISTINCT * FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteMETsAPRMAY`;`

`## repeating dedupe for minuteMETsAPRMAY ##
CREATE OR REPLACE TABLE `bellabeats-452721.Fitabase_MARAPRMAY.dailyactivity_MARAPRMAYdedupe` AS
SELECT DISTINCT * FROM `bellabeats-452721.Fitabase_MARAPRMAY.dailyactivity_MARAPRMAY`;``

`**Merging March-Apr to Apr-May**
``#combining METs MARAPR and APRMAR##
CREATE OR REPLACE TABLE `bellabeats-452721.Fitabase_MARAPRMAY.minuteMETs_MARAPRMAY` AS
SELECT * FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteMETsAPRMAYdedupe`
UNION ALL
SELECT * FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteMETsMARAPRdedupe`;`
`
##combining minutesleep MARAPR and APRMAR##
CREATE OR REPLACE TABLE `bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_MARAPRMAY` AS
SELECT * FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_APRMAYdedupe`
UNION ALL
SELECT * FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_MARAPR`;`

**Cleaning METS**
``## 1 - dedupe complete``
`## 2 - Standardize data - datetime has already been converted, no need to convert text or trim spaces.`
`## 3 - Missing or Null values`
`#Checking for Null#`
`SELECT ActivityMinute_datetime, COUNT(*) AS null_count `
FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteMETs_MARAPRMAY` `
WHERE ActivityMinute_datetime IS NULL 
GROUP BY ActivityMinute_datetime;`
`# confirmed no Null values ID, ActivityMinute, METs, ActivityMinute_datetime #`
`## 4 - Remove Unwanted Characters (checked - not applicable)`
`## 5 - Validate Data Integrity`
`SELECT * FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteMETs_MARAPRMAY`  WHERE METs < 0; #negatives check complete`
`## 6 - Normalize data (break columns out or merge columns where necessary. `
`## Breaking DATETIME to DATE and TIME
CREATE OR REPLACE TABLE `bellabeats-452721.Fitabase_MARAPRMAY.minuteMETs_MARAPRMAYdate` AS
SELECT 
    *,  
    DATE(ActivityMinute_datetime) AS date_only,  
    TIME(ActivityMinute_datetime) AS time_only  
FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteMETs_MARAPRMAY`;`
`## 7 - Standardize Categorical data (not applicable)`
`## 8 - Check for outliers (checked no concerning outliers)`
`SELECT
MAX(METs),
from `bellabeats-452721.Fitabase_MARAPRMAY.minuteMETs_MARAPRMAYdate`;
#result 189 = acceptable. 
SELECT
MIN(METs),
from `bellabeats-452721.Fitabase_MARAPRMAY.minuteMETs_MARAPRMAYdate`;`
`#result 0 = expected`
``## 9 - Remove unnecessary columns``
`## no columns to be removed`
`## 10 - Create cleaned dataset - 
##already complete due to no changes required ##`

`#Cleaning METs Complete#`

**Cleaning Sleep**
`##1 - dedupe complete
##2 - Standardize data - datetime has already been converted, no need to convert text or trim spaces.
##3 - Missing or Null values
#Checking for Null#
SELECT ActivityMinute_datetime, COUNT(*) AS null_count 
FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteSleepMARAPRMAY_MARAPRMAY` 
WHERE ActivityMinute_datetime IS NULL 
GROUP BY ActivityMinute_datetime;`
`# confirmed no Null values ID, ActivityMinute, METs, ActivityMinute_datetime #
## 4 - Remove Unwanted Characters (checked - not applicable)
## 5 - Validate Data Integrity`
`SELECT * FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteSleepMARAPRMAY_MARAPRMAY`  WHERE METs < 0; #negatives check complete
## 6 - Normalize data (break columns out or merge columns where necessary.` 
`## Breaking DATETIME to DATE and TIME
CREATE OR REPLACE TABLE `bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_MARAPRMAYdate` AS
SELECT 
    *,  
    DATE(ActivityMinute_datetime) AS date_only,  
    TIME(ActivityMinute_datetime) AS time_only  
FROM `bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_MARAPRMAY`;
``## 7 - Standardize Categorical data (not applicable)``
``## 8 - Check for outliers (checked no concerning outliers)``
SELECT
MAX(METs),
from `bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_MARAPRMAYdate`;
#result 189 = acceptable. 
SELECT
MIN(METs),
from `bellabeats-452721.Fitabase_MARAPRMAY.minuteSleep_MARAPRMAYdate`;
#result 0 = expected`

`## 9 - Remove unnecessary columns `
`## no columns to be removed`

`## 10 - Create cleaned dataset - `
`##already complete due to no changes required ##`

Cleaning Sleep Complete`



**














#### - Have you ensured your data's integrity?
#### - What steps have you taken to ensure the data is clean?
#### - How can you verify that your data is clean and ready to analyze?
#### - Have you documented your cleaning process so you can review and share those results?

for March-April and April-May.  I will review, clean, and then consolidate each dataset into a March - May overview.  I will also

### Key tasks 
#### - Check data for errors.

For each dailyActivity_merged.csv file reviewed and approved


#### - Transform the data so you can work with it efficiently.
#### - Document the cleaning process

### Deliverable 
Document the cleaning process

## Step 4) Analyze
### Guiding Questions
#### How should you organize your data to perform analysis on it?
#### Has your data been properly formatted?
#### What surprises did you discover in the data?
#### What trends or relationships did you find in the data?
#### How will these insights help answer your business questions?

### Key Tasks
#### Aggregate your data so it's useful and accessible.
#### Organize and format your data.
#### Perform calculations
#### Identify trends and relationships

### Deliverable
#### A summary of your analysis.

## Step 5) Share 
### Guiding Questions
#### Were you able to answer the business questions?
#### What story does your data tell?
#### How do your findings relate to your original question?
#### Who is your audience? What is the best way to communicate with them?
#### Can data visualization help you share your findings?
#### Is your presentations accessible to your audience?

### Key tasks
#### Determine the best way to share your findings
#### Create effective data visualizations
#### Present your findings
#### Ensure your work is accessible

### Deliverable 
#### Supporting Visualizations and Key Findings. 

## Step 6) Act
### Guiding Questions
#### What is your final conclusion based on your analysis?
#### How could your team and business apply your insights?
#### What next steps would you or your stakeholders take based on your findings?
#### Is there additional data you could use to expand on your findings?

### Key Tasks
#### Create portfolio
#### Add your case study
#### Practice presenting

### Deliverable
#### Your top high-level insights based on your analysis
