# SQL Module 2:  Using SQL in R’s SQLDF Package 

This SQL module will focus on the version of SQL used in R’s sqldf package. The examples and sample code shown throughout the SQL Modules can typically be adapted to other software that includes SQL (e.g., Microsoft SQL Server) with just a few changes.  This is because many “flavors” of SQL are based on T-SQL or “Transact-SQL.”

After a brief introduction to sqldf the module will follow the data analyst’s workflow.

#### **Data Analyst Workflow**

The figure below provides a reasonable visual representation of the data analyst's workflow.  The first step is **Importing** the data into the data analyst's preferred software package.  The second step is **Tidying** the data.  This step is sometimes referred to as "munging" or "wrangling" but they all generally mean the same thing:  preparing the data for analysis.  The remaining time is spent **Exploring** the data, often through summary statistics, regressions, or more advanced techniques like Machine Learning and Artificial Intelligence.  The final step is to **Communicate** insights through figures, charts, tables, and other visuals.

The true workflow is not as linear as the figure implies.  Nevertheless, it provides a useful framework for how to approach a project and for learning a new software package.  

![Data](https://d33wubrfki0l68.cloudfront.net/795c039ba2520455d833b4034befc8cf360a70ba/558a5/diagrams/data-science-explore.png)

##### Notes:  Wickham, Hadley, and Garrett Grolemund. R for data science: import, tidy, transform, visualize, and model data. " O'Reilly Media, Inc.", 2016. (available at https://r4ds.had.co.nz/). #####

Programming often follows the Pareto Principle:  80% of the work can be done by 20% (or less) of the available features.  This module is designed to introduce the fewest number of concepts necessary to be proficient on the vast majority of tasks.

## The sqldf Package in R

[[About the sqldf package]]

To load the sqldf package do the following:

```r
install.packages("sqldf")
library(sqldf)
```

To query a database using sqldf you will need to first make sure your data is stored as a dataframe.

```r
df <- as.data.frame(df) 
```

> **BEST PRACTICE NOTE:**  
> The R packages dplyr and sqldf have very similar capabilities. Depending upon the use case, dplyr may be a better option than sqldf and vice versa.

Almost every SQL query you will write will include a SELECT keyword and a FROM keyword.  The SELECT keyword is followed by a list of fields that you want the resulting dataframe to contain.  The FROM keyword is followed by a list of databases that you want to query. For example, if you want to create a new dataframe 'mpgdf' containing only the column 'mpg' from the 'mtcars' data frame that is included in base R you would write:

```r
install.packages("sqldf")
library(sqldf)
df <- mtcars
mpg <- sqldf(
    "SELECT mpg 
    FROM df"
  )
-- [[insert number of observations]]
head(mpg)
```

> **BEST PRATICE NOTES:**
> It is convention to use all caps when writing a SQL keyword (so “SELECT” not “select”). Some variants of SQL are case sensitive while others are not (sqldf is not).
> Always record the number of rows returned from each query at the bottom to help Quality Control work. To do this use '--' to comment out individual lines. Use /* to open and */ to close multiple line comments.


A WHERE keyword allows you to include only those records that satisfy a certain condition. Importantly, the FROM and WHERE keywords are activated at the beginning of the SQL query so that only those records that satisfy the WHERE condition are processed as part of the SELECT keyword. For example, if you wanted to create a data frame that only includes cars with 25 or more mpg you could specify:

```r
mpg25 <- sqldf(
    "SELECT * 
    FROM df 
    WHERE mpg >= 25"
  )
print(mpg25)
```

The asterisk '*' allows you to SELECT all columns in the data frame without having to name all of the columns in the data frame.
The order of the statements SELECT…FROM…WHERE is very important. A query where the statements are out of order (e.g., SELECT * WHERE mpg >= 25 FROM df) will result in an error.


* [Basic sql with sqldf in R](https://www.youtube.com/watch?v=TRGLODyf-6s) (4:35)

## Import
Because sqldf uses dataframes you will likely import your data through R packages like read_csv or read_tsv. However, you may not always use SQL in R so it is useful to understand how to add data using SQL syntax.

### Inserting Data into a SQL Database

> library(sqldf)
> BOD
  Time demand
1    1    8.3
2    2   10.3
3    3   19.0
4    4   16.0
5    5   15.6
6    7   19.8
> New <- BOD[1, ]
> BOD1 <- BOD[2:3,]
> sqldf(c("insert into New select * from BOD1", "select * from New"))
  Time demand
1    1    8.3
2    2   10.3
3    3   19.0


### Stacking Data with UNION and UNION ALL


## Wrangle
#### Creating Subsets of Data
#### Casting Variables

## Analyze
#### Frequencies
#### Descriptive Statistics

## Communicate

SQL is a querying language. It does not, by itself, have graphics or visualization capabilities. For this reason, output of SQL query is often “moved” to a different software package (e.g., ggplot2 in R, or Excel in order to communicate the results.

## Other Important Concepts (Optional)
COALESCE - [[]]
HAVING - command is used instead of WHERE with aggregate functions

Source: https://www.w3schools.com/sql/sql_quickref.asp


> **BEST PRACTICE:**
> When writing SQL code (or for any coding language) keep in mind the following best practices:
> -   Always save your SQL queries in a program or "script"
> -   Include comments throughout the script to explain what each individual query does
> -   After each query in a script include a comment for results and/or conclusions
