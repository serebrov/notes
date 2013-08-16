How to setup git server on ubuntu with push email notifications
============================================

Git Server
--------------------------------------------
Prerequisites are git and ssh-server (apt-get install openssh-server).

The installation process is described in [the Pro Git book](http://git-scm.com/book/en/Git-on-the-Server-Setting-Up-the-Server).
Below is the setup process with some comments and updates.

Add git user, set some password (you will be asked for it):

    $ sudo adduser git

Log in as git user and setup authorized ssh keys:

    $ su git
    git@localname$ cd ~
    git@localname$ mkdir .ssh

For each user who need an access to the server add user's public key into ~/.ssh/authorized_keys
to generate new key-pair for the user use ssh-keygen see [github manual](https://help.github.com/articles/generating-ssh-keys) for details.

    git@localname$ cat /home/usera/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    git@localname$ cat /home/userb/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

Change permissions for .ssh folder and authorized_keys file:

    git@localname$ chmod 600 ~/.ssh/authorized_keys
    git@localname$ chmod 755 ~/.ssh

Make a dir for repositories and init bare repositories:

    git@localname$ mkdir ~/server
    git@localname$ cd server
    git@localname$ mkdir project.git
    git@localname$ cd project.git
    git@localname$ git --bare init

Sequrity - set git-shell for the git user:

    # check where git-shell is
    $ which git-shell
    /usr/bin/git-shell

    # edit the etc/passwd
    $ sudo vim /etc/passwd

    # find the string for git user:
        git:x:1000:1000::/home/git:/bin/sh
    # change shell for git user:
        git:x:1000:1000::/home/git:/usr/bin/git-shell

On the user side - clone the repository or add a remote to existing repository:

    # clone
    $ git clone ssh://git@server.host.name/home/git/re/sugar.git

    # or add remote
    $ cd project
    $ git remote add origin git@server.host.name:/home/git/re/sugar.git

For 'server.host.name' there are several options:
* if the server has a domain name then just use it
* use sever IP address instead of host name
* use any host name you like and add it to local hosts file to map to the IP address

### Possible problems and solutions

#### git-shell shows the "Interactive git shell is not enabled."

This is OK and it will work, additional setup can be done to allow the git user to log-in via ssh
and execute special commands like "list" to get repository listing.
See:
* main git-shell
* cat /usr/share/doc/git/contrib/git-shell-commands/README
* http://serverfault.com/questions/285324/git-shell-not-enabled

#### When the user tries to execute a server operation git asks for git user password instead of ssh key passphrase.

Check auth log (/var/log/auth.log) for errors.
In my case there were errors related to git user's .ssh and authorized_keys permissions:

    Aug 15 15:25:24 seb-ubu sshd[4561]: Authentication refused: bad ownership or modes for file /home/git/.ssh/authorized_keys
    ...
    Aug 15 15:47:48 seb-ubu sshd[7145]: Authentication refused: bad ownership or modes for directory /home/git/.ssh

Fixing access permissions as described above fixed the issue.

#### How to make existing git repository bare

If you already have a repository and want to put it on the server then it seems to be safe to just
copy the .git folder from the existing repository:

    # source: project/.git
    $ cp -r project/.git project.git
    $ cd project.git
    $ git config --bool core.bare true

Git Server - email notifications on push
--------------------------------------------

Git already has a script to handle email notifications after push.
Check the /usr/share/doc/git/contrib/hooks/post-receive-email for instructions:

    $ sudo chmod a+x /usr/share/git-core/contrib/hooks/post-receive-email
    $ cd /path/to/your/repository.git
    $ ln -sf /usr/share/git-core/contrib/hooks/post-receive-email hooks/post-receive

Configure notifications:

    $ cd /path/to/your/repository.git

    # who should receive notifications
    $ git config hooks.mailinglist "user1@example.com user2@example.com"

    # send emails from
    $ git config hooks.envelopesender git@algo-rithm.com

    # email subject prefix
    $ git config hooks.emailprefix "[Git]"

    # project name - edit 'description' file in the git repository folder
    $ vim description

The post-receive-email script requires sendmail to work.
Below is a description of the sendmail setup process.

Setup sendmail on Ubuntu
--------------------------------------------

Install it:
    $ sudo apt-get install sendmail

Check your hosts file - in my case sendmail was incredibly slow and this was fixed by following
line in /etc/hosts:

    127.0.0.1 localhost.localdomain localhost myhostname <--- order matters!!!

Note that you need to use the same order as above - localhost.localdomain, localhost and then
myhostname (replace myhostname with your real host name, check the output of 'hostname' command).

Send a test email:

    $ echo "My test email being sent from sendmail" | /usr/sbin/sendmail myemail@domain.com

If you have problems with emails then check the log: /var/log/mail.log and error log: /var/log/mail.err.

If you want to setup SMTP server for you emails do the following:

    $ cd /etc/mail
    $ sudo mkdir auth
    $ sudo chmod 700 auth
    $ sudo vim client-info

Enter following line into the client-info file:

    AuthInfo:smtp.server.com "U:mymail@server.com" "I:mymail@server.com" "P:mypassword"

See details about parameters above [here](http://www.scalix.com/wiki/index.php?title=Configuring_Sendmail_with_smarthost_Ubuntu_Gutsy).
Continue:

    $ sudo bash -c "makemap hash client-info < client-info"

Edit the /etc/mail/sendmail.mc and add following lines before the "MAILER_DEFINITIONS" line:

    define('SMART_HOST','smtp.server.com')dnl
    define('confAUTH_MECHANISMS', 'EXTERNAL GSSAPI DIGEST-MD5 CRAM-MD5 LOGIN PLAIN')dnl
    FEATURE('authinfo','hash /etc/mail/auth/client-info')dnl

Process the sendmail.mc with m4:

    $ sudo bash -c "m4 sendmail.mc > sendmail.cf"

Restart sendmail:

    $ sudo service sendmail restart

Resources:

* [sendmail: how to configure sendmail on ubuntu?](http://stackoverflow.com/questions/10359437/sendmail-how-to-configure-sendmail-on-ubuntu)
* [Configuring Sendmail with smarthost Ubuntu Gutsy](http://www.scalix.com/wiki/index.php?title=Configuring_Sendmail_with_smarthost_Ubuntu_Gutsy)
* [Sendmail startup slow, "unqualified hostname unknown; sleeping for retry"](http://forums.fedoraforum.org/archive/index.php/t-85365.html)

Links
--------------------------------------------
* [gitosis vs gitolite? (and vs simple git server)](http://stackoverflow.com/questions/10888300/gitosis-vs-gitolite)
* [Pro Git: Git on the Server - Setting Up the Server](http://git-scm.com/book/en/Git-on-the-Server-Setting-Up-the-Server)
* [git push email notification](http://stackoverflow.com/questions/552360/git-push-email-notification)
* [Setting Up Git Commit Email Notifications](http://www.fclose.com/tutorial/1473/setting-up-git-commit-email-notification/)
