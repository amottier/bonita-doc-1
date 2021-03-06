# Custom Bonita Platform Setup

How to set up a Bonita Platform from scratch in a specific environment (Ubuntu, Openjdk, Tomcat, PostgreSQL).

## Operating System

Install a Ubuntu Linux distribution.
Simply follow a standard Ubuntu installation procedure. You won't need any specific settings here.

Note: make sure you got a text editor installed such as `nano`. In order to install it: `sudo apt install nano`

Note: we use `apt` command line tool to install packages such as OpenJDK or PostgreSQL. You can also use different tool such
as `apt` or a graphical package manager such as `synaptic`.

## Java Virtual Machine

To install the OpenJDK JVM you need to install the `openjdk-8-jre-headless` package:

* Run the following command line: `sudo apt install openjdk-8-jre-headless`
* If needed, type your Ubuntu user password
* Type _**Enter**_ to confirm that you want to continue installation

To check that Java is correctly setup in a console/terminal, type: `java -version`.
You should get a similar message to the one below:  
```
openjdk version "1.8.0_111"
OpenJDK Runtime Environment (build 1.8.0_111-8u111-b14-2ubuntu0.16.04.2-b14)
OpenJDK 64-Bit Server VM (build 25.111-b14, mixed mode)
```

## Database

### Install

To install PostgreSQL 11 you need to install the `postgres-11` package:

* Run the following command line: `sudo apt install postgresql-11`
* If needed, type your Ubuntu user password
* Type _**Enter**_ to confirm that you want to continue installation

### Create databases and user

You now need to create: a new PostgreSQL user and two new databases for Bonita, and grant the user the appropriate privileges to access
the databases. In a terminal or console, type following commands:

* Switch to a shell for `postgres` user account: `sudo -u postgres -i`
* To create a new database user named `bonita_db_user` type: `createuser -P bonita_db_user`
* Enter (twice) the password for the newly created database user. E.g.: `bonita_db_password`
* Create a new database (for Bonita Engine) named `bonita_db` and give ownership to `bonita_db_user`: `createdb -O bonita_db_user bonita_db`
* Create another database (for BDM) named `bonita_bdm` and give ownership to `bonita_db_user`: `createdb -O bonita_db_user bonita_bdm`
* Exit `postgres` session, type command: `exit`

You can now verify that user and database are correctly created. In a terminal or console type following commands:

* Start PostgreSQL client program by typing: `psql -d bonita_db -h 127.0.0.1 -U bonita_db_user`
* Type the PostgreSQL user password. E.g.: bonita\_db\_password
* You can run test query that should return `1`: `SELECT 1;`
* Now you can quit PostgreSQL client, type: `\q`

### Enable prepared transactions

Prepared transactions is disabled by default in PostgreSQL. You need to enable it:

* Edit the `postgresql.conf` configuration file: `sudo nano /etc/postgresql/11.2/main/postgresql.conf`
* Look for the line with `max_prepared_transactions` (line 117)
* Uncomment the line: remove `#` character
* Change the value from `zero` to `100`
* Save modification and quit text editor
* Restart PostgreSQL server: `sudo service postgresql restart`

## Application Server

### Install Tomcat 8.5

Run the following command lines:

* `sudo apt install curl`
* `cd /tmp/`
* `wget http://apache.mirrors.ionfish.org/tomcat/tomcat-8/v8.5.40/bin/apache-tomcat-8.5.40.tar.gz`
* `sudo tar xzvf apache-tomcat-8.5.40.tar.gz -C /usr/share/tomcat8 --strip-components=1`

Tomcat 8.5 should now be installed on your computer. Now you should enable tomcat to be run as a service

* `sudo nano /etc/systemd/system/tomcat.service`
* Copy the following code into the file
```
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking

Environment=JAVA_HOME=Your Java Home
Environment=CATALINA_PID=/usr/share/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/usr/share/tomcat
Environment=CATALINA_BASE=/usr/share/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/usr/share/tomcat/bin/startup.sh
ExecStop=/usr/share/tomcat/bin/shutdown.sh

User=yourUser
Group=yourGroup
UMask=0007
RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```

* Save and close the file
* Reload the systemd daemon: `sudo systemctl daemon-reload`

In case of problems with this part refer to the documentations pages [How to Install and Configure Apache Tomcat 8.5 on Ubuntu 16.04](https://www.howtoforge.com/tutorial/how-to-install-apache-tomcat-8-5-on-ubuntu-16-04/)

### Add JDBC driver

You need to include JDBC driver in Tomcat classpath:

* Change to Tomcat libraries directory: `cd /usr/share/tomcat8/lib`
* Install `wget` tool in order to be able to download JDBC driver: `sudo apt install wget`
* Download the JDBC driver: `sudo wget http://jdbc.postgresql.org/download/postgresql-42.2.5.jar`

## Bonita Platform

### Download and unzip the Bonita deploy bundle

Download the Bonita Tomcat bundle from the [Customer Portal](https://customer.bonitasoft.com/)
(Subscription editions) or get the [Community edition](http://www.bonitasoft.com/downloads-v2). Instructions
below will be given for Bonita Subscription. You can easily adapt files and folder names to the Community edition.

* Go to Bonitasoft [Customer Portal](https://customer.bonitasoft.com/)
* In **Download** menu, click on _**Request a download**_
* Select your version and click on _**Access download page**_ button
* On the download page, go to the **Deploying Server Components** section
* Click on _**Download BonitaSubscription-x.y.z.zip**_ link. If your server only has a terminal available
you can copy the link and use `wget` to download the file or use SSH with `scp` command to copy
the file from another computer.
* Make sure that the `BonitaSubscription-x.y.z.zip` file is located in your home folder (e.g. `/home/osuser`).
If you type `cd ~ && ls` you should see the file listed.
* Make sure the `unzip` command is installed: `sudo apt install unzip`
* Unzip the Tomcat bundle: `unzip -q BonitaSubscription-x.y.z.zip`

* Change folders and files ownership: `sudo chown -R tomcat8:tomcat8 /opt/bonita`

### JVM system variables

To define JVM system properties, you need to use a new file named `setenv.sh`:

* Copy the file from Tomcat bundle to Tomcat installation `bin` folder: `sudo cp ~/BonitaSubscription-x.y.z/server/bin/setenv.sh /usr/share/tomcat8/bin/`, where "x.y.z" stands for your current product version.
* Make the file executable: `sudo chmod +x /usr/share/tomcat8/bin/setenv.sh`
* Edit `setenv.sh` file: `sudo nano /usr/share/tomcat8/bin/setenv.sh`
* Change `sysprop.bonita.db.vendor` from `h2` to `postgres`
* Change `com.arjuna.ats.arjuna.common.propertiesFile` from `${CATALINA_HOME}/conf/jbossts-properties.xml` to `/opt/bonita/conf/jbossts-properties.xml`

### Add extra libraries to Tomcat

Bonita needs extra libraries such as Narayana, in order to run on Tomcat:

* Change to the Deploy bundle Tomcat lib folder: `cd ~/BonitaSubscription-x.y.z-deploy/Tomcat-8.5.z/lib`, where "y.z" stands for the last digits of the product version
* Copy the libraries (.jar files) from the Deploy bundle to your Tomcat folder: `sudo cp *.jar /usr/share/tomcat8/lib/`
* Remove default tomcat-dbcp.jar from folder `/usr/share/tomcat8/lib/`, as Bonita provides a newer version.

### Configure Bonita to use PostgreSQL

You need to configure the data source for Bonita Engine.

Warning: make sure you stop Tomcat before performing following operations: `sudo service tomcat8 stop`

* Create new folders in order to store Arjuna configuration and work files: `sudo mkdir -p /opt/bonita/conf && sudo mkdir -p /opt/bonita/arjuna/tx-object-store`
* Set the ownership of the Arjuna configuration folder: `sudo chown -R tomcat8:tomcat8 /opt/bonita/arjuna/tx-object-store`
* Copy the Arjuna configuration files to `/opt/bonita/conf` folder: `sudo cp ~/BonitaSubscription-x.y.z/server/conf/jbossts-properties.xml /opt/bonita/conf/`
* Edit `jbossts-properties.xml` file, and update the `objectStoreDir` property to point to Arjuna work dir (created above) by replacing:  
  `<entry key="com.arjuna.ats.arjuna.objectstore.objectStoreDir">${catalina.base}/work/bonita-tx-object-store</entry>`  
  by  
  `<entry key="com.arjuna.ats.arjuna.objectstore.objectStoreDir">/opt/bonita/arjuna/tx-object-store</entry>`
* Save and quit: `CTRL+X, Y, ENTER`
* Copy the `bonita.xml` file (Bonita web app context configuration): `sudo cp ~/BonitaSubscription-x.y.z/server/conf/Catalina/localhost/bonita.xml /etc/tomcat8/Catalina/localhost/`
* Edit the `bonita.xml` file by commenting the h2 datasource configuration and
uncomment PostgreSQL example and update username, password and database name (bonita in the URL property) to match your
configuration (e.g. `bonita_db_user`, `bonita_db_password` and `bonita_db`): `sudo nano /etc/tomcat8/Catalina/localhost/bonita.xml`
* Also in `bonita.xml` file update data base configuration for BDM to match your configuration (e.g. `bonita_db_user`, `bonita_db_password` and `bonita_bdm`)
* Save and quit: `CTRL+X, Y, ENTER`
* Copy and overwrite `logging.properties` file: `sudo cp ~/BonitaSubscription-x.y.z/server/conf/logging.properties /etc/tomcat8/logging.properties`
* Copy and overwrite `context.xml` file: `sudo cp ~/BonitaSubscription-x.y.z/server/conf/context.xml /etc/tomcat8/context.xml`
* Copy and overwrite `server.xml` file: `sudo cp ~/BonitaSubscription-x.y.z/server/conf/server.xml /etc/tomcat8/server.xml`
* Edit `server.xml` (`sudo nano /etc/tomcat8/server.xml`) and comment out h2 listener line
* Fix ownership on the copied files: `sudo chown -R root:tomcat8 /etc/tomcat8`

### License

If you run the Subscription Pack version, you will need a license:

* Generate the key in order to get a license:
  * Change the current directory to license generation scripts folder: `cd ~/BonitaSubscription-x.y.z/tools/request_key_utils-x.y-z`
  * Make sure the license generation script is executable: `chmod u+x generateRequestKey.sh`
  * Run the script: `./generateRequestKey.sh`
  * For `License type:`
    * enter `1` to select `1 - Case counter license.` if your subscription is case-based.
    * enter `2` to select `2 - CPU core license.` If your subscription type is cpu based, please refer to the [knowledge-base](https://customer.bonitasoft.com/knowledgebase) in the customer portal.
    * enter `3` to select `3 - Enterprise license.` if your subscription is Enterprise.
  * You will get a license key (of the type `(9fIyr5O+e2Z8MwDiEPC23sfTfAXv7Y6K)`) that you can copy. Make sure that you keep the brackets. If the key is separated by a linebreak, remove it and put the key on a single line.
* Connect to Bonitasoft [Customer Portal](https://customer.bonitasoft.com/)
* Go to Licenses \> **Request a license**
* Fill in the license request forms
* You should receive the license file by email
* Copy the license file to the Bonita license folder: `sudo cp BonitaSubscription-x.y-Your_Name-ServerName-YYYYMMDD-YYYYMMDD.lic /opt/bonita/<SETUP_TOOL_FOLDER>/platform_conf/licenses/`
* Change folders and files ownership: `sudo chown -R tomcat8:tomcat8 /opt/bonita`

### Deployment

Deploy the Bonita web application:

Copy `bonita.war` to Tomcat `webapps` folder: `sudo cp ~/BonitaSubscription-x.y.z/server/webapps/bonita.war /var/lib/tomcat8/webapps/`

Take care to set the proper owner: `sudo chown tomcat8:tomcat8 /var/lib/tomcat8/webapps/bonita.war`

Start Tomcat: `sudo service tomcat8 start`

### First connection

You can access the Bonita Portal using your web browser, just type the following URL `http://<your_server_hostname>:8080/bonita` (your\_server\_hostname can be either an IP address or a name).  
You can log in using the tenant administrator login: `install` and password: `install`.  
The first step is to create at least one user and add it to "administrator" and "user" profiles.
