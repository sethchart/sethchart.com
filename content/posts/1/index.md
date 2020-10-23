---
title: "Data Engineering or: How I Learned to Stop Worrying and Love the Database"
date: 2020-10-23
slug: ""
description: ""
keywords: [Data Science, Database]
draft: true
tags: [Data Science, Database]
math: false
toc: false
---

## The Problem
For my first project in the Flatiron data science bootcamp, I was asked to explore a data set with information about movies.
The goal was to clean and summarise the data.
You can see my git hub repository [here](https://github.com/sethchart/Movie_Market_Analysis).

We were provided with data organized as a folder full of spreadsheets that that had been scraped from several different online resources.
As I started work on the project I noticed that I was spending a huge amount of time cleaning data, combining data from multiple sources, cleaning the combined data, and whittling down the resulting data so that I could produce a particular summary.
Then, I would come up with another question about the data that I wanted to answer.
I would immediately realize that the data processing that I had just completed wasn't quite right for answering my new question and I would start over from the raw data and build up a new processed data set to answer my question.
After working through this cycle a few times I got fed up.
It was time to build some better tools.

I am currently a database manager for a non-profit organization so I like databases and using SQL to query them.
I decided that I would organize the provided spreadsheets into an sqlite database and use that as my primary data source.
Having worked with sqlite and python together earlier in the module I knew that there was a fair amount of repetitive code associated with accessing a sqlite database, so I knew that I also wanted to write some helper functions to simplify accessing the data.
Finally, I had recently been exposed to some object oriented programming in python and I wanted to try to encapsulate my helper functions into a class.
This was my first attempt at writing a class, so I had some thinking to do.

## Designing the Database
The first problem that I needed to solve was how to reorganize the data that I was given in spreadsheets and store it in a usable database.
Loading each spreadsheet into the database as a new table was not going to make my life easier, I needed to try to find the underlying structure that was hiding in the spreadsheets.

The first thing that I noticed was that I had several spreadsheets from IMDb, all of these spreadsheets had IMDb identifiers, which are unique strings that IMDb assigns to movies and people in the movie industry.
This was fantastic, I had primary keys!

### Missing Values

Next, I noticed that in each IMDb spreadsheet there were some columns that would be filled for every row while other columns were blank for many rows.
The columns with missing values described attributes that were not available or relevant for every row.
When we view data in a spreadsheet it is tempting to deal with missing values in one of two ways.
First, fill missing values with some value that seams reasonable.
Second, drop the rows of the sheet that have missing values.
The first option embeds our assumptions about the data.
The second option throws out data that does not comply with our current requirements.

The great thing about a relational database is that it provides a third option that resolves the issue of missing values without embedding our opinions or throwing out data.
We can store the data that is available for every row in one table and then store data that is only available for some rows in a second table, we can link the second table to the first using the primary key of the first table as a foreign key for the second table.
Once we have organized our data in this way we can access it in many different ways by querying the database with different types of joins.

That got a little heavy, let's work through a toy example to see what I am talking about.
Suppose that someone hands you a spreadsheet with information about customers for their website.
Everyone who uses the website must provide an email address, username and password.
Some users also provide a mailing address when they buy merchandise from the store.

| ID | Email | Username | Password | Mailing Address | 
|---|---|---|---|---|
| 1 | seth@mailinator.com | seth | !@#$123d | |
| 2 | salimah@mailinator.com | salimah | 4321!$@# | 1234 Faux Road, Atlantis SE 54321|
| 3 | nate@mailinator.com | nate | !@#$123d | |
| 4 | leslie@mailinator.com | leslie| !@#$123d | |
| 5 | tom@mailinator.com | tom | !@#$123d | 5432 Fake Street, Moon City SP 09786|

This spreadsheet can be recast as the two database tables below:

 * User Table

| ID | Email | Username | Password | 
|---|---|---|---|
| 1 | seth@mailinator.com | seth | !@#$123d |
| 2 | salimah@mailinator.com | salimah | 4321!$@# |
| 3 | nate@mailinator.com | nate | !@#$123d |
| 4 | leslie@mailinator.com | leslie| !@#$123d | |
| 5 | tom@mailinator.com | tom | !@#$123d | 

 * Mailing Address Table 

| ID | User ID | Mailing Address | 
|---|---|---|
| 1 | 2 | 1234 Faux Road, Atlantis SE 54321|
| 2 | 5 | 5432 Fake Street, Moon City SP 09786|

Notice that we have not dropped any data, we have not filled any missing values with our best guess, and there are no missing values in our tables.
If we wanted to recreate our original table, we could do an outer join on our two database tables.
If we wanted to drop rows for users without an address, we could do an inner join on our tables.

### Overloaded Fields

Another issue that I noticed with the provided data had to do with the genre of a movie. In one of the spreadsheets, each row contained information about a single movie and one of the cells contained a list of up to five genres that were relevant to the movie in question separated by commas. Any analysis that included genre would require preprocessing the genre field to extract the multiple datum contained within. 

There are several ways to tag data in a database with one or more relevant categories. For the purposes of a small scale data analysis I opted to one hot encode genres in a database table. This is most likely less efficient than other options, but for every application I had in mind one-hot encoding was the most useful representation of the data.

Most of our database design amounted to applying the observation from the example above to reorganize spreadsheets into smaller database tables without missing values. 

## Data Cleaning

In our data set there were several columns spread across our various spreadsheets that were not properly formatted. For example, most of the financial data was stored as strings with commas and dollar signs. For every application that we were interested in, it was more useful to store this data as number instead of a formatted string. 

When working with spreadsheets as the primary data source, it is possible to do all of the necessary data cleaning and then save the resulting data in a new spreadsheet. Unfortunately, I found myself dealing with missing values and merging anyway, so it hardly seemed worth creating new spreadsheet files to store the cleaned data is I would just need to manipulate it further before I could use it to answer a question. 

On the other hand, if my primary data sources is a well designed database, then missing values are no longer an issue and merging data is a matter of writing an appropriate database query.
In this case it is worthwhile to clean the data as it is loaded into the database so that my primary data source has data in the appropriate format and I don't need to repeat cleaning steps every time I access the data.
In addition, I can specify the data type for each column in my database so that my data requirements are enforced by the database.


## Loading the Data

Having resolved where and how to store the data, I needed to load the data from the provided spreadsheets into the database. For each spreadsheet I wrote a function that would read a row of the sheet, clean the contained data, and write it to the appropriate database tables. Loading the data was then a simple matter of iterating over the rows of each spreadsheet executing the appropriate function. 

## The Data API

Once the data was loaded into the database, I needed methods that would allow me to quickly access the stored data and use it as inputs for summary statistics and visualizations. These methods formed my MoviesDb class.
