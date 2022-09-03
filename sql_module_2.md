# SQL Module 2:  Using SQL in R’s sqldf Package 

The examples and sample code shown throughout the SQL Modules can typically be adapted to other software that includes SQL (e.g., Microsoft SQL Server) with just a few changes.  This is because many “flavors” of SQL are based on T-SQL or “Transact-SQL.” This SQL module will focus on the version of SQL used in R’s sqldf package. 

The next section of the module introduces the sqldf package in R. It then (re)introduces the Data Analyst's Workflow. The remaining sections of this module will follow the Data Analyst's Workflow to introduce basic SQL concepts. By the end of this module you should be able to query an R dataframe using SQL statements.

## The sqldf Package in R

sqldf is an R package for running SQL statements on R data frames, optimized for convenience. To make this work, behind the scenes, sqldf relies upon another package, RSQLite. RSQLite is a "lite" version of a SQL database. When a new SQL query is submited via sqldf, the data frame is converted into a SQL database, processed, and returned as an R dataframe. The SQL database is deleted once the query results are returned. According to the package creators, "[s]urprisingly this can at times be even faster than the corresponding pure R calculation (although the purpose of the project is convenience and not speed)." While these backend mechanics aren't important for new users, understanding this will help with future modules. 

To load the sqldf package do the following:

```r
install.packages("sqldf")
library(sqldf)
```

To query a database using sqldf you will need to first make sure your data are stored as a data frame.

```r
df <- as.data.frame(df) 
```

In R, data frames often include a rowname. A rowname is not a column in the data frame but an identifier. Because they are not variables, rownames cannot be included in SQL queries. More often than not, you will want to convert the rowname to a variable so that it can be used in SQL queries. To do this you will use the rownames_to_column function in R's tibble package.

```r
install.packages("tibble")
library(tibble)
library(dplyr)

x <- USArrests %>%
  rownames_to_column(var = "State")
```

> **BEST PRACTICE NOTE:**  
> The R packages dplyr and sqldf have very similar capabilities. Depending upon the use case, dplyr may be a better option than sqldf and vice versa.

Almost every SQL query you write will include a SELECT keyword and a FROM keyword.  The SELECT keyword is followed by a list of columns that you want the resulting dataframe to contain.  The FROM keyword is followed by a list of databases that you want to query. For example, if you want to create a new dataframe 'df' containing only the column 'mpg' from the 'mtcars' data frame that is included in base R you would write:

```r
library(sqldf)
library(tibble)

df <- mtcars %>%
  rownames_to_column(var = "car_name")

mpg <- sqldf(
    "SELECT mpg 
    FROM df
    -- 32 observations"
  )

head(mpg)
```

> **BEST PRATICE NOTES:**
> It is convention to use all caps when writing a SQL keyword (so “SELECT” not “select”). Some variants of SQL are case sensitive while others are not. sqldf does not care if you use "SELECT" or "select."
> Always go back and record the number of rows returned from each query at the bottom to help Quality Control work. To do this use '--' to comment out individual lines. Use /* to open and \*/ to close multiple line comments.

A WHERE keyword allows you to include only those records that satisfy a certain condition. Importantly, the FROM and WHERE keywords are activated at the beginning of the SQL query so that only those records that satisfy the WHERE condition are processed as part of the SELECT keyword. For example, if you wanted to create a data frame that only includes cars with 25 or more mpg you could specify:

```r
mpg25 <- sqldf(
    "SELECT * 
    FROM df 
    WHERE mpg >= 25
    -- 6 observations"
  )
print(mpg25)
```

The asterisk '\*' allows you to select all columns in the data frame without having to name all of the columns in the data frame.

The order of the statements SELECT…FROM…WHERE is very important. A query where the statements are out of order (e.g., SELECT * WHERE mpg >= 25 FROM df) will result in an error.

* [Basic sql with sqldf in R](https://www.youtube.com/watch?v=TRGLODyf-6s) (4:35)

## Data Analyst Workflow

The figure below provides a reasonable visual representation of the data analyst's workflow.  The first step is **Importing** the data into the data analyst's preferred software package.  The second step is **Tidying** the data.  This step is sometimes referred to as "munging" or "wrangling" but they all generally mean the same thing:  preparing the data for analysis.  The remaining time is spent **Exploring** the data, often through summary statistics, regressions, or more advanced techniques like Machine Learning and Artificial Intelligence.  The final step is to **Communicate** insights through figures, charts, tables, and other visuals.

The true workflow is not as linear as the figure implies.  Nevertheless, it provides a useful framework for how to approach a project and for learning a new software package.  

![Data](https://d33wubrfki0l68.cloudfront.net/795c039ba2520455d833b4034befc8cf360a70ba/558a5/diagrams/data-science-explore.png)

##### Notes:  Wickham, Hadley, and Garrett Grolemund. R for data science: import, tidy, transform, visualize, and model data. " O'Reilly Media, Inc.", 2016. (available at https://r4ds.had.co.nz/). #####

Programming often follows the Pareto Principle:  80% of the work can be done by 20% (or less) of the available features.  This module is designed to introduce the fewest number of concepts necessary to be proficient on the vast majority of tasks.

## Import
Because sqldf uses dataframes you will likely first import your data through R packages like read_csv or read_tsv. However, you may not always use SQL in R so it is useful to understand how to add data using SQL syntax.

### Inserting Data into a SQL Database

[[fill]]

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

Stacking two or more datasets together in SQL is called a UNION. Somewhat confusingly there are two ways to do this, UNION and UNION ALL. UNION stacks the datasets together but only keeps _unique_ rows from both datasets.  UNION ALL stacks the datasets together and keeps _all_ rows from both datasets, even duplicates. Most of the time you will want to use UNION ALL rather than UNION. This is because duplicate values may not, in fact, be observations you want to drop.

> **BEST PRACTICE NOTE:**
> It is *very* easy for new programmers to mistakenly type UNION rather than UNION ALL (and vice versa). Including comments in your code that indicate row counts for each dataset allows for a simple check. For example, say you are trying to stack data set "a" with dataset "b" to create dataset "c." If the resulting dataset "c" has fewer rows than "a" rows + "b" rows and that isn't what you expected you will quickly be alerted to a potential problem.

To illustrate the difference between UNION and UNION ALL we are going to create two subsets of the USArrests data. The first subset will include the first 30 observations. The second subset will include the last 30 observations. This means there will be 10 observations that appear in both datasets.

```r
library(sqldf)
library(dplyr)
library(tibble)

us_arrests <- USArrests %>%
  rownames_to_column(var = 'State')

first30 <- us_arrests %>% select (State) %>% slice_head(n=30) %>% arrange(State)
last30 <- us_arrests %>% select (State) %>% slice_tail(n=30) %>% arrange(State)

stack_union <- sqldf("
        SELECT * FROM first30

                UNION

        SELECT * FROM last30
        -- dataset has 50 observations, 3 columns
")

stack_unionall <- sqldf("
        SELECT * FROM first30

                UNION ALL

        SELECT * FROM last30
        -- dataset has 60 observations, 3 columns
")
```

### JOINs
A JOIN (aka merge) of two datasets is another way to combine datasets in SQL. There are multiple different types of joins:

* LEFT JOIN - Likely this will be the JOIN used most frequently. Returns all rows from the left table, and the matching rows from the right table. 

<p align="center">
  <img src="https://www.w3schools.com/sql/img_leftjoin.gif" alt="Venn Diagram - LEFT JOIN" width = "400" height="auto">
</p>

* RIGHT JOIN - Returns all rows from the right table, and the matching rows from the left table.

<p align="center">
  <img src="https://www.w3schools.com/sql/img_rightjoin.gif" alt="Venn Diagram - RIGHT JOIN" width = "400" height="auto">
</p>

* JOIN - Joins tables together

<p align="center">
  <img src="https://www.w3schools.com/sql/img_innerjoin.gif" alt="Venn Diagram - JOIN" width = "400" height="auto">
</p>

* FULL OUTER JOIN - Returns all rows when there is a match in either left table or right table.

<p align="center">
  <img src="https://www.w3schools.com/sql/img_fulljoin.gif" alt="Venn Diagram - FULL (OUTER) JOIN" width = "400" height="auto">
</p>

##### Notes: https://www.w3schools.com/sql/sql_join.asp.

You will use a different type of JOIN depending upon what you want to accomplish. To illustrate the differences, we will again divide the USArrests data into two parts. This time, however, we will keep the fields State and Murder in first30 and the fields State and Assault in last30. Because State is the common variable, that is variable on which we JOIN the datasets together.

> **BEST PRACTICE NOTE:**
> You can assign "nicknames" to a dataset and use that nickname to specify the source for each column. For example, "FROM first30 as a LEFT JOIN last30 as b" assigns the nicknames "a" to the first30 dataset and "b" to the last30 dataset. To specify a variable contained in first30 you would type "a.Murder." To specify a variable contained in last30 you would type "b.Assault."

```r
library(sqldf)
library(dplyr)
library(tibble)

us_arrests <- USArrests %>%
        rowname_to_column(value = 'State')

first30 <- us_arrests %>% select (State, Murder) %>% slice_head(n=30) %>% arrange(State)
last30 <- us_arrests %>% select (State, Assault) %>% slice_tail(n=30) %>% arrange(State)

# LEFT JOIN
  left_join <- sqldf("
    SELECT a.State
      , a.Murder
      , b.Assault
    FROM first30 as a LEFT JOIN last30 as b
    ON a.State = b.State
    -- 30 observations, 3 variables
    -- limited to records appearing in first30
    ")

  # States in both first30 and last30 have values for Assault. 
  # States only appearing in first30 have "NA" in the Assault column since those data were not available in last30.

# RIGHT JOIN
  right_join <- sqldf("
      SELECT b.State
        , a.Murder
        , b.Assault
      FROM first30 as a RIGHT JOIN last30 as b
      ON a.State = b.State
      -- 30 observations, 3 variables
      -- limited to records appearing in last30
      ")
  # States in both first30 and last30 have values for Assault. 
  # States only appearing in last30 have "NA" in the Murder column since those data were not available in last30.


# JOIN
join <- sqldf("
    SELECT a.State
      , a.Murder
      , b.Assault
    FROM first30 as a JOIN last30 as b
    ON a.State = b.State
    -- 10 observations, 3 variables
    -- contains records appearing in both first30 and last30
    ")

  # All observations contain values for Murder and Assault because all information is available for overlapping states.

# FULL JOIN
full_join <- sqldf("
    SELECT a.State
      , a.Murder
      , b.Assault
    FROM first30 as a FULL JOIN last30 as b
    ON a.State = b.State
    -- 50 observations, 3 variables
    -- contains records appearing in either first30 and last30, but combines records for overlapping states.
    ")

  # Overlapping states have values for Murder and Assault.
  # States only appearing in first30 have values for Murder but not Assault.
  # States only appearing in last30 have values for Assault but not Murder.

```
JOINS can have additional functionality by including either INNER or OUTER to JOIN along with a few other conditions. Illustrations of this can be found in the graphic below.

<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/9/9d/SQL_Joins.svg" alt="Venn Diagram of SQL Joins" width = "500" height="auto">
</p>


## Wrangle
#### Creating Subsets of Data
It is relatively common to analyze or view a subset of data rather than an entire dataset. This can be accomplished in SQL in several different ways. One common way is to include a WHERE keyword in the the SQL query. 

For example, lets assume you want to view arrest statistics for just one state, Virginia. This can be accomplished via the query:

```r
x <- USArrests %>%
  rownames_to_column(var = "State")

result <- sqldf("
    SELECT *
    FROM x 
    WHERE State = 'Virginia'
  ")
```

If you want to view more than one state you can write:

```r
x <- USArrests %>%
  rownames_to_column(var = "State")

result <- sqldf("
    SELECT *
    FROM x 
    WHERE State IN ('Virginia', 'Maryland', 'South Carolina')
  ")
```

Prior to selecting the data, the SQL query tests whether values of the variable "State" are equal to 'Virginia,' 'Maryland,' **or** 'South Carolina'. Any row that meets this criteria (the equality is TRUE rather than NOT TRUE) will then be processed as part of the SELECT statement. Use of the IN operator allows for code that is more simple to read and write and is equivalent to:

```r
WHERE State = 'Virginia' OR State = 'Maryland' OR State = 'South Carolina'
```

The use of the equality condition (i.e., State = 'Virginia' or State IN ('Virginia')) implies you know AND correctly type the contents of the State field.


> **BEST PRATICE NOTE:**
> Be careful with your WHERE statement. For example, if you type "Marilandd," the equality will always be NOT TRUE and your SQL query will not return values for Maryland.

If you are unfamiliar with the contents of the field (or your typing ability) you can use the LIKE operator. The below would select all states that have "North" in their name.

```r
WHERE State LIKE '%North%'

# which is equivalent to

WHERE State IN ('North Dakota', 'North Carolina')
```

> **BEST PRATICE NOTE:**
> The LIKE operator has some great benefits but it means the computer does the thinking rather than the programmer. Sometimes that has disasterous consequences. When you use the LIKE operator (or any other "wildcard" operator) assume the results are erroenous until you confirm the opposite.

Other useful logical operators (i.e., those where the result is TRUE or NOT TRUE) recognized by SQL include:

* AND - TRUE if all the conditions separated by AND is TRUE 
* BETWEEN - TRUE if the operand is within the range of comparisons  
* IN - TRUE if the operand is equal to one of a list of expressions  
* NOT - Displays a record if the condition(s) is NOT TRUE 
* OR - TRUE if any of the conditions separated by OR is TRUE 

##### Notes: https://www.w3schools.com/sql/sql_operators.asp

#### The CASE Expression

There are occasions where you will want to create a new variable where the contents are conditional on the contents of another variable. This can be accomplished using the CASE expression. The CASE expression goes through conditions and returns a value when the first condition is met (like an if-then-else statement). So, once a condition is true, it will stop reading and return the result. If no conditions are true, it returns the value in the ELSE clause.

If there is no ELSE part and no conditions are true, it returns NULL.

For example, you may want to create a variable that contains the census region for each state.

```r
us_arrests <- USArrests %>%
  rownames_to_column(var = "State")

df <- sqldf("
  SELECT * 
    , CASE
        WHEN state IN ('Hawaii', 'Alaska') THEN 'Pacific'
        WHEN state IN ('Washington', 'Oregon','California', 'Idaho', 'Montana', 'Wyoming', 'Nevada', 'Utah', 'Colorado', 'Arizona', 'New Mexico') THEN 'West'
        WHEN state IN ('North Dakota', 'South Dakota', 'Nebraska', 'Kansas', 'Missouri', 'Iowa', 'Minnesota', 'Wisconsin', 'Illinois', 'Indiana', 'Michigan', 'Ohio') THEN 'Midwest'
        WHEN state IN ('Maryland', 'Delaware', 'West Virginia', 'Virginia', 'North Carolina', 'South Carolina', 'Georgia', 'Kentucky', 'Florida', 'Tennessee', 'Mississippi', 'Alabama', 'Oklahoma', 'Arkansas', 'Louisiana', 'Texas') THEN 'South'
        WHEN state IN ('Maine', 'New Hampshire', 'Vermont', 'Massachusetts', 'Connecticut', 'Rhode Island', 'Pennsylvania', 'New Jersey', 'New York') THEN 'Northeast'
        ELSE 'Missing'
      END AS region
  FROM us_arrests
  ORDER BY region
    ")

print(df)
```

You can also use values you create to create aggregated summaries. For example, if you wanted to calculate the average number of murder arrrests in each state contained in a region:

```r
df <- sqldf("
  SELECT CASE
    WHEN state IN ('Hawaii', 'Alaska') THEN 'Pacific'
    WHEN state IN ('Washington', 'Oregon','California', 'Idaho', 'Montana', 'Wyoming', 'Nevada', 'Utah', 'Colorado', 'Arizona', 'New Mexico') THEN 'West'
    WHEN state IN ('North Dakota', 'South Dakota', 'Nebraska', 'Kansas', 'Missouri', 'Iowa', 'Minnesota', 'Wisconsin', 'Illinois', 'Indiana', 'Michigan', 'Ohio') THEN 'Midwest'
    WHEN state IN ('Maryland', 'Delaware', 'West Virginia', 'Virginia', 'North Carolina', 'South Carolina', 'Georgia', 'Kentucky', 'Florida', 'Tennessee', 'Mississippi', 'Alabama', 'Oklahoma', 'Arkansas', 'Louisiana', 'Texas') THEN 'South'
    WHEN state IN ('Maine', 'New Hampshire', 'Vermont', 'Massachusetts', 'Connecticut', 'Rhode Island', 'Pennsylvania', 'New Jersey', 'New York') THEN 'Northeast'
    ELSE 'Missing'
    END AS region
    , max(Murder) as max_murder
  FROM us_arrests
  GROUP BY region
  ORDER BY max_murder DESC
  -- 4 observations
  -- The South contains the state with the highest arrests for murder.
  ")
print(df)
```

Finally, you can use CASE..WHEN inside a function to create variables whose values are conditional:

```r
df <- sqldf("
  SELECT 'murders' as arrests
    , min(CASE WHEN state IN ('Washington', 'Oregon','California', 'Idaho', 'Montana', 'Wyoming', 'Nevada', 'Utah', 'Colorado', 'Arizona', 'New Mexico') THEN Murder ELSE NULL END) as min_west_murders
    , min(CASE WHEN state IN ('Maine', 'New Hampshire', 'Vermont', 'Massachusetts', 'Connecticut', 'Rhode Island', 'Pennsylvania', 'New Jersey', 'New York') THEN Murder ELSE NULL END) as min_northeast_murders
    , min(CASE WHEN state IN ('Maryland', 'Delaware', 'West Virginia', 'Virginia', 'North Carolina', 'South Carolina', 'Georgia', 'Kentucky', 'Florida', 'Tennessee', 'Mississippi', 'Alabama', 'Oklahoma', 'Arkansas', 'Louisiana', 'Texas') THEN Murder ELSE NULL END) as min_south_murders
    , min(CASE WHEN state IN ('Washington', 'Oregon','California', 'Idaho', 'Montana', 'Wyoming', 'Nevada', 'Utah', 'Colorado', 'Arizona', 'New Mexico') THEN Murder ELSE NULL END) as min_midwest_murders
    , min(CASE WHEN state IN ('Hawaii', 'Alaska') THEN Murder ELSE NULL END) as min_pacific_murders
  FROM us_arrests
  GROUP BY arrests
  -- 1 observation
")
```

## Analyze
#### Frequencies
Frequencies are commonly used to understand the contents of the data. These can be calculated in SQL using the GROUP BY keyword. The GROUP BY keyword indicates the level at which calculated variables should be aggregated. Frequencies are calculated using the "count" function. While you can name the resulting variable anything, it is best practice to name variables in a way that makes it easy for others to understand. 

A variable name is assigned using "as *new_variable_name*"

```r
x <- USArrests %>%
  rownames_to_column(var = "State")

freq_murder <- sqldf(
    "SELECT murder
      , count(murder) as count_murder
    FROM x
    GROUP BY murder
    -- 43 observations
    "
)
print(freq_murder)
```

By default, SQL will order the results in ascending order by the GROUP BY variable (e.g., murder). Sometimes, however, you will want the query results to be ordered differently. This can be managed using the ORDER BY keyword. The ORDER BY keyword must follow the GROUP BY keyword or your SQL query will result in an error. 

Adding an optional DESC after the ORDER BY variable will sort the results in descending order by that variable.

```r
x <- USArrests %>%
  rownames_to_column(var = "State")

freq_murder <- sqldf(
    "SELECT murder
      , count(murder) as count_murder
    FROM x
    GROUP BY murder
    ORDER BY murder DESC
    -- 43 observations
    "
)
print(freq_murder)
```

It is also possible to order the results using the variable that is calculated (e.g., count_murder). 

```r
x <- USArrests %>%
  rownames_to_column(var = "State")

freq_murder <- sqldf(
    "SELECT murder
      , count(murder) as count_murder
    FROM x
    GROUP BY murder
    ORDER BY count DESC
    -- 43 observations
    "
)
print(freq_murder)
```

#### Descriptive Statistics
SQL is a database query language and not a statistical package. That said, it is possible to use SQL to create some summary statistics like average, min, max, mode, and sum. 

```r
x <- USArrests %>%
  rownames_to_column(var = "State")

summary_murder <- sqldf(
    "SELECT count(murder) as count_murder
      , avg(murder) as mean_murder
      , min(murder) as min_murder
      , max(murder) as max_murder
      , mode(murder) as mode_murder
      , sum(murder) as sum_murder
    FROM x
    -- 1 observation
    "
    # Note that by not specifying GROUP BY, SQL will aggregate across the whole data frame
)
print(freq_murder)
```

## Communicate

SQL is a querying language. It does not, by itself, have graphics or visualization capabilities. For this reason, output of a SQL query is often “moved” to a different software package (e.g., ggplot2 in R, or Excel) to communicate the results.

That said, sometimes you will want to share the SQL code and the raw output. To do that, you'll want to make sure your program (or script) is saved to a file, comments are included in your code, and conclusions drawn are included as comments.

> **BEST PRACTICE:**
> When writing SQL code (or for any coding language) keep in mind the following best practices:
> -   Always save your SQL queries in a program or "script"
> -   Include comments throughout the script to explain what each individual query does
> -   After each query in a script include a comment for results and/or conclusions

## Other SQL Concepts (Optional)

The HAVING clause was added to SQL because the WHERE keyword cannot be used with aggregate functions.
HAVING - command is used instead of WHERE with aggregate functions
[[fill in]]


Source: https://www.w3schools.com/sql/sql_quickref.asp










