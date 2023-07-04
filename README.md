# 🐍 📄 Data Analysis using PySpark  

Data Analysis using PySpark is a comprehensive and powerful approach to extracting insights from large datasets. PySpark, the Python library for Apache Spark, enables data scientists and analysts to perform distributed computing and parallel processing on big data. With its robust set of tools and functions, PySpark allows users to manipulate, transform, and analyze data efficiently and at scale. Whether you are dealing with structured or unstructured data, PySpark provides a user-friendly interface to perform tasks such as data cleaning, aggregation, statistical analysis, machine learning, and more. 


This quick reference guide serves as a handy resource for beginners and experienced practitioners alike, offering a glimpse into the essential concepts, techniques, and functions involved in data analysis using PySpark.

First we will learn PySpark, the Perform Data Anlysis on Yahoo! Finance dataset with Spark.



#### Table of Contents

- [Common Patterns](#common-patterns)
    - [Importing Functions & Types](#importing-functions--types)
    - [Filtering](#filtering)
    - [Joins](#joins)
    - [Column Operations](#column-operations)
    - [Casting & Coalescing Null Values & Duplicates](#casting--coalescing-null-values--duplicates)
- [String Operations](#string-operations)
    - [String Filters](#string-filters)
    - [String Functions](#string-functions)
- [Number Operations](#number-operations)
- [Date & Timestamp Operations](#date--timestamp-operations)
- [Array Operations](#array-operations)
- [Aggregation Operations](#aggregation-operations)
- [Advanced Operations](#advanced-operations)
    - [Repartitioning](#repartitioning)
    - [UDFs (User Defined Functions](#udfs-user-defined-functions)

- [Analyzing Historical Stock Prices using PySpark](#analyzing-historical-stock-prices-using-pyspark)

If you can't find what you're looking for, check out the [PySpark Official Documentation](https://spark.apache.org/docs/latest/api/python/pyspark.sql.html) and add it here!

## Common Patterns

#### Importing Functions & Types

```python
# Easily reference these as F.my_function() and T.my_type() below
from pyspark.sql import functions as F, types as T
```

#### Filtering

```python
# Filter on equals condition
df = df.filter(df.is_adult == 'Y')

# Filter on >, <, >=, <= condition
df = df.filter(df.age > 25)

# Multiple conditions require parentheses around each condition
df = df.filter((df.age > 25) & (df.is_adult == 'Y'))

# Compare against a list of allowed values
df = df.filter(col('first_name').isin([3, 4, 7]))

# Sort results
df = df.orderBy(df.age.asc()))
df = df.orderBy(df.age.desc()))
```

#### Joins

```python
# Left join in another dataset
df = df.join(person_lookup_table, 'person_id', 'left')

# Match on different columns in left & right datasets
df = df.join(other_table, df.id == other_table.person_id, 'left')

# Match on multiple columns
df = df.join(other_table, ['first_name', 'last_name'], 'left')

# Useful for one-liner lookup code joins if you have a bunch
def lookup_and_replace(df1, df2, df1_key, df2_key, df2_value):
    return (
        df1
        .join(df2[[df2_key, df2_value]], df1[df1_key] == df2[df2_key], 'left')
        .withColumn(df1_key, F.coalesce(F.col(df2_value), F.col(df1_key)))
        .drop(df2_key)
        .drop(df2_value)
    )

df = lookup_and_replace(people, pay_codes, id, pay_code_id, pay_code_desc)
```

#### Column Operations

```python
# Add a new static column
df = df.withColumn('status', F.lit('PASS'))

# Construct a new dynamic column
df = df.withColumn('full_name', F.when(
    (df.fname.isNotNull() & df.lname.isNotNull()), F.concat(df.fname, df.lname)
).otherwise(F.lit('N/A'))

# Pick which columns to keep, optionally rename some
df = df.select(
    'name',
    'age',
    F.col('dob').alias('date_of_birth'),
)

# Remove columns
df = df.drop('mod_dt', 'mod_username')

# Rename a column
df = df.withColumnRenamed('dob', 'date_of_birth')

# Keep all the columns which also occur in another dataset
df = df.select(*(F.col(c) for c in df2.columns))

# Batch Rename/Clean Columns
for col in df.columns:
    df = df.withColumnRenamed(col, col.lower().replace(' ', '_').replace('-', '_'))
```

#### Casting & Coalescing Null Values & Duplicates

```python
# Cast a column to a different type
df = df.withColumn('price', df.price.cast(T.DoubleType()))

# Replace all nulls with a specific value
df = df.fillna({
    'first_name': 'Tom',
    'age': 0,
})

# Take the first value that is not null
df = df.withColumn('last_name', F.coalesce(df.last_name, df.surname, F.lit('N/A')))

# Drop duplicate rows in a dataset (distinct)
df = df.dropDuplicates()

# Drop duplicate rows, but consider only specific columns
df = df.dropDuplicates(['name', 'height'])

# Replace empty strings with null (leave out subset keyword arg to replace in all columns)
df = df.replace({"": None}, subset=["name"])

# Convert Python/PySpark/NumPy NaN operator to null
df = df.replace(float("nan"), None)
```

## String Operations

#### String Filters

```python
# Contains - col.contains(string)
df = df.filter(df.name.contains('o'))

# Starts With - col.startswith(string)
df = df.filter(df.name.startswith('Al'))

# Ends With - col.endswith(string)
df = df.filter(df.name.endswith('ice'))

# Is Null - col.isNull()
df = df.filter(df.is_adult.isNull())

# Is Not Null - col.isNotNull()
df = df.filter(df.first_name.isNotNull())

# Like - col.like(string_with_sql_wildcards)
df = df.filter(df.name.like('Al%'))

# Regex Like - col.rlike(regex)
df = df.filter(df.name.rlike('[A-Z]*ice$'))

# Is In List - col.isin(*cols)
df = df.filter(df.name.isin('Bob', 'Mike'))
```

#### String Functions

```python
# Substring - col.substr(startPos, length)
df = df.withColumn('short_id', df.id.substr(0, 10))

# Trim - F.trim(col)
df = df.withColumn('name', F.trim(df.name))

# Left Pad - F.lpad(col, len, pad)
# Right Pad - F.rpad(col, len, pad)
df = df.withColumn('id', F.lpad('id', 4, '0'))

# Left Trim - F.ltrim(col)
# Right Trim - F.rtrim(col)
df = df.withColumn('id', F.ltrim('id'))

# Concatenate - F.concat(*cols)
df = df.withColumn('full_name', F.concat('fname', F.lit(' '), 'lname'))

# Concatenate with Separator/Delimiter - F.concat_ws(delimiter, *cols)
df = df.withColumn('full_name', F.concat_ws('-', 'fname', 'lname'))

# Regex Replace - F.regexp_replace(str, pattern, replacement)[source]
df = df.withColumn('id', F.regexp_replace(id, '0F1(.*)', '1F1-$1'))

# Regex Extract - F.regexp_extract(str, pattern, idx)
df = df.withColumn('id', F.regexp_extract(id, '[0-9]*', 0))
```

## Number Operations

```python
# Round - F.round(col, scale=0)
df = df.withColumn('price', F.round('price', 0))

# Floor - F.floor(col)
df = df.withColumn('price', F.floor('price'))

# Ceiling - F.ceil(col)
df = df.withColumn('price', F.ceil('price'))

# Absolute Value - F.abs(col)
df = df.withColumn('price', F.abs('price'))

# X raised to power Y – F.pow(x, y)
df = df.withColumn('exponential_growth', F.pow('x', 'y'))

# Select smallest value out of multiple columns – F.least(*cols)
df = df.withColumn('least', F.least('subtotal', 'total'))

# Select largest value out of multiple columns – F.greatest(*cols)
df = df.withColumn('greatest', F.greatest('subtotal', 'total'))
```

## Date & Timestamp Operations

```python
# Convert a string of known format to a date (excludes time information)
df = df.withColumn('date_of_birth', F.to_date('date_of_birth', 'yyyy-MM-dd'))

# Convert a string of known format to a timestamp (includes time information)
df = df.withColumn('time_of_birth', F.to_timestamp('time_of_birth', 'yyyy-MM-dd HH:mm:ss'))

# Get year from date:       F.year(col)
# Get month from date:      F.month(col)
# Get day from date:        F.dayofmonth(col)
# Get hour from date:       F.hour(col)
# Get minute from date:     F.minute(col)
# Get second from date:     F.second(col)
df = df.filter(F.year('date_of_birth') == F.lit('2017'))

# Add & subtract days
df = df.withColumn('three_days_after', F.date_add('date_of_birth', 3))
df = df.withColumn('three_days_before', F.date_sub('date_of_birth', 3))

# Add & Subtract months
df = df.withColumn('next_month', F.add_month('date_of_birth', 1))

# Get number of days between two dates
df = df.withColumn('days_between', F.datediff('start', 'end'))

# Get number of months between two dates
df = df.withColumn('months_between', F.months_between('start', 'end'))

# Keep only rows where date_of_birth is between 2017-05-10 and 2018-07-21
df = df.filter(
    (F.col('date_of_birth') >= F.lit('2017-05-10')) &
    (F.col('date_of_birth') <= F.lit('2018-07-21'))
)
```

## Array Operations

```python
# Column Array - F.array(*cols)
df = df.withColumn('full_name', F.array('fname', 'lname'))

# Empty Array - F.array(*cols)
df = df.withColumn('empty_array_column', F.array([]))

# Array Size/Length – F.size(col)
df = df.withColumn('array_length', F.size('my_array'))

# Flatten Array – F.flatten(col)
df = df.withColumn('flattened', F.flatten('my_array'))

# Unique/Distinct Elements – F.array_distinct(col)
df = df.withColumn('unique_elements', F.array_distinct('my_array'))
```

## Aggregation Operations

```python
# Row Count:                F.count()
# Sum of Rows in Group:     F.sum(*cols)
# Mean of Rows in Group:    F.mean(*cols)
# Max of Rows in Group:     F.max(*cols)
# Min of Rows in Group:     F.min(*cols)
# First Row in Group:       F.alias(*cols)
df = df.groupBy('gender').agg(F.max('age').alias('max_age_by_gender'))

# Collect a Set of all Rows in Group:       F.collect_set(col)
# Collect a List of all Rows in Group:      F.collect_list(col)
df = df.groupBy('age').agg(F.collect_set('name').alias('person_names'))

# Just take the lastest row for each combination (Window Functions)
from pyspark.sql import Window as W

window = W.partitionBy("first_name", "last_name").orderBy(F.desc("date"))
df = df.withColumn("row_number", F.row_number().over(window))
df = df.filter(F.col("row_number") == 1)
df = df.drop("row_number")
```

## Advanced Operations

#### Repartitioning

```python
# Repartition – df.repartition(num_output_partitions)
df = df.repartition(1)
```

#### UDFs (User Defined Functions)

```python
# Multiply each row's age column by two
times_two_udf = F.udf(lambda x: x * 2)
df = df.withColumn('age', times_two_udf(df.age))

# Randomly choose a value to use as a row's name
import random

random_name_udf = F.udf(lambda: random.choice(['Bob', 'Tom', 'Amy', 'Jenna']))
df = df.withColumn('name', random_name_udf()) 

``` 



## Analyzing Historical Stock Prices using PySpark 

First, we import the necessary libraries. pyspark is the Python API for Apache Spark, a distributed computing framework. yfinance is a library for fetching historical stock price data. We also import specific functions from pyspark.sql.functions that we will use later for calculating mean, standard deviation, and correlation. 

```python
import pyspark
import yfinance as yf
from pyspark.sql.functions import mean, stddev, corr
``` 

Next, we set the variables ticker, start_date, and end_date to specify the stock ticker symbol (in this case, "AAPL" for Apple), the start date, and the end date for the historical stock price data we want to fetch. We then use the yf.download() function from the yfinance library to retrieve the stock price data within the specified date range. 

```python
ticker = "AAPL" 
start_date = "2010-01-01"
end_date = "2023-07-04"

stock_prices_data = yf.download(ticker, start=start_date, end=end_date) 
``` 


The following line saves the downloaded stock price data to a CSV file named "stock_prices_data.csv" in a folder called "Data".

```python
# Save file
stock_prices_data.to_csv("Data/stock_prices_data.csv")
``` 

The following displays the first 5 rows of the downloaded stock price data.

```python
stock_prices_data.head(5)
```

Then, we use spark.read.csv() to read the previously saved CSV file into a Spark DataFrame named stock_prices_df. We set the options "header" and "inferSchema" to true to indicate that the CSV file has a header row and that Spark should infer the column types automatically. 

```python 
stock_prices_df = spark.read\
                .option("header", "true")\
                .option("inferSchema", "true")\
                .csv("Data/stock_prices_data.csv")


```

In the next step, we calculate the mean, standard deviation, and correlation between the "Close" column (stock price) and the "Volume" column (trading volume) of the stock_prices_df DataFrame. We use the select() function along with the mean(), stddev(), and corr() functions from pyspark.sql.functions to perform these calculations. The .first()[0] retrieves the first row of the result as a single value. Finally, we print out the mean price, standard deviation price, and correlation between price and volume.

```python
# Calculate mean, standard deviation, and correlation
mean_price = stock_prices_df.select(mean("Close")).first()[0]
stddev_price = stock_prices_df.select(stddev("Close")).first()[0]
corr_price_volume = stock_prices_df.select(corr("Close", "Volume")).first()[0]

print("Mean Price:", mean_price)
print("Standard Deviation Price:", stddev_price)
print("Correlation between Price and Volume:", corr_price_volume)
``` 

And lastly, we extract the "Date" and "Close" columns from the stock_prices_df DataFrame and collect the data as a list of rows. We then create separate lists for dates and prices by iterating over the collected rows. Finally, we use matplotlib.pyplot (assumed to be imported earlier) to plot the daily closing prices over the years, with dates on the x-axis and prices on the y-axis. The plot is displayed using plt.show(). 

```python 
# Daily closing prices over the years
prices = stock_prices_df.select("Date", "Close").collect()
dates = [row.Date for row in prices]
prices = [row.Close for row in prices]

plt.plot(dates, prices)
plt.xlabel("Date")
plt.ylabel("Price")
plt.title("Daily Stock Prices")
plt.show()
```

**These are the fundamental steps in utilising Apache Spark to analyse stock price data. Of fact, many additional techniques and methodologies, such as time series analysis, regression analysis, and machine learning, can be utilised to extract insights from data.**

