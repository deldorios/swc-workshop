# R for Spatial Data Analysis

Some of the material is repurposed from [the Data Carpentry lesson on Ecology](http://www.datacarpentry.org/R-ecology-lesson/01-intro-to-r.html), which is released under an open license.
Some is adapted from [Tracy K. Teal's lesson](http://tracykteal.github.io/r-novice-gapminder/) and [John Moreau's workshop at the Federal Reserve Board](https://github.com/JohnRMoreau/2016-04-07-FederalReserveBoard/wiki) under a CC-By-SA license.
The rest is original; [licensing information is available in the main repository](https://github.com/arthur-e/swc-workshop).

## Overview

This lesson is intended for people new to R that want to get a general introduction to R for spatial data analysis.
Like the [Data Carpentry lesson on R](http://www.datacarpentry.org/R-ecology-lesson/01-intro-to-r.html), it is not intended to be an introduction to the R language.
Rather, it introduces R as *an environment for data analysis;* it focuses on libraries that, once installed, can be used for general purpose data analysis (`dplyr` and `tidyr`).
In this lesson, I tie in spatial data that can be visualized with `ggplot2` using additional libraries (`sp` and `rgdal`).

### About the Data

In this workshop, we will be using a subset of [the Gapminder dataset](http://www.gapminder.org/).
This is world-wide statistical data collected and curated to allow for a "fact based worldview."
These data includes things like employment rates, birth rates, death rates, percent of GDP that's from agriculture and more.
There are currently 519 variables overall, each as a time series.

You can see some examples of Gapminder's visualizations [in this TED video](http://www.ted.com/talks/hans_rosling_shows_the_best_stats_you_ve_ever_seen).
In this workshop, we'll focus just on **life expectancy at birth** and **per-capita GDP** by country, continent, and total population.

## Introduction to R and RStudio

There are two tools we're going to be using for this part of the workshop.
One is the R programming language.
**A programming language contains keywords and a prescribed grammar for linking words together** in a way that makes sense, just like a natural human language.
Programming languages differ in that **they describe a task or a series of tasks we would like a computer to execute.**

The other tool we're using is called RStudio.
**RStudio is what we would call an interactive development environment, or IDE.**
It is *interactive* because we can issue it commands in the R language and the program responds with output.
It is a *development environment* because we can use its features to develop scripts or programs in R that may consist of multiple R commands.

------------------------------

## Review of R

Let's quickly review programming in R so everyone's on the same page before we dive into more advanced topics.

### Data Types

There are five (5) atomic data types in R.
We can use the `typeof()` to determine the type of a data value.

```r
# Double-precision (decimal numbers)
typeof(3.14159)

# Integers
typeof(5)
typeof(5L)

# Character strings
typeof("Hello, world!")

# Logical (Boolean)
typeof(TRUE)

# Complex numbers
typeof(2+3i)
```

### Machine Precision

When performing numerical calculations in a programming environment like R, particularly when working with important quantities like money, it's important to be aware of how the computer represents decimal numbers.
For various reasons we won't go into, computers typically only approximate decimal numbers when they're stored.
If we don't guard against this, we may see some surprising results.
Numbers that seem to be equal may actually have different underlying representations and therefore be different by a small margin of error (called machine numeric tolerance).

```r
0.1 == 0.3 / 3
```

**In general, don't use `==` to compare numbers unless they are integers. Instead, use the all `all.equal` function.**

```r
all.equal(0.1, 0.3 / 3)
```

### Variables and Assignment

```r
x <- 1/2
```

Variable names in R can contain any letters, numbers, the underscore, or the period character.
R is probably the only programming language that allows you to use the period or dot character in a variable name.
For this reason, some R experts advise against using such variable names.

```r
x2 <- 3/4
my.parameter <- 'something'
my_parameter <- 'something else'
```

Using the right assignment operator is important.
**We may be tempted to use the `=` form because it is shorter, however, there are problems with mixing these two assignment operators, so it's best to get in the habit of using the arrow form for variable assignment.**
Plus, RStudio makes it easier to type this operator with a keyboard shortcut: [Alt] + [-].

### Functions

```r
pow10 <- function (value) {
  10^value
}

pow10(1)

# Using the return() function
pow10 <- function (value) {
  return(10^value)
}

pow10(1)

# Chaining function application
pow10(log10(100))
```

------------------------------

# Getting Started with Data

Let's load in the Gapminder data.

```r
surveys <- read.csv('data/gapminder-FiveYearData.csv',
  stringsAsFactors = FALSE)
```

Typically, the first thing we want to do after reading in some data is to take a quick look at it to verify its integrity.

```r
head(surveys)
tail(surveys)
```

**The default data structure in R for tabular data is referred to as a data frame.**
It has rows and columns just like a spreadsheet or a table in a database.
A data frame is a collection of vectors of identical lengths.
Each vector represents a column, and each vector can be of a different data type (e.g., characters, integers, factors).

**In R, the columns of a data frame are represented as vectors.**
Vectors in R are a sequence of values with the same data type.
We can use the `c()` function to create new vectors.

```r
x1 <- c(1, 2, 3)
x2 <- c('a', 'b', 'c')
```

We can construct data frames as a collection of vectors using the `data.frame()` function.
Note that if we try to put unlike data types in a single vector, they get coerced to the most flexible data type.

```r
df <- data.frame(
  x1 = c(TRUE, FALSE, TRUE),
  x2 = c(1, 'red', 2))
```

A data structure in R that is very similar to vectors is the `list` class.
**Unlike vectors, lists can contain elements of varying data types and even varying classes.**

```r
list(99, TRUE 'balloons')
list(1:10, c(TRUE, FALSE))
```

A data frame is like a list in that it is a collection of multiple vectors, each with a different data type.
We can confirm this with the `str()` function.

```r
str(surveys)
```

We can ask R about data structures using the `class()` function.

```r
class(list())
class(surveys)
```

**R provides several ways to access the columns of a data frame.**
It's important to understand the differences between them and what they each return.

```r
class(surveys$year) # Returns a numeric vector
class(surveys['year']) # Returns a data frame
class(surveys[['year']]) # Returns a numeric vector
class(surveys[,4]) # Returns a numeric vector
```

**Challenge: Understanding Data Frames**

Based on the output of the `str()` function, answer the following questions.

1. How many rows and columns are there in the data frame?
2. How many species have been recorded in this data frame?

As you can see, many of the columns consist of integers, however, the columns `species_id` and `sex` are of a special class called `factor`.
Before we learn more about the `data.frame` class, let's talk about factors.

## Data Frames

**There are several questions we can answer about our data frames with built-in functions.**

```r
dim(surveys)
nrow(surveys)
ncol(surveys)
names(surveys)
rownames(surveys)
summary(surveys)
```

Most of these functions are "generic;" that is, they can be used on other types of objects besides `data.frame`.

## Sequences and Indexing

**Recall that in R, the colon character is a special function for creating sequences.**

```r
1:10
```

This is a special case of the more general `seq()` function.

```r
seq(1, 10)
seq(1, 10, by = 2)
```

**Integer sequences like these are useful for extracting data from our data frames.**
Our survey data frame has rows and columns (it has 2 dimensions), if we want to extract some specific data from it, we need to specify the "coordinates" we want from it.
Row numbers come first, followed by column numbers, **though when we provide only one number it is interpreted as denoting a column.**

```r
# First column of surveys
surveys[1]

# First element of first column
surveys[1, 1]

# First element of fifth column
surveys[1, 5]

# First three elements of the fifth column
surveys[1:3, 5]

# Third row
surveys[3, ]

# Fifth column
surveys[, 5]

# First six rows
surveys[1:6, ]
```

We can also use negative numbers to exclude parts of a data frame.

```r
# The first column removed
head(surveys[, -1])

# The second through fourth columns removed
head(surveys[, -2:-4])
```

Recall that to subset the data frame's entire columns we can use the column names.

```r
surveys['year']   # Result is a data frame
surveys[, 'year'] # Result is a vector
surveys$year      # Result is a vector
```

**Challenge**

The function `nrow()` on a `data.frame` returns the number of rows. Use it, in conjunction with `seq()` to create a new `data.frame` that includes every 100th row of the `surveys` data frame starting at row 100 (100, 200, 300, ...).

## Subsetting and Aggregating Data

The Gapminder data contain surveys of each country's per-capita GDP and life expectancy every five years.
Let's learn how to use R's advanced data manipulation and aggregation features to answer a few questions about the Gapminder data.

1. Which countries had a life expectancy lower than 30 years of age in 1952?
2. What was the mean per-capita GDP between 2002 and 2007 for each country?
3. What is the range in life expectancy for each continent?

**To answer these questions, we'll combine relatively simple tasks in R together, progressively building towards a more complex answer.**

### Subsetting Data Frames

To answer our first question, we need to learn about subsetting data frames.
Recall our comparison operators; we want to find those entries of the `lifeExp` column that are less than 30.

```r
surveys$lifeExp < 30
```

That's a lot of output!
In fact, there's one logical value for every row in the data frame.
This makes sense; we basically performed a calculation on the `lifeExp` column, comparing every value in that column to 30.
The result is a logical vector with `TRUE` wherever the `lifeExp` value is less than 30 and `FALSE` elsewhere.

**Thus, when we subset the rows of `gapminder` with this logical vector, we obtain only those rows that matched.**
Finally, we specified we just wanted the `country` column in this result.

```r
surveys[surveys$lifeExp < 30, 'country']
```

This is the best way to subset data frames when we're writing a reusable R script.

**Question:** Why does the conditional expression go in the first slot inside the brackets, before the comma?

**Challenge: Subsetting**

Subset the `surveys` data frame to just those entries between from years between 1990 and 2000, inclusive. **Bonus:** Combine the condition on `year` with the original condition on `lifeExp`, so that you return just the entries between 1990 and 2000 just where the life expectancy is less than 30 years.

## Analyzing Data with dplyr

**R packages** are basically sets of additional functions that let you do more stuff.
The functions we've been using so far, like str() or data.frame(), come built into R; packages give you access to more of them.
Before you use a package for the first time you need to install it on your machine, and then you should import it in every subsequent R session when you need it.
R packages can be installed using the `install.packages()` function.
Let's try to install the `dplyr` package, which provides advanced data querying and aggregation.

```r
install.packages('dplyr')
```

Now that we've installed the package, we can load it into our current R session with the `library()` function.

```r
library(dplyr)
```

### What is dplyr?

The package `dplyr` provides easy tools for the most common data manipulation tasks.
It is built to work directly with data frames.
As you might expect, `dplyr` is an improvement on the `plyr` package, which has been in use for some time but can be slow in some use cases.
`dplyr` addresses this by porting much of the computation to C++.
An additional feature is the ability to work directly with data stored in an external database.
The benefits of doing this are that the data can be managed natively in a relational database, queries can be conducted on that database, and only the results of the query returned.

This addresses a common problem with R in that all operations are conducted in memory and thus the amount of data you can work with is limited by available memory.
The database connections essentially remove that limitation in that you can have a database of many 100s of gigabytes, conduct queries on it directly, and pull back just what you need for analysis in R.

### Selecting and Filtering with dplyr

We're going to learn some of the most common `dplyr` functions: `select()`, `filter()`, `mutate()`, `group_by()`, and `summarize()`.

To select columns from a data frame, use `select()`.
The first argument to this function is the data frame (`surveys`), and the subsequent arguments are the columns to keep.

```r
output <- select(surveys, country, year, lifeExp)
head(output)
```

To choose rows, use `filter()`.

```r
filter(surveys, country == 'China')
```

**Challenge: Stepwise Function Application**

Combine the last two operations: We want to `filter()` the `surveys` data frame to just those rows from China and we also want to display only the `country`, `year`, and `lifeExp` columns.

### Pipes

How can we combining `select()` and `filter()` so as to do both at the same time?
There are three ways to do this, two of which we've already seen:

- Use intermediate steps; recall how we would save a subset of our data frame as a new, temporary data frame. This can clutter up our workspace with lots of objects.
- Nested functions; we saw this most recently with the `with()` function. This is handy, but can be difficult to read if too many functions are nested together.
- Pipes.

The last option, pipes, are a fairly recent addition to R.
Pipes let you take the output of one function and send it directly to the next, which is useful when you need to do many things to the same data set.
In R, the pipe operator, %>%, is available in the `magrittr` package, which is installed as part of `dplyr`.
**Let's get some practice using pipes.**

```r
surveys %>%
  filter(country == 'China') %>%
  select(year, lifeExp)
```

If we want to save the output of this **pipeline,** we can assign it to a new variable.

```r
china.surveys <- surveys %>%
  filter(country == 'China') %>%
  select(year, lifeExp)
```

**Why does one of the following example not work?**

```r
surveys %>%
  select(year, lifeExp) %>%
  filter(country == 'China')
```

**Challenge: Pipes**

Modify the previous pipeline to show only records from the following two countries:

- `Korea Dem. Rep.`
- `Korea Rep.`

Make sure the `country` column is displayed in the output!

### Mutating Data with dplyr

Frequently you'll want to create new columns based on the values in existing columns, for example to do unit conversions, or find the ratio of values in two columns.
For this we'll use `mutate()`.
**For instance, we might want to convert per-capita GDP to total GDP.**

```r
surveys %>%
  mutate(gdpTotal = gdpPercap * pop)
```

Woops.
This is a lot to look at.
Luckily, we can pipe the results into the `head()` function.

```r
surveys %>%
  mutate(gdpTotal = gdpPercap * pop) %>%
  head()

surveys %>%
  mutate(gdpTotal = gdpPercap * pop) %>%
  tail()
```

### Split-Apply-Combine with dplyr

`dplyr` introduces the **split, apply, combine** workflow to our skill set.
This workflow allows us to split apart a data frame based on the levels (or unique values) of one or more columns, apply a function to those subgroups, and combine the results together.

**To split apart a data frame, we need to introduce the `group_by()` function, which identifies the groups by which we'll split the data.**
`group_by()` is often used together with `summarize()`, which collapses each group into a single-row summary of that group.
`group_by()` takes as argument the column names that contain the categorical variables for which you want to calculate the summary statistics.

```r
surveys %>%
  group_by(year) %>%
  summarize(median(lifeExp))
```

**Here, we've first split apart the data by `year`, that is, into groups of entries with the same value in the `year` column. Then, we applied a function, the `median()` function, to each part's `lifeExp` column. Finally, we combined together the results of each `median(lifeExp)` calculation.**

Note that we can group by more than one variable at a time.

```r
surveys %>%
  group_by(continent, year) %>%
  summarize(median(lifeExp)) %>%
  View
```

**Challenge: Split-Apply-Combine**

Using `filter`, `group_by`, and `summarize`, answer our second question: **What was the mean per-capita GDP between 2002 and 2007 for each country?**

### More on dplyr

**When we use the `summarize()` function in `dplyr`, we can see that the output appears truncated; instead of running off the screen, we get just the first few rows and a count of how many remain to be seen.**
That's because `dplyr`, instead of a `data.frame`, has returned an instance of the `tbl_df` class.
This is a data structure that's very similar to a data frame; for our purposes the only difference is that it won't automatically show too many rows.
It also displays the data type for each column under its name.
If you want to display more data on the screen, you can add the `print()` function at the end with the argument `n` specifying the number of rows to display.

```r
surveys %>%
  group_by(continent, year) %>%
  summarize(median(lifeExp)) %>%
  print(n = 15)
```

------------------------------

## Checkpoint: Data Analysis in R

**Now you should be familiar with the following:**

* How to read in a CSV as an R `data.frame`
* Numeric sequences for indexing vectors and data frames
* Subsetting data frames
* The split-apply-combine workflow
* How to use `dplyr` and pipes in a data analysis workflow

Next, you'll see how to create plots of your data for checking assumptions, validating the data, and answering data analysis questions.

------------------------------

## Data Visualization in R

*Disclaimer: We will be using the functions in the ggplot2 package. R has powerful built-in plotting capabilities, but for this exercise, we will be using the ggplot2 package, which facilitates the creation of highly-informative plots of structured data.*

`ggplot2` **is a plotting package that makes it simple to create complex plots from data in a dataframe.**
It uses default settings, which help creating publication quality plots with a minimal amount of settings and tweaking.
The "gg" in `ggplot2` refers to the "grammar of graphics," which is a design philosophy for how to describe visualizations with computer code.

`ggplot` graphics are built step-by-step by adding new elements; in the `ggplot2` documentation, these elements include what are called "geometric objects," which are things like points, lines, and polygons that correspond to your data.
**We'll first use `ggplot2` to create a scatter plot of hindfoot length and weight across the surveys.**
To create this plot with `ggplot2` we need to:

1. Bind the plot to a specific data frame using the `data` argument;
2. Define aesthetics (`aes`) that map variables in the data to axes on the plot or to plotting size, shape, color, etc.;
3. Add geometric objects; the graphical representation of the data in the plot (points, lines, polygons). To add a geometric objector or `geom`, use the `+` operator.

```r
china.surveys <- surveys %>%
  filter(country == 'China') %>%
  select(year, lifeExp)

ggplot(data = china.surveys)
```

If this is all we instruct R to do, we see that the "Plots" tab in RStudio has appeared and a background has been rendered but nothing else.
Basically, `ggplot2` has set up a plotting environment but nothing more.

```r
ggplot(data = china.surveys, aes(x = year, y = lifeExp))
```

After we add the aesthetics, we can see that the `x` and `y` axes have been set up with the appropriate ranges and drawn on the plot along with a grid that defines the coordinate space they span.

```r
ggplot(data = china.surveys, aes(x = year, y = lifeExp)) +
  geom_line()
```

After adding a line layer, we can see the data plotted.

```r
ggplot(data = china.surveys, aes(x = year, y = lifeExp)) +
  geom_point() +
  geom_line()
```

The `+` in the `ggplot2` package is particularly useful because it allows us to modify existing `ggplot` objects.
**This means we can easily set up plot "templates" and conveniently explore different types of plots,** so the above plot can also be generated with code like this:

```r
base <- ggplot(data = china.surveys, aes(x = year, y = lifeExp))
base + geom_bar(stat = 'identity')
```

Some things to note:

- Anything you put in the `ggplot()` function can be seen by any `geom` layers that you add. i.e. these are universal plot settings. This includes the `x` and `y` axis you set up in `aes()`.
- You can also specify aesthetics for a given geom independently of the aesthetics defined globally in the `ggplot()` function.

```r
ggplot(data = surveys_complete) +
 geom_point(aes(x = weight, y = hindfoot_length))
```

## Conclusion and Summary

### Other Resources

* [Quick-R](http://www.statmethods.net/): "An easily accessible reference...for both current R users, and experienced users of other statistical packages...who would like to transition to R."
* [RStudio Cheat Sheets](https://www.rstudio.com/resources/cheatsheets/): A list of informational graphics that help remind you how to use some advanced features in RStudio.
* [The R Inferno](http://www.burns-stat.com/pages/Tutor/R_inferno.pdf): A humorous and informative guide to some of the more advanced features and confounding behavior.
* [Writing (Fast) Loops in R](http://faculty.washington.edu/kenrice/sisg/SISG-08-05.pdf)
* [dplyr Cheat Sheet](http://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf)
* A longer introduction to `plyr` and `dplyr` [using the same Gapminder data](http://stat545.com/block009_dplyr-intro.html).
* More on using `dplyr` [to analyze the Gapminder data](http://stat545.com/block010_dplyr-end-single-table.html).