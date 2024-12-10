Top-3 and Bottom-3 makers for the fiscal years 2023 and 2024 in terms of the number of 2-wheelers sold.
```bash
WITH RankedSales AS (
    SELECT d.fiscal_year,m.maker,SUM(m.electric_vehicles_sold) AS total_electric_vehicles_sold,
        ROW_NUMBER() OVER (PARTITION BY d.fiscal_year ORDER BY SUM(m.electric_vehicles_sold) DESC) AS rank_desc,
        ROW_NUMBER() OVER (PARTITION BY d.fiscal_year ORDER BY SUM(m.electric_vehicles_sold) ASC) AS rank_asc
    FROM electric_vehicle_sales_by_makers m JOIN dim_date d ON m.date = d.date
    WHERE d.fiscal_year IN (2023, 2024) AND m.vehicle_category = '2-Wheelers'
    GROUP BY d.fiscal_year, m.maker
)

SELECT fiscal_year,maker,total_electric_vehicles_sold,
    if(rank_desc <= 3,'Top 3','Bottom 3')AS rank_type
FROM RankedSales WHERE rank_desc <= 3 OR rank_asc <= 3
ORDER BY fiscal_year, rank_type, total_electric_vehicles_sold DESC;
```
Identify the top 5 states with the highest penetration rate in 2-wheeler and 4-wheeler EV sales in FY 2024.
```bash
select * from(select d.fiscal_year, s.state,s.vehicle_category,(sum(s.electric_vehicles_sold)/sum(s.total_vehicles_sold))*100 as penetration_rate
FROM electric_vehicle_sales_by_state s JOIN dim_date d ON s.date = d.date
WHERE d.fiscal_year IN (2024) AND s.vehicle_category = '2-Wheelers'
group by s.state,s.vehicle_category
ORDER BY 
    penetration_rate DESC
LIMIT 5) as two
Union all
select * from (select d.fiscal_year, s.state,s.vehicle_category,(sum(s.electric_vehicles_sold)/sum(s.total_vehicles_sold))*100 as penetration_rate
FROM electric_vehicle_sales_by_state s JOIN dim_date d ON s.date = d.date
WHERE d.fiscal_year IN (2024) AND s.vehicle_category = '4-Wheelers'
group by s.state,s.vehicle_category
order by penetration_rate desc limit 5) as four;
```
List the states with negative penetration (decline) in EV sales from 2022 to 2024?
```bash
WITH PenetrationRates AS (
    SELECT s.state,d.fiscal_year,(SUM(s.electric_vehicles_sold) / SUM(s.total_vehicles_sold)) * 100 AS penetration_rate
    FROM electric_vehicle_sales_by_state s
    JOIN dim_date d ON s.date = d.date
    WHERE d.fiscal_year IN (2022, 2024)
    GROUP BY s.state, d.fiscal_year
),
PenetrationComparison AS (
    SELECT p22.state,p22.penetration_rate AS penetration_rate_22,p24.penetration_rate AS penetration_rate_24,
        p24.penetration_rate - p22.penetration_rate AS penetration_change
    FROM PenetrationRates p22 JOIN PenetrationRates p24 
    ON p22.state = p24.state 
    AND p22.fiscal_year = 2022
    AND p24.fiscal_year = 2024
)
SELECT state,penetration_rate_22,penetration_rate_24,penetration_change
FROM PenetrationComparison
WHERE penetration_change<0
ORDER BY penetration_change ASC;
```
How do the EV sales and penetration rates in Delhi compare to Karnataka for 2024?

```bash
WITH Delhi_prate AS (
SELECT state, a.fiscal_year,
SUM(c.electric_vehicles_sold) AS total_EV_sold,
sum(c.total_vehicles_sold) AS total_vehicle_sold,
SUM(c.electric_vehicles_sold)/sum(c.total_vehicles_sold)*100 AS penetration_rate
FROM electric_vehicle_sales_by_state AS c
JOIN dim_date AS a
on c.date=a.date
WHERE a.fiscal_year=2024 AND c.state="Delhi"
),
karnataka_prate AS ( SELECT state, a.fiscal_year,
SUM(c.electric_vehicles_sold) AS total_EV_sold,
sum(c.total_vehicles_sold) AS total_vehicle_sold,
SUM(c.electric_vehicles_sold)/sum(c.total_vehicles_sold)*100 AS penetration_rate
FROM electric_vehicle_sales_by_state AS c
JOIN dim_date AS a
on c.date=a.date
WHERE a.fiscal_year=2024 AND c.state="karnataka"
)
SELECT *
FROM Delhi_prate
UNION
SELECT *
FROM karnataka_prate;
```
What are the peak and low season months for EV sales based on the data from 2022 to 2024?
```bash
(
    SELECT 
        'Peak Month' AS season,
        DATE_FORMAT(a.date,'%M') AS month,
        SUM(c.electric_vehicles_sold) AS total_sales
    FROM 
        electric_vehicle_sales_by_state AS c
    JOIN 
        dim_date AS a USING(date)
    WHERE 
        a.fiscal_year BETWEEN 2022 AND 2024 
    GROUP BY 
        month
    ORDER BY 
        total_sales DESC
    LIMIT 1
)
UNION ALL
(
    SELECT 
        'Low Month' AS season,
        DATE_FORMAT(a.date,'%M') AS month,
        SUM(c.electric_vehicles_sold) AS total_sales
    FROM 
        electric_vehicle_sales_by_state AS c
    JOIN 
        dim_date AS a USING(date)
    WHERE 
        a.fiscal_year BETWEEN 2022 AND 2024 
    GROUP BY 
        month
    ORDER BY 
        total_sales ASC
    LIMIT 1
);
```
What are the quarterly trends based on sales volume for the top 5 EV makers (4-wheelers) from 2022 to 2024?
```bash
WITH Top5Makers AS (
    SELECT maker FROM electric_vehicle_sales_by_makers AS e
    JOIN dim_date AS d ON e.date = d.date
    WHERE d.fiscal_year BETWEEN 2022 AND 2024 AND e.vehicle_category = '4-Wheelers'
    GROUP BY maker
    ORDER BY SUM(electric_vehicles_sold) DESC LIMIT 5
)
SELECT d.quarter,e.maker,SUM(e.electric_vehicles_sold) AS quarterly_sales
FROM electric_vehicle_sales_by_makers AS e
JOIN dim_date AS d ON e.date = d.date
JOIN Top5Makers AS t ON e.maker = t.maker
WHERE d.fiscal_year BETWEEN 2022 AND 2024 AND e.vehicle_category = '4-Wheelers'
GROUP BY d.quarter,e.maker
ORDER BY e.maker,d.quarter;
```
List down the compounded annual growth rate (CAGR) in 4-wheeler units for the top 5 makers from 2022 to 2024.
```bash
WITH Top5Makers AS (
    SELECT maker 
    FROM electric_vehicle_sales_by_makers AS e
    JOIN dim_date AS d ON e.date = d.date
    WHERE d.fiscal_year BETWEEN 2022 AND 2024 
    AND e.vehicle_category = '4-Wheelers'
    GROUP BY maker
    ORDER BY SUM(electric_vehicles_sold) DESC 
    LIMIT 5
),
AnnualSales AS (
    SELECT e.maker, d.fiscal_year, SUM(e.electric_vehicles_sold) AS annual_sales
    FROM electric_vehicle_sales_by_makers AS e
    JOIN dim_date AS d ON e.date = d.date
    JOIN Top5Makers AS t ON e.maker = t.maker
    WHERE d.fiscal_year IN (2022, 2024)
    AND e.vehicle_category = '4-Wheelers'
    GROUP BY e.maker, d.fiscal_year
),
CAGR_Calculation AS (
    SELECT
        a.maker,
        round(power((SUM(CASE WHEN a.fiscal_year = 2024 THEN a.annual_sales END) / 
         SUM(CASE WHEN a.fiscal_year = 2022 THEN a.annual_sales END)),(1.0/2)) - 1 ,2)AS CAGR
    FROM AnnualSales AS a
    GROUP BY a.maker
)
SELECT maker, CAGR AS CAGR
FROM CAGR_Calculation
ORDER BY CAGR DESC;
```
List down the top 10 states that had the highest compounded annual growth rate (CAGR) from 2022 to 2024 in total vehicles sold.
```bash
WITH Top10states AS (
    SELECT state
    FROM electric_vehicle_sales_by_state AS s
    JOIN dim_date AS d ON s.date = d.date
    WHERE d.fiscal_year BETWEEN 2022 AND 2024 
    GROUP BY state
    ORDER BY SUM(total_vehicles_sold) DESC 
    LIMIT 10
),
AnnualSales AS (
    SELECT s.state, d.fiscal_year, SUM(s.total_vehicles_sold) AS annual_sales
    FROM electric_vehicle_sales_by_state AS s
    JOIN dim_date AS d ON s.date = d.date
    JOIN Top10states AS t ON s.state = t.state
    WHERE d.fiscal_year IN (2022, 2024)
    GROUP BY s.state, d.fiscal_year
),
CAGR_Calculation AS (
    SELECT
        a.state,
        round(power((SUM(CASE WHEN a.fiscal_year = 2024 THEN a.annual_sales END) / 
         SUM(CASE WHEN a.fiscal_year = 2022 THEN a.annual_sales END)),(1.0/2)) - 1 ,2)AS CAGR
    FROM AnnualSales AS a
    GROUP BY a.state
)
SELECT state, CAGR AS CAGR
FROM CAGR_Calculation
ORDER BY CAGR DESC;
```
What is the projected number of EV sales (including 2-wheelers and 4- wheelers) for the top 10 states by penetration rate in 2030, based on the compounded annual growth rate (CAGR) from previous years?
```bash
WITH Top10States AS (
    SELECT state, 
           SUM(electric_vehicles_sold) / SUM(total_vehicles_sold) AS penetration_rate
    FROM electric_vehicle_sales_by_state AS e
    JOIN dim_date AS d ON e.date = d.date
    WHERE d.fiscal_year = 2024 -- assuming 2024 as the latest year
    GROUP BY state
    ORDER BY penetration_rate DESC
    LIMIT 10
),
AnnualSales AS (
    SELECT e.state, d.fiscal_year, SUM(e.electric_vehicles_sold) AS annual_sales
    FROM electric_vehicle_sales_by_state AS e
    JOIN dim_date AS d ON e.date = d.date
    JOIN Top10States AS t ON e.state = t.state
    WHERE d.fiscal_year IN (2022, 2024) -- assuming sales data available for these years
    GROUP BY e.state, d.fiscal_year
),
CAGR_Calculation AS (
    SELECT
        a.state,
        round(power((SUM(CASE WHEN a.fiscal_year = 2024 THEN a.annual_sales END) / 
         SUM(CASE WHEN a.fiscal_year = 2022 THEN a.annual_sales END)),(1.0/2)) - 1 ,2)AS CAGR
    FROM AnnualSales AS a
    GROUP BY a.state
),
ProjectedSales AS (
    SELECT 
        c.state,
        c.CAGR,
        a.annual_sales * POWER(1 + c.CAGR, 2030 - 2024) AS projected_sales_2030
    FROM AnnualSales AS a
    JOIN CAGR_Calculation AS c ON a.state = c.state
    WHERE a.fiscal_year = 2024
)
SELECT state, projected_sales_2030
FROM ProjectedSales
ORDER BY projected_sales_2030 DESC;
```
