Elastic Beanstalk - cron command and RDS DB access
============================================

Problem
--------------------------------------------
I have a console command in php which needs an access to DB.
The command need to be launched via cron.

The DB connection string looks like like this

    'connectionString' => 'mysql:host='.$_SERVER['RDS_HOSTNAME'].';port='.$_SERVER['RDS_PORT'].';dbname='.$_SERVER['RDS_DB_NAME'],

where RDS_xxx parameters come from environment variables.

The problem is that cron launches the command with a clean environment (there are no RDS_xx variables).
So the command fails to access the database.

Solution
--------------------------------------------
Solution is to set the required environment variables before launching the command and this can be done with '/opt/elasticbeanstal/support/envvars' script:

    0 3 * * * . /opt/elasticbeanstalk/support/envvars; /var/www/html/console/yiic mycommand

