+++
Categories = ["Server"]
Date = "2015-03-22"
Tags = ["Server", "GitLab", "Ruby On Rails", "Apache", "MySQL", "Debian"]
Title = "Install GitLab (7.9) with Apache and MySQL on Debian 7"
Description = "Installation of GitLab 7.9 on Debian 7 with Apache and MySQL."
+++

In this article, we'll see what is a version control, for newbies, and then
we'll proceed to the installation of Gitlab on a Debian 7 machine with MySQL
and Apache.
<!--more-->
*I'm not the most qualified person concerning Ruby on Rails, so this tutorial may
not be the most perfect one.*

### A version control system, what is this ?
During shared development, it's really useful, and even necessary, to
use a version control system. Every programmer working in group is one day or
an other confronted to this kind of comments (especially during their studies):

* "Anthony, don't change the *main* file. I'm modifying it. You'll erase my
modifications!";
* "Who has the last version of the project ? Mine is completely f*cked up!".

To avoid those annoying situations, some tools exist with their pros and cons,
whose famous [SVN](http://en.wikipedia.org/wiki/Apache_Subversion) and [Git](http://en.wikipedia.org/wiki/Git_\(software\)).
With them, you can modify the project without compromising it.

### Principle
First, the system allows you to keep every source code modifications. Then,
unlike a classic cloud software which erases the distant copy of the
project by a local one, it does the merge of the local copy and the distant copy.
Therefore, two developers can write on a same file and save their modifications
without erasing their work.

From this statement, it's possible to create 2 types of version control systems:

* Centralized : The server keeps every modification brought to the source code
and the project integrity while clients only conserve the last version of the
code. Every modification are transmitted to the server then clients sync to the
server.
* Distributed : Clients keep every project change and sync between them.
Theoretically there is no need for a server, but, for ease of synchronization,
it's common to use one as a "meeting point" between clients.

SVN is part of the first category whereas Git is part of the second. So, in the
rest of this article, we will focus on distributed systems.

#### Git and GitHub, a winning couple for open source software, but not proprietary source
This model is a good one for open source developers : Everyone can contribute
to a project. There are some collaborative websites, based on Git, such as
GitHub. Important projects are located on this one, like Symfony, Ruby on Rails
or even JQuery.

Unfortunately,
[like OVH](https://www.ovh.com/fr/a1136.interview-github-octave-klaba-ovh), it
is necessary to keep your source code secret from the entire world. By example,
if you write some proprietary code. In this case, you can pay GitHub to host your
secret code or use GitLab to do the same job freely on your own server, via
Community Edition (CE). Indeed, we will see how to install this one.

### GitLab installation : Let's get started

#### Packages and dependencies

    # Add necessary repositories (as root)
    echo 'deb http://http.debian.net/debian wheezy-backports main' > /etc/apt/sources.list.d/wheezy-backports.list

    # Update (as root)
    apt-get update -y
    apt-get upgrade -y

    # Sudo installation
    apt-get install sudo -y
    # Add your username to the sudo group
    adduser username sudo
    # It's necessary to reconnect to apply modification about sudo group

    # Installation of vim as default editor
    # If you use an other text editor, you can skip this part
    sudo apt-get install -y vim
    sudo update-alternatives --set editor /usr/bin/vim.basic

    # Installation of required packages
    sudo apt-get install -y build-essential zlib1g-dev libyaml-dev libssl-dev libgdbm-dev libreadline-dev libncurses5-dev libffi-dev curl openssh-server redis-server checkinstall libxml2-dev libcurl4-openssl-dev libicu-dev logrotate python-docutils pkg-config cmake libkrb5-dev libxslt1-dev nodejs

    # Git installation
    sudo apt-get install -y git-core

    # Postfix installation
    sudo apt-get install -y postfix

#### Ruby

    # Delete ruby1.8
    sudo apt-get remove ruby1.8

    # Ruby 2.1.5 installation
    mkdir /tmp/ruby && cd /tmp/ruby
    curl -L --progress http://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.5.tar.gz | tar xz
    cd ruby-2.1.5
    ./configure --disable-install-rdoc
    make
    sudo make install

    # Bundler Gem installation
    sudo gem install bundler --no-ri --no-rdoc

#### Users

    # Gitlab user creation
    sudo adduser --disabled-login --gecos 'GitLab' git

#### MySQL database

    # MySQL Installation
    sudo apt-get install -y mysql-server mysql-client libmysqlclient-dev

    # Connection to MySQL database
    mysql -u root -p

    # Gitlab user creation
    # Replace 'yourPassword' by a password of your choice
    mysql> CREATE USER 'gitlab'@'localhost' IDENTIFIED BY 'yourPassword';

    # Gitlab table creation
    mysql> CREATE DATABASE IF NOT EXISTS `gitlabhq_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;

    # Rights management of the new table
    mysql> GRANT SELECT, LOCK TABLES, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON `gitlabhq_production`.* TO 'gitlab'@'localhost';

    # Disconnect from mysql console
    mysql> \q

    # Verification of recent modifications
    sudo -u git -H mysql -u gitlab -p -D gitlabhq_production

    # If you can connect, everything is correct and you can leave the mysql console
    mysql> \q

#### Redis

    # Redis server installation
    sudo apt-get install redis-server

    # Copy server configuration to use sockets
    sudo cp /etc/redis/redis.conf /etc/redis/redis.conf.orig

    # Deactivate Redis listening on TCP by setting 'port' parameter to 0
    sed 's/^port .\*/port 0/' /etc/redis/redis.conf.orig | sudo tee /etc/redis/redis.conf

    # Activate Redis socket for default path under Debian and sub-distros
    echo 'unixsocket /var/run/redis/redis.sock' | sudo tee -a /etc/redis/redis.conf

    # Give access to socket for every member of redis group
    echo 'unixsocketperm 770' | sudo tee -a /etc/redis/redis.conf

    # Socket directory creation
    mkdir /var/run/redis
    chown redis:redis /var/run/redis
    chmod 755 /var/run/redis
    # Make the directory persistent, if possible
    if [ -d /etc/tmpfiles.d ]; then
      echo 'd  /var/run/redis  0755  redis  redis  10d  -' | sudo tee -a /etc/tmpfiles.d/redis.conf
    fi

    # Activate modifications of redis.conf
    sudo service redis-server restart

    # Add git to Redis group
    sudo usermod -aG redis git

##### Gitlab configuration

    # Go to gitlab directory
    cd /home/git/gitlab

    # Copy GitLab example configuration
    sudo -u git -H cp config/gitlab.yml.example config/gitlab.yml

    # Change configuration as follow :
    # Change 'host' value by the web address of your gitlab (example: 'gitlab.yourwebsite.com')
    # Change 'email_from' value by your gitlab adress (example: 'gitlab@votresite.com')
    sudo -u git -H editor config/gitlab.yml

    # Give write access on log and tmp to Git
    sudo chown -R git log/
    sudo chown -R git tmp/
    sudo chmod -R u+rwX,go-w log/
    sudo chmod -R u+rwX tmp/

    # Directories creation for satellites
    sudo -u git -H mkdir /home/git/gitlab-satellites
    sudo chmod u+rwx,g=rx,o-rwx /home/git/gitlab-satellites

    # Give write access to tmp/pids, tmp/sockets and public/uploads
    sudo chmod -R u+rwX tmp/pids/
    sudo chmod -R u+rwX tmp/sockets/
    sudo chmod -R u+rwX  public/uploads

    # Copy unicorn configuration
    sudo -u git -H cp config/unicorn.rb.example config/unicorn.rb

    # Display number of processors
    nproc

    # Authorize cluster mode if you expect a lot of charge on your server
    # Correct the number of processus to what `nproc` displays
    sudo -u git -H editor config/unicorn.rb

    # Copy Rack Attack configuration example
    sudo -u git -H cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb

    # Configuration of global parameters for git user, usefull for web edition.
    # Change user.email by the address you have written in gitlab.yml
    sudo -u git -H git config --global user.name "GitLab"
    sudo -u git -H git config --global user.email "example@example.com"
    sudo -u git -H git config --global core.autocrlf input

    # Copy Redis configuration
    sudo -u git -H cp config/resque.yml.example config/resque.yml

    # Change path to socket if you do not use basic configuration (if you have
      installed Redis for the first time, there is no need for this).
    sudo -u git -H editor config/resque.yml

#### Gitlab database configuration

    # Copy MySQL configuration
    sudo -u git cp config/database.yml.mysql config/database.yml

    # Update username and password in config/database.yml
    # You just need to adapt production parameters (first part)
    # If you have correctly followed this tutorial, do as follow :
    # Change username value (actually'git') by 'gitlab'
    # Change password value (actually 'secure password') by the password you have given to MySQL table
    # You can keep quotes around password
    sudo -u git -H editor config/database.yml

    # Make the database configuration readable only for git (protecting the password)
    sudo -u git -H chmod o-rwx config/database.yml

#### Gems installation

    # Gems installation for MySQL (option say 'without ... postgres ...')
    sudo -u git -H bundle install --deployment --without development test postgres aws

#### GitLab Shell Installation

    # Launch gitlab-shell installation (replace 'REDIS_URL' if needed)
    sudo -u git -H bundle exec rake gitlab:shell:install REDIS_URL=unix:/var/run/redis/redis.sock RAILS_ENV=production

    # By default, gitlab-shell configuration is generated from Gitlab main configuration
    # You can read it (and modify it) by executing the following command:
    sudo -u git -H editor /home/git/gitlab-shell/config.yml

#### Database Initialisation and activation of advanced options

    # Launch database installation
    sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production

    # Enter 'yes' when it will be asked : It will only create tables
    # Once the script terminated, take note of login informations after 'Administrator account created:'

#### Initialisation script installation

    # Copy initialisation script
    sudo cp lib/support/init.d/gitlab /etc/init.d/gitlab

    # Update system to launch Gitlab at reboot
    sudo update-rc.d gitlab defaults 21

#### Post-Installation

    # Set up logs
    sudo cp lib/support/logrotate/gitlab /etc/logrotate.d/gitlab

    # GitLab configuration verification
    sudo -u git -H bundle exec rake gitlab:env:info RAILS_ENV=production

    # Restart GitLab
    sudo /etc/init.d/gitlab restart

    # Compile static files to accelerate GitLab (it can take a moment)
    sudo -u git -H bundle exec rake assets:precompile RAILS_ENV=production

#### Apache configuration

You just have to configure Apache and you will be able to enjoy your installation
So, we will create a file "gitlab.yourwebsite.com" in `/etc/apache2/sites-available`
with following configuration (do not forget to change the website address in this
configuration).

    <VirtualHost \*:80>
      ServerName gitlab.example.com
      ServerSignature Off

      ProxyPreserveHost On

      <Location />
        Order deny,allow
        Allow from all

        ProxyPassReverse http://127.0.0.1:8080
        ProxyPassReverse http://gitlab.example.com/
      </Location>

      RewriteEngine on
      RewriteCond %{DOCUMENT_ROOT}/%{REQUEST_FILENAME} !-f
      RewriteRule .* http://127.0.0.1:8080%{REQUEST_URI} [P,QSA]

      # needed for downloading attachments
      DocumentRoot /home/git/gitlab/public

      # Set up apache error documents, if back end goes down (i.e. 503 error) then a maintenance/deploy page is thrown up.
      ErrorDocument 404 /404.html
      ErrorDocument 422 /422.html
      ErrorDocument 500 /500.html
      ErrorDocument 503 /deploy.html
    </VirtualHost>

Then, you only have to activate the configuration with Apache restart :

    # Website activation
    sudo a2ensite gitlab.nomDeVotreSite.com

    # Modules activation (to match with the previous configuration)
    sudo a2enmod proxy_http
    sudo a2enmod proxy
    sudo a2enmod rewrite

    # Restart Apache
    sudo service apache2 restart

Voilà ! Now, you can log into your new installation, with login informations
displayed previously, on `gitlab.yourwebsite.com` and go on ! (Do not forget to
change your password).

Sources :
[gitlab.com (section 'installation')](https://github.com/gitlabhq/gitlabhq/blob/master/doc/install/installation.md), [ezerdeniz.fr](http://eserdeniz.fr/articles/view/4/installer-gitlab-sous-debian-7-avec-nginx-et-mysql)
