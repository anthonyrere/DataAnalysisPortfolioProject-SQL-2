# Analyzing Global Layoffs-Data Cleaning and Exploration using MySQL

In this project, I explored the global trends of layoffs across various industries, locations, and companies using SQL. The main goal was to identify insights about layoffs by cleaning, transforming, and exploring the dataset.

The dataset used in this project contains data from 2020 to March 2023, providing insights into global layoff trends during this period.

---

## **1. Data Cleaning Process:**

The raw data had several inconsistencies, duplicates, and missing values, which could skew analysis if left unaddressed. Here’s a breakdown of the data cleaning process:

### **Step 1: Removing Duplicates**
Duplicates in the dataset can distort the accuracy of insights. Using a common table expression (CTE), I identified and removed duplicate records based on key fields such as company, location, industry, and total layoffs:
```sql
WITH duplicate_cte AS
(
SELECT *,
ROW_NUMBER() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, date, stage, country, funds_raised_millions) as row_num
FROM layoffs2
)
DELETE FROM duplicate_cte
WHERE row_num > 1;
```
This ensured that each layoff event was unique in the database.

### **Step 2: Standardizing Text Fields**
To avoid inconsistencies in text fields like company names, industry, and country, I used SQL functions to trim unnecessary spaces and unify similar values:
```sql
UPDATE layoffs_staging2
SET company = TRIM(company);
```
In the case of industries, I standardized similar values (e.g., "Crypto Startups" was unified under "Crypto"):
```sql
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```

### **Step 3: Converting Dates**
Dates were in different formats across the dataset. To ensure uniformity, I converted them to a standard date format:
```sql
UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');
```
This enabled easy querying and analysis based on date ranges.

### **Step 4: Handling Missing Values**
Some rows had missing data for key fields like industry or total layoffs. Where possible, I imputed missing values using data from similar entries:
```sql
UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
    ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL;
```
This ensured that gaps in the data did not hinder analysis.

### **Step 5: Removing Irrelevant Data**
Some records contained rows with both `total_laid_off` and `percentage_laid_off` missing. These rows were removed from the dataset to maintain data integrity:
```sql
DELETE 
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

---

## **2. Data Exploration Process:**

With a clean dataset, I began the data exploration phase to uncover insights into global layoff trends. Here are some of the key queries I ran:

### **1. Total Layoffs by Company**
```sql
SELECT company, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY company
ORDER BY 2 DESC;
```
This query helped identify which companies were responsible for the highest number of layoffs. The results showed significant layoffs concentrated in a few large corporations.

### **2. Layoffs by Industry**
```sql
SELECT industry, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY industry
ORDER BY 2 DESC;
```
Analyzing the layoffs by industry revealed that certain sectors, such as consumer, retail and transportation were hit particularly hard by layoffs.

### **3. Layoffs by Country**
```sql
SELECT country, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY country
ORDER BY 2 DESC;
```
This query provided a geographical breakdown of layoffs, showing which countries were most affected.

### **4. Trends Over Time**
I analyzed layoffs over time to understand how they evolved, especially during crises like the COVID-19 pandemic:
```sql
SELECT YEAR(`date`), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY YEAR(`date`)
ORDER BY 1 DESC;
```
The results showed spikes in layoffs during specific years, helping to correlate layoffs with global events.

### **5. Rolling Total of Layoffs**
To get a more nuanced view of layoff trends over time, I calculated the rolling total of layoffs by month:
```sql
WITH Rolling_Total AS
(
SELECT SUBSTRING(`date`, 1,7) AS `MONTH`, SUM(total_laid_off) AS total_off
FROM layoffs_staging2 
GROUP BY `MONTH`
)
SELECT `MONTH`, SUM(total_off) OVER(ORDER BY `MONTH`) AS ROLLING_TOTAL  
FROM Rolling_Total;
```
This revealed cumulative layoff numbers month by month, giving a clear sense of how layoffs accumulated over time.

### **6. Top 5 Companies with the Most Layoffs by Year**
```sql
WITH Company_Year_Rank AS
(
SELECT company, YEAR(`date`), SUM(total_laid_off),
DENSE_RANK() OVER (PARTITION BY YEAR(`date`) ORDER BY SUM(total_laid_off) DESC) AS Ranking
FROM layoffs_staging2
GROUP BY company, YEAR(`date`)
)
SELECT *
FROM Company_Year_Rank
WHERE Ranking <= 5;
```
This query identified the top 5 companies with the highest layoffs for each year between 2020 and 2023. It offered insight into which companies were most affected during specific periods.

### **Conclusion:**

Through this project, I demonstrated proficiency in SQL for data cleaning and exploration. By cleaning the raw dataset and extracting insights, I gained a better understanding of the impact of layoffs globally. This project is a testament to my ability to handle messy data and derive actionable insights, which I can apply to future analytical challenges.


