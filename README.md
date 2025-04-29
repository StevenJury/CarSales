# CarSales
Today we will be analyzing car sales across the United States. 
Data is from https://www.kaggle.com/datasets/syedanwarafridi/vehicle-sales-data
I am using SSMS with a flat file import. 
* Query:
* ![image](https://github.com/user-attachments/assets/67cea8ca-fd39-4082-9e98-60ef03ab39cc)

Lets see how many of each model car there is
```
select count(*) as amount_of_models, model  from dbo.carsales
Group by model
ORDER BY count(*) desc
```

![image](https://github.com/user-attachments/assets/ed0f9fdf-167c-4b94-b7f4-cefb54c02680)


Lets do the same to see car sales by state
```
select count(*), state from dbo.carsalesnew
GROUP BY state
```

![image](https://github.com/user-attachments/assets/841ec6ae-5fa3-4cc9-9f13-6a026880c677)
```

Now lets see our average values by Make, as we can see our luxury car brands are at the top 


select AVG(mmr) as averagevalue, make from dbo.carsalesnew
Group by Make
ORDER BY AVG(MMR) DESC
```

![image](https://github.com/user-attachments/assets/48553508-2c78-4762-907a-e3103dd5368b)


Now I want to see how many cars there are in certain set parameters, I am thinking of classifying price ranges are 'cheap 'expensive' etc. Lets use
a CASE statement to categorize these prices and see how many cars fall within them 

```
select count(*) as totalcars, CASE WHEN mmr > 70000 THEN 'expensive'
WHEN mmr BETWEEN 40000 AND 70000 THEN 'Midrange' 
WHEN mmr < 40000 THEN 'Cheap' 
END AS Carvalue 
from dbo.carsalesnew
GROUP BY CASE WHEN mmr > 70000 THEN 'expensive'
WHEN mmr BETWEEN 40000 AND 70000 THEN 'Midrange' 
WHEN mmr < 40000 THEN 'Cheap' END
ORDER BY carvalue desc
```
![image](https://github.com/user-attachments/assets/c497e69d-b864-4126-9208-e7510d964a27)


As expected we have the most 'cheap' care. the NULL value is because our dataset doesnt contain car value for every row. 



Now I want to see what the Average price of the cars are based on milage. Im going to create a CASE statement in order to group my odometer readings
```
SELECT 
  CASE
    WHEN odometer > 100000 THEN 'Above100k'
    WHEN odometer BETWEEN 80000 AND 100000 THEN '80to100K'
    WHEN odometer BETWEEN 50000 AND 79999 THEN '50to79K'
    WHEN odometer BETWEEN 30000 AND 49999 THEN '30to49K'
    WHEN odometer BETWEEN 10000 AND 29999 THEN '10to29K'
    ELSE 'Under10K'
  END AS Mileage,
AVG(mmr) AS AvgMMR
FROM dbo.carsalesnew
GROUP BY 
  CASE
    WHEN odometer > 100000 THEN 'Above100k'
    WHEN odometer BETWEEN 80000 AND 100000 THEN '80to100K'
    WHEN odometer BETWEEN 50000 AND 79999 THEN '50to79K'
    WHEN odometer BETWEEN 30000 AND 49999 THEN '30to49K'
    WHEN odometer BETWEEN 10000 AND 29999 THEN '10to29K'
    ELSE 'Under10K'
  END
ORDER BY AvgMMR asc
```
But upon running this query I am running across a problem 
![image](https://github.com/user-attachments/assets/3f1454ab-c6f2-4b0a-8e7d-173b3adea933)



What could this be? 
Upon review my Odometer and MMR datatypes are both INT, so it is not a string issue. When SQL is calculating my AVG, they are doing an internal SUM first
Maybe that number is too big to classify as an INT? Lets try casting it as a FLOAT to see if that fixes it. 
```
SELECT 
  CASE
    WHEN odometer > 100000 THEN 'Above100k'
    WHEN odometer BETWEEN 80000 AND 100000 THEN '80to100K'
    WHEN odometer BETWEEN 50000 AND 79999 THEN '50to79K'
    WHEN odometer BETWEEN 30000 AND 49999 THEN '30to49K'
    WHEN odometer BETWEEN 10000 AND 29999 THEN '10to29K'
    ELSE 'Under10K'
  END AS Mileage,
AVG(CAST(mmr AS FLOAT)) AS AvgMMR
FROM dbo.carsalesnew
GROUP BY 
  CASE
    WHEN odometer > 100000 THEN 'Above100k'
    WHEN odometer BETWEEN 80000 AND 100000 THEN '80to100K'
    WHEN odometer BETWEEN 50000 AND 79999 THEN '50to79K'
    WHEN odometer BETWEEN 30000 AND 49999 THEN '30to49K'
    WHEN odometer BETWEEN 10000 AND 29999 THEN '10to29K'
    ELSE 'Under10K'
  END
ORDER BY AvgMMR asc
```
![image](https://github.com/user-attachments/assets/98ee4cbf-e8fa-446b-af87-b54ffc7dbb04)

Looks like that worked. 


Now lets add together car models so we can see our average prices per car models with the milage bucket
```
SELECT 
  CASE
    WHEN odometer > 100000 THEN 'Above100k'
    WHEN odometer BETWEEN 80000 AND 100000 THEN '80to100K'
    WHEN odometer BETWEEN 50000 AND 79999 THEN '50to79K'
    WHEN odometer BETWEEN 30000 AND 49999 THEN '30to49K'
    WHEN odometer BETWEEN 10000 AND 29999 THEN '10to29K'
    ELSE 'Under10K'
  END AS Mileage,
AVG(CAST(mmr AS FLOAT)) AS AvgMMR,
make
FROM dbo.carsalesnew
GROUP BY 
  CASE
    WHEN odometer > 100000 THEN 'Above100k'
    WHEN odometer BETWEEN 80000 AND 100000 THEN '80to100K'
    WHEN odometer BETWEEN 50000 AND 79999 THEN '50to79K'
    WHEN odometer BETWEEN 30000 AND 49999 THEN '30to49K'
    WHEN odometer BETWEEN 10000 AND 29999 THEN '10to29K'
    ELSE 'Under10K'
  END,
 make
ORDER BY AvgMMR asc
```
![image](https://github.com/user-attachments/assets/dce8ee6b-a37c-4b01-be44-a404120ad255)



From here we can mess around with using WHERE statements to get some more in depth detail on specific makes

![image](https://github.com/user-attachments/assets/20ec4b08-ba74-42db-9abf-f716a098d927)

Now I want to know how many cars are being sold under market value. This is an easy query, we just need to find out what cars have a higher selling price than MMR and then we can group by our model. 

```
select count(*) as totalcars, model from dbo.carsalesnew
WHERE mmr > sellingprice
GROUP BY model
ORDER BY count(*) desc
```

![image](https://github.com/user-attachments/assets/813e202e-85d1-4a35-9774-db655280b0dd)


Lets put it together in Tableau to show the top selling brand by state using the RANK function 

https://public.tableau.com/app/profile/steven.jury/viz/Cardata_17459492172650/CarSales#1

![image](https://github.com/user-attachments/assets/ee87910f-2858-451f-806f-64b41d57943d)

I also made sure to rerun some queries like 
```
select SUM(sellingprice), make  from dbo.carsalesnew
WHERE state = 'NM'
GROUP BY Make
ORDER BY sum(sellingprice) desc
```
In order to make sure Tableau was calculating correctly.


