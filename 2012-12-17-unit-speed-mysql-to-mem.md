Speedup unit tests by moving MySql data to memory [Ubuntu]
============================================

There are several ways to speedup slow unit tests which interact with database:
 * Refactor code and tests and do not interact with db in unit tests
 * Use sqlite db in memory instead of MySql
 * Use MySql MEMORY engine
 * Move MySql data to memory

It is better to try other listed approaches and I think of last method as of quick temporary hack, but here it is:
 * stop mysql
 * move /var/lib/mysql to /dev/shm/mysql
 * link /var/lib/mysql to /dev/shm/mysql
 * start mysql

In Ubuntu there is also a problem with apparmor which will not allow mysql to read from /dev/shm.
To fix this it is recommended to add following to the `/etc/apparmor.d/usr.sbin.mysqld`:

    /dev/shm/mysql/ r,
    /dev/shm/mysql/** rwk,

But it doesn't work for me and I disabled apparmor for mysql (not recommended):

    sudo mv /etc/apparmor.d/usr.sbin.mysqld /etc/apparmor.d/disable

Below are shell scripts to move MySql data to /dev/shm and back, restore backed up data and check db state.

Move db to memory script
--------------------------------------------

    #!/bin/sh
    #Check if run as root
    if [ `whoami` != root ]
    then
        echo "You must be root to do that!"
        exit 1
    fi

    service mysql stop
    if [ ! -s /var/lib/mysql.backup ]
    then
        cp -pRL /var/lib/mysql /var/lib/mysql.backup
    fi
    mv /var/lib/mysql /dev/shm/mysql
    chown -R mysql:mysql /dev/shm/mysql
    ln -s /dev/shm/mysql /var/lib/mysql
    chown -h mysql:mysql /var/lib/mysql
    service mysql start


Move db to disk script
--------------------------------------------

    #!/bin/sh
    #Check if run as root
    if [ `whoami` != root ]
    then
        echo "You must be root to do that!"
        exit 1
    fi

    service mysql stop
    rm /var/lib/mysql
    if [ ! -s /dev/shm/mysql ]
    then
        cp -pRL /var/lib/mysql.backup /var/lib/mysql
    else
        mv /dev/shm/mysql /var/lib/mysql
    fi
    service mysql start


Restore db backup script
--------------------------------------------

    #!/bin/sh
    #Check if run as root
    if [ `whoami` != root ]
    then
        echo "You must be root to do that!"
        exit 1
    fi

    service mysql stop
    if [ ! -s /var/lib/mysql.backup ]
    then
        exit -1
    fi
    rm /var/lib/mysql
    cp -pRL /var/lib/mysql.backup /var/lib/mysql
    rm -rf /dev/shm/mysql
    service mysql start

Check db state script
--------------------------------------------

    #!/bin/sh
    #Check if run as root
    if [ `whoami` != root ]
    then
        echo "You must be root to do that!"
        exit 1
    fi

    if [ -L /var/lib/mysql ]
    then
        echo "Mem db"
        exit 0
    else
        echo "File db"
        exit 1
    fi


Links
--------------------------------------------
[Unit test application including database is too slow](http://stackoverflow.com/questions/9500032/unit-test-application-including-database-is-too-slow)

[Force an entire MySQL database to be in memory](http://stackoverflow.com/questions/4894850/force-an-entire-mysql-database-to-be-in-memory)

[How to run django's test database only in memory?](http://stackoverflow.com/questions/3096148/how-to-run-djangos-test-database-only-in-memory)

[Script to put mysqld on a ram disk in ubuntu 10.04. Runs on every hudson slave boot](https://gist.github.com/1152547)

[MySQL tmpdir on /dev/shm](http://www.indimon.co.uk/2012/mysql-tmpdir-on-devshm/)

[How can I get around MySQL Errcode 13 with SELECT INTO OUTFILE?](http://stackoverflow.com/questions/2783313/how-can-i-get-around-mysql-errcode-13-with-select-into-outfile)
