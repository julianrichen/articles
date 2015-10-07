# Create a LEMP Stack on Debian/Ubuntu + Security Tweaks
## Setup a robust and powerful LEMP server tweaked for security.

**Published:**  2015/04/09 (April 9 2015)

## Introduction
In this article we are going to cover a wide range of topics when it comes to creating a web server for your application. Most people know what a LEMP stack is and how to install a basic one but actually configuring it and more importantly securing it is over looked. Simple things like allowing directory traversal, showing software version number, and not restricting directory permissions can land you in a whole world hurt when you are targeted. As always it is up to you to stay up-to-date with security vulnerabilities but having a secure baseline to work off is always nice.

## Server Information
For this tutorial I'll be using [Digital Ocean](https://www.digitalocean.com/?refcode=8823ef2f59e4) running a basic droplet with:
* Debian 7x64
* 1 CPU core
* 512MB RAM
* 20GB of SSD disk space

More then enough for a few sites with low to median load.

## Create sudo account for your self
First thing we are going to do is create our own account. Go ahead and login with your root account and `sudo su` in.
```bash
$ ssh root@YOUR_IP_ADDRESS
$ sudo su
```

Once that is done let's create our own account:
```bash
$ sudo adduser <username>
```
Enter a strong password (8+ alphanumeric with special characters at a minimum) and if you so desire all the other information.

Next we want to add our self to the sudeors, this allows use to have the same permissions as root but without using root.
```bash
$ sudo adduser <username> sudo
```

Adding the user to the sudo group is better then doing `visudo` and editing the `/etc/sudeors` because sudeors already grants permissions to the sudo group. Adding yourself to `/etc/sudeors` defeats the purpose of having elevated permissions with the `sudo` command.

At this point you can logout and login with your new account. Before continuing to the next step make sure you can `sudo su` and can gain access to all root privileges. Now go ahead and put very strong password to the root account (if you have not already) and leave it be, however, it is recommend you go ahead and remove remote SSH login.

This is highly recommended since root is a common username many people will attempt to gain access to your server, using your own account will allow you to monitor roots status and see if anyone has attempted to gain access.

To disable root SSH edit the `sshd_config`.
```bash
$ sudo nano /etc/ssh/sshd_config
```

Find `PermitRootLogin` and change it to no:
```
PermitRootLogin no
```

Restart the `sshd` daemon.
```bash
$ sudo service ssh restart
```

If you need to access root user you can use the built in terminal with many VPS solutions, however, you will be using your root account you created above for everything to follow.

## Before we continue
Lets update & upgrade our system with the latest packages.
```bash
$ sudo apt-get update && sudo apt-get upgrade
```

When the prompt comes up select `Y` to install new updates. Make sure you resolve any broken repositories or issues that are pointed out from the above command.

## Installing nginx
Nginx is your web server similar to apache. Installing nginx is as simple as `apt-get`-ing the package `nginx`.
```bash
$ sudo apt-get install nginx
```

Nginx might not have started, run the follow to start it.
```bash
$ sudo service nginx start
```

You should now be able navigate to the web server IP and see the default static page. If you need to find your server's ip you can run:
```bash
$ ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```

Once we confirm that everything is working go ahead and add it to the default daemons to load on boot.
```bash
$ sudo update-rc.d nginx defaults
```

### Change Web Root
If you are moving from apache then you are familiar with `/var/www`, Nginx uses `/usr/share/nginx/www` or `/usr/share/nginx/html`. I personally like `/var/www` more and will be using it for the rest of this tutorial. Our files will be stored in a sub directory of `public_html` and any vhost will be `/var/www/example.com/public_html`. To change the root follow these steps.

First make your directory & apply the appropriate permissions.
```bash
$ sudo mkdir /var/www
$ sudo chown -R www-data:www-data /var/www
$ sudo chmod -R 775 /var/www
```

We set the permissions as www-data:www-data as that is the user that runs nginx. If you are unsure what user runs nginx and believe it might be different you can check the first line of the `/etc/nginx/nginx.conf` file.

Now open up the default sites-available.
```bash
$ sudo nano /etc/nginx/sites-available/default
```

And replace any instance of `/usr/share/nginx/www` with `/var/www`, most comanly it will be seen after the `root` parameter inside a `server { }` block:
```
server {
    . . .
    root /var/www;
    . . .
}
```

Save and restart nginx.
```bash
$ sudo service nginx restart
```

### Improve nginx
To improve the quality of our nginx server we are going to do a few tweaks. First run the following commands, notice the lack of `sudo` on the second command. `ulimit` will not run with `sudo`.
```bash
$ sudo grep processor /proc/cpuinfo | wc -l
$ ulimit -n
```
The first will return the number of cores and the second the amount of connections we can have (ex. 512MB ram server is generally ~= 1024)

Now open up the `nginx.conf`.
```bash
$ sudo nano /etc/nginx/nginx.conf
```

and change the `worker_processes & `worker_connections` according to what is returned above. Here is an example with 1 core and 512MB ram machine:
```
worker_processes 1;
worker_connections 1024;
```

You can also attempt to set `worker_processes` to `auto` as it takes into account CPU core(s), disk array, etc... Leaving it the default is also not a bad thing.

`worker_processes`
The number of process (or cores) of the machine

`worker_connections`
The number of simultaneous connections the server can handle

It is also recommended you lower the `keepalive_timeout` timeout to say 10 ~ 15 instead of the default 65. `keepalive_timeout` tells nginx to keep a connection alive for 65 seconds then kill it. Lowering it will help against Timeout/DDOS attacks.
```
keepalive_timeout = 15;
```

Next I recommend hiding the nginx version number from the outside world. This can be done by setting the `server_tokens` to `off`:
```
server_tokens off;
```

While this seems trivial, knowing your server version can give attackers an idea of what vulnerability are already known of that version of nginx. Never give more information then you need to.

Next I would implement Gzip compression to save on bandwidth. Don't go crazy on these settings or you will overload the CPU. Here are some good settings on server I listed above.
```
gzip             on;
gzip_comp_level  2;
gzip_min_length  1000;
gzip_proxied     expired no-cache no-store private auth;
gzip_types       text/plain application/x-javascript text/xml text/css application/xml;
```

With those settings changed go ahead and save the file and jump over to `/etc/nginx/sites-available/default`.
```bash
$ sudo nano /etc/nginx/sites-available/default
```

Along side gzip you might also consider setting an expire time for images, css & js. This will cache the data for the user on the next visits. Add the following snippet inside the `server {}` block. This will target a specific site on your server and can be customized for each site.
```
# Add:
location ~* .(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 365d;
}
```

The above are the main configs I would change to see a difference in your server. You can find more specific configs on the nginx site or use [this article](https://www.digitalocean.com/community/tutorials/how-to-optimize-nginx-configuration).


Apply your changes by restart nginx.
```bash
$ sudo service nginx restart
```

## Installing MariaDB
You might be asking your self two questions:
1. What is MariaDB?
2. I know what MariaDB is but why use it over MySQL?

The simplest answer is MariaDB is a drop-in replacement for MySQL with improvements around the board. It works with MySQL databases and has the same queries as MySQL. A few years back Oracle/Sun took over MySQL and many of the people working on MySQL including the creator left because they did not like the direction the company was taking. They forked MySQL and made an alternative which is faster, more secure and has big backers like Google working on it.

If you are transitioning a server from MySQL to MariaDB you need to purge the old files, this does not apply to fresh installs.
```bash
$ sudo apt-get purge mysql*
$ sudo apt-get autoremove
```

There are two methods of installing MariaDB (2nd method preferred). First you *might* be able to get the latest stable version from your existing source-list by running the following command:
```bash
$ sudo apt-get install mariadb-server mariadb-client
```

This method might work, however, it is suggested to follow the second method.

The second method to installing MariaDB is adding the MariaDB repository straight from MariaDB.org. [Visit the download section of MariaDB](https://downloads.mariadb.org/mariadb/repositories/) and choose your distro, release version, MariaDB version & mirror. Here is an example for Debian 7 using MariaDB 10.0 repository from Digital Ocean. I chose MariaDB version 10.0 since it is the latest and greatest, version 5.5 is a back-port of MySql 5.6.
```bash
$ sudo apt-get install python-software-properties
$ sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 0xcbcb082a1bb943db
$ sudo add-apt-repository 'deb http://nyc2.mirrors.digitalocean.com/mariadb/repo/10.0/debian wheezy main'
```

Once the repository is added refresh your source-list and install MariaDB server & client.
```bash
$ sudo apt-get update
$ sudo apt-get install mariadb-server mariadb-client
```

You will be asked to enter a root password for MariaDB and then the instillation will finish. Go ahead and check the mysql version & and your login credentials with:
```bash
$ sudo mysql -v -u root -p
```

Now let's lock down MariaDB, `mysql_install_db` installs a basic database layout and `mysql_secure_installation` secures mysql.
```bash
$ sudo mysql_install_db
# Follow prompts

$ sudo mysql_secure_installation
# Enter password
# Change the root password?     -> N (You should have already done this)
# Remove anonymous user?        -> Y
# Disallow root login remotely? -> Y (unless you need it, normally you don't)
# Remove test database?         -> Y
# Reload privilege tables?      -> Y (must be done for changes to take effect!)
```

Finally lets check the status of mysql to make sure everything is good:
```bash
$ sudo service mysql status
```

If everything checks out we can install PHP.

## Installing PHP
If you have installed a LAMP stack on another machine then you might find install PHP on nginx a little bit more complicated but fret not! Installing PHP is only a few extra steps.

First install the following packages:

* **[php5-fpm](https://packages.debian.org/sid/php5-fpm):** The core php code using using FastCGI (recommended over php5 for nginx).
* **[php5-mysql](https://packages.debian.org/sid/php5-mysql):** Includes mysql, mysqli and pdo_* modules.
* **[php5-mcrypt](https://packages.debian.org/sid/php5-mcrypt):** Provide mcyrpt module for hashing passwords.

```bash
$ sudo apt-get install php5-fpm php5-mysql php5-mcrypt
```

Second we need to do a security tweak, open up your php.ini file:
```bash
$ sudo nano /etc/php5/fpm/php.ini
```

Next find the line with `cgi.fix_pathinfo`, a CTRL+W will allow you to search in `nano`. The line should be commented out with a semicolon `;` and set to 1. Uncomment the line and set it to 0:
```
cgi.fix_pathinfo=0
```

Restart php to apply the changes.
```bash
$ sudo service php5-fpm restart
```

Now we need to configure nginx to use PHP, open up your default sites-available file:
```bash
$ sudo nano /etc/nginx/sites-available/default
```

We are about to make a number of changes and below is the full changed file, edit the settings as needed.

The changes summarizes are:
1. Adding `index.php` to the list of index files.
2. Adding 404, 500, 502, 503, 504, etc.. error pages.
3. Adding fastcgi_* to enable php files.
4. Change exmaple.com to your domain name or use an ip address.

The fifth and optional setting is to change `/usr/share/nginx/html` to `/var/www` like I mentioned above. I and many other users prefer to have the web servers root in the commonly located `/var/www` location, however, this is entirely preference. You could also set the root to a users home directory like many share hosts do but again this is up to you.

I have attempted to comment out as much as possible to explain all the functions.
```
server {
    # You can only have IPv4 or IPv6 enabled at one time
    listen 80 default_server; # IPv4
    # listen [::]:80 default_server ipv6only=on; #IPv6

    # Root for files
    root /var/www;
    # index; The file that is searched for in each directory as the index
    # This could be anything like default.html, which is similar to an aspx environments
    index index.php index.html index.htm;

    # Domain name or the server's ip
    # Can accept more then one domain by separating by a space
    # ex. server_name exmaple.com example.net;
    server_name example.com;

    # Default location {} block for routing
    location / {
        # Try to find file, else throw 404 error
        try_files $uri $uri/ =404;
    }

    # Depending on the error redirect
    # Make sure you have these files
    # 404 error, redirect to /404.html
    error_page 404 /404.html;
    # 500-504 error, redirect to /50x.html
    error_page 500 502 503 504 /50x.html;

    # Route php with FastCGI
    # This MUST be done for php to work
    # Match all php files
	location ~ \.php$ {
		# Try to find file, else throw 404
		try_files $uri =404;
		# FastCGI magic
		fastcgi_split_path_info ^(.+\.php)(/.+)$;
		fastcgi_pass unix:/var/run/php5-fpm.sock;
		fastcgi_index index.php;
		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
		include fastcgi_params;
	}
}
```

Restart nginx to enable the changes.
```bash
$ sudo service nginx restart
```

### Testing php
You can do a qucik test to see if PHP is working by creating a standard phpinfo.php file
```bash
$ sudo nano /var/www/phpinfo.php
<?php echo phpinfo(); ?>
```

Save the file, navigate to your server `YOUR_IP_ADDRESS/phpinfo.php`.

If the the php configuration returns and no errors are shown go ahead and remove the file for security reasons.
```bash
$ rm /var/www/phpinfo.php
```

The next section we learn how to configure multiple domains on one server.

## Setting-up up a vhost for each site
If you are asking your self "What's vhost" then chances are you don't need it, however, for scaling reasons I suggest you read this section and implement it anyway even if you only have one site.

A vhost allows you to host several websites on one server, even if you only plan on having one site/application on the server you might as well setup a vhost. If you plan on adding a new site/application it makes it much easier, as well you get to customize each vhost to have specific settings for each site/application.

First we need to create the individual directories for each site, following the changes made above we will be using `/var/www` instead of the default nginx directory. In this example we are adding the site example.com to our vhost so we create `example.com/public_html` in our `/var/www`.
```bash
$ sudo mkdir -p /var/www/example.com/public_html
$ sudo chmod -R 775 /var/www
```

The `/var/www/example.com/public_html` will be the root of the site, make sure example.com reflects your domain name. You generally want a `public_html` directory that way you can store sensitive data outside the public web root of your site/application.

There are two methods of setting-up vhosts, while I prefer the first method but I will explain both.

### First method
First you can use the `/etc/nginx/sites-available/default` file and have multiple `server {}` blocks for each domain, this makes it easy to manage every domain since it is one file. This is an example:

```
server {
    listen 80;
    #listen [::]:80 ipv6only=on;

    root /var/www/example.com/public_html;
    index index.php index.html index.htm;

    server_name example.com;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
server {
    listen 80;
    #listen [::]:80 ipv6only=on;

    root /var/www/example.net/public_html;
    index index.php index.html index.htm;

    server_name example.net;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
server {
    . . . domain 3 information . . .
}
server {
    . . . domain 4 information . . .
}
```

A specific `root` & `server_name` are required. As well if you have an SSL cert then change the port 80 to port 443. You should also only include the blocks you need, for example if the site is only made up of html, css, javascript and image you don't need the PHP/FastCGi block.

### Second method
The second method which in my opinion is the cleaner method but more fragmented is great if you want to split the vhost `server { }` blocks up into different files. First copy (command `cp`) the default sites-available file to a new file, notice we are using "example.com". Make sure the domain name reflects your site, this really helps to keep everything organized.
```bash
$ sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/example.com
$ sudo nano /etc/nginx/sites-available/example.com
```

Make the appropriate changes to `root`, `server_namer` and `listen` (port number).

When you are ready, save the file and added it to the sites-enable with:
```bash
$ sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/example.com
```

Some people recommend removing the default to avoid conflicting server name error. If you receive the error go ahead and remove it with:
```bash
$ sudo rm /etc/nginx/sites-enabled/default
```

Or simply rename it and create a back-up a back-up in the process.
```bash
$ sudo mv /etc/nginx/sites-enabled/default /etc/nginx/sites-enabled/default.backup
```

Now restart the nginx daemon:
```bash
$ sudo service nginx restart
```

To add more vhosts simply follow the steps above for either option.

### Force www before domain (www.example.tld)
You might expect the following to force www.
```
server {
    . . .
    server_name example.com www.example.com;
    . . .
}
```

This will accept www & non-ww but won't force www. In order to force www you need to make two `server {}` blocks and have the non-www force to www, like so:
```
server {
    server_name  example.com;
    rewrite ^(.*) http://www.example.com$1 permanent;
}
server {
    server_name www.example.com;
    . . .
}
```

You can also do the reverse and force non-www. Don't forget to restart nginx before testing.
```bash
$ sudo service nginx restart
```

### Error: could not build the server_names_hash
> could not build the server_names_hash, you should increase server_names_hash_bucket_size: 32
([Source](http://nginx.org/en/docs/http/server_names.html#optimization))

If you receive the above error then you either have a long domain name, like `too.long.server.name.example.org` or to many domains.

To fix the issue open up the nginx.conf and uncomment the `server_names_hash_bucket_size` if it is not already. Next increase the value it is equal to 32, 64, or another value depending on CPU cache line size. The values are powers of 2 on 32 (32, 64, 128, etc...).
```bash
$ sudo nano /etc/nginx/nginx.conf
```
In the `http {}` block look for the following and increase the value.
```
http {
    . . .
    server_names_hash_bucket_size 64;
    . . .
}
```

Restart nginx and the issue should go away.
```bash
$ sudo service nginx restart
```

## SFTP - Secure File Transfer Protocol
At this point SFTP is not an option but a requirement, FTP is just too insecure. If you are worried this is going to be more work to setup then don't worry, it's actually easier. First off decide what user you want to allow ftp access.

### Adding an existing user with all access
If the user you want to allow ftp access already has access to the server via SSH then you are in luck just do the following:
```bash
$ sudo useradd <username> www-data
```

This will make the user part of the www-data group and have permissions in the /var/www folder. Now jump down to the section about "Connecting via SFTP".

### Adding a user, only allow sftp access & limit folders
Before this section starts I want to give a massive shout-out to the user [Time Sheep](http://superuser.com/users/180824/time-sheep) over at [SuperUser.com](http://superuser.com) for [this answer](http://superuser.com/questions/669690/chrooted-sftp-user-with-write-permissions-to-var-www). He came up with a fantastic solution for jailing users while still keeping www-data permissions for `/var/www`.

Basically we are going to jail the user to their home directory (ex. `/home/john/www`) and then mount (link) that www folder to the site the user has access to edit. We also supply the `g+s` & use `setfacl` to keep the new written data under the www-data permissions. When that is all done we save the mount in the fstab file for when the server reboots.

First make sure you have acl's enabled in your system. Depending on the host (like Digital Ocean) the fstab file might be different and won't show acl's are set even though they are. You can skip this section about enabling them and if you have an error come back and follow it.
```bash
$ nano /etc/fstab
```

Look for the acl flag
```
. . .
. . .
# /dev/sda1
UUID=<_________fs_name__________>       /    ext4 acl,realtime,errors=remount....
. . .
. . .
```

If it is not, go ahead and add it and remount the system.
```bash
$ sudo mount / -o remount
```

Second check to see if `/usr/sbin/nologin` or `/sbin/nologin` is a valid shell.
```bash
$ less /etc/shells
```

If in the return list it does not exists `CTRL+Z` to exit out and add it with:
```
# Most systems:
echo "/sbin/nologin" >> /etc/shells
# Debian/Ubuntu:
echo "/usr/sbin/nologin" >> /etc/shells
```

Re-run a less on shell to see if it was added
```bash
$ less /etc/shells
```

SFTP will only work if the shell used on the user (in this case `/usr/sbin/shell`) is in the `/etc/shells`.

Third we need to add a group for our users, we could call this anything but keep it lower-case and without spaces or numbers. For example `developer`, `sitename`, `content-advisor`, etc.. We are going to use `sftp-only` for these examples.
```
sudo groupadd sftp-only
```

Next we need to setup the sftp-server Subsystem in sshd_config, open up the file:
```bash
$ sudo nano /etc/ssh/sshd_config
```

Locate the following line in the file and comment it out (if it is not already):
```
#Subsystem       sftp    /usr/libexec/openssh/sftp-server
```

and under it add
```
Subsystem       sftp    internal-sftp
```

after that add the following at the **VERY END** of the file.
```
Match Group sftp-only
    ChrootDirectory /home/%u
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```
It must go at the very end or an error will be produced!

Save the file and restart ssh.
```bash
$ sudo service ssh restart
```

Finally we need to setup a chroot environment, this will lock the user to the `/home/<username>/www` folder which is mounted to `/var/www/example.com`. To add a new user run:

```bash
$ sudo useradd <username>
$ sudo usermod -g sftp-only -s /usr/sbin/nologin <username>
$ sudo chown root:root /home/<username>
$ sudo mkdir -p /home/<username>/www
$ sudo mount --bind /var/www/example.com /home/<username>/www
$ sudo chmod g+s /home/<username>/www
$ sudo chown -R <username>:www-data /home/<username>/www
$ sudo setfacl -d -m g::rwx /home/<username>/www
$ sudo nano /etc/fstab
# Add:
/var/www/example.com        /home/<username>/www        none        bind        0        0
```

Make sure to replace `<username>` with your user you are creating and replace the `/var/www/example.com` with the site you want the user to have access to. The user could also have access to multiple sites by linking multiple directories and creating the edits in `/etc/fstab`.

```bash
$ sudo mount --bind /var/www/example.com /home/<username>/site1
$ sudo mount --bind /var/www/example.net /home/<username>/site2
$ sudo mount --bind /var/www/example.org /home/<username>/site3
```

Make sure to apply the changes to /etc/fstab or it won't save on reboot

Make sure to set the proper permissions & add the lines in `/etc/fstab`. As well the chroot directory must be own by root for sftp to work so the line `$ sudo chown root:root /home/<username>` is very much required!

You can repeat the above for an existing user, just ignore the `useradd` and proceed to `usermod`.

I also recommend using `/usr/sbin/nologin` over `/bin/false` as the user will get a polite message if they try to SSH saying "This account is currently not available" while `/bin/false` immediately exists the user out of shell. As well we added `/usr/sbin/nologin` above to the `/etc/shell` so might as well stay consistent.

### Connecting via SFTP
You can now use your terminal and/or ftp client like FileZilla to access your server via sftp.

Make sure you test if the user is jail to the specified directory, they should also not be able to issue commands in the terminal.

Connect via terminal using sftp
```bash
$ sftp <username>@<your server's ip or domain>
```

If you want to use some kind of gui ftp client then I recommended [FileZilla](https://filezilla-project.org/). When you setup FileZilla or any ftp client use the following:
```
# Host
sftp://<server ip>
# Username
<username>
# Password
<password>
# Port
22
# Optional setting (good if you are root user and want to be redirect to /var/www by default)
# Site Manager -> Select Site -> Advanced Tab | Under "Default remote directory" add
/var/www/
```

## Installing phpMyAdmin (optional)
phpMyAdmin is a user interface written in php that allows easy access and editing of MySQL (MariaDB) via a web panel. While it is not required on your webserver it makes it very easy to manage databases.

To install phpMyAdmin `apt-get` the package
```bash
$ sudo apt-get install phpmyadmin
```

Proceed to follow the setup guide and hit `tab` to bypass the apache/lighttpd. Nginx is not currently available in the listing but will work regardless. If you are unable to skip this step you can select either apache/lighttpd, nothing will change.

Next you will be asked to configure `dbconfig-common`, select yes and follow the setup guide again. Once phpMyAdmin is installed we need to make a symbolic link, this can be done with:
```bash
$ sudo ln -s /usr/share/phpmyadmin /var/www
```

Notice we are using `/var/www` instead of the default nginx root, you might have chosen the default so change accordingly.

Once done restart nginx:
```bash
$ sudo service nginx restart
```

You should now be able to navigate to http://example.com/phpmyadmin or my.ip.address/phpMyAdmin.

### Secure phpMyAdmin
After installing phpMyAdmin it is highly recommended that you secure it, while this is not a 100% full proof solution it is better then leaving it in the open. This solution will require an extra login field to even load phpMyAdmin.

First run the following command:
```bash
$ openssl passwd
```

You will then be asked to enter a password to use and a confirmation of the password. Once done a hashed string will be created, like so:
```
.nIQQxLo3uHO6
```

Second create a password file, in this case we will call it `phpmyadmin_password`:
```bash
$ sudo nano /etc/nginx/phpmyadmin_password
```

Enter the username you wish to use alongside the newly created hash from before, in this case we are using john as our demo user:
```
john:.nIQQxLo3uHO6
```

Finally we need to make phpMyAdmin use the new security measure, open up the default config in sites-available (or the file you are using for routing):
```bash
$ sudo nano /etc/nginx/sites-available/default
```

Next add a `location {}` block for phpmyadmin. Remember that the `server {}` you place it in will be the domain you connect to. If you place it in the master `server {}` block then the ip of the server will be used. You can always place it in every `server {}` block and allow every domain to access it.
```
server {
    . . .

    location /phpmyadmin {
        auth_basic "Please authenticate yourself";
        auth_basic_user_file /etc/nginx/phpmyadmin_password;
    }
    . . .
}
```

Restart ngninx
```bash
$ sudo service nginx restart
```

If all went well navigating to http://example.com/phpmyadmin or my.ip.address/phpMyAdmin should prompt you for a model asking for authentication. Enter your credentials and then attempt to login to phpMyAdmin.

## Verify mail works (and maybe fix it)
Of the many systems I have configured I always find mail to be the worst offender. Why does it never work? Why so many daemon options like `postfix`, `sendmail` & `exim4`? And why does no one have a good solution? I feel you pain and after much research have found this to be the best solution.

Let's break down the plan.
1. Uninstall all MTA's and start from scratch
2. Use a remote smtp solution (really do it)
3. Configure everything
4. Test it!

### Mailgun
I know what your thinking and trust me NO. As much as a running your own mail server sounds romantic it's not. Yes you might be independent from *the man* but the amount of work and server resources it cost is not worth it, at least for a small site. If yo plan on scaling up then I might consider a local mail server solution. Most mail servers need at least 2GB ram, a 2 core CPU & ssl certification. Since our little VPS is only running a single core with 512MB of ram running a local mail server is not the best idea.

That's where Mailgun comes in, with a free mail account you can send up to 10,000 emails! Afraid of the pricing? Don't. 15,000 emails is only $2.50 and 50,000 emails/mo is $20 but with a free account you should be fine in most cases. As well emails sent from the server retain your domain like `noreply@example.com` or `pr@example.net` and replys can be forward to another account like `myemail@gmail.com`.

Still not convinced? Take a look at Mailgun's impressive collection of users, which include GitHub, uservoice, stripe and many others. Plus you get the finical backing of Rackspace, one of the largest storage suppliers.

If you are convinced continue on and install postfix & configure for Mailgun. I'll include the info for gmail/Google Apps just in case you need an alternative.

If you are still determined to setup a local server I recommend finding another tutorial as I will not be going into detail about it.

### Postfix + Mailgun
First go ahead and remove any traces of any MTA:
```bash
$ apt-get --purge remove qmail && apt-get --purge remove exim4 && apt-get --purge remove sendmail && apt-get --purge remove postfix
# Repeat for any you have installed
```

Next we are go to work with `libsasl2-modules` (authentication package) & `postfix` from scratch (reason why we purged it above).
```bash
$ sudo apt-get install libsasl2-modules
$ sudo apt-get install postfix
# Select Internet site
# Enter domain name
```

Follow the setup guide and when done add your smtp info into `sasl_passwd`:
```bash
$ sudo nano /etc/postfix/sasl_passwd

# Example
# [mail.isp.tld]:port username:password
# Gmail
# [smtp.gmail.com]:587 USERNAME@gmail.com:PASSWORD
# MailGun
[smtp.mailgun.org]:587 username@domain.com:PASSWORD
```

Map smftp info
```bash
$ sudo postmap /etc/postfix/sasl_passwd
```

Protect sasl_* info.
```
sudo chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
sudo chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
```

Next configure the main.cf file.
```bash
$ sudo nano /etc/postfix/main.cf
# Locate following and add your domain
myhostname = example.com

# enable SASL authentication
smtp_sasl_auth_enable = yes

# disallow methods that allow anonymous authentication.
smtp_sasl_security_options = noanonymous

# where to find sasl_passwd
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd

# Enable STARTTLS encryption
smtp_use_tls = yes

# where to find CA certificates
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt

# Add a reley host, examples:
# ISP
# relayhost = [mail.isp.tld]:port <-- Normally 587 (check)
# Gmail
# relayhost = [smtp.gmail.com]:587
# MailGun
relayhost = [smtp.mailgun.org]:587
```

Restart
```bash
$ sudo service postfix restart
```

Configure PHP, find `sendmail_path`
```bash
$ sudo nano /etc/php5/fpm/php.ini
; For Unix only.  You may supply arguments as well (default: "sendmail -t -i").
; http://php.net/sendmail-path
sendmail_path = "/usr/sbin/sendmail -t -i"
```

Restart
```bash
$ sudo service nginx restart && sudo service php-fpm restart
```

Test via terminal
```bash
$ mail -s "Testing new email" "example@example.com" <<EOF
Hello World!
Just testing the new email from my server.
EOF
```

## Create another mysql user (optional)
Before you start configuring your application with the default root mysql user you might want to consider using multiple mysql accounts. This will help mitigate worse damage if the user gains access to your application files and see the mysql username & password. While they may gain access to the data in the database they at least won't gain access to the rest of the server.

First login to mysql in the terminal, in this case we will use root:
```bash
$ mysql -u root -p
```

You will be prompted for a password, once authenticated you will be in the MySQL terminal denoted by the `mysql>`. Do not include the `mysql>` into commands.
```bash
mysql> CREATE USER '<users username>'@'localhost' IDENTIFIED BY '<users password>';
mysql> GRANT ALL PRIVILEGES ON * . * TO '<users username>'@'localhost';
mysql> FLUSH PRIVILEGES;
mysql> exit
```

Create as many users as you need for each application, make sure you grant the proper permissions for each user. For example you might only need `SELECT`, `UPDATE` & `INSERT` so you could do the following instead of the above:
```bash
mysql> GRANT SELECT, UPDATE, INSERT PRIVILEGES ON * . * TO '<users username>'@'localhost';
```

You can also select specific databases & tables too effect:
```bash
mysql> GRANT ALL PRIVILEGES ON database.table1 TO '<users username>'@'localhost';
```

Remember you can always add privilege later if you need them, no need to add more then you need!

## Setting up iptables
In GNU/Linux your firewall is called iptables. Iptables can be a daunting task as they look scary but fear not, [the Debian wiki has documentation about it](https://wiki.debian.org/iptables).

You can list the current rules you have for your ip table by doing
```bash
$ iptables -L
# Which will return (something like) the following
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

Let's add some rules for testing, to do so create a iptables.test.rules file.
```bash
$ nano /etc/iptables.test.rules
```

Allow port 80 (http), 443 (https), 22 (ssh), 20:21 (FTP inbound & outbound), ping, 25 (mail), log all attempts & reject everything else unless it's pre-established:
```
*filter

# Allow all loopback
-A INPUT -i lo -j ACCEPT
-A INPUT ! -i lo -d 127.0.0.0/8 -j REJECT

# Allow all established inbound connections
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow all outbound traffic
-A OUTPUT -j ACCEPT

# Allow HTTP/HTTPS
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 443 -j ACCEPT

# Allow inbound SSH
-A INPUT -p tcp --dport 22 -j ACCEPT

# Allow FTP/SFTP
-A INPUT -p tcp --dport 20:21 -j ACCEPT

# Allow ping
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT

# Mail
-A INPUT -p tcp --dport 25 -j ACCEPT
-A OUTPUT -p tcp --dport 25 -j ACCEPT

# Log iptables denied calls
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7

# Reject all other inbound
-A INPUT -j REJECT
-A FORWARD -j REJECT

COMMIT
```

Add the new rules for testing.
```bash
$ iptables-restore < /etc/iptables.test.rules
```

Check your new rules.
```bash
$ iptables -L
```

Everything looking good? Save the new rules and add command in boot-up iptable.
```bash
$ iptables-save > /etc/iptables.up.rules
$ nano /etc/network/if-pre-up.d/iptables
```

Go ahead and add the following to the file:
```bash
#!/bin/sh
/sbin/iptables-restore < /etc/iptables.up.rules
```
Once done save the file and give it execute permissions.
```bash
$ chmod +x /etc/network/if-pre-up.d/iptables
```

Go ahead and reboot your server and rerun the iptables command.
```bash
$ sudo shtudown -r now
$ iptables -L
```

If the rules where saved then good, everything worked! To change rules simply repeat the above but make sure to test the rules before saving them as you can lock your self out. Rules will not be applied till you save them to the `/etc/iptables.up.rules`.

## Security
### Enable automatic security updates
WARNING! While this is great for updating your system there is a risk of breaking some packages. The Debian team has gotten much better since Debian 6.0 but if you are worried you might not want to do this. If you feel confident in Debian and won't have time to check & update the system your self you, follow these steps.
```bash
$ sudo apt-get install unattended-upgrades
$ sudo nano /etc/apt/apt.conf
# Add the following
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

With Debian 6.0 and lower you an use
```bash
$ sudo dpkg-reconfigure -plow unattended-upgrades
```

### Install chkrootkit & rkhunter
While Linux is generally known to be a "safer" system then Windows & Mac it does not put it out of the way of rootkits, malware and so one. `chkrootkit` & `rkhunter` are two packages designed to help find and remove rootkits, malware and so on.
```bash
$ sudo apt-get install chkrootkit rkhunter
```

To run `chkrootkit` & `rkhunter` use these commands.
```bash
# chkrootkit
# Run a system scan
$ sudo chkrootkit

# Scan a directory scan
$ sudo chkrootkit -p /dir/to/check

# rkhunter
# Update
$ sudo rkhunter --update

# Run a system scan
$ sudo rkhunter --check

# CRun a directory scan
$ sudo rkhunter --bindir /dir/to/check

# Check logs
$ sudo nano /var/log/rkhunter.log
```

When you install a new program `rkhunter` will generally throw a `[WARNING]` about an unknown new program. If the program that alerts rkhunter is ok you can run rkhunter with the `--propupd` argument to ok for the next scan.

Example:
```bash
# Ok warnings
$ sudo rkhunter --propupd
# Update
$ sudo rkhunter --update
# Re Run
$ sudo rkhunter --check
```

It is recommended to put `chkrootkit` & `rkhunter` on a cron-job and email yourself a report, we will do this below!

### Install ClamAV
ClamAV is a virus scanner that is cross platform and will search for virus, malware, etc... even if it is not Linux specific. The database is also updated every hour (sometimes minutes for pro users) and is considered one of the best anti-virus for free users. First we need to install ClamAV.
```bash
$ sudo apt-get install clamav
```

Second we need to update the ClamAV database
```bash
$ sudo freshclam
```

Third run a scan
```
# Check entire system
$ sudo clamscan -r /

# Check /var/www
$ sudo clamscan -r /var/www

# Log to file
$ sudo mkdir /var/log/clamscans
$ sudo clamscan -r / -l /var/log/clamscans/`date +%Y-%m-%d`.log
$ sudo nano /var/log/clamscans/`date +%Y-%m-%d`.log

# Check entire system and alert when something is found
$ sudo clamscan -r --bell -i /

# Check entire system & remove infected files
$ sudo clamscan -r --remove /

# Kitchen sink
# Get a fresh database, full scan, sound bell, show only infected & remove infected from entire system
$ sudo freshclam | sudo clamscan -r -bell -i --remove /

# More info
$ sudo clamscan --help

```

Run when possible, especially if users can upload their own data & files. I should also point out that you might receive this error:
```
LibClamAV Warning: fmap: map allocation failed
LibClamAV Error: CRITICAL: fmap() failed
```

This error is caused when you try to scan files larger than 2.7GB and don't have enough RAM. Nothing you can do but upgrade your server with more ram.

### Fail2ban
You SSH connection will be one of the best attack vectors for malicious users, lucky we have Fail2Ban which helps protect SSH. Fail2Ban audits log files searching for users that repeat offending actions like failed login attempts.

Let's install `fail2ban` and create a copy of the default fail2ban configuration files.
```bash
$ sudo apt-get install fail2ban
$ cd /etc/fail2ban
$ sudo cp jail.conf jail.local
$ sudo nano jail.local
```

When you open the new `jail.local` you will see a section called `[DEFAULT]`. The default settings are well configured already but let's take a look at a few we might want to change.

| Config      | Description
|-------------|-------------
| `ignoreip`  | The IPs fail2ban should ignore, for example your home and office IPs. Each IP is seperated with a space.
| `bantime`   | The default ban time is 10 minutes but you can raise/lower it.
| `maxretry`  | The number of tries before a ban, 3 is the default.
| `banaction` | The action to take when a user is banned. This can be found in `/etc/fail2ban/action.d`.
| `action`    | The action fail2ban will take, by default it is `action_*` and will call an action based off port, ip, program, etc...
| `destemail` | The email to send a ban report to.
| `mta`       | The mail delivery program fail2ban should use.

Later in the file you will start to notice application specific sections.
```
[application_name]
. . .
```

You can always create your own application section with unique configs.
```
[nginx]
. . .
. . .
. . .
. . .
```


Change your settings (if you need to) & restart fail2ban.
```bash
$ service fail2ban restart
```

You can see the effected iptables that fail2ban added (if any)
```bash
$ sudo iptables -L
```

Fail2ban will override your old iptables and dynamically block request. If you are wondering why we added iptables above then did this solution then let me explain. Fail2ban will build off the iptables, as well if fail2ban fails and goes down we always have our stable static iptables to fall back on!

Enjoy a greater sense of security.

### Logwatch (very optional)
I would personally ignore Logwatch since we will implement our own solution but I figured it would be handy for some users.

Logwatch is a powerful log analyser that helps to locate issues reported in logs. It's recommended to run logwatch daily for issues and send reports. Logwatch is a very minimal program and no performance hit should be noticed on the server.

Go ahead and install logwatch with `apt-get` & start it.
```bash
$ sudo apt-get install logwatch
$ sudo service logwatch start
```

The configs for logwatch can be found in `logwatch.conf`, go ahead and open it up.
```bash
$ nano /usr/share/logwatch/default.conf/logwatch.conf
```

| Config        | Description
|---------------|-------------
| `MailTo`      | The person log watch should send its reports to
| `MailFrom`    | The "From" field in the emails
| `Range`       | Default is `Yesterday`, sends reports from yesterday. I recommend `Today`.
| `Detail`      | The amount of detail in the log report, options are `Low`, `Median` & `High`.
| `Service`     | The services logwatch should track, `All` is the default. For a full list of all the programs check out `/usr/share/logwatch/scripts/services`. You can also do a list of services, see below for details.
| `DailyReport` | If you do not wan to have daily reports you should uncomment this line and supply No; ex. `DailyReport = No`. If you want daily reports simply comment the line out `# DailyReport = No`.

To select a list of services to watch you can do.
```
Service = sudo
Service = ssh
Service = nginx
Service = mysql
Service = sendmail
```

Once down configuring restart logwatch.
```bash
$ sudo service logwatch restart
```

You can also run logwatch manually by doing
```bash
$ sudo logwatch --detail Median --mailto email@address.tld --service All --range today
```

Run a `man` command on `logwatch` if you wish to see all the options.

### Get weekly server reports
Combining everything under the security section lets send our selves a weekly server update about:
1. Server uptime
2. Packages needing updating
3. Reboot required?
4. Rootkit scans (rkhunter & chkrootkit)
5. Malware scans (rkhunter & chkrootkit)
6. General virus scan (ClamAV)
7. Fail2ban bans

First we are going to create the `security_report.sh` file. Lets make a directory in the root of our server called `/security_reports/` to limit the access.
```bash
$ mkdir /security_reports/
$ cd /security_reports/
$ nano security_report.sh
# Inset bash script located below
# NOTE! Make sure to update the email variables with your emails.
$ chmod +x security_report.sh
```

Next let's test the script by running it, It will take ~ 5 or so minutes to complete. The script will spit out it's progress in the terminal.
```bash
$ ./security_report.sh
```

The script is set by default to not rerun commands/scans if the log file exists, this is great for testing but you might want to enable it if you are doing more than one scan a day since the files are stored as `/var/log/security_reports/YYYY-MM-DD.program_name.log`.

Before we continue and automate cron let's understand cron and what it is doing:
```
*   *   *   *   * /command/that/you/want/to/execute
|   |   |   |   |
|   |   |   |   +-- Day of the Week  (0 - 7)
|   |   |   +------ Month            (1 - 12)
|   |   +---------- Day of the month (1 - 31)
|   +-------------- Hour             (0 - 23)
+------------------ Minute           (0 - 59)
```

You can view the current cron jobs with the `-l` command:
```bash
crontab -l
```

Using `crontab -l` will only show the current users cron, if you need to see a specific user as root you can use the `-u` command. Say you are setting-up the security reports on a user called "Audit Bot" (`auditbot`) you can run:
```bash
crontab -e -u auditbot -l
```

This will list all cron jobs for the user auditbot.

Now let's have it automated in cron.weekly (or .daily, .hourly, .monthly). First open the cron file using crontab:
```bash
crontab -e
```

This will use the default editor & scan for errors/syntax before adding it to the list of cron jobs when you save. To change the default editor you can do:
```bash
# export EDITOR=/usr/bin/emacs
# export EDITOR=/usr/bin/vim
# and so...

export EDITOR=/usr/bin/nano
crontab -e
```

If you have any more questions check out the [Debian help page about the issue](https://www.debian-administration.org/article/56/Command_scheduling_with_cron).

Now we understand cron we can start editing. Inside cron add a record like so:
```
0   0   *   *   0 /security_reports/security_report.sh
```

This cron will run the `/security_reports/security_report.sh` command every Sunday at 00:00.

Once you save your cron you should see the following:
```
crontab: installing new crontab
```

Go ahead and verify that everything was set with:
```bash
crontab -l
```

Now wait for Sunday night and enjoy that sweet report!

#### security_report.sh
```bash
#!/bin/bash
# ------------------------------------------------------------------------------
# 1. Name the script
#    security_script.sh
# 2. Allow execute
#    $ chmod +x /path/to/filesecurity_report.sh
# 3. Place in cron
#    crontab -e
# ------------------------------------------------------------------------------

# ------------------------------------------------------------------------------
# System Information
# ------------------------------------------------------------------------------
HOSTNAME=$(hostname)
DATE=$(date +%Y-%d-%m)
# Re-run scans if they exists
FORCE_RUN=false

# ------------------------------------------------------------------------------
# Email Information
# ------------------------------------------------------------------------------
EMAIL_SUBJECT="Weekly server review for \"${HOSTNAME}\" (${DATE})."
EMAIL_FROM="~~~~~~~~~~~~~~~~~~~~PLEASSE CHANGE THIS FIELD~~~~~~~~~~~~~~~~~~~~"
EMAIL_TO="~~~~~~~~~~~~~~~~~~~~PLEASSE CHANGE THIS FIELD~~~~~~~~~~~~~~~~~~~~"
EMAIL_MESSAGE=""

# ------------------------------------------------------------------------------
# Directories
# ------------------------------------------------------------------------------
DIR_LOG="/var/log/security_reports/"
DIR_LOG_OVERVIEW="/var/log/security_reports/overview/"

# ------------------------------------------------------------------------------
# Make security directory
# ------------------------------------------------------------------------------
mkdir -p $DIR_LOG
mkdir -p $DIR_LOG_OVERVIEW

# ------------------------------------------------------------------------------
# Log location
# ------------------------------------------------------------------------------
LOG_CLAMAV="${DIR_LOG}${DATE}.clamav.log"
LOG_RKHUTNER="${DIR_LOG}${DATE}.rkhunter.log"
LOG_CHKROOTKIT="${DIR_LOG}${DATE}.chkrootkit.log"
LOG_FAIL2BAN="${DIR_LOG}${DATE}.fail2ban.log"
LOG_REPORT="${DIR_LOG_OVERVIEW}${DATE}.log"

# ------------------------------------------------------------------------------
# Remove existing log report
# ------------------------------------------------------------------------------
rm $LOG_REPORT

# ------------------------------------------------------------------------------
# Start
# ------------------------------------------------------------------------------
echo "Running security report..."

# ------------------------------------------------------------------------------
# CHKROOTKIT
# ------------------------------------------------------------------------------
if [ ! -f "$LOG_CHKROOTKIT" ] || [ "$FORCE_RUN" = true ]; then
	echo "Running CHKROOTKIT"
	chkrootkit > "$LOG_CHKROOTKIT"
	echo "Finished CHKROOTKIT"
else
    echo "CHKROOTKIT report already exists"
fi
# ------------------------------------------------------------------------------
# RKHUNTER
# ------------------------------------------------------------------------------
if [ ! -f "$LOG_RKHUTNER" ] || [ "$FORCE_RUN" = true ]; then
	echo "Running RKHUNTER"
	# rkhunter --update --check --logfile "$LOG_RKHUTNER"
	rkhunter --update --check --quiet --logfile "$LOG_RKHUTNER"
	echo "Finished RKHUNTER"
else
    echo "RKHUNTER report already exists"
fi
# ------------------------------------------------------------------------------
# CLAMAV
# ------------------------------------------------------------------------------
if [ ! -f "$LOG_CLAMAV" ] || [ "$FORCE_RUN" = true ]; then
	echo "Updating ClamAV"
    if ps ax | grep -v grep | grep freshclam > /dev/null
    then
        echo "ClamAV already updated"
    else
        freshclam
        echo "Updating complete"
    fi
	echo "Running ClamAV"
	# clamscan -r / --log="$LOG_CLAMAV"
	clamscan -r / --exclude-dir=/sys/ --quiet --log="$LOG_CLAMAV"
	echo "Finished ClamAV"
else
    echo "ClamAV report already exists"
fi

# ------------------------------------------------------------------------------
# FAIL2BAN
# ------------------------------------------------------------------------------
# Prints: [YYYY-MM-DD] [SERVICE] Banned "00.111.222.333"
if [ ! -f "$LOG_FAIL2BAN" ] || [ "$FORCE_RUN" = true ]; then
	grep -e "Ban" -e "$(date --date "-7 days")" /var/log/fail2ban.log | awk '{print "["$1"] "$5" Banned \"" $7 "\""}' > "$LOG_FAIL2BAN"
    echo "Creating Fail2Ban report"
else
    echo "Fail2Ban report already exists"
fi

# ------------------------------------------------------------------------------
# Compile report
# ------------------------------------------------------------------------------
echo "Creating report file"

echo -e "
----------------------------------------
Weekly server review for \"${HOSTNAME}\"
$(date +%Y-%d-%m)
----------------------------------------" >> "$LOG_REPORT"

# ------------------------------------------------------------------------------
#  Uptime?
# ------------------------------------------------------------------------------
echo -e "


Uptime
----------------------------------------" >> "$LOG_REPORT"
awk '{print "The system has been up for about " int(($1/3600)/24) " days; Exact time: " int($1/3600) " hour(s) " int(($1%3600)/60) " minute(s) and " int($1%60) " second(s)."}' /proc/uptime >> "$LOG_REPORT"

# ------------------------------------------------------------------------------
# Check for updates
# ------------------------------------------------------------------------------
echo -e "


Possible Upgrades?
----------------------------------------" >> "$LOG_REPORT"
echo "Possible Upgrade(s): $(apt-get -s -o Debug::NoLocking=true upgrade | grep -c ^Inst)" >> "$LOG_REPORT"
echo -e "\n" >> "$LOG_REPORT"

# ------------------------------------------------------------------------------
# Check for Reboot
# ------------------------------------------------------------------------------
echo -e "


Reboot Required?
---------------------------------------" >> "$LOG_REPORT"
if [ -f /var/run/reboot-required ]; then
	echo 'The system requires a reboot.' >> "$LOG_REPORT"
else
	echo 'No reboot required.' >> "$LOG_REPORT"
fi

# ------------------------------------------------------------------------------
# Look for Infected, Warnings & Errors
# ------------------------------------------------------------------------------
echo -e "


CHKROOTKIT Report
----------------------------------------" >> "$LOG_REPORT"
grep "INFECTED" "$LOG_CHKROOTKIT" >> "$LOG_REPORT"


echo -e "


RKHunter Report
----------------------------------------" >> "$LOG_REPORT"
grep "Warning" "$LOG_RKHUTNER" >> "$LOG_REPORT"


echo -e "


ClamAV Report
-----------------------------------------" >> "$LOG_REPORT"
grep "FOUND" "$LOG_CLAMAV" >> "$LOG_REPORT"


echo -e "


Fail2Ban Report
----------------------------------------" >> "$LOG_REPORT"
cat "$LOG_FAIL2BAN" >> "$LOG_REPORT"

# ------------------------------------------------------------------------------
# Email
# ------------------------------------------------------------------------------
EMAIL_MESSAGE="From: ${EMAIL_FROM}
Subject: ${EMAIL_SUBJECT}
Importance: High
X-Priority: 1
$(<"$LOG_REPORT")"

# ------------------------------------------------------------------------------
# Send
# ------------------------------------------------------------------------------
echo "Sending email report"
/usr/sbin/sendmail "$EMAIL_TO" <<EOF
From: $EMAIL_FROM
Subject: $EMAIL_SUBJECT
Importance: High
X-Priority: 1
$(<"$LOG_REPORT")
EOF
```

## Best of luck!
If you made it to this point then congratulations! Your server is in much better condition to fend off attacks and serve your users. While security risk still exist like Shellshock, Glibc GHOST & Heart Bleed you are at least handling the major every day problems that can cause havoc on your server. Remember to stay alert, read the latest news about security & audit your server.

Best of luck with your newly installed server!
