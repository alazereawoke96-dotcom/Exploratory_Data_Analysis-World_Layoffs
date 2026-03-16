-- exploratory mysql data analysis project

-- maximum number of employees laid off & percentage of employees laid off in one day 
SELECT MAX(total_laid_off), MAX(percentage_laid_off)
FROM layoffs_staging_2;

-- laid off 100% of their employees, have laid off the highest number of employees
select * from layoffs_staging_2
where percentage_laid_off = 1
order by total_laid_off desc;

-- Companies with the biggest single Layoff
SELECT company, total_laid_off
FROM world_layoffs.layoffs_staging_2
ORDER BY 2 DESC
LIMIT 5;

-- Companies with the most Total Layoffs
SELECT company, SUM(total_laid_off)
FROM world_layoffs.layoffs_staging_2
GROUP BY company
ORDER BY 2 DESC
LIMIT 10;



-- location that five most number of employees
SELECT location, SUM(total_laid_off) as total_laid_off
FROM layoffs_staging_2
GROUP BY location
ORDER BY 2 DESC
limit 5;

-- stage that most number of employees
SELECT stage, SUM(total_laid_off) total_laid_off
FROM layoffs_staging_2
GROUP BY stage
ORDER BY 2 DESC;

-- country that five most number of employees
SELECT country, SUM(total_laid_off) total_laid_off
FROM layoffs_staging_2
GROUP BY country
ORDER BY 2 DESC
limit 5;

-- year that the most number of layoffs
SELECT YEAR(date)  years, SUM(total_laid_off) total_laid_off
FROM layoffs_staging_2
GROUP BY  YEAR(date)
ORDER BY 2 desc;

-- industries that have experienced the highest number of layoffs
SELECT industry, sum(total_laid_off) total_laid_off from layoffs_staging_2
group by industry
order by 2 desc;

-- date with the most Total Layoffs
SELECT date, SUM(total_laid_off)  total_laid_off
FROM layoffs_staging_2
GROUP BY  date
ORDER BY 2 desc;

-- Date range of the dataset
select min(`date`),max(`date`) from layoffs_staging_2;


-- How many layoffs occurred in each month, and what is the running total of layoffs up to that month
WITH rolling_total AS
(
SELECT SUBSTRING(`date`,1,7)  AS `MONTH`, SUM(total_laid_off) AS total_off  
FROM layoffs_staging_2
WHERE SUBSTRING(`date`,1,7) IS NOT NULL
GROUP BY `MONTH`
ORDER BY 1 ASC
)
SELECT `MONTH`, total_off, SUM(total_off) OVER(order by `MONTH`) AS Rolling_total
 FROM rolling_total;


-- top five companies with the highest number of layoffs for each year based on the total layoffs recorded
with company_year(company, years, total_laid_off) as
(
select company,year(`date`), sum(total_laid_off) from layoffs_staging_2
group by company, year(`date`)
) , company_years_rank as
(
select *,dense_rank() OVER(PARTITION BY years ORDER BY total_laid_off desc) as ranking
from company_year
where years is not null
order by years ASC
)
select * from company_years_rank
where ranking <=5;


-- the total layoffs for each company by year, and how are the companies ranked based on the number of layoffs within each year
with cte_company_year(compnay,year,total_laid_off) as (
select company,extract(year from date)  ,sum(total_laid_off)  from layoffs_staging_2
group by company,extract(year from date)
) 
select *, dense_rank() over(partition by year order by total_laid_off desc) ranks
from cte_company_year
where year is not null;


-- total number of layoffs for Amazon each year, and how do these yearly totals rank in descending order
select company,year(`date`) as year, sum(total_laid_off) as total_laid_off
from layoffs_staging_2
where company = "Amazon"
group by company,year(`date`)
having sum(total_laid_off)
order by total_laid_off desc;


