-- DATA CLEANING 

SELECT *
FROM layoffs; 


-- 1. Remove Duplicates
-- 2. Standardize the Data 
-- 3. Null Values or blank values
-- 4. Remove Any Columns 


-- 1. remove duplicates
Create table layoffs_staging 
like layoffs; 

select *
from layoffs_staging;

insert layoffs_staging
select *
from layoffs; 

select * , 
row_number() over(partition by company, industry, total_laid_off, percentage_laid_off, `date`)
from layoffs_staging;

with duplicate_cte as 
(
select * , 
row_number() over(
partition by company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) as row_num
from layoffs_staging
)
select *
from duplicate_cte
where row_num > 1; 

select*
from layoffs_staging
where company='casper'; 


CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` int 
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

select*
from layoffs_staging2;

Insert into layoffs_staging2 
select *,
row_number() over(
partition by company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) as row_num
from layoffs_staging;

delete
from layoffs_staging2
where row_num  > 1;

-- standardizing data 
select company, TRIM(company)
from layoffs_staging2;

UPDATE layoffs_staging2
set company=trim(company);

select distinct industry
from layoffs_staging2
order by 1; 


update layoffs_staging2
set industry='Crypto'
where industry like 'Crypto%'; 

select distinct country
from layoffs_staging2
order by 1; 

select *
from layoffs_staging2
where country like 'United states%'
order by 1; 

select distinct country, TRIM( TRAILING '.' from country)
from layoffs_staging2
order by 1; 

update layoffs_staging2
set country= TRIM( TRAILING '.' from country)
where country like 'United states%';

SELECT `date`
from layoffs_staging2; 

update layoffs_staging2
set `date`=STR_TO_DATE(`DATE`,'%m/%d/%Y');

ALTER TABLE layoffs_staging2
modify column `date` date;

 select *
 from layoffs_staging2; 
 
 -- 3.remove null and blank values 
 
select *
from layoffs_staging2
where total_laid_off is null
and percentage_laid_off is null; 
 
 
 select *
 from layoffs_staging2
 where industry is null
 or industry =''; 
 
 select *
 from layoffs_staging2
 where company like 'Bally%' ; 
 
 
 Select * 
 from layoffs_staging2 t1
 join layoffs_staging2 t2
	ON t1.company=t2.company
where (t1.industry is null)
and t2.industry is not null ;

update layoffs_staging2
set industry = null
where industry = '' ; 


update layoffs_staging2 t1
join layoffs_staging2 t2
	ON t1.company=t2.company
set t1.industry=t2.industry
where t1.industry is null 
and t2.industry is not null ;
 
 Select * 
 from layoffs_staging2;
 -- 4. dropping columns 
 select *
from layoffs_staging2
where total_laid_off is null
and percentage_laid_off is null; 

delete
from layoffs_staging2
where total_laid_off is null
and percentage_laid_off is null; 
 
select *
from layoffs_staging2;

ALTER TABLE layoffs_staging2
drop column row_num; 
