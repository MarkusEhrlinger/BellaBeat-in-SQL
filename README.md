# BellaBeat-in-SQL
My capstone project for my Google Analytics Course
Scenario
You are a junior data analyst working on the marketing analyst team at Bellabeat, a high-tech manufacturer of health-focused
products for women. Bellabeat is a successful small company, but they have the potential to become a larger player in the
global smart device market. Urška Sršen, cofounder and Chief Creative Officer of Bellabeat, believes that analyzing smart
device fitness data could help unlock new growth opportunities for the company. You have been asked to focus on one of
Bellabeat’s products and analyze smart device data to gain insight into how consumers are using their smart devices. The
insights you discover will then help guide marketing strategy for the company. You will present your analysis to the Bellabeat
executive team along with your high-level recommendations for Bellabeat’s marketing strategy.

Personal Note:
I'm not coseplaying this, but rather try to show my approach how and why I did what I did. 
------------------------------------------------------------------------------------------


First I loaded the files to BigQuery as I wanted to do most of the work with SQL as some CSV-files are quite large.
-------------------------------------------------------------------------------------------------------------------


How many Ids are in the dataset?
```sql
SELECT
 COUNT(DISTINCT(Id)) as Id_count
FROM
  capstone2-fitbit.FitBit_Data.dailyActivity
```
Answer: 33 in dailyActivity & dailyItensity, 24 in sleepDay
-> Very small number compared to the amount of users
At this point, I would have asked my more experienced colleagues.

I continued my work.
-----------------------------------------------------------------

At what time of the day is the most activity?

Firstly I needed to clean the timestamp in the file. I experienced some difficulties, time didn't convert, but then found out, I must change the UNICODE (UTF-8) to (UTF-7) in Liblre Office Calc, now the change was possible.
I chose steps as an activity measurement. 

```sql
SELECT
   DISTINCT(Time),
   AVG(StepTotal)
FROM
    capstone2-fitbit.FitBit_Data.hourlySteps
GROUP BY
   TIME
```
![Overall activity per hour](https://user-images.githubusercontent.com/132265260/235455328-f0ed81b3-0660-416a-af77-8104e9d442cb.png)

------------------------------------------------------------------




