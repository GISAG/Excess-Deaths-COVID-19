<p align="center">
<b>Dashboard: https://arcg.is/05jD4K0 </b>
<p align="left">
 
# Excess-Deaths-COVID-19
Monitoring the spread of the emerging Covid-19 pandemic has taken the world by storm. From the initial reports of a novel virus spreading in Wuhan, China, to the Fall of 2020, people around the world have turned to geographic information systems (GIS) and online maps to track its spread. To control the pandemic, scientists have begun to analyze and model the spread of cases to evaluate factors which may influence the rate of growth and predict future outbreaks or hotspots. Unfortunately, many early Covid-19 cases went undetected due to lack of testing and disparities in access. Additionally, testing rates and efficacy are not consistent across geographies and time. For many reasons, current counts of Covid-19 cases represent a biased and considerable under count which can negatively impact predictions and estimates moving forward. We estimate excess deaths using the The CDC National Center for Health Statistics and Provisional Mortality Data and the Provisional COVID-19 Death Counts by Week Ending Data and State to find estimates of excess deaths at a county level.

# Data

NCHS, National Vital Statistics System Provisional Mortality Data 2015-2020. Used to calculate historical weekly seasonality value (2015-2019), in addition to the 2019 observation and seasonality values (Obs_2019.csv and Os_2019_weekly.csv) used to check accuracy of forecast model. [https://data.cdc.gov/NCHS/Weekly-counts-of-deaths-by-jurisdiction-and-age-gr/y5bj-9g5w]

CDC National Center for Health Statistics Mortality Data. The data is used to estimate 2019 and 2020 deaths at the county-level. It is also used to determine the proportionalty value for each county.[https://wonder.cdc.gov/]

The CDC Provisional Death Counts in the United States by County. The data is used for county-level observed death totals and COVID-19 totals. [https://data.cdc.gov/NCHS/Provisional-COVID-19-Death-Counts-in-the-United-St/kn79-hsxy]

The CDC NCHS Provisional Death Counts by Week Ending and State. This data is used for state-level observed death totals and COVID-19 totals. [https://data.cdc.gov/NCHS/Provisional-COVID-19-Death-Counts-by-Week-Ending-D/r8kw-7aab]


# Excess Deaths and Methodology
To determine the Excess Deaths by county we (1) project the number of expected deaths at the county-level by week, (2) apply weekly mortality totals to the respective counties (3) identify all observed and expected deaths which occur between February 1 and 4 weeks prior to the last day of observation, and (4) Calculate the difference between expected and observed number of deaths, determine the number of deaths which exceed the expected value for the respective time, and identify the number of deaths which exceed expected values with COVID-19 deaths removed. 

## Expected Deaths
We project the number of expected deaths at the county-level by using the CDC's National Center for Health Statistics WONDER historical mortality dataset from 2012-2018. The expected number of deaths per county is calculated using the MMWR historical data to provide expected deaths for 2019 and 2020. The the expected deaths is forecasted using the ARIMA script in R. The values at the upper bound (95%) of the model result is used for 2020 expected numbers. The 2019 projections are used to test the model accuracy using CDC provincial 2019 totals. For accuracy assessment, we parse out weekly mortality numbers by identifying the seasonality value from the NCHS Provisional Mortality data from 2019. 

## Observed Deaths 
The CDC Provisional Death Counts in the United States by County, updated weekly, provides the public with the total number of all deaths and the total number of COVID-19 reported deaths per county. This dataset omits all counties with less then 10 reported COVID-19 deaths and also omits deaths which occur in the first 4 weeks of the year. As the dataset omits a substantial number of counties in the US, we developed a method to apply values provided weekly by the CDC to supplement the death counts for the omitted counties. The CDC NCHS Provisional Death Counts by Week Ending is used to de-aggregate the continously updated totals to the respective counties. 

State reporting times vary and is noted to take several weeks to tabulate final totals, therefore, death counts change frequently as numbers are revised. The counts level off and we determine the point in time where changes to the counts do not increase or decrease by more then 10 percent as the earliest lag time for the state.  Furthermore, the county mortality data begins February 1, 2020 and only includes counties where deaths recorded as COVID-19 number more then 10. As such, many counties are omitted from the dataset. A solution to this issue can be found by using the CDC Daily Updates of Totals by Week and State to de-aggregate the state totals using the average proportion of deaths that fall within each county per week to substitute for the missing counties. As the number of deaths observed at the county is a proportion of the state's total, we use the average proportion value from historical to parse out the appropriate number per county.It is important to note, when using state totals to substitute death counts for all counties where county specific totals are omitted from the CDC County Provisional totals, the number of COVID-19 cases must be removed from the state total prior to de-aggregation. If not removed, the COVID-19 associated cases will duplicated. [line 315]

### Proportionality 
We calculate the proportionality value to deaggregate state totals into the component parts (counties). We simply calculate the proportion of total state deaths which occur within county per week as follows;

![alt text](equations/eq_2.PNG)

where x is the number of deaths at county (i) for each year (t) over the sum of deaths for all counties within its state. The county average is calculated for 6 years (2012 – 2018) and the average Ps value for each county is assigned and used for deaggregating state totals.  
### Seasonality 
When given total mortality counts per county, we calculate the expected number of deaths per week using our seasonality value. This county specifc value is derived by using the weekly observations per county over the number of all deaths per year to determine the average number of deaths which occur per week. The average seasonality value for all years (2012-2018) (Cs) used to estimate the expected number of deaths per week per county. The calculation is as follows;

![alt text](equations/eq_1.PNG)

Where x is the number of deaths at county i over the total deaths for all weeks in the year (n). 

## Output Variables
County observed death numbers (described above) is located in the result: f_death_obs

The expected deaths represent the model results for both 4 weeks prior to date of observation and 8 weeks prior. The two categories are to account for the difference in lag time. This lag time varies by state, as states differ in the average amount of time it takes to report the mortality figures to the CDC. The death numbers coded as 4_week_est and 8_week_est. 

Excess deaths are calculated by the difference in total observed deaths and the expected number for each week in the United States and represent all deaths which exceed the number of expected deaths. 

Unaccounted deaths include all deaths over expected numbers per county and may reflect deaths which have been 'missed' in the official COVID-19 counts. 

# Required Files - Estimation Folder
## 1. Folder: prov_
Archive of downloaded Provisional Death Counts in the United States by County. We combine all available weekly reports regarding provisional deaths by county into one database. 
## 2.	County_Deaths_2012_2018.csv
The input file is derived from a download of county-level national mortality data from 2012-2018 obtained using the WONDER online database. WONDER is a public service operated by the CDC NCHS and provides aggregated death certificate records for United States residents where each record contains a single underlying cause of death and/or multiple causes. All counties with fewer than 10 deaths are suppressed for privacy purposes. The mortality data is derived from reported death certificate information filed throughout all fifty states and the District of Columbia. Data is coded by the states and provided to the NCHS through the Vital Statistics Cooperative Program or coded by NCHS from copies of the original death certificates provided to NCHS by the State registration offices.
## 3. 3.	Deaths_byWeek 2015_2020
The input data is derived from the CDC WONDER online database which provides weekly data on the number of deaths from all causes by jurisdiction and week with age group. Age groups are combined in the file. 
## 4. DOY_Week.csv
The reference table for obtaining the week number which corresponds to the 'week-end-date' provided with the CDC MMWR data.
## 5. Obs_2019.csv & Obs_2019_Weekly.csv
Input data tables created using the CDC Excess Deaths file and used to check for accuracy. Data compiled using CDC Provisional Deaths by week and state, and CDC Provisional Mortality by county. 

# Citation
## Code
Rahman, Md. Ishfaq Ur., Panozzo, Kimberly A. Excess Deaths by County (2020, December 1). doi: 10.13140/RG.2.2.10056.34566
## Methodology and Works
Panozzo, Kimberly A., Rahman, Md. Ishfaq Ur. Estimation of Excess Deaths During a Global Pandemic. (2020, December 1). DOI: 10.13140/RG.2.2.31145.83047

Panozzo, Kimberly A., Rahman, Md. Ishfaq Ur. Excess Deaths in the United States, A Geographic Approach.(2020, November 30). doi:10.13140/RG.2.2.34504.55046


