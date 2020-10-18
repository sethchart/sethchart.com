---
title: "Movie Market Analysis"
date: 2020-09-17T15:52:30-04:00
categories:
- Project
- DataScience
- Flatiron
draft: true
---
This blog post is an account of my first project in the Flatiron Data Science boot camp. I reserve the right to ramble.

## The Problem

Our hypothetical client was Microsoft. The premiss is that are launching a movie production company. They need to know what movies have historically performed well at the box office and what factors contribute to success. For this project we focused on data cleaning and exploratory data analysis. 

## Data Evaluation
Below are some of my impressions of the data as it was provided. Broadly, the data was clean and correct, but the formatting and organization of the data made difficult to work with. 

### Data Sources

The data was provided in a collection of spreadsheets. All of the data was collected from the publicly available data sources below. My best guess is that the provided data was collected in late 2018 or early 2019.  
 * [IMDb](https://www.imdb.com/interfaces/)
 * [Box Office Mojo](https://www.boxofficemojo.com/)
 * [Rotten Tomatoes](https://developer.fandango.com/rotten_tomatoes)
 * [The Movie Database](https://www.themoviedb.org/documentation/api)
 * [The Numbers](https://www.the-numbers.com/movies/report-builder)

Because data was stored in a collection of spreadsheets, relationships between variables stored in different sheets were not explicitly reflected in the data. To investigate data from multiple sheets directly from the provided data required loading data from multiple sources, cleaning each source data separately, identifying a strategy for combining data the sources into a single data frame or spreadsheet, and possibly cleaning the resulting combined data. The result was generally a data source that was usable to answer a single question, then the process needed to be repeated to load, clean, combine, and clean for the next question. This approach leads to lots of disposable code and a substantial barrier to analysis. 

### Data Quality

The provided data was essentially correct and consistently formatted. In general, the issues below applied across all of the provided data. 
 * Numerical data generally needed to be converted from strings to numerical data types. 
 * Genres were stored as a comma separated string in an overloaded column. 
 * Financial data needed to be corrected for inflation.

These issues are not unusual, but if we treat the provided spreadsheets as our primary data source, resolving these issues becomes a perennial part of the load, clean, combine, clean workflow described above. 

### Data Completeness

Generally each of the provided spreadsheets reported one or two variables for all or almost all rows and reported a few other variables where available. This means that most spreadsheets had some columns with very large numbers of missing values. Missing values cause issues. If they are allowed to persist, they require exception handling, otherwise they require explicit cleaning.

Again, using the provided sheets as our primary data source means that managing missing values is part of every single load clean combine clean cycle. 


## Data Engineering

From the discussion above, we notice that when we use the provided spreadsheets as a primary data source, every time we want to answer a new question about the data we end needing to perform a tedious load, clean, combine, clean workflow. We repeat processing related to cleaning and handling missing values and we need to constantly devise question specific combinations of the data. This is not an efficient way to work. But what is the alternative?

Saving cleaned data in new spreadsheets can eliminate the need to repeat precessing related to data cleaning. But we are still dealing with spreadsheets so we have not fixed the problems with recombining data for each question. Also this approach does not support preprocessing missing values since the approach to handling missing values is almost always dependant on the question that we are answering. So we can solve one out of three problems and end up creating lots and lots of disposable spreadsheet files that all differ slightly and we are now responsible for managing them. Great! Wonderful! This sucks!

Instead, we will load our data into a relational database. We can handle data cleaning tasks once and for all as we load the data. A relational database provides a powerful framework for making the relationships between variables explicit and powerful methods for combining data. Finally, if we structure our database carefully, we can eliminate missing values by splitting data across database tables when appropriate. This solves all three of our problems. Additionally, it will provide an opportunity to build reusable methods for interacting with the data.   

For this project we chose to use a sqlite3 database and encapsulate database interaction in the `moviesdb` python library . 
### The `MoviesDb` Class
The core functionality of the `moviesdb` library is encapsulated in the `MoviesDb` class. When we instantiate a `MoviesDb` object a connection and cursor are opened on the `movies.sqlite` database. We provide the following methods to facilitate interaction with the database. 
 
 * `close`
 * `list_tables`
 * `list_column_names`
 * `load_query_as_df`
 * `load_table_as_df`
 * `create_table`
 * `write_row_to_table`

## Exploring the Data

## Feature Engineering
