# cartodb-install-centos7.x
Note to install CartoDB on Centos 7.x (Core)



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
export PATH=/usr/pgsql-9.3/bin:$PATH
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
Install PostGIS
```
yum -y install postgis2_93 postgis2_93-client postgis2_93-devel postgis2_93-utils
```
Initialize template postgis database. We create a template database in postgresql that will contain the postgis extension. This way, every time CartoDB creates a new user database it just clones this template database.

```
createdb -T template0 -O postgres -U postgres -E UTF8 template_postgis
createlang plpgsql -U postgres -d template_postgis
psql -U postgres template_postgis -c 'CREATE EXTENSION postgis;CREATE EXTENSION postgis_topology;'
ldconfig
```
Run an installcheck to verify the database has been installed properly
```
PGUSER=postgres make installcheck
```
Restart PostgreSQL after all this process
```
systemctl start postgresql-9.3.service
```

___
**Coming Soon**
Refered to [CartoDB Docs](http://cartodb.readthedocs.io/en/latest)
