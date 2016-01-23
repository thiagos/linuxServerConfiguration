#Linux Server Configuration

In this project, an Ubuntu Virtual Machine was created, and the server was
configured from scratch in order to host a python web application, following
best practices on server security.

The steps below were executed on the server:

##Server security
- users *student* and *grader* created
- password login disabled, so that private key authentication is required
- sudo permission added to both users
- linux packages updated
- ssh port changed to 2200 instead of 22
- Uncomplicated Firewall (UFW) configured to allow connections only from
ports 2200 (SSH), 80 (HTTP) and 123 (NTP)
- local timezone configured to UTC
- disabled root login

##Web server configuration
- `apache` and `mod_wsgi` installed using `apt-get`, to make the server host
web applications

##Database configuration
- postgresql installed using apt-get, so that the web applications above
can use as their database
- user *catalog* created in linux
- user *catalog* (not a superuser) created in postgres - this will be our 
application db user
- added permission to create database to this user: `alter role catalog createdb`
- create database *catalogdb*: `sudo -u catalog createdb catalogdb`

##Application configuration
- package `git` installed, so that repository can be downloaded
- package `Flask` installed using `pip`, as it is the framework used on 
the Catalog application
- package `Flask-SQLAlchemy` installed using `pip`, as SQLAlchemy framework
is used for database interactions from the Flask application
- package `oauth2client` installed using `pip`, as it is required for 
google authentication

###Apache configuration
- edit `/etc/apache2/apache2.conf` to not serve json files, as they have important 
authentication informatino to google and facebook
- within /etc/apache2/sites-available, create file catalogApp.conf, indicating
the application URL, the location of the wsgi application, log config, etc
- enabled the new site created, and disabled the default 000-default.conf:
`sudo a2ensite catalogApp.conf`
`sudo a2dissite 000-default.conf`
- started apache: `sudo service apache2 start`

###External authentication configuration
- Logged in developers console for google and facebook, adding the amazon URL information
needed for application and redirect sites
- Made fb app public, so external logins can be properly executed

###Application DB configuration
- package *psycopg* installed, as the python adapter for postgresql
- edit file `database_setup.py` to make SQLAlchemy connect to postgresql,
using user *catalog* and database *catalogdb*:
`engine = create_engine('postgresql://catalog:<password>@localhost:5432/catalogdb')`
- execute `database_setup.py` so that tables are created properly

##Accessing the server
- The ubuntu server hosting the application ip address is **52.35.167.125**, and ssh service
is running on port **2200**. To login, execute: 
`ssh -i <private key> grader@52.35.167.125 -p 2200`

##Running the application
- access the URL http://ec2-52-35-167-125.us-west-2.compute.amazonaws.com/ , and all functionality
from the Restaurant Catalog app ([here!](https://github.com/thiagos/RestaurantCatalogApp)) is 
available.

##Issues faced
- The login page was not opening, due to the following error: 
`RuntimeError: the session is unavailable because no secret key was set. Set the 
secret_key on the application to something unique and secret.`

Even though this was being set as `app.secret_key = 'some key'`

After some research, the change that fixed the problem was to set it directly in the `app`
config attribute: `app.config['SECRET_KEY'] = 'some key'`

I could not find the exact same problem or explanation why the first line, which worked when
running the application locally, did not work here, maybe some apacke+wsgi scoping issue.

- My database_setup had a table called 'user', which is a reserved word in postgresql (reference
[here](http://stackoverflow.com/questions/22256124/cannot-create-a-database-table-named-user-in-postgresql)).

Probably `SQLAlchemy` is not using the table name in quotes, which would work, per the link above.

To solve the issue, I renamed the table as *catalog_user* instead, and recreated the schema. 

##Next steps

- To setup automatic upgrades, package `unattended-upgrades` ([here](https://wiki.debian.org/UnattendedUpgrades)) looks interesting.
- To protect against multiple failed logins, package `fail2ban` is the one.
- To monitor the application, go with `glances`! 