# Roar server setup


## Overview
The Roar Application Stack comprises the following distinct layers:

- **Web**  
  _Apache+PHP web layer_  
  Handles incoming HTTP requests and routes to the App layer.

- **Leaderboard**  
  _PHP+MySQL leaderboard module_  
  A sub-module dedicated to handling leaderboards.

- **Application**  
  _C++ Roar socket application_  
  The core game engine (the server itself).

- **DB**  
  _Postgres app DB layer_  
  Used to persist application data (eg. players, items, etc).

Instructions to setup these components are detailed below. To **configure** Roar's mechanics via the XML files in `/bin/data`, please refer to the [configuration file docs](https://github.com/roarengine/roar-docs/tree/master/docs/configuration_files).

#### Scaling up
We recommend running Roar in AWS with the following starting config:

- Elastic Loadbalancer
- 1x EC2 medium node (web layer)
- 1x EC2 x-large node (application + db + leaderboard)

Beyond this, depending on your application usage profile, scale out the webnodes, put the leaderboard server on its own cluster, and place the Postgres DB in its own cluster. And of course, vertically scaling the capacity of the boxes running the services will go along way to scaling your service.


## Server OS

Roar requires and is supported only on **Ubuntu Server 12.04 64bit**. We recommend using AWS for running Roar, although you may use any service you wish (including your own dedicated hosting).

If you're running on the AWS Ubuntu 12.04 LTS image you will need to do the following to get everything up to the right stage:


    sudo apt-get update
    sudo apt-get install postgresql postgresql-client-common apache2 mysql-server  libapache2-mod-php5 php5-mysql
    sudo a2enmod php5
    sudo /etc/init.d/apache2 restart

Once that is setup, you can begin installing Roar. 


## Install and Build process

* Install **Ubuntu server 12.04 64bit**:
    * Select opens server, LAMP server. Postgresql database
* Install **required packages**:

     ~~~
     sudo apt-get update
     sudo apt-get install git make g++ libboost-dev libpq-dev \
         libprotobuf-dev protobuf-compiler \
         python-protobuf scons libreadline-dev libncurses-dev libxml2-dev libboost-all-dev \
         libcurl4-openssl-dev libgoogle-perftools-dev python-lxml
    ~~~

    On the **web server(s)** we need **PHP Pear**:

    ~~~
    sudo apt-get install php-pear
    ~~~

    **NOTE:** We can probably get away with a lighter package than lib boost-all-dev though.

* Install mailutils (used by `restarter.bash`):

    sudo apt-get install mailutils

* Generate a server **ssh key** with `ssh-keygen`  and add `~/.ssh/id_rsa.pub` to your github user.
* Clone the **roarengine repository**:
    ~~~
    git clone git@github.com:roarengine/roarengine
    cd roarengine
    git checkout master
    git submodule init ; git submodule update
    cd ..
    ~~~

* Download, patch and install **Lua**:

    ~~~
    curl http://www.lua.org/ftp/lua-5.1.tar.gz -o lua-5.1.tar.gz
    tar xzf lua-5.1.tar.gz
    cd lua-5.1
    patch -p0 < roarengine/lua_patch.patch
    make linux
    sudo make install
    ~~~

* Build the **fastjson** module:

    ~~~
    cd roarengine/fastjson ; make ; cd ..
    ~~~

* **Build Roar**:
    ~~~
    cd ..
    ./build.bash clean
    mv src/roarengine .
    ~~~

* Setup **Apache**:
    * Install PHP packages

        ~~~
         sudo pear install channel://pear.php.net/XML_Serializer-0.20.2
        ~~~

    * Install postgres drivers ( the install is supposed to restart apache, but doesn't in all cases, so try an explicit restart )

        ~~~
        sudo apt-get install php5-pgsql
        sudo /etc/init.d/apache2 restart
        ~~~

    * Disable default site and enable roar site

        ~~~
        sudo cp apache-roar /etc/apache2/sites-available/roar
        sudo a2dissite default
        sudo a2ensite roar
        sudo service apache2 reload
        ~~~

        You can ignore any warnings about `DocumentRoot [/var/www/chameleon] does not exist` later steps will ensure this is setup.

* Setup **databases**
    * Setup core postgres database and user.
         The actual usernames, passwords and database names can be changed in the `engine/linux.conf` file.
         This just uses the default values we use for testing.

        ~~~
        > sudo -u postgres psql
        # create database "ChameleonTest";
        # create user roar with password 'awesomesauce';
        # grant all privileges on database "ChameleonTest" to roar;
        # \q
        > psql --host=localhost --username=roar -W ChameleonTest < database/core_schema.sql
        ~~~

   * Setup auxiliary lua postgres database and user.
         The actual usernames, passwords and database names can be changed in the `engine/linux.conf` file.
         This just uses the default values we use for testing.

        ~~~
        > sudo -u postgres psql
        # create database luadb;
        # create user roarlua with password 'awesomesaucier';
        # grant all privileges on database luadb to roarlua;
        ~~~

    * Setup leaderboard mysql database

        * Install memcache related modules

             ~~~
             sudo apt-get install memcached php5-memcache
             ~~~

        * Set the root password

           ~~~
           sudo dpkg-reconfigure mysql-server-5.5
           ~~~

        * Create the database

           ~~~
           > mysql -u root -p
           mysql> CREATE DATABASE Leaderboard;
           mysql> exit;
           > mysql -u root -p Leaderboard < leaderboard_server/leaderboard_schema.sql
           ~~~

        * Add the default leaderboard user info. In config `config/linux.conf` file there is something

        ~~~
          "leaderboard": {
           "app_id":"4321",
           "user_id":"777",
           "auth_token":"1234",
        ~~~

        These values need to be inserted into the Leaderboard database (you can change them if you wish).

        ~~~
        > mysql -u root -p Leaderboard
        mysql> INSERT INTO apps ( app_id ) VALUES ( 4321 );
        mysql> INSERT INTO users ( user_id, auth_token ) VALUES ( 777, 1234 );
        mysql> INSERT INTO permissions ( user_id, app_id, resource_id, permission ) VALUES ( 777, 4321, 0, "ALL" );
        mysql> INSERT INTO permissions ( user_id, app_id, resource_id, permission ) VALUES ( 777, 0, 0, "SUPER" );
        ~~~

    * If you use any non-standard database names / passwords you will need to change the `linux.conf` and `serverconfig.php` files in the config directory.

* Create your **keys.json** file

    ~~~
    cp config/default/keys.template.json config/default/keys.json
    ~~~

* Run the **Install script**

    ~~~
    sudo ./install.bash
    ~~~

* And finally, **start the engine...**

    ~~~
    ./roarengine config/tests/linux.conf
    ~~~

Note: Refer to the full [documentation on correct startup and shutdown for Roar](https://github.com/roarengine/roar-docs/blob/master/docs/running.md) in **production**.

You now have a functional Roar Engine server. To start doing interesting things with it, you'll need to setup the **configuration XML files** located in `/bin/data`. For information on how to setup these files, refer to the [configuration file docs](https://github.com/roarengine/roar-docs/tree/master/docs/configuration_files).

