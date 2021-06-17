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
> The combination of sessionId and itemInSession is unique, and we need results based on sessionId and itmeInSession, so these two columns could be the Composite Primary Key of the table.
> We need the artist name, song title and song length based on the sessionId and itemInSession Based on the above requirements, we could design the data model as follows:
```
Create table music_library
    (
        sessionId INT,
        itemInSession INT,
        artist_name TEXT,
        song_title TEXT,
        song_length FLOAT,
        PRIMARY KEY (sessionId, itemInSession)  
    )
  ```
2. Give me only the following: name of artist, song (sorted by itemInSession) and user (first and last name) for userid = 10, sessionid = 182
> Since we need results based on userid and sessionid, it could make good perfomance to use userid and sessionid as partition key to make sure the sessions belonging to the same user could be distributed to same node
> Since the data needs to be sorted by itemInSession, the itemInSession should be the clustering key
> Besides Primary Key, we need artist name, song title and user name for this query Based on the above requirements, we could design the data model as follows:
```
Create table artist_user_library
    (
         artist_name TEXT, 
         song_title TEXT, 
         itemInSession INT, 
         user_firstname TEXT,
         user_lastname TEXT,
         userId INT, 
         sessionId INT,
         PRIMARY KEY((userId, sessionId), itemInSession)) 
         WITH CLUSTERING ORDER BY (itemInSession DESC) # not necessary
     )
```
3. Give me every user name (first and last) in my music app history who listened to the song 'All Hands Against His Own'
> Since we need the results based on song name, firstName and lastName might be not unique, we need to use song name and userid as Primary Key
> Use the song name as partition key so the users data listened to the same song could be distributed on the same node, it will improve the query performance
> we need firatName and lastName as well for this query Based on the above requirements, we could design the data model as follows:
```
Create table user_song
    (
        song_title TEXT,
        userId INT,
        user_firstname TEXT,
        user_lastname TEXT,  
        PRIMARY KEY (song_title, userId)  
    )
```
## Build ETL Pipeline
1. Implement the logic in section Part I of the notebook template to iterate through each event file in `event_data` to process and create a new CSV file in Python
2. Make necessary edits to Part II of the notebook template to include Apache Cassandra `CREATE` and `INSERT` statements to load processed records into relevant tables in your data model
3. Test by running `SELECT` statements after running the queries on your database
