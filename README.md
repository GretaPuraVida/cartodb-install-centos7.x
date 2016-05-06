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
yum -y install postgresql93 postgresql93-server postgresql93-contrib postgresql93-libs postgresql93-python
```
Start PostgreSQL
```
systemctl enable postgresql-9.3
/usr/pgsql-9.3/bin/postgresql93-setup initdb
systemctl start postgresql-9.3
```
Locate the `pg_hba.conf` an set this with no password access from localhost  

 TYPE  DATABASE        USER            ADDRESS                 METHOD  
 "local" is for Unix domain socket connections only  
local   all             all                                     trust  
 IPv4 local connections:  
host    all             all             127.0.0.1/32            trust  
 IPv6 local connections:  
host    all             all             ::1/128                 ident  

For these changes to take effect, you'need to restart postgres:
```
systemctl restart postgresql-9.3
```

**Coming Soon**
