
# Tutorial Log Aggregation with Python3

## Introduction
In this tutorial, you will be given two log files of users which tried to attempt to log in to a web site. Both log files have different information but one column in common. Our Goal is to merge this files in to one and find information about the IP Addresses, for example from which country the users tried to access. Finally, we want wo visualize everything in google maps.

### What you'll learn
* How to merge two files in to one with Python3 by a common indicator
* How to perform a DNS-Look up with Python3
* How to receive Geolocation Data about an IP Address with Python3
* How to create a KML File and display it on Google Maps with Python3

### What you'll need
* The following Log Files: acces.log, forensic.log

## Subtask 1: Log Normalization

In this Task our goal is to merge both log Files into one. 

The log file acces.log consists of the following data: ID, IP, Timestamp, HTTP Method(POST or GET).

The log file forensic.log is a bit more complex there is a different structure for a POST and GET Method. For example a POST Method consists of: ID, Method, Accept-Encoding, Content-Length, Host, Content-Type, Connection, User-Agent whereas a GET ususally consists of the following structure: ID, Method, Host, Connection, Upgrade-Insecure-Request, User-Agent, Accept, Referer, Accept-Encoding, Accept-Language.

It is importance to notice that even the structure of the GET Requests can be different but we they all will start with the ID of the entry.

### Step 1: Read the Files by its delimiter

We will start with reading the forensic.log file. If you investigate the file you will notice that every entry has the delimiter '|'






## Subtask 2: Log Enrichment

## Subtask 3: Display Location on Google Maps
