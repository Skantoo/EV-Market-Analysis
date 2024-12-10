# **Electric Vehicle Market Analysis**

This project showcases the powerful synergy between SQL and Power BI in analyzing and visualizing electric vehicle sales data. SQL was employed to perform comprehensive data queries, including ranking makers, evaluating growth patterns, and identifying market trends. Power BI brought these insights to life through interactive and visually engaging dashboards. Together, these tools highlight the potential of combining advanced data analysis with intuitive visual storytelling.



### Dataset Description  

#### **electric_vehicle_sales_by_state.csv**  
| Column Name             | Description                                                                                     | Data Type     |  
|--------------------------|-------------------------------------------------------------------------------------------------|---------------|  
| `date`                  | The date on which the data was recorded. Format: DD-MMM-YY. (Data is recorded on a monthly basis) | Date          |  
| `state`                 | The name of the state where the sales data is recorded, indicating the geographical location within India. | String        |  
| `vehicle_category`      | The category of the vehicle, specifying whether it is a 2-Wheeler or a 4-Wheeler.               | String        |  
| `electric_vehicles_sold`| The number of electric vehicles sold in the specified state and category on the given date.     | Integer       |  
| `total_vehicles_sold`   | The total number of vehicles (including both electric and non-electric) sold in the specified state and category on the given date. | Integer |  

---

#### **electric_vehicle_sales_by_makers.csv**  
| Column Name             | Description                                                                                     | Data Type     |  
|--------------------------|-------------------------------------------------------------------------------------------------|---------------|  
| `date`                  | The date on which the sales data was recorded. Format: DD-MMM-YY. (Data is recorded on a monthly basis) | Date          |  
| `vehicle_category`      | The category of the vehicle, specifying whether it is a 2-Wheeler or a 4-Wheeler.               | String        |  
| `maker`                 | The name of the manufacturer or brand of the electric vehicle.                                 | String        |  
| `electric_vehicles_sold`| The number of electric vehicles sold by the specified maker in the given category on the given date. | Integer |  

---

#### **dim_date.csv**  
| Column Name             | Description                                                                                     | Data Type     |  
|--------------------------|-------------------------------------------------------------------------------------------------|---------------|  
| `date`                  | The specific date for which the data is relevant. Format: DD-MMM-YY. (Data is recorded on a monthly basis) | Date          |  
| `fiscal_year`           | The fiscal year to which the date belongs. This is useful for financial and business analysis.  | Integer       |  
| `quarter`               | The fiscal quarter to which the date belongs. Fiscal quarters are typically divided as Q1, Q2, Q3, and Q4. | String |  
