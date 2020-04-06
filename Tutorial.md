 

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

The open Function takes two parameter: the first parameter is the path to the file and the second parameter in which mode the file should be opend. There exists multiple mode but the most important are: 

* r: Opens the file in read-only mode. Starts reading from the beginning of the file and is the default mode for the open() function. 

* r+: Opens a file for reading and writing, placing the pointer at the beginning of the file. 

* w: Opens in write-only mode. The pointer is placed at the beginning of the file and this will overwrite any existing file with the same name. It will create a new file if one with the same name doesn't exist. 

* w+: Opens a file for writing and reading.  

* a: Opens a file for appending new information to it. The pointer is placed at the end of the file. A new file is created if one with the same name doesn't exist. 

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

After we have merged the forensic.log with the access.log in the previous step, we now want to enrich the ip address in the log with it's unique geo location. Thus, we need to lookup the geo location per ip address. We will use the library geoip2 for this task. The lookup could be done against an online resource - but for the sake of this tutorial, we will lookup against a local copy of the GeoIP database.

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

It is important to notice that the use of the .search() function for the pattern will return a Match Object. The matched string can be extracted with the .group() function.

This Code Snippet illustrates this:

```python 
import re
pattern = re.compile("your pattern")
ip = pattern.search("string").group(0)
``` 
The Parameter 0 simply specifies that the whole string should be returned as a result. It is possible to receive subgroups of the matched pattern by entering another number as a parameter but this is not required for this task.

Now write a function that extracts the IP address from each line and stores it in a variable. Additionally, all geo data should be stored in variables. Print the solution on the console to see if it worked. This function will be extended in the next steps. 

### Important:

A free database is used in this task. It is not 100% accurate and does not always find the corresponding country or city. However, the longitude and latitude are always found. If, the city or country is not found the value of the variable will be set to "None"

It is possible that for a certain IP no infromation is found in the database. (This happens for 2 IP Addresses in our log file). In this case, the GeoIP2 will throw an error. This snippet shows which error is thrown and how it is catched:

```python
from geoip2.errors import AddressNotFoundError
try :
    response = reader.city(ip)
except AddressNotFoundError:
  #do something
```

### Step 2:

#### Theory DNS-Look up

The function of Step 1 will now be extended by querying the associated DNS name for the IP address and saving in a variable.

To achieve this the socket library of Python can be used.  A DNS name to the IP address can be queried with the function "gethostbyaddr()". Note that if no DNS name is found, an exception is thrown. This exception must be caught and the value of the DNS variable must be set to None.

The following snippet illustrates this:

```python
import socket

 try:
     DNS = socket.gethostbyaddr(ip)[0]
  except socket.herror:
     DNS = None
```
The Indexing with [0] is used because the method returns a tuple with: Host Name, Alias list for the IP address if any, IP address of the host.

Extend the Function from step 1 with querying the DNS Name and print the result on the console to see if it works.

### Step 3

Now the normalized log file needs to be extended with the queried data. To achieve this as easy as possible, we use the in_place library. It is not possible to safely write to a file and read it at the same time. Any change you make could overwrite the content that has not yet been read. To do it safely the file must be read into a buffer, updating any lines as required, and then re-write the file.

The in_Place Libary will take care of the buffering and therefore makes the coding much easier. The following code illustrates how to use the library:

```python
import in_place

 with in_place.InPlace('../normalized.log') as file:
        for line in file:
        newline = line.rstrip()
        # append the queried data to the new line variable
        file.write(newline)
```
In the above code the file.write() will automatically overwrite the old line. Therefore it is best to save the old line, remove the line break with rstrip() and then extend the string with the data.

Since, it is possible that the country, city and DNS variable can be NONE check if they are not NONE and only append them to the newline variable if the check results in true. The format should be the following "|Counry: Switzerland|"  Illustrated by this snippet:

```python

     if country is not None:
       newline += ('|Country: ' + country)

```
A resulting Log entry in the normalized File should look like this:

XbuV-n8AAAEAABAxX@oAAABF 114.5.212.30 - - [01/Nov/2019:03:18:38 +0100] "GET /misc/js/hllogin-o.js HTTP/1.1" 200 817|GET /misc/js/hllogin-o.js HTTP/1.1|Host:www.hacking-lab.com|Connection:keep-alive|User-Agent:Mozilla/5.0 (Linux; U; Android 8.1.0; en-US; SM-G610F Build/M1AJQ) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/57.0.2987.108 UCBrowser/12.13.4.1214 Mobile Safari/537.36|Accept:*/*|Referer:https%3a//www.hacking-lab.com/user/login/|Accept-Encoding:gzip, deflate, br|Accept-Language:id,en-US;q=0.8|Cookie:HLSSL=3FpK/ZpvSNrqdjUvxDo861sOu5V5jEJT; JSESSIONID=4A2C41D73A7BA40ADC9D1E75B24D82E5|DNS:114-5-212-30.resources.indosat.com| Country: Indonesia|City: Jakarta|Latitude:-6.1741|Longitude:106.8296

Keep in mind that we have explained in Step 1 how to catch the AddressNotFound Exception from GeoIP2. If this exception occures write the fhe following sentence "|No Geo Data found" instead of the geo information. This is an Example for such an log entry:

XgbDCH8AAAEAAEzwAgoAAADF 151.217.218.42 - - [28/Dec/2019:03:50:48 +0100] "GET / HTTP/1.1" 302 222|GET / HTTP/1.1|Host:www.hacking-lab.com|User-Agent:Mozilla/5.0 (X11; Linux x86_64; rv%3a60.0) Gecko/20100101 Firefox/60.0|Accept:text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8|Accept-Language:en-US,en;q=0.5|Accept-Encoding:gzip, deflate, br|Connection:keep-alive|Upgrade-Insecure-Requests:1|No Geo Data found


In the normalized file there is again a large amount of data. It is best to copy some lines into a test file to see if the solution works. 

## Visualizing log files on Google Maps

### Theory KML-Files

In Google Maps or Google Earth data can be displayed with KML-Files. KML uses a tag-based structure with nested elements and attributes and is based on the XML standard.

This is an example of a KML-File with the information of one point:

```
<?xml version="1.0" encoding="UTF-8"?>
<kml xmlns="http://www.opengis.net/kml/2.2">
  <Placemark>
    <name>Simple placemark</name>
    <description>Attached to the ground. Intelligently places itself 
       at the height of the underlying terrain.</description>
    <Point>
      <coordinates>-122.0822035425683,37.42228990140251,0</coordinates>
    </Point>
  </Placemark>
</kml>
```

To create a KML file with Python, the library simplekml can be used. A point should contain the IP address as the name and a description with the contents of the log entry. For the coordinate data the latitude has always to be specified first and then the longitude.

This code snippet shows how simplekml is used to create a KML file:

```python
kml = simplekml.Kml() #creates the kml file
kml.newpoint(name=ip, coords=[(longitude, latitude)]).description = "Example"  #create a point with the ip as name and log entry as description
kml.save('../locations.kml') #save the kml file with the specified name

```

### Step 1

To solve the task, the following data must be extracted from the normalized.log file:

* IP address
* Longitude
* Latitude

For the IP Address the Regex Pattern from Step1 of the Log File Data Enrichment subtask can be used. If, the Log entry of the normalized.log has the format showed in Step 3 of the previous subtask, it is best to extract the latitude with a regex pattern but for the longitude simply the partition string function can be used.

Write a function that extracts the required data from the .normalized log File. Output the data to the console to see if everything worked.

### Step 2

Our KMl file should have the following information per point:
* IP address and how many times accessed as name
* Entire log entry as description

In the KML file each IP address should only appear once, but in addition it should be shown how many times it has been accessed from this IP address.

The string for the name of the point should look like this : "192.1.1.1 (127)" This means the IP 192.1.1.1 tried to access 127 times. The description should only contain the log entry of the first IP found.

This problem can be solved with dictionaries. A Dictionary contains Key/Value pairs. Each key is separated from its value by a colon (:), the items are separated by commas, and the whole thing is enclosed in curly braces. An empty dictionary without any items is written with just two curly braces, like this: {}.

Keys are unique within a dictionary while values may not be. The values of a dictionary can be of any type, but the keys must be of an immutable data type such as strings, numbers, or tuples.

Example:

```python
dict = {'Name': 'Simon', 'Age': 10}
print ("dict['']: ", dict['Name'])
print ("dict['Age']: ", dict['Age'])
```

which gives the following result:

```python
dict['Name']:  Simon
dict['Age']:  10
```
To solve our problem, we can use two dictionaries. In both dictionaries you store the IP address as a key. The first dictionary contains of: 
Key=IP-Adress / Value= Number of times the IP tried to acces and 
The second dictionary contains of: Key=IP-Address / Value=False. The value of the second dictionary is set to True when a point was created. This prevents the KML file from having duplicated IP addresses. 

Firstly, open the normalized.log file iterate over it and create the two dictionaries. Secondy, open the normalized.log file iterate over it and create the KML file. 

### Step 3

The created KML File can be uploaded in Google Maps in the following way:

Open your browser and navigate to Google Maps.Next, Click on the 3 dashes in the upper left corner (Menu). Select "your places" from the list. Under the point "your places" select "Maps". Click on the point "Create Map" at the very bottom. Then use the import point to upload your KML file.
