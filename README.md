# Linux Server

This project is a guide for how I configured a linux server to deploy my [Outdoor Catalog project](https://github.com/caseyr003/catalog.git/)

## Server Info

* IP Address: 34.205.141.166
* URL: http://ec2-34-205-141-166.compute-1.amazonaws.com/

## Built With

* [Bootstrap](http://getbootstrap.com/)
* [Flask](http://flask.pocoo.org/)
* [Python 2.7](https://www.python.org/)
* [SQLAlchemy](https://www.sqlalchemy.org/)

## Summary of Configurations

### Set up firewall

Install uncomplicated firewall
* `sudo apt-get install ufw`

Set default with
* `sudo ufw default deny incoming`
* `sudo ufw default allow outgoing`

Allow incoming ports
* `sudo ufw allow 2200/tcp`
* `sudo ufw allow 80/tcp`
* `sudo ufw allow 123/tcp`

Deny incoming ports
* `sudo ufw deny 22`

Enable firewall
* `sudo ufw enable`

### Make sure packages are upgraded

Check for updates then updgrade
* `sudo apt-get update`
* `sudo apt-get upgrade`

### Add new user

Create user for grader
* `sudo adduser grader`

Give grader sudo permissions
* add `grader` file to `/etc/sudoers.d/` folder
* add `grader ALL=(ALL:ALL) ALL` to `grader` file

### Add Key Login for grader

* Use `ssh-keygen` to generate key on local machine for grader
* Store public key in `/home/grader/.ssh/authorized_keys`

### Configure SSH

Configure `/etc/ssh/sshd_config` file
* Change port from 22 to 2200
* Disable user password login
* Change PermitRootLogin to no.

Restart
* `sudo service ssh restart`

### Set up Postgresql database

Install Postgresql
* `sudo apt-get install postgresql postgresql-contrib`

Configure
* Check `/etc/postgresql/9.1/main/pg_hba.conf` doesn't allow remote connections

Create database and user
* `sudo --user=postgres psql -c "CREATE DATABASE outdoorcatalog"`
* `sudo -u postgres createuser -P catalog`

Grant previledges to catalog on outdoorcatalog database
* `sudo --user=postgres psql -c "GRANT ALL ON DATABASE outdoorcatalog TO catalog"`

### Deploy Flask App

Install and enable mod_wsgi
* `sudo apt-get install libapache2-mod-wsgi python-dev`
* `sudo a2enmod wsgi`

Add project from github
* `sudo apt-get install git`
* `git clone https://github.com/caseyr003/catalog.git/` to `var/www/App`
* Rename folder from `catalog` to `App`
* Rename `project.py` file in App to `__init__.py`

Add Flask
* `sudo apt-get install python-pip`

Add virtual environment
* `sudo pip install virtualenv`
* `sudo virtualenv venv`
* `source venv/bin/activate`
* `sudo pip install Flask`
* `deactivate`

Configure virtual host and enable
* `sudo nano /etc/apache2/sites-available/App.conf`
* `sudo a2ensite App`
* run `sudo a2dissite <filename>` on all other `.conf` files

Configure .wsgi file
* create and configure `outdoor.wsgi` file in `/var/www/App`

Setup database
* `cd /var/www/App/App`
* `python database_setup.py`
* `python populate.py`

Restart
* `sudo service apache2 restart`

## JSON API

The JSON API allows retrieving data from the Outdoor Catalog database_setup

* `http://ec2-34-205-141-166.compute-1.amazonaws.com/category/JSON`
(returns a list of all categories available)
* `http://ec2-34-205-141-166.compute-1.amazonaws.com/category/<category_id>/list/JSON`
(returns a list of all items in the provided category id)
* `http://ec2-34-205-141-166.compute-1.amazonaws.com/category/<category_id>/list/<item_id>/JSON`
(returns a item for the provided item id and category id)

## Resources
[Digital Ocean](https://www.digitalocean.com)

[Digital Ocean Deploy Flask](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

## License

This project is licensed under the MIT License
