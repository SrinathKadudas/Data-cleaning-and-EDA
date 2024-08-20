# Data-cleaning-and-EDA
This is a mysql project on which focuses on data cleaning and EDA of a layoff dataset
select * from layoffs; 


use world_layoffs; -- use the database world_layoffs.

-- The first step is to clean the data. we use 4 steps to clean the data. 
-- 1. remove duplicates
-- 2. standarize the data
-- 3. remove null values and blank values
-- 4. remove any columns that are not necessary
-- first create a similar table like layoffs to fire your queries

create table layoffs_staging like layoffs;
insert into layoffs_staging select * from layoffs; -- we have created a clone of layoffs table. This has to be done in real world project so we dont miss on our original dataset

select * from layoffs_staging;

-- 1. Remove duplicates from the data

-- since we dont have a unique identifying factor for each row we use a row number.
select *, row_number() 
over(partition by company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) 
as rn from layoffs_staging;

with duplicate_cte as
(
select *, row_number() 
over(partition by company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) 
as rn from layoffs_staging)
select * from duplicate_cte where rn >1;


/* we have 5 duplicates as per created CTE, and they can not be directly removed using delete statement in CTE. so we create another table similar
to layoff_staging and delete where rn >1*/

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
  `rn` int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
select * from layoffs_staging2;

insert into layoffs_staging2 
select *, row_number() 
over(partition by company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) 
as rn from layoffs_staging;

select * from layoffs_staging2;
select * from layoffs_staging2 where rn >1;
Delete from layoffs_staging2 where rn > 1; 

/* we removed all the duplicates from the data. if we had a unique column it would be easier. 
since we didnot had unique column we need to create a rn using cte and delete the duplicates*/

-- standardizing data

select * from layoffs_staging2;

select distinct(company) from layoffs_staging2;

select company, trim(company) from layoffs_staging2; -- remove the white spaces at from company column

update layoffs_staging2 
set company = trim(company); 

select distinct(industry) from layoffs_staging2 order by 1;
select * from layoffs_staging2 where industry like 'Crypto%';

update layoffs_staging2
set industry = 'Crypto' where industry like 'Crypto%';

select distinct(industry) from layoffs_staging2;

select distinct(location) from layoffs_staging2 order by 1;
select distinct(country) from layoffs_staging2 order by 1;

update layoffs_staging2 
set country = 'United States' where country like 'United States%';

describe layoffs_staging2;
-- we have date parameter in text format and we need to change it to date format. 
update layoffs_staging2 set date = str_to_date(date, '%m/%d/%Y');
alter table layoffs_staging2 modify column `date` date; 

-- we have created a column rn and we need to delete it, since it is not use full now
select * from layoffs_staging2;


alter table layoffs_staging2 drop rn ;


-- 3. remove null values and blank values
select * from layoffs_staging2 where 
total_laid_off is null and
percentage_laid_off is null and industry is null or industry = '';

select * from layoffs_staging2 where industry is null or industry = ''; 

select * from layoffs_staging2 where company like 'Bally%';

select * from layoffs_staging2 where company in( 'Juul', 'Carvana' , 'Airbnb' , "Bally's Interactive");
-- there are few null values in industry column and few columns are blank. first change all the blanks to null values
update layoffs_staging2 set industry = null where industry = ''; 


select t1.industry, t2.industry from layoffs_staging2 t1 join layoffs_staging2 t2 on
t1.company = t2.company where t1.industry is null  and t2.industry is not null;

update layoffs_staging2 t1 join layoffs_staging2 t2 on
t1.company = t2.company set t1.industry = t2.industry where t1.industry is null and t2.industry is not null;

select * from layoffs_staging2 where total_laid_off is null and percentage_laid_off is null;
delete from layoffs_staging2 where total_laid_off is null and percentage_laid_off is null;
select * from layoffs_staging2;
