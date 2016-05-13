# Note to install CartoDB on Centos 7.x


######Start with the Centos 7.x Core
From the first step use `sudo su` to change to root and `yum -y update` to upgrade your CentOS system software to the latest version.

######System locales
Installations assume you use UTF8. You can set the locale by doing this:
```
localectl status # to display locale settings
localectl set-locale LANG=en_US.utf8 # to set the Language
localectl list-locales # to lists locales
locale list-keymaps # list keyboard mappings
```

######Install Develpment Tools
If Development Tools are not installed in your system by default, you can install the latest available from the repositories as follows:
```
yum -y groups install "Development Tool"
```

######Install Git
```
yum -y install git
```

######Install Python and pip
```
yum -y install python python-devel
easy_install pip
```

######Install PostgreSQL
Add the PostgreSQL 9.3 Repository
```
rpm -iUvh http://yum.postgresql.org/9.3/redhat/rhel-7-x86_64/pgdg-centos93-9.3-1.noarch.rpm
```
Install PostgreSQL
```
yum -y update 
yum -y install postgresql93 postgresql93-server postgresql93-contrib postgresql93-libs postgresql93-devel postgresql93-python postgresql93-plpython
```
Start PostgreSQL
```
systemctl enable postgresql-9.3
/usr/pgsql-9.3/bin/postgresql93-setup initdb
systemctl start postgresql-9.3.service
```
Locate the `pg_hba.conf` an set this with no password access from localhost  

```bash
local   all             postgres                                trust
local   all             all                                     trust
host    all             all             127.0.0.1/32            trust
host    all             all             ::1/128                 trust
```
For these changes to take effect, you'need to restart postgres:
```
systemctl restart postgresql-9.3.service
```

Create some users in PostgreSQL. These users are used by some CartoDB apps internally.
```
createuser publicuser --no-createrole --no-createdb --no-superuser -U postgres
createuser tileuser --no-createrole --no-createdb --no-superuser -U postgres
```

Install CartoDB postgresql extension. It must be assumed that the local folder where git clone the repository is `/opt/`
```
git clone https://github.com/CartoDB/cartodb-postgresql.git
cd cartodb-postgresql/
```
At this point the official documentation recommends to use `git checkout` to match the `<LATEST cartodb-postgresql tag>`.  
Use `git describe --tags` (0.16.3) to check the current tag version of master. If there are multiple tags (local or remote) for the HEAD of master you will get one tag from this command, not a list. 
It recommend using the `-–match` parameter to make sure you get the tag you expect.
```
git describe  --match="v*" 
git checkout <latest cartodb-postgresql tag>
export PATH=$PATH:/usr/pgsql-9.3/bin/
make all install
```
######Install GIS dependencies
Install PROJ.4 
```
yum -y install proj proj-devel proj-epsg proj-nad proj-static
```
Install JSON
```
yum -y install json-c json-c-devel python-simplejson
```
Install GEOS
```
yum -y install geos geos-devel geos-python
```
Install GDAL
```
yum -y install gdal gdal-libs gdal-devel gdal-python
```

######Install PostGIS
Install XML Library
```
yum -y install libxml2 libxml2-devel
```
Install PostGIS extension
```
yum -y install postgis2_93 postgis2_93-client postgis2_93-devel postgis2_93-utils
```
Initialize template postgis database. We create a template database in postgresql that will contain the postgis extension. This way, every time CartoDB creates a new user database it just clones this template database.

```
createdb -T template0 -O postgres -U postgres -E UTF8 template_postgis
createlang plpgsql -U postgres -d template_postgis
psql -U postgres template_postgis -c 'CREATE EXTENSION postgis;CREATE EXTENSION postgis_topology;'

```
Run an installcheck to verify the database has been installed properly
```
PGUSER=postgres make installcheck
```
Restart PostgreSQL after all this process
```
systemctl start postgresql-9.3.service
```

######Install Redis
Redis 3+ is needed. The version contained in the EPEL repository is `2.8.19-2`. We proceed by installing redis 3.x from stable source package.  

Download Redis 3.x stable source package

``` 
yum -y install wget
wget http://download.redis.io/releases/redis-3.2.0.tar.gz
```
Untar downloaded Redis Tar ball

``` 
tar -xvzf redis-3.2.0.tar.gz
```
Compiling of Redis from source

``` 
cd redis-3.2.0/deps/
make hiredis lua jemalloc linenoise
cd .. && cd src
make
make install
```
Testing Redis source installation
```
yum -y install tcl
make test
```
Install init script
```
cd .. && cd utils
./install_server.sh

Welcome to the redis service installer
This script will help you easily set up a running redis server

Please select the redis port for this instance: [6379] 
Selecting default: 6379
Please select the redis config file name [/etc/redis/6379.conf] 
Selected default - /etc/redis/6379.conf
Please select the redis log file name [/var/log/redis_6379.log] 
Selected default - /var/log/redis_6379.log
Please select the data directory for this instance [/var/lib/redis/6379] 
Selected default - /var/lib/redis/6379
Please select the redis executable path [/usr/local/bin/redis-server] 
Selected config:
Port           : 6379
Config file    : /etc/redis/6379.conf
Log file       : /var/log/redis_6379.log
Data dir       : /var/lib/redis/6379
Executable     : /usr/local/bin/redis-server
Cli Executable : /usr/local/bin/redis-cli
Is this ok? Then press ENTER to go on or Ctrl-C to abort.
Copied /tmp/6379.conf => /etc/init.d/redis_6379
Installing service...
Successfully added to chkconfig!
Successfully added to runlevels 345!
Starting Redis server...
Installation successful!
```
Setting Redis persistence
Edit Redis Config file `/etc/redis/6379.conf`. Look for the following line `appendonly no` and change it to `appendonly yes`. You have just enabled Redis’ AOF (Append Only File) persistence. You are ready to go. For more information check [redis.io/topics/persistenc](http://redis.io/topics/persistenc)

Redis start/stop/restart/status
```
#To check status of Redis Server
systemctl status redis_6379.service

# To start Redis Server
systemctl start redis_6379.service

# To stop Redis Server
systemctl stop redis_6379.service

# To restart the Redis Server
systemctl restart redis_6379.service
```

######Install NodeJS 
```
yum -y install nodejs npm
```

You can verify the installation went as expected with `node -v` and `npm -v`.  
If you need to update npm use `npm install npm -g` and global dependencies use `npm update -g`.  


######Install SQL API
Download API
```
git clone https://github.com/CartoDB/CartoDB-SQL-API.git
cd CartoDB-SQL-API
git checkout master
```
Install NPM dependencies
```
npm install
```
Test SQL API
Create configuration. The name of the filename of the configuration must be the same than the environment you are going to use to start the service. Let’s assume it’s development. You may find the `./configure` script useful to make an edited copy for you.
```
PGUSER=postgres make check
```
Start SQL API

Start the service. The second parameter is always the environment if the service. Remember to use the same you used in the configuration.
```
node app.js <environment>
```
Supported values are development, test, staging, production

######Install MAPS API
Download API
```
git clone https://github.com/CartoDB/Windshaft-cartodb.git
cd Windshaft-cartodb
git checkout master
```
######Install Cairo
```
yum -y install cairo cairo-devel
```
######Install JPEG Library
```
yum -y install libjpeg-turbo*
```
######Install GIF Library
```
yum -y install giflib*
```
######Install TIFF Library
```
yum -y install libtiff*
```
Install NPM dependencies
```
npm install
```
Test SQL API
Create configuration. The name of the filename of the configuration must be the same than the environment you are going to use to start the service. Let’s assume it’s development. You may find the `./configure` script useful to make an edited copy for you.
```
PGUSER=postgres make check
```
Start SQL API

Start the service. The second parameter is always the environment if the service. Remember to use the same you used in the configuration.
```
node app.js <environment>
```
Supported values are development, test, staging, production

* 1 failing (multilayer unknown text-face-name)
This error indicates that you are using a text-face-name value in your stylesheet that references a font that does not exist on your file system or is mis-spelled.

######Install Ruby47
Install Required Packages
```
yum -y install readline-devel libyaml-devel libffi-devel sqlite-devel
```

Install Ruby Version Manager (RVM) 
```
curl -sSL https://rvm.io/mpapis.asc | gpg --import -
curl -L get.rvm.io | bash -s stable
source /etc/profile.d/rvm.sh
rvm reload
```
Verify Dependencies
```
rvm requirements run 
```
Install Ruby 2.2 
```
rvm install 2.2.3
```
Setup Default Ruby Version and Check Current Ruby Version
```
rvm use 2.2.3 --default
ruby -v
```
Install Ruby Gems
```
gem install bundler
gem install compass
```

######Install Editor
Download Editor
```
git clone --recursive https://github.com/CartoDB/cartodb.git
cd cartodb
```

Install dependencies
```
yum -y install ImageMagic libicu libicu-devel

RAILS_ENV=development bundle install
npm install
pip install --no-use-wheel -r python_requirements.txt
```

If this fails due to the installation of the gdal package not finding Python.h, you’ll need to do this:
```
export CPLUS_INCLUDE_PATH=/usr/include/gdal
export C_INCLUDE_PATH=/usr/include/gdal
export PATH=$PATH:/usr/include/gdal
```
After this, re-run the pip install command, and it should work.

Add the grunt command to the PATH
```
export PATH=$PATH:$PWD/node_modules/grunt-cli/bin
```

Install all necesary gems
```
bundle install
```

Precompile assets. Note that the last parameter is the environment used to run the application. It must be the same used in the Maps and SQL APIs  
```
bundle exec grunt --environment development
```
Create Configuration File
``` 
cp config/app_config.yml.sample config/app_config.yml
cp config/database.yml.sample config/database.yml
```
######First running, setting up user
```
cd cartodb
export SUBDOMAIN=development

# Add entries to /etc/hosts needed in development
echo "127.0.0.1 ${SUBDOMAIN}.localhost.lan" | sudo tee -a /etc/hosts

# Create a development user
sh script/create_dev_user ${SUBDOMAIN}
```


######Running all the processes

Start the resque daemon (needed for import jobs):
```
bundle exec script/resque
```
Finally, start the CartoDB development server on port 3000:
```
bundle exec thin start --threaded -p 3000 --threadpool-size 5
```
Node apps
```
cd cartodb-sql-api && node app.js
cd windshaft-cartodb && node app.js
```
You should now be able to access `http://<mysubdomain>.localhost.lan:3000` in your browser and login with the password specified above.

Enjoy
