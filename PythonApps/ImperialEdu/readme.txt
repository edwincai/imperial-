# Paul Institute Documentation #

author: Pauli
author email: paulnaokii@gmail.com


------------------------------------------------------------------------------------------------------------------------
date: 24 August 2016
version: 1.0

## Introduction ##

Paul.Institute is a marketing tool and an online store. It first collects as much user info as possible when it sells cool stuff 
such as 'music', and then reuse those user info to other marketing purposes.  This repository contains all source code of paul.institute, 
if you are a developer for this site, it is compulsory to read this document. This project is written in python3, css, jquery/javascript and sql. 
Key components and frameworks of
this project are:

1. django 1.8: webserver backend
2. mysql/sqlite3: user data storage
   note: mysql is for deployment and sqlite3 is for testing, you can s
3. uwsgi: webserver/wsgi
4. celery: task queue for various purposes
5. (optional) nginx: webserver to work with uwsgi.
   note: this is is installed but we do not have time to make it working due to time constraint. If you want nginx
   working as the default webserver then you have to change uwsgi settings in the file "uwsgi.ini" to work as the wsgi.
 
##Project Setup: ##

Main Module: PaulInstitute

1. this is the startup module of the project, the static folder contains all the static files for this project, when you
deploy this site to the production server, you need to run this at the shell at the project directory:

sudo python3 manage.py collectstatic

and enter 'y' for yes

2. the 'views' folder is the logic behind each http request

_init__.py & celery.py: contains celery setup code.
settings.py: settings for the project, constant etc.
notice: don't forget to let DEBUG=False when deploy the project to production server, otherwise it will expose access to the backend data
urls: contains top level url mapping from each http request to the logical view in the 'view' folder, or shortcut views of the django framework
      also, it will include url patterns from other apps.
wsgi.py: wsgi setup code. uwsgi will look into this when it is connected.


## Workflow ##

When a new user comes to the website at the root url '/'，we find out where he is coming from by using maxmind Geo Ip api. 
The ip to localtion mapping data is stored in the dir GeoDB under the project root dir. If the user is coming from certain 
countries, we redirect them to a page to register with their phone number if we have a virtual number in that country, 
otherwise redirect them to another page if we support email registration only. Hence, we have 2 separate workflows for these 
2 type of users, and we label their activity when they navigate through the site by using the url pattern /local/* for phone users, 
for instances: /local/register, local/login and local/activate/complete. And pattern /global/* is for email users.

1 Workflow for users who registered via phone number
Once user has been redirected to /local/register and give away their phone, we send them some thing
called 'passport id' via sms using virtual numbers. Virtual numbers are provided by Telerivet and
sms service by Nexmo. Then they will be redirected to an internal online shop page /local/members where they need 
to login using the 'passport id' and the last 4 digits of their phone number.

The whole phone registration workflow is done in a module called 'PhoneRegistration', where its user model is build from 
another module called 'PhoneNumberAsUserName'. You can check url pattern file see how each request is handled at PhoneRegistration/urls.py. 
In the root setup url pattern file url.py, an prefix of 'local' is added to indicate these are workflow for phone users.

2 Workflow for users who registered via email address
This workflow is very similar to the phone one, and they have the same url pattern. Once they registered, they need to login with their email 
and 'passport id' so that they can see the onlien store at /global/members. Email registration module is called 'registration', and its user 
model is defined at 'EmailAsUserName'. Please check its registration workflow at registration/backends/default/urls.py. In the root setup url 
pattern file url.py, an prefix of 'global' is added to indicate these are workflow for email users.

3 the default online store page is called '/members', where it will redirect users to either '/local/members' or '/global/members' depending on 
how they were registered.

4 The passport id is generated using two pre-defined word lists in English. Each passport id has 2 words with each word is selected from each of 
those 2 lists randomly. The first list only contains adjectives and the second contains only nouns. The total phrase length of the combination 
of those 2 words is in between 6 and 12 including the white space. You have to supply enough passport ids to the sql database so that it can be 
assigned to those newly registered users, otherwise those 2 registration workflows will stop working. 
You can use this method 'MySQLObjectTestCase.test_add_data' defined in the file '/RemoteUserDB/Test/MyDBTest.py' to generate a few k of those ids 
and upload to the database in one go, but it may take 5-10 mins.

Those keywords list are stored as csv files under project directory. Nouns are in 'Noun.csv' and adjective is in 'adjective.csv'. Also, there is a
list of reserved words(reserved.csv) for internal use, words generated from those 2 lists will filter through reserved words in the method 
'MySQLObjectTestCase.test_add_data'.

------------------------------------------------------------------------------------------------------------------------