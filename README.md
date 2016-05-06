# cartodb-install-centos7.x
Note to install CartoDB on Centos 7.x (Core)



######Start with the Centos 7.x Core
From the first step use `sudo su` to change to root and `yum update` to upgrade your CentOS system software to the latest version.

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
yum groups install "Development Tool"
```

######Install Git
```
yum install git
```

######Install Python and pip
```
yum install python python-devel
easy_install pip
```

######Install PostgreSQL
Import RPM Repository
```
rpm -ivh http://yum.postgresql.org/9.3/redhat/rhel-7-x86_64/pgdg-centos93-9.3-1.noarch.rpm
```

```
yum update
yum list postgre93*
```

**Coming Soon**
