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
I used mainly BigQuery, but did some cleaning and quick graphs in Google-Sheets/Libre Office Calc, the visualizations I did in Tableau on Tableau Public


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

Next question was, how is the overall activity on different workdays?

```sql
SELECT
   
   ROUND(AVG(TotalSteps),2) AS avg_steps,
   ROUND(AVG(TotalDistance), 2) AS avg_distance,
   ROUND(AVG(Calories), 2) AS avg_calories,
  
   CASE
     WHEN EXTRACT(DAYOFWEEK FROM ActivityDate) = 1 THEN "Sunday"
     WHEN EXTRACT(DAYOFWEEK FROM ActivityDate) = 2 THEN "Monday"
     WHEN EXTRACT(DAYOFWEEK FROM ActivityDate) = 3 THEN "Tuesday"
     WHEN EXTRACT(DAYOFWEEK FROM ActivityDate) = 4 THEN "Wednesday"
     WHEN EXTRACT(DAYOFWEEK FROM ActivityDate) = 5 THEN "Thursday"
     WHEN EXTRACT(DAYOFWEEK FROM ActivityDate) = 6 THEN "Friday"
     ELSE "Saturday"
   END AS day_of_week  
FROM
    capstone2-fitbit.FitBit_Data.dailyActivity

GROUP BY
    day_of_week
```
![Overall activity per weekday](https://user-images.githubusercontent.com/132265260/235456033-1e7026d5-f82d-45a6-bdd6-7640b04da53f.png)

I took AVG(calories) and AVG(TotalDistance) into the query to have a quick look on the relation.
No surprises here, more steps -> more distance -> more calories
So I stayed with the step counts.
--------------------------------------------------------------------------------

I asked myself, how can I create a sort of tier system for different levels of activity?

I kept it simple and simply added the different activity minutes per ID, then looked at the maximum and simply split it into 3 sections, high, medium and low activity.

```sql
SELECT
     DISTINCT(Id),
     SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes ) AS status_minutes,
     
     CASE
         WHEN SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes ) <= 3000 
         THEN "low active"

         WHEN SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes ) > 3000 
         AND SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes ) <= 7000 
         THEN "medium active"

         ELSE "high active"
     END AS activity_status    
FROM
    capstone2-fitbit.FitBit_Data.dailyActivity      

GROUP BY
     Id

ORDER BY
    activity_status
```
-----------------------------------------------------------------------------------------

These groups I combinded then with the hourly and weekday steps.

For the steps per hour, I could simply join the tables I created before.

```sql
SELECT
    DISTINCT(EXTRACT(TIME FROM  steps.ActivityHour)) AS time,
    AVG(steps.StepTotal) AS avg_steps,
    activity.activity_status,

FROM
    capstone2-fitbit.FitBit_Data.steps_per_hour AS steps

    JOIN
        capstone2-fitbit.FitBit_Data.activity_status AS activity
    ON steps.Id = activity.Id    
    
GROUP BY
    time,
    activity_status   
                
ORDER BY
    time                
```  
![Activity per status per hour](https://user-images.githubusercontent.com/132265260/235458439-c8aebdc1-a731-4b2a-afb2-fc82ea34adf2.png)

For the weekdays, it wasn't so easy, I had to combine the queries into a new one and for some reason needed to change the activity-level a bit.

```sql
SELECT
   Id,
   SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes ) AS status_minutes,
   ROUND(AVG(TotalSteps),2) AS avg_steps,
   
     CASE
         WHEN SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes ) <= 600 
         THEN "low active"

         WHEN SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes ) > 600 
         AND SUM(VeryActiveMinutes + FairlyActiveMinutes + LightlyActiveMinutes ) <= 1300 
         THEN "medium active"

         ELSE "high active"

     END AS activity_status,

     CASE
     WHEN EXTRACT(DAYOFWEEK FROM ActivityDate) = 1 THEN "Sunday"
     WHEN EXTRACT(DAYOFWEEK FROM ActivityDate) = 2 THEN "Monday"
     WHEN EXTRACT(DAYOFWEEK FROM ActivityDate) = 3 THEN "Tuesday"
     WHEN EXTRACT(DAYOFWEEK FROM ActivityDate) = 4 THEN "Wednesday"
     WHEN EXTRACT(DAYOFWEEK FROM ActivityDate) = 5 THEN "Thursday"
     WHEN EXTRACT(DAYOFWEEK FROM ActivityDate) = 6 THEN "Friday"
     ELSE "Saturday"
   END AS day_of_week, 

FROM
    capstone2-fitbit.FitBit_Data.dailyActivity

GROUP BY
    Id,
    day_of_week
```
![Activity per status per weekday](https://user-images.githubusercontent.com/132265260/235458805-cf281a14-8b06-43ff-90e6-507859558a51.png)

---------------------------------------------------------------------------------

I then created a dashboard in Tableau to have these different graphs in one spot and to show some of my Tableau skills.

![Dashboard 1](https://user-images.githubusercontent.com/132265260/235461715-d560cbeb-a8cc-4384-8dfd-72e8e7985629.png)

https://public.tableau.com/app/profile/markus.ehrlinger/viz/Fitbit-4-sheets/Dashboard1#1

--------------------------------------------------------------------------------------

My conclusion is, that the most active days are Saturday and Tuesday. The lowest activity is on Sunday and Thursday. So maybe a motivational reminder on those days could be a thing.
More active users work out after work, while medium users prefer the lunchbreak. Least active users prefer the morning. 
My suggestion would be to gather more data, as this dataset is very small, old and most likely biased. 
I think, next steps should be to get indivudaliced dashboards, so users can see they're own progress. I would recommend to create a level/tier system in which users can level up according to their activity.
