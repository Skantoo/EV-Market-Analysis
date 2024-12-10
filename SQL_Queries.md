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
