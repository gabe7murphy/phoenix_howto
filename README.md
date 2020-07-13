# Open Event Data Alliance Phoenix Pipeline Install & Test

A step-by-step guide to running Pheonix for the purpose of updating the dictionaries for Actors, Agents, and Issues.

SAMPLE CHANGE

These instructions will set up the following tools:

* MongoDB: a popular database tool
* scraper: crawls RSS newsfeeds and stores the articles in a MongoDB database
* stanford_pipeline: reads sentences from the articles in MongoDB and builds sentence parse trees using the Stanford Core NLP library.
* phoenix_pipeline: reads the sentences and parse trees and uses the Petrarch 2 library to create coded events using the CAMEO codebook and the Actor,Agent, and Issues dictionaries.

## PreRequisites

These instructions were tested on Ubuntu 18.04.  You should have basic knowledge of Linux, Git, and Python.

Make sure you have at least X GB of disk space available.


### Install Git

```sudo apt-get install git```

Check:

```git --version```

### Install virtualenv

```pip install virtualenv```

Check:

```virtualenv --version```

### Install python2
Comes with Ubuntu.

Check:

```python2 --version```

### Make a project dir
Create a directory for this project:

```mkdir oeda```

### Notes on Forked Repos

The Git repos listed here were originally forked from the OEDA Github account here:
https://github.com/openeventdata

## Install MongoDB

Familiarize yourself with the purpose and operation of the MongoDB database:

https://www.mongodb.com/

Follow these instructions for installing MongoDB on Ubuntu:

https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/

Check: run the Mongo command-line interface to make sure it's installed

```mongo```  to enter the MongoDB CLI.

```quit()``` to exit MongoDB.


### MongoDB User Interface: Robo 3T
Install the "Robo 3T" user-interface for MongoDB:

Visit: ```https://www.robomongo.org/download``` and download the 
Robo 3T tool only. 
*Skip the Studio 3T download.*

Copy the tar.gz file to your home directory and unzip it.

After unzipping, find the file `robo3t` in the bin directory and launch it. E.g.

```~/robo3t-1.3.1-linux-x86_64-7419c406/bin/robo3t```

In the MongoDB Connections panel use the Create link to make a new database connection. 

In the Connection tab Name field enter the name "Event Pipeline"

Keep all the other connection defaults and click Save.

Check:  Select the Event Pipeline connection to make sure you can connect to your new empty MongoDB.

## Install & Test Scraper

Scraper crawls RSS feeds to gather news content and store it in a MongoDB instace. View the scraper README on github: https://github.com/dgmurphy/scraper

### Clone the Scraper Repository

Change to the oeda directory:

```cd oeda```

In the oeda directory clone the forked version of the scraper repo:

```git clone https://github.com/dgmurphy/scraper.git```

```cd scraper```

### Create Python Environment & Install libraries

Create a Python2 virtual environment:

```virtualenv -p /usr/bin/python2.7 venv```

Activate the virtual environment:

```source venv/bin/activate```

Check: make sure your linux command prompt starts with '(venv)'

Install the setuptools library (this step required for nltk install)

```pip install setuptools==9.1```

Unzip the nltk library file:

```tar -xzvf nltk-2.0.4.tgz```

Install the nltk 2.0.4 library:

```pip install ./nltk-2.0.4```

*Q: Why not use the requirments.txt file to install the nltk library?*

*A: This is an old library version that requires a manual installation.*

Install the other required libraries for the scraper project:

```pip install -r requirements.txt```

### Do a Test Run of Scraper

Verify that the file `default_config.ini` has this line which will specify a short list of URLs to scrape for test purposes:

```file = whitelist_urls_short.csv```

Run the scraper:

`python scraper.py`

When it completes, inspect the log to verify that it added entries:

```vim scraper_log.log```

You should see a number of "added entry" lines with news article urls.

### View the Scraped Stories in MongoDB

Open the Robo 3T application and from the Connection dialog select the Event Pipeline connection.

Check: Robo 3T should connect and you should see a database called "event_scrape"

Expand the Collections folder under the Event Scrape item. You should see a collection called "stories".

Double-click the stories collection to view the entries in the database. 

Check: Expand some of the stories to browse them. Right click a story item and select View Document so see the complete item content. You should see the news article content here. Notice the field called "stanford" has value zero becuase no parse trees have been created for this story.

### Deactivate

Before proceeding, deactivate your scraper virtual environment:

```deactivate```

Check: makes sure your command prompt no longer start with (venv)


## Install and Test the Stanford Pipeline

This tool parses sentences into parse trees (sentence diagrams). This step is crucial for allowing the later parts of the event coding pipeline to work.
Read about the stanford_pipeline tool here: 
https://github.com/dgmurphy/stanford_pipeline

### Clone the Repo

In the oeda directory:

```git clone https://github.com/dgmurphy/stanford_pipeline.git```

```cd stanford_pipeline```

### Download the Core NLP Models

```
wget http://nlp.stanford.edu/software/stanford-corenlp-full-2014-06-16.zip
unzip stanford-corenlp-full-2014-06-16.zip
mv stanford-corenlp-full-2014-06-16 stanford-corenlp
cd stanford-corenlp
wget http://nlp.stanford.edu/software/stanford-srparser-2014-07-01-models.jar
```

In the file default_config.ini edit the line:

`stanford_dir = ~/stanford-corenlp`

to use the full path to the stanford-corenlp dir, e.g.:

`stanford_dir = /home/dmurphy/dev/oeda/stanford_pipeline/stanford-corenlp`

### Create Python Environment & Install libraries

In the stanford_pipeline directory create a Python2 virtual environment:

```virtualenv -p /usr/bin/python2.7 venv```

Activate the virtual environment:

```source venv/bin/activate```

Check: make sure your linux command prompt starts with '(venv)'

### Install Python Libraries

```pip install -r requirements.txt```


### Execute a Test Run

```python process.py```

Up to a minute of [Errno 111] Connection refused messages are normal during startup.

Check: The command line should show "processing story" messages.

### View Stories in MongoDB

Open the Event Pipeline connection in Robo 3T and view a document in the story collection. Verify that the field 'stanford' = 1 and that sentence parse trees are visible.

Example of a parse tree:

```
(ROOT (S (NP (NP (NNP Yangon) (NNP Region) (NNP Hluttaw))
(PRN (-LRB- -LRB-) (NP (NNP Parliament)) (-RRB- -RRB-))) 
(NP (NNP Speaker) (NN Tin) (NNP Maung) (NNP Tun)) 
(VP (VBD announced) (SBAR (IN that) (S (NP (NP (DT an) 
(NN emergency) (NN meeting)) (PP (IN of) (NP (DT the) 
(JJ regional) (NN parliament)))) (VP (MD will) (VP (VB be) 
(VP (VBN held) (PP (IN on) (NP (NNP June) (CD 18))))))))) (. .)))"
```

### Deactivate

Before proceeding, deactivate your scraper virtual environment:

```deactivate```


## Install and Test the Phoenix Pipeline

The Phoenix Pipeline takes the sentence content and the parse trees and produces events using the Petrarch event coder. Petrach includes a number of data dictionaries including the CAMEO event codes, Actor, Agent and Issues dictionaries, and a Discards list. Read more about Petrarch here:

https://petrarch.readthedocs.io/

Read more about the Phoenix Pipeline here:

https://github.com/dgmurphy/phoenix_pipeline

Note: this version of the pipeline has Geocoding disabled. Events will not be location coded but the source sentence will be saved with each event so a seperate process can be used to perform sentence-level geocoding.


## Clone the Repo

In the oeda directory:

```git clone https://github.com/dgmurphy/phoenix_pipeline.git```

### Create Python Environment & Install libraries

In the phoenix_pipeline directory perform the usual steps to create & activate the virtual environment, then pip install -r requirements.txt.


### Edit the Config File

The pipeline will process events on a per-day basis by checking the date fields of the stories in MongoDB.
To process events for a particular day we need to specify that day in the config file.


In the file `PHOX_config.ini` :

`run_date = 20200713`

NOTE: To process events on the same day they were collected, set the run_date to today's date plus one day.

### Do a Test Run of the Pipeline

```python pipeline.py```




