 

# Tutorial Log Aggregation with Python3 

  

## Introduction 

This challenge is about manipulating (log) files with Python3. Learn how to correlate, merge, enrich and visualize with Python3. You will get two files (see picture below)


### Learn how to ...

* merge and correlate the two given log files into a single file

* enrich the resulting log file by performing DNS lookups

* enrich the resulting log file by adding GEO location information (to ip address)

* visualize the resuluting log file on Google Maps.


## Preperation

### Step1
Please download the webserver-logs.zip from RESOURCES to /home/hacker/Downloads

LOG

```
╭─root@hlkali  /home/hacker/Downloads  
╰─$ ls -al 
total 3700
drwxr-xr-x  2 hacker hacker    4096 Mar 20 11:24 .
drwxr-xr-x 35 hacker hacker    4096 Mar 20 09:44 ..
-rw-r--r--  1 hacker hacker 3776821 Mar 20 11:24 webserver-logs.zip


╭─root@hlkali  /home/hacker/Downloads  
╰─$ md5sum webserver-logs.zip 
d479b45c92411a6853d1e83358e0b5bc  webserver-logs.zip


╭─root@hlkali  /home/hacker/Downloads  
╰─$ unzip webserver-logs.zip 
Archive:  webserver-logs.zip
   creating: webserver-logs/
  inflating: webserver-logs/access.log  
  inflating: webserver-logs/forensic.log  
╭─root@hlkali  /home/hacker/Downloads  


╰─$ ls -al webserver-logs
total 55448
drwxr-xr-x 2 root   root       4096 Mar 20  2020 .
drwxr-xr-x 3 hacker hacker     4096 Mar 20 11:24 ..
-rw-r--r-- 1 root   root   11062573 Mar  4 13:26 access.log
-rw-r--r-- 1 root   root   45706362 Mar  4 13:24 forensic.log

```

you should have your access.log and forensic.log available.

### Step2

Please have a closer look at both log files. May you want to see the last 5 log entries of both log files using the following command tail -5 access.log and tail -5 forensic.log

LOG

```
╭─root@hlkali  /home/hacker/Downloads  
╰─$ cd /home/hacker/Downloads/webserver-logs                                                                            1 ↵


╭─root@hlkali  /home/hacker/Downloads/webserver-logs  
╰─$ tail -5 access.log 
Xgq5@H8AAAEAAH743ucAAAAL 66.249.76.144 - - [31/Dec/2019:04:01:12 +0100] "GET /misc/css/tabs/ui.progressbar.css HTTP/1.1" 200 130
Xgq5@X8AAAEAAH743ugAAAAL 66.249.76.144 - - [31/Dec/2019:04:01:13 +0100] "GET /misc/css/tabs/ui.dialog.css HTTP/1.1" 200 447
Xgq5@X8AAAEAAH743ukAAAAL 66.249.76.144 - - [31/Dec/2019:04:01:13 +0100] "GET /misc/css/tabs/ui.core.css HTTP/1.1" 200 630
Xgq6KX8AAAEAAH743uoAAAAU 212.254.246.102 - - [31/Dec/2019:04:02:01 +0100] "GET /cron/vmcontrol.html?job=getList HTTP/1.1" 200 64
Xgq6KX8AAAEAAH91usgAAACS 212.254.246.103 - - [31/Dec/2019:04:02:01 +0100] "GET /cron/vmcontrol.html?job=getList HTTP/1.1" 200 64


╭─root@hlkali  /home/hacker/Downloads/webserver-logs  
╰─$ tail -5 forensic.log 
-Xgq5@X8AAAEAAH743ukAAAAL
+Xgq6KX8AAAEAAH743uoAAAAU|GET /cron/vmcontrol.html?job=getList HTTP/1.1|Accept-Encoding:identity|Host:www.hacking-lab.com|Connection:close|User-Agent:Python-urllib/2.7
-Xgq6KX8AAAEAAH743uoAAAAU
+Xgq6KX8AAAEAAH91usgAAACS|GET /cron/vmcontrol.html?job=getList HTTP/1.1|Accept-Encoding:identity|Host:www.hacking-lab.com|Connection:close|User-Agent:Python-urllib/2.7
-Xgq6KX8AAAEAAH91usgAAACS
```

in the access.log, every log entry is on one single line. In the forensic.log you will find log entries in between the forensicID. Such a log entry starts with the +<number> and closes with -<number>
 
### Step3

We have a pipenv python3 skeleton for you. please run the following commands (e.g. Hacking-Lab LiveCD) and setup your python3 environment.

```
mkdir -p /opt/git
cd /opt/git
git clone https://github.com/ibuetler/p3s-log-aggregation.git
cd /opt/git/p3s-log-aggregation
pipenv --python 3 sync
pipenv --python 3 shell
```

LOG

```
╭─root@hlkali  /home/hacker/Downloads  
╰─$ mkdir -p /opt/git                                            


╭─root@hlkali  /opt/git  
╰─$ cd /opt/git 


╭─root@hlkali  /opt/git  
╰─$ git clone https://github.com/ibuetler/p3s-log-aggregation.git
Cloning into 'p3s-log-aggregation'...
remote: Enumerating objects: 50, done.
remote: Counting objects: 100% (50/50), done.
remote: Compressing objects: 100% (47/47), done.
remote: Total 50 (delta 13), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (50/50), 14.77 KiB | 540.00 KiB/s, done.


╭─root@hlkali  /opt/git  
╰─$ cd p3s-log-aggregation 


╭─root@hlkali  /opt/git/p3s-log-aggregation  ‹master› 
╰─$ pipenv shell
Creating a virtualenv for this project…
Using /usr/bin/python3 (3.7.7) to create virtualenv…
⠋Already using interpreter /usr/bin/python3
Using base prefix '/usr'
New python executable in /root/.local/share/virtualenvs/p3s-log-aggregation-UEkZgZS6/bin/python3
Also creating executable in /root/.local/share/virtualenvs/p3s-log-aggregation-UEkZgZS6/bin/python
Installing setuptools, pkg_resources, pip, wheel...done.

Virtualenv location: /root/.local/share/virtualenvs/p3s-log-aggregation-UEkZgZS6
Creating a Pipfile for this project…
Spawning environment shell (/bin/bash). Use 'exit' to leave.
root@hlkali:/opt/git/p3s-log-aggregation# . /root/.local/share/virtualenvs/p3s-log-aggregation-UEkZgZS6/bin/activate
(p3s-log-aggregation-UEkZgZS6) root@hlkali:/opt/git/p3s-log-aggregation# 

```

you should now have your python3 environment ready for this exercise.

## Log File Normalization

### Step1

First, we need to merge and normalize the two given log files.

The access.log consists of the following information

* ID
* IP
* timestamp
* HTTP Method (POST or GET)
* URL

example: XbuUln8AAAEAABEHgLsAAACR 212.254.246.102 - - [01/Nov/2019:03:12:38 +0100] "POST /cron/vmcontrol.html?job=updateList HTTP/1.1" 200 -

The forensic.log is more comprehensive and complex. There are different formats for the POST and GET Method. The log format of a POST entry contains: ID, Method, Accept-Encoding, Content-Length, Host, Content-Type, Connection, User-Agent whereas the log format of a GET consists of ID, Method, Host, Connection, Upgrade-Insecure-Request, User-Agent, Accept, Referer, Accept-Encoding, Accept-Language.

It is importance to notice that even the structure of the GET Requests can be different but they all will start with the +forensicID of the entry and end with the -forensicID.

 

### Step 2 

The original forensic.log is not very easy to merge and correlate with the access.log. As the forensic.log log entries announce a new log entry with the +forensicID and closing the log entry with a -forensicID, it is preferred to convert the forensic.log into a new log where every log entry is on a single line. This is recommended, because the access.log contains one log entry per line.

A log entry in the forensic.log looks like this:

+XbuUln8AAAEAABEHgLsAAACR|POST /cron/vmcontrol.html?job=updateList HTTP/1.1|Accept-Encoding:identity|Content-Length:1899|Host:www.hacking-lab.com|Content-Type:application/x-www-form-urlencoded|Connection:close|User-Agent:Python-urllib/2.7
-XbuUln8AAAEAABEHgLsAAACR

It should look like this, after the first normalization:

XbuUln8AAAEAABEHgLsAAACR|POST /cron/vmcontrol.html?job=updateList HTTP/1.1|Accept-Encoding:identity|Content-Length:1899|Host:www.hacking-lab.com|Content-Type:application/x-www-form-urlencoded|Connection:close|User-Agent:Python-urllib/2.7

### Theory Python 3 File Manipulation
  
In Python3 it is best to open a file with the "with" keyword. It is used in exception handling to make the code cleaner and much more readable. It simplifies the management of common resources like file streams. There is for example no need to use the close function, because the "with" keyword will take care of.

The following code snippet opens a file with the keyword with and iterates over the lines:

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

The following code snipet shows how to nest multiple with statements (one file in write mode and one in read mode):

```python 

 with open('filename', 'w') as f1:
        with open('filename', 'r') as f2:
            for line in f2:
                  f1.write(line)

``` 
  

For more information, please consult the reading/writing manual:

* https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files

Hint: Please have a closer look at the following string manipulation library

* http://python-ds.com/python-3-string-methods


You will need this to delete the leading + from the forensicID and delete the line with the leading -forensicID.


### Step 3


Please merge the optimized forensic.log file with access.log file. Compare the forensicID's and if they match write both lines into a new normalized.log file.

Write first the line from the access.log file with separating it with the '|' chracter at the end. This will ensure that you have a common delimiter over the whole file which will make the next steps easier.

A resulting log entry should look like this:

XbuUln8AAAEAABEHgLsAAACR 212.254.246.102 - - [01/Nov/2019:03:12:38 +0100] "POST /cron/vmcontrol.html?job=updateList HTTP/1.1" 200 -|POST /cron/vmcontrol.html?job=updateList HTTP/1.1|Accept-Encoding:identity|Content-Length:1899|Host:www.hacking-lab.com|Content-Type:application/x-www-form-urlencoded|Connection:close|User-Agent:Python-urllib/2.7

Hint: The forensic.log and access.log have many different entries. Comparing a line with each line of another file will take a lot of time. Maybe copy two or three lines in a different file to see if your solution works.
  

## Log File Data Enrichment

### Step 1

fter we have merged the forensic.log with the access.log in the previous step, we now want to enrich the ip address in the log with it's unique geo location. Thus, we need to lookup the geo location per ip address. We will use the library geoip2 for this task. The lookup could be done against an online resource - but for the sake of this tutorial, we will lookup against a local copy of the GeoIP database.

* Country
* City
* latitude
* longitude
* DNS name
  

### Theory GeoIP Lookup

Assuming you have previously download the GeoLite database, you can load the database with the following code:

```python 
 import geoip2.database as database
 reader = database.Reader('GeoLite2-City.mmdb')
``` 

In the following documentation you will find all the information how to receive all the data you will need for this step:

https://geoip2.readthedocs.io/en/latest/

Hint: Extracting the IP you could use a regex pattern. You can create a pattern with the following snippet:

```python 
import re
pattern = re.compile("your pattern")
``` 

The Syntax for creating such a pattern can be found in this python documentation:
https://docs.python.org/3/library/re.html#re.findall

It is important to notice than when you will use the .search() function for your pattern you will receive a Match Object but you can extract the matched string with the .group() function.

This Code Snippet illustrates this:

```python 
import re
pattern = re.compile("your pattern")
m = pattern.search(line)
ip = m.group(0)
``` 
The Parameter 0 simply specifies that we want the whole string as a result. You could also specify to receive subgroups of the matched pattern by entering another number as a parameter but this is not required for our task.


### Step 2:

#### Theory DNS-Look up


  

## Visualizing log files on Google Maps

 

 
