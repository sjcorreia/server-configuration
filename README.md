# Linux Server Configuration Project

This repository contains my version of the Linux Server Configuration Project as part of the Udacity Full Stack Nanodegree Program. This project is intended to demonstrate the concepts learned during part five of the nanodegree program, 'Deploying to Linux Servers'.

This project has tasked student with deploying a Flask web application to the web by configuring a Linux system as our host server. The main goal of this project is to gain understanding of user management techniques and basic security practices while setting up a Linux system to host a web application.

As recommended, I have created a new Linux instance with [AWS Lightsail](https://aws.amazon.com/lightsail/), runnung Ubuntu 16.04 LTS. This instance provides a clean foundation for this project. My instance has the following public IP Address: `18.221.163.29`. If you visit the link [http://18.221.163.29/](http://18.221.163.29/) in your favorite browser, you can see the Item Catalog Application created for Project 3 of the Full Stack Nanodegree progam **while this instance is active**.

## Project Guidelines

The requirements for this project were comprised of a few steps needed to proper set up the server, install additional required tools, and configure the Apache web server to host a web application.

### Server Configuration and Security

As part of the requirements for this project, and to allow the Udacity graders to review the project itself, a new user `grader` was created and given `sudo` access. The grader can access the server via `ssh` from a terminal of her/his choice. Key-based authentication using a key created with the `ssh-keygen` tool and configured for the user `grader`. The private key will be provided in a different step of the project review process.

In order to get a foundational understanding of server security, a few changes were made to the default `ssh` settings and the `ufw` tool, or Uncomplicated Firewall, was enabled. The first changes were to change the port for `ssh` to 2200. The default port for `ssh` is 22, but it is important to know how to make these changes as a means of increasing the server sercurity. In the AWS Lightsail settings for the instance, the port 2200 was added for TCP connections. The next step was to modify the configuration settings in the file `/etc/ssh/sshd_config`. The port number was changed to `Port 2200` and the Authentication setting for root login was set to `PermitRootLogin no`.

The tool `ufw` was then configured to allow connections to `ssh`, `http`, and `ntp` via ports 2200, 80, and 123, respectively. The following commands accomplished those configurations.

    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw allow ssh
    sudo ufw allow 2200
    sudo ufw allow www
    sudo ufw allow ntp
    sudo ufw deny 22
    sudo ufw enable

After all of these steps, the `grader` will be able to log in to the server via the terminal using the command

    ssh -i grader_key.pem grader@18.221.163.29 -p 2200

As mentioned above, the private key will be provided at another step in the submission process.

### Web Server Installation and Set Up

After running the commands `sudo apt-get update` and `sudo apt-get upgrade` to update all packages, the packages for an Apache HTTP web server and the Python 3 mod_wsgi package were installed before configuring the server.

    sudo apt-get install apache2
    sudo apt-get install libapache2-mod-wsgi-py3

Enable mod_wsgi using the command `sudo a2enmod wsgi`

The next step was to install `git` and clone the repositiory for the [Catalog Application](https://github.com/sjcorreia/catalog-project) created earlier in this nanodegree program. The web server needs to host a web application and this is a good example of a site with CRUD functionality to demonstrate several topics covered throughout this program. The destination repository for the web application is in the `/var/www/` directory, according to the set up directions for an Apache server.

The Apache web server needs to have a virtual host file for the configuration of each site being hosted. The directory for these files is located in `/etc/apache2/sites-available`. In this directory, I needed to create a virtual host file for the Catalog app, which I created using the command `sudo touch /etc/apache2/sites-available/catalogApp.conf`. The contents of this file were modified from the `000-default.conf` file and a tutorial mentioned below. After the virtual host file was configured correctly, the following command enables the virtual host.

    sudo a2ensite catalogApp

A WSGI Application Script file is used to start the application on the Apache server, so a `catalogApp.wsgi` file is created in the directory `/var/www/catalog-project/vagrant/catalog`, the same directory containing the Catalog application from GitHub. The contents of `catalogApp.wsgi` have been modified from a [tutorial](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps). Since mod_wsgi requires that the WSGI application entry point be called ‘application’, a few changes were made to the Catalog application files to work properly in this environment, such as the main Flask application moving from `application.py` to `__init__.py`.

There is a new requirement to host the Item Catalog app and connect it to a PostgreSQL database. After installing the needed packages for PostgreSQL, a user `catalog` and password `udacity` were created to access the database. SQLAlchemy uses the form `dialect+driver://username:password@host:port/database` to access databases. Adding the username and password, along with the database `catalog_db` created for this project, SQLAlchemy can access the database by creating its engine with the use of the database URL `postgresql+psycopg2://catalog:udacity@localhost/catalog_db`. The Item Catalog app required a few modifications to be able to access the `catalog_db` using this database URL. A few other changes were made to the Catalog application in order to get it working properly with PostgreSQL, but the functionalty of the application remains the same. The use of authentication with Google was removed due to errors with the host name.

Once the virtual host file and WSGI application script have been properly set up, it is necessary to restart the Apache server to apply the changes. The server can be restarted using the command

    sudo service apache2 restart

While setting up the server, it was useful to check the contents of the error log and fix any issues in the application. The path for the error log file (as defined in the virtual host file) is `/var/log/apache2/error.log` and can be viewed using the `cat` or `more` command, if the user has the proper permissions.

### Installed Packages

Several packages have been installed to the Linux machine to provide a host for the web application and properly set up the environment needed. These packages include:

- `finger`
- `postgresql` and `postgresql-contrib`
- `python3-dev`
- `apache2`
- `git`
- `libapache2-mod-wsgi-py3`
- `python-sqlalchemy`
- `python3-sqlalchemy`
- `python3-pip`

Additional Python 3 packages/modules were installed to set up the environment needed by the Python Flask application. Those packages include:

- `virtualenv`
- `flask`
- `passlib`
- `sqlalchemy`
- `flask-sqlalchemy`
- `psycopg2` and `psycopg2-binary`
- `itsdangerous`

## Resources Used

- [AWS Lightsail](https://aws.amazon.com/lightsail/)
- [Connect to Lightsail via terminal](https://stackoverflow.com/questions/46028907/how-do-i-connect-to-a-new-amazon-lightsail-instance-from-my-mac)
- [Add a new user to Lightsail](https://aws.amazon.com/premiumsupport/knowledge-center/new-user-accounts-linux-instance/)
- Digital Ocean Tutorial [Installing Apache web serber on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-16-04)
- [Create a new user in postgresql](https://www.postgresql.org/docs/9.5/static/app-createuser.html)
- Digital Ocean Tutorial [Postgresql - Deny Remote connections](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
- [pip install error](https://stackoverflow.com/questions/36394101/pip-install-locale-error-unsupported-locale-setting)
- Digital Ocean Tutorial [How To Deploy a Flask App on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- Digital Ocean Tutorial [How To Configure the Apache Web Server on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps)
- Digital Ocean Tutorial [How To Secure PostgreSQL on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
- [mod_wsgi Quick Configuration Guide](http://modwsgi.readthedocs.io/en/develop/user-guides/quick-configuration-guide.html)
- [Flask Help: Installing mod_wsgi](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/#installing-mod-wsgi)
- [Generating your own ssh key and importing it to AWS](https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ec2-key-pairs.html#how-to-generate-your-own-key-and-import-it-to-aws)
- [Ubuntu ufw help page](https://help.ubuntu.com/community/UFW)
- [Linux man page for sshd_config](https://linux.die.net/man/5/sshd_config)
- [Don't Publicly Expose Git](https://en.internetwache.org/dont-publicly-expose-git-or-how-we-downloaded-your-websites-sourcecode-an-analysis-of-alexas-1m-28-07-2015/)
- Stack Overflow: [How To Make a Git Directory inaccessible on the web](https://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible)
- [How to use htaccess](http://www.htaccess-guide.com/how-to-use-htaccess/)
- Stack Overflow: [WSGI Apache ImportError](https://stackoverflow.com/questions/43330231/500-internal-server-error-mod-wsgi-apache-importerror-no-module-named-django)
- Stack Overflow: [Apache server and mod_wsgi File Not Found error](https://stackoverflow.com/questions/28654930/apache-mod-wsgi-error-file-not-found-when-the-file-exists)
- Stack Overflow: [Secret Key in a Flask session](https://stackoverflow.com/questions/26080872/secret-key-not-set-in-flask-session)
- Digital Ocean Tutorial [How To Structure Large Flask Applications](https://www.digitalocean.com/community/tutorials/how-to-structure-large-flask-applications)
- Stack Overflow: [PostgreSQL explicit type casts](https://stackoverflow.com/questions/42465402/sqlalchemy-postgres-you-might-need-to-add-explicit-type-casts-on-merge)
- Stack Overflow: [PostgreSQL type mismatch](https://stackoverflow.com/questions/21349378/operator-does-not-exist-character-varying-bigint-in-gnuhealth-project)
- [mod_wsgi Issue: Target WSGI script cannot be loaded as Python module](https://github.com/GrahamDumpleton/mod_wsgi/issues/156)
- Udacity Forum [Deploying to the Server](https://discussions.udacity.com/t/troubles-with-project-5-deploying-to-server/370603)
- Udacity Forum [Deploying Item Catalog Project](https://discussions.udacity.com/t/deploying-item-catalog-project/227189)
- AWS Forum [How To Change SSH Port](https://forums.aws.amazon.com/thread.jspa?threadID=160352)
- AWS Forum [Operation Timeout](https://forums.aws.amazon.com/thread.jspa?threadID=66813)
- AWS Docs [Default Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html#default-security-group)
- [Update MOTD Ubuntu](https://serverfault.com/questions/262751/update-ubuntu-10-04/262773#262773)
- [Automatic Updates on Ubuntu](https://help.ubuntu.com/lts/serverguide/automatic-updates.html)
