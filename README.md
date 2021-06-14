# Data-Modeling-with-Apache-Cassandra
## Introduction
A startup called Sparkify wants to analyze the data they've been collecting on songs and user activity on their new music streaming app. The analysis team is particularly interested in understanding what songs users are listening to. Currently, there is no easy way to query the data to generate the results, since the data reside in a directory of CSV files on user activity on the app.<br>
They'd like a data engineer to create an Apache Cassandra database which can create queries on song play data to answer the questions. 
## Project Datasets
Working with one dataset: event_data. The directory of CSV files partitioned by date. Here are examples of filepaths to two files in the dataset:
```
event_data/2018-11-08-events.csv
event_data/2018-11-09-events.csv
```
Below is an example of what an original single event data file, *2018-11-08-events.csv*, looks like.
```
{
    "artist": "Slipknot", 
    "auth": "Logged In", 
    "firstName": "Aiden", 
    "gender": "M", 
    "itemInSession": 0, 
    "lastName": "Ramirez", 
    "length": 192.57424, 
    "lvevel": "paid", 
    "location": "New York-Newark-Jersey City,NY-NJ-PA"
    "method": "PUT"    
    "page": "NextSong"
    "regisgration":1.54028E+12
    "sessionId":19
    "song":"Opinum Of The People"
    "status":200
    "ts":1.5416E+12
    "userId":20    
}
```
## ETL Pipeline for Pre-Processing the Files
Processing the original event data files to create the event_datafile_new.csv that will be used for creating Apache Casssandra tables

The event_datafile_new.csv contains the following columns:
```
artist
firstName
gender
itemInSession
lastName
length
level
location
sessionId
song
userId
```
## Data Modeling
With Apache Cassandra, the database tables are modelled on the requested queries.

1. Give me the artist, song title and song's length in the music app history that was heard during sessionId = 338, and itemInSession = 4
>> The combination of sessionId and itemInSession is unique, and we need results based on sessionId and itmeInSession, so these two columns could be the Composite Primary Key of the table.
Since itemInSession could be skewed to some specific value, it's better to use sessionId as Priamary key so the data could be evenly distributed on the nodes
We need the artist name, song title and song length based on the sessionId and itemInSession Based on the above requirements, we could design the data model as follows:
