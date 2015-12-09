# What To Do

Below, you'll find the beginning of an article on setting up MediaWiki with the lighttpd web server on Ubuntu 14.04.  The introduction, prerequisites, and some section headers are included to get you started.  Edit these if you need to, but use them as a guideline for what your tutorial's scope should be.

An outdated version of MediaWiki is available through Ubuntu's repository, but you should tell users how to set up the latest stable version instead.

If you wish to expand the scope of the article, consider adding information about one of these:

* Securing your wiki with SSL
* Storing your wiki data in a remote database
* Creating rewrites to configure pretty URLs

Good luck!

----------------------

### Introduction

MediaWiki is a popular open source wiki platform that can be used for public or internal collaborative content publishing.  MediaWiki is used for many of the most popular wikis on the internet including Wikipedia, the site that the project was originally designed to serve.

In this guide, we will be setting up the latest version of MediaWiki on an Ubuntu 14.04 server.  We will use the `lighttpd` web server to make the actual content available, `php-fpm` to handle dynamic processing, and `mysql` to store our wiki's data.

## Prerequisites

To complete this guide, you should have access to a clean Ubuntu 14.04 server instance.  On this system, you should have a non-root user configured with `sudo` privileges for administrative tasks.  You can learn how to set this up by following our [Ubuntu 14.04 initial server setup guide](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04).

When you are ready to continue, log into your server with your `sudo` user and get started below.

# What To Do

Below, you'll find the beginning of an article on setting up MediaWiki with the lighttpd web server on Ubuntu 14.04.  The introduction, prerequisites, and some section headers are included to get you started.  Edit these if you need to, but use them as a guideline for what your tutorial's scope should be.

An outdated version of MediaWiki is available through Ubuntu's repository, but you should tell users how to set up the latest stable version instead.

If you wish to expand the scope of the article, consider adding information about one of these:

* Securing your wiki with SSL
* Storing your wiki data in a remote database
* Creating rewrites to configure pretty URLs

Good luck!

----------------------

### Introduction

MediaWiki is a popular open source wiki platform that can be used for public or internal collaborative content publishing.  MediaWiki is used for many of the most popular wikis on the internet including Wikipedia, the site that the project was originally designed to serve.

In this guide, we will be setting up the latest version of MediaWiki on an Ubuntu 14.04 server.  We will use the `lighttpd` web server to make the actual content available, `php-fpm` to handle dynamic processing, and `mysql` to store our wiki's data.

## Prerequisites

To complete this guide, you should have access to a clean Ubuntu 14.04 server instance.  On this system, you should have a non-root user configured with `sudo` privileges for administrative tasks.  You can learn how to set this up by following our [Ubuntu 14.04 initial server setup guide](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04).

When you are ready to continue, log into your server with your `sudo` user and get started below.

### Install Lighttpd
Lighttpd is a lightweight alternative to the Apache web server.  It is capable of serving high traffic loads while using less server resources compared to other web servers.  It does this by caching static files to make them readily available when requested.

Before we install lighttpd, lets first update and upgrade Ubuntu to make sure it has the latest security patches.  Run `sudo apt-get update` to update the package catalog for the machine (or the list of software and versions that your machine has internally) and then `sudo apt-get upgrade` to actually install any updates.

We can now install lighttpd, which can be done using `sudo apt-get install lighttpd`.  That command will not only install lighttpd, but will also configure it to run as the web server for your machine, or the piece of software that handles incoming requests from users and serves up pages.  You can verify that lighttpd was installed and configured correctly by directing a web browser to the IP address of your droplet like this `http://xxx.xxx.xxx.xxx`, and substituting the IP address of your droplet.  

If lighttpd was installed correctly, you will see the lighttpd “Placeholder Page.”

###Installing MySQL
We’ll do more configuration of lighttpd shortly.  Let’s next install the MySQL database software, or the database software that MediaWiki will use.  This can be done using `sudo apt-get install mysql-server`.  During this process, you will be given a full-page prompt to set a root MySQL password.  Choose a strong, secure password and remember it, you will be using that password to create databases.

Once that process completes, we will run a MySQL script that secures our database installation.  This script removes anonymous user accounts, disables root logins outside of localhost, and removes test databases.  These sensible steps are the MySQL equivalent to locking your house before leaving, and are highly recommended.  This script can be run using the command `sudo mysql_secure_installation` and follow all the prompts.  It is best to answer `Yes` to all the prompts.

## Configure MySQL and Create Credentials for MediaWiki
While we are working with MySQL, let’s create the database that MediaWiki will use.
   1. Open up the MySQL terminal.
      `mysql -u root -p`
   2. Create the database that MediaWiki will use.
      `CREATE DATABASE wikidb;`
   3. Create a MySQL user for MediaWiki.  Substitute “password” for an actual password that    MediaWiki will use to access the database.
      `CREATE USER wikiuser@localhost IDENTIFIED BY 'password';`
   4. Grant permissions to MediaWiki to access and modify the database.
      `GRANT index, create, select, insert, update, delete, alter, lock tables on mediawikidb.* TO mediawikiuser@localhost IDENTIFIED BY 'mediawikipassword';`
     5. Let’s finish by flushing MySQL privileges by running `FLUSH PRIVILEGES;` and then exit the MySQL console by using `exit`.  After exiting the console, run `service mysql restart` to restart the MySQL service.

## Configure PHP-FPM and Lighttpd

###Install PHP and Required Modules
To assure Lighttpd and MediaWiki work correctly together, we need to install both PHP and a few components that PHP will use to work with Lighttpd and get the most out of the web server’s caching functionality.  We will also be installing a module that will allow PHP to work with MySQL.  All together, we will be installing php5, php5-fpm and php5-mysql.  We can install all of these using the command `sudo apt-get install php5 php5-fpm php5-mysql`.  

###Configure Lighttpd to work with php5-fpm
We need to configure lighttpd to work with php5-fpm.  Change into the directory for the lighttpd configuration file by using `cd /etc/lighttpd/conf-available` 

This directory contains all of the config files for lighttpd modules.  The one we are concerned with will be named `15-fastcgi-php.conf`.  We will be editing this file to instruct lighttpd to use php5-fpm.  Before we do though, let’s create a backup of the original config file just in case.  Create the backup using the command `sudo cp 15-fastcgi-php.conf 15-fastcgi-php.conf.backup`.  That command is the copy command, which basically duplicates the first file listed and names it with the second file name listed.  
Now that we have a backup, let’s use the built in text editor to edit the config file.  Run `sudo nano 15-fastcgi-php.conf` and you will be brought into the file itself.
All you need to do is to replace all the PHP code below the `##Start an FastCGI server for php` with the following code:

```php
fastcgi.server += ( ".php" =>
        ((
                "socket" => "/var/run/php5-fpm.sock",
                "broken-scriptfilename" => "enable"
        ))
)
```

Once you have done that, save and exit out of the editor using `CTRL+X`, then `Y`.
We need now to enable the lighttpd’s CGI modules by using the following commands:
```
sudo lighttpd-enable-mod fastcgi
sudo lighttpd-enable-mod fastcgi-php
```
Finally, restart lighttpd using `sudo service lighttpd force-reload` and make sure Lighttpd is running using `sudo service lighttpd status`.
  
## Install MediaWiki
We’ll download and build MediaWiki from source, since many times, the version of the software in the Ubuntu package manager is very out of date.  First, get the source from MediaWiki’s servers by running `curl -O https://releases.wikimedia.org/mediawiki/1.26/mediawiki-1.26.0.tar.gz`.

We decompress and build the source using the `tar` command, which is similar to an Zip program on Windows.  The full command is `tar zxvf mediawiki-*.tar.gz`.

Move MediaWiki to the web server’s root document by using `sudo mv mediawiki-1.xx.x/* /var/www/` (substituting the applicable version number).

Finally, point a browser window to the root URL of your server (for example http://123.456.789.012), and you should be greeted with a MediaWiki page.  You’ll see an error message “LocalSettings.php not found, please set up wiki first.”  Click the link and run through the setup process.  

It will guide you through the process of setting up MediaWiki.  You’ll be asked again for your MySQL database credentials (the ones we created before) so make sure to have them handy.

At the very end, MediaWiki will generate a LocalSettings.php file.  Download that to your PC and open it up with a text editor of some sort.  This is a config file for MediaWiki, and needs to placed at the root level of your web server. 

This can be done very simply.  Copy all of the text from the generated LocalSettings.php file.  In a console window, run `sudo nano /var/www/LocalSettings.php`.  Paste the entire text from the generated file into the file you are creating on your server.  Save the file (`CTRL-X` and then `Y`) and refresh the window on your browser.




## Finishing Up/ Securing Your Droplet
If you were able to see the default front page for MediaWiki at the root URL of your server, you have configured everything correctly.  But before you carry on, we want to secure your server with some easy security settings.

Let’s first disable responses to ICMP requests, more commonly known as ping.  People looking to attack servers will often run bulk ping scans of all IP addresses, and if your server doesn’t reply, it’s less likely to be attacked.

We can disable ICMP/Ping by running the following command:
```
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all
```
If you run a continuous ping on your server while running this command (`ping <your server IP> -t` on Windows or simply `ping <your server IP>` on Mac or Unix), you should notice the server appears to stop responding.  If it does, you have correctly executed the command.

Finally, we want to enable some firewall rules to block all but the traffic we want, that being traffic to and from our webserver.  We could edit our IPTables, but there is a much simpler program that will make our lives easier, and it’s called `ufw` or Uncomplicated FireWall.

First, make sure `ufw` is installed by using `sudo apt-get install ufw`.  Assuming it is, we can move on to the next step.

You will want to do the final firewall configuration form the Digital Ocean browser console access.  We will temporarily be blocking all IP traffic, so it will end your SSH session if you are using one.  To access the console application, log into your Digitial Ocean account, select the droplet you are working on, and under the “Access” section, click the blue “Console Access” button.  This would be the equivalent of plugging a console cable into the front of the server, allowing us to configure firewall rules without interruption.

After the page loads, press your return key a time or two, and you will be prompted to login.  After logging in, we can configure `ufw`.

Let’s first configure `ufw` to deny all incoming and outgoing traffic by default.  We can do this using by running the command `sudo ufw default deny`.  It’s best to deny everything and then re-allow only what we need, that way we’re sure to not leave any IP ports open by mistake.

To test that the firewall is working, try visiting your MediaWiki site and also creating a new SSH session.  If nothing works, your firewall is fully enabled.


We now want to re-allow ports 80, 443, and 22 (the ports for regular http, https, and SSH traffic respectively.)  We could manually allow them using a command like `ufw allow 80`, but ufw has pre-defined sets that will do all of that for us.  Execute the following commands:
```
sudo ufw allow “lighttpd full”
sudo ufw allow “OpenSSH”
```

If you executed those commands correctly, both your web page and SSH session should work again (you may need to open a new SSH session).

With that, you have now set up a functioning and secure MediaWiki server!  Please feel free to consult further documentation on DigitalOcean or on the MediaWiki website if you have further questions, or stop by our forums.  Happy Hacking!


