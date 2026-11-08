# Data Analyst Job Market Analysis

## Introduction

This project uses PostgreSQL to explore data analyst job postings, reported salaries, and the skills employers request. The goal is to identify common skills, compare salary patterns, and find skills that combine strong demand with competitive pay.

## Background

The analysis uses four CSV files: job postings, company information, skill descriptions, and the job-to-skill mapping. The included snapshot contains **787,686 job postings**. Of these, **13,331** are classified as remote Data Analyst jobs, and **604** of the remote Data Analyst postings have a reported annual salary.

The project focuses on these questions:

- Which Data Analyst postings have the highest reported annual salaries?
- Which skills are most requested in remote Data Analyst postings?
- Which skills are associated with higher average salaries?
- Which skills have both meaningful demand and strong average pay?

## Tools I used

- **PostgreSQL** to create tables, join datasets, and run the analyses.
- **pgAdmin** or **psql** to connect to the database and execute SQL.
- **CSV files** in [`csv_files/`](csv_files/) as the source data.

### Set up the database

Run the database creation script while connected to a PostgreSQL maintenance database, then create the tables in `sql_course`:

```powershell
psql -U postgres -d postgres -f .\sql_load\1_create_database.sql
psql -U postgres -d sql_course -f .\sql_load\2_create_tables.sql
```

Load the CSVs in this order to satisfy the foreign-key constraints: `company_dim`, `skills_dim`, `job_postings_fact`, then `skills_job_dim`.

The `COPY` commands in [`3_modify_tables.sql`](sql_load/3_modify_tables.sql) use a machine-specific absolute path. Update it for your computer before running the script. Alternatively, in `psql` connected to `sql_course`, use client-side `\copy` commands with the correct local path:

```text
\copy public.company_dim FROM 'D:/path/to/csv_files/company_dim.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',', ENCODING 'UTF8');
\copy public.skills_dim FROM 'D:/path/to/csv_files/skills_dim.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',', ENCODING 'UTF8');
\copy public.job_postings_fact FROM 'D:/path/to/csv_files/job_postings_fact.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',', ENCODING 'UTF8');
\copy public.skills_job_dim FROM 'D:/path/to/csv_files/skills_job_dim.csv' WITH (FORMAT csv, HEADER true, DELIMITER ',', ENCODING 'UTF8');
```

Client-side `\copy` can help when PostgreSQL's server process cannot read the CSV files directly. Once the data is loaded, open a script from [`project_sql/`](project_sql/) in pgAdmin or run it with `psql`.

## The analysis

The project contains five analysis scripts:

| Script | What it answers |
| --- | --- |
| [`1_top_paying_jobs.sql`](project_sql/1_top_paying_jobs.sql) | What are the 10 highest-paying Data Analyst postings marked as `Anywhere`? |
| [`2_top_paying_job_skills.sql`](project_sql/2_top_paying_job_skills.sql) | Which skills are listed for those high-paying postings? |
| [`3_top_in-demand_skills.sql`](project_sql/3_top_in-demand_skills.sql) | What are the five most-requested skills in remote Data Analyst postings? |
| [`4_top_paying_skills.sql`](project_sql/4_top_paying_skills.sql) | Which skills have the highest average reported annual salary in remote postings? |
| [`5_optimal_skills.sql`](project_sql/5_optimal_skills.sql) | Which skills combine demand (more than 10 salary-reported job-to-skill matches) with high average salary? |

### Most-requested skills

This chart follows the filters and top-five result of `3_top_in-demand_skills.sql`. Demand is counted as job-to-skill matches, so one posting can contribute to multiple skill counts.

<img src="./assets/top_skills_demand.svg" alt="Most-requested skills in remote Data Analyst postings: SQL, Excel, Python, Tableau, and Power BI" width="900">

### High-paying skills with meaningful demand

This chart follows the salary and demand filters in `5_optimal_skills.sql`. Average salary uses remote Data Analyst postings with a reported annual salary; only skills with more than 10 matching job-skill rows are included. Results are ordered by average salary, then demand.

<img src="./assets/top_skills_salary.svg" alt="Top 10 average salaries for skills with more than 10 salary-reported remote Data Analyst matches" width="900">

The charts are static images calculated from the included CSV snapshot. Re-run the SQL queries and refresh the charts if the source data changes.

## What I learned

- **SQL is the most frequently requested skill** in the remote Data Analyst sample, followed by Excel, Python, Tableau, and Power BI.
- **High average salary does not necessarily mean high demand.** Script 4 can rank skills with very few salary-reported matches, so those averages should be interpreted cautiously.
- **Combining salary with a minimum demand threshold gives a more balanced comparison.** In this snapshot, Go, Confluence, Hadoop, Snowflake, and Azure are among the top results that meet script 5's threshold.
- **The filters matter.** The in-demand analysis includes remote postings regardless of salary, while salary analyses require a reported annual salary. The top-paying-jobs query uses the literal location `Anywhere`, which is not the same filter as `job_work_from_home`.

## Conclusion

The results suggest that SQL, Excel, Python, Tableau, and Power BI are strong foundational skills to investigate because they appear frequently in remote Data Analyst postings. Specialized tools such as Go, Hadoop, Snowflake, and Azure also have high average salaries among the postings that meet the project's demand threshold. These findings describe this dataset snapshot, not a guarantee of salary or future demand; use the SQL scripts to re-check results for updated data.
