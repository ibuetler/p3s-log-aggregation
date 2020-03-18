 

# Tutorial Log Aggregation with Python3 

  

## Introduction 

In this tutorial, you will be given two log files of users which tried to attempt to log in to a web site. Both log files have different information but one column in common. Our Goal is to merge this files in to one and find information about the IP Addresses, for example from which country the users tried to access. Finally, we want wo visualize everything in google maps. 

  

### What you'll learn 

* How to merge two log files in to one 

* How to perform a DNS-Look up with Python3 

* How to receive Geolocation Data about an IP Address with Python3 

* How to enrich a log file with data 

* How to create a KML File and display it on Google Maps with Python3 

  

### What you'll need 

* The following Log Files: acces.log, forensic.log 
* Geoip2 Database

  

## Subtask 1: Log Normalization 

  

In this Task our goal is to merge both log Files into one.  

  

The log file acces.log consists of the following data: ID, IP, Timestamp, HTTP Method (POST or GET). 

  

The log file forensic.log is a bit more complex there is a different structure for a POST and GET Method. For example a POST Method consists of: ID, Method, Accept-Encoding, Content-Length, Host, Content-Type, Connection, User-Agent whereas a GET usually consists of the following structure: ID, Method, Host, Connection, Upgrade-Insecure-Request, User-Agent, Accept, Referer, Accept-Encoding, Accept-Language. 

  

It is importance to notice that even the structure of the GET Requests can be different but they all will start with the ID of the entry and end with the ID. 

  

### Step 1: Optimize Forensic.log File 

  

The Forensic.log File is in its current state not very useful to merge with the acces.log. Since the forensic has always two lines starting with the +ID and then ending with a line with the -ID. I would recommend to create a new log file with writing only the lines which start with + and at the same time delete the leading + from the ID. This will make the merging much easier in the next steps. 

  

In Python 3 it is best to open a file with the "with" keyword. It is used in exception handling to make the code cleaner and much more readable. It simplifies the management of common resources like file streams. There is for Example no need to use the close function which the "with" will take care of. 

  

The following code snippet opens a file and iterates over the lines: 

  

```python 

with open("yourfile.txt", "w") as f: 

    for line in lines: 

            //do something 

``` 

The open Function takes two parameter: the first parameter is the path to the file and the second parameter in which mode the file should be opend. There exists the following modes: 

  

* r: Opens the file in read-only mode. Starts reading from the beginning of the file and is the default mode for the open() function. 

* rb: Opens the file as read-only in binary format and starts reading from the beginning of the file. While binary format can be used for different purposes, it is usually used when dealing with things like images, videos, etc. 

* r+: Opens a file for reading and writing, placing the pointer at the beginning of the file. 

* w: Opens in write-only mode. The pointer is placed at the beginning of the file and this will overwrite any existing file with the same name. It will create a new file if one with the same name doesn't exist. 

* wb: Opens a write-only file in binary mode. 

* w+: Opens a file for writing and reading. 

* wb+: Opens a file for writing and reading in binary mode. 

* a: Opens a file for appending new information to it. The pointer is placed at the end of the file. A new file is created if one with the same name doesn't exist. 

* ab: Opens a file for appending in binary mode. 

* a+: Opens a file for both appending and reading. 

* ab+: Opens a file for both appending and reading in binary mode. 

  

For further help here is the documentation of reading/writing files in python https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files 

  

Hint: In the documentation is also explained how to format a string which you will need to delete the + from the ID String 

  

Now use this knowledge to create a new optimized Forensic Logfile. 


### Step 2: Merging Files 

  

In the next step you should now merge the optimized forensic.log file with acces.log file. Compare the ID's and if they match write both lines in a new normalized.log file 

  

## Subtask 2: Log Enrichment 

In this subtask we want to receive the following information about the IP Adresses and enrich the log file with it:
* Country
* City
* latitude
* longitude
* DNS name
  

### Step 1: Receive Geo Location Data 

We will use the library geoip2 to receive all the geolocation data. Try to acces the database and save all information in variables.

you can load the database with the following code:

```python 
 import geoip2.database as database
 reader = database.Reader('GeoLite2-City.mmdb')
``` 

In the following documentation you will find all the information how to receive all the data you will need for this step:

https://geoip2.readthedocs.io/en/latest/

### Step 2: DNS look up

### Step 3: Enrich File

  

## Subtask 3: Display Location on Google Maps 

 

 