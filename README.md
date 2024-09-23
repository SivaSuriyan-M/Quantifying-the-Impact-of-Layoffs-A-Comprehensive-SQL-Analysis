# Quantifying-the-Impact-of-Layoffs-A-Comprehensive-SQL-Analysis

# Exploratory Data Analysis (EDA) on Global Layoffs Data

## Overview
This project performs **Exploratory Data Analysis (EDA)** on the global layoffs dataset stored in `world_layoffs.layoffs_staging2`. The aim is to explore the data, identify trends, patterns, and outliers, and derive meaningful insights, such as companies with significant layoffs, industries and countries most affected, and layoff trends over time.

The EDA is executed using SQL queries to gain insights such as:
- Maximum layoffs in a single event.
- Companies that laid off 100% of their workforce.
- Layoffs grouped by country, location, industry, and company stage.
- Trends of layoffs over the years and cumulative monthly layoffs.

## Dataset Information
The dataset `world_layoffs.layoffs_staging2` contains the following key columns:
- **`company`**: Name of the company.
- **`total_laid_off`**: Total number of employees laid off.
- **`percentage_laid_off`**: Percentage of the company laid off (1 = 100%, 0 = 0%).
- **`funds_raised_millions`**: Total funds raised by the company.
- **`location`**: Location where layoffs occurred (e.g., city).
- **`country`**: Country where layoffs occurred.
- **`date`**: Date of the layoffs.
- **`industry`**: Industry classification of the company.
- **`stage`**: Stage of the company (e.g., startup, growth, mature).

---

## SQL Queries and Insights

### 1. Data Overview
Retrieve all data from the dataset:
```sql
SELECT * 
FROM world_layoffs.layoffs_staging2;
```

### 2. Maximum Layoffs in a Single Event
Identify the largest number of layoffs in a single event:
```sql
SELECT MAX(total_laid_off)
FROM world_layoffs.layoffs_staging2;
```

### 3. Layoff Percentages
Analyze the highest and lowest layoff percentages:
```sql
SELECT MAX(percentage_laid_off), MIN(percentage_laid_off)
FROM world_layoffs.layoffs_staging2
WHERE percentage_laid_off IS NOT NULL;
```

### 4. Companies with 100% Layoffs
Find companies that laid off 100% of their workforce:
```sql
SELECT *
FROM world_layoffs.layoffs_staging2
WHERE percentage_laid_off = 1;
```

### 5. Companies with 100% Layoffs and Funding
Check how large these companies were by analyzing their funds raised:
```sql
SELECT *
FROM world_layoffs.layoffs_staging2
WHERE percentage_laid_off = 1
ORDER BY funds_raised_millions DESC;
```

### 6. Largest Single Layoff Events
Find companies with the largest layoffs in a single event:
```sql
SELECT company, total_laid_off
FROM world_layoffs.layoffs_staging2
ORDER BY total_laid_off DESC
LIMIT 5;
```

### 7. Companies with the Most Total Layoffs
List companies with the highest total layoffs across all events:
```sql
SELECT company, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY company
ORDER BY SUM(total_laid_off) DESC
LIMIT 10;
```

### 8. Layoffs by Location
Analyze layoffs by location (e.g., city):
```sql
SELECT location, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY location
ORDER BY SUM(total_laid_off) DESC
LIMIT 10;
```

### 9. Layoffs by Country
List countries with the most layoffs:
```sql
SELECT country, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY country
ORDER BY SUM(total_laid_off) DESC;
```

### 10. Layoffs by Year
Analyze layoffs over the years:
```sql
SELECT YEAR(date), SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY YEAR(date)
ORDER BY YEAR(date) ASC;
```

### 11. Layoffs by Industry
Identify which industries were hit hardest by layoffs:
```sql
SELECT industry, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY industry
ORDER BY SUM(total_laid_off) DESC;
```

### 12. Layoffs by Company Stage
Analyze layoffs based on the stage of the company:
```sql
SELECT stage, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging2
GROUP BY stage
ORDER BY SUM(total_laid_off) DESC;
```

---

## Advanced Analysis

### 13. Layoffs by Company and Year
Find the companies with the most layoffs per year:
```sql
WITH Company_Year AS 
(
  SELECT company, YEAR(date) AS years, SUM(total_laid_off) AS total_laid_off
  FROM world_layoffs.layoffs_staging2
  GROUP BY company, YEAR(date)
),
Company_Year_Rank AS (
  SELECT company, years, total_laid_off, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
  FROM Company_Year
)
SELECT company, years, total_laid_off, ranking
FROM Company_Year_Rank
WHERE ranking <= 3
ORDER BY years ASC, total_laid_off DESC;
```

### 14. Rolling Total Layoffs by Month
Calculate the total layoffs per month and track the rolling cumulative total:
```sql
WITH DATE_CTE AS 
(
  SELECT SUBSTRING(date, 1, 7) AS dates, SUM(total_laid_off) AS total_laid_off
  FROM world_layoffs.layoffs_staging2
  GROUP BY dates
  ORDER BY dates ASC
)
SELECT dates, SUM(total_laid_off) OVER (ORDER BY dates ASC) AS rolling_total_layoffs
FROM DATE_CTE
ORDER BY dates ASC;
```

---

## Conclusion
This EDA project has uncovered key insights about global layoffs:
- **Companies** with the largest layoffs, including those that laid off 100% of their workforce.
- **Industries** most affected by layoffs.
- **Countries** and **locations** where layoffs were concentrated.
- **Layoff trends** over time, providing insights into how layoffs evolved over the years and months.

These insights can help in understanding economic trends, the impact of layoffs on industries, and the specific vulnerabilities of companies during periods of economic distress.
