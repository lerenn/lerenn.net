+++
Categories = ["Serveur"]
Date = "2015-03-22"
Tags = ["Serveur", "GitLab", "Ruby On Rails", "Apache", "MySQL", "Debian"]
Title = "Installer GitLab (7.9) avec Apache et MySQL sur Debian 7"
Description = "Installation de GitLab 7.9 sous Debian 7 avec Apache et MySQL."
+++

Dans cet article, nous allons aborder le sujet des gestionnaires de version pour les néophytes puis nous passerons à l'installation proprement dite de GitLab sur une Debian 7 avec MySQL et Apache. <!--more-->

N'étant pas la personne la plus à l'aise avec Ruby on Rails, il se peut que ce tuto ne soit pas la perfection même, mais je reste ouvert à vos propositions !

### Un gestionnaire de versions, késaco ?
Avant de rentrer dans le vif du sujet, remémorons-nous (ou découvrons) un peu le sujet : Lors du développement mutualisé de programmes, il est fort utile voir indispensable d'utiliser un gestionnaire de version. Tout programmeur se retrouvant à travailler en groupe est un jour ou l'autre confronté à ce genre de phrases :

* "Anthony, ne change pas le fichier main car je suis en train de le modifier : Tu vas écraser mes modifications !";
* "Qui a gardé la dernière version du projet ? La mienne est complètement boguée !".

Pour éviter ce genre de situations désagréables, certains logiciels existent avec leur avantages et leurs inconvénients dont les célèbres [SVN](http://fr.wikipedia.org/wiki/Apache_Subversion) et [Git](http://fr.wikipedia.org/wiki/Git). Ils permettent à chacun de modifier le projet sans pour autant gêner le travail des autres.

#### Principe de fonctionnement
Tout d'abord, le gestionnaire permet de conserver toutes les modifications apportées au code source. Ensuite, contrairement à un logiciel de Cloud classique qui écrase la copie distante du projet par la copie locale, il effectue la fusion de la copie locale et de la copie distante. Ainsi deux développeurs peuvent écrire sur un même fichier et sauvegarder leurs modifications sans écraser mutuellement leur travail.

A partir de cela, il est possible de créer 2 types de gestionnaires de versions :

* Centralisé : Le serveur conserve toutes les modifications et l'intégralité du projet, tandis que les clients ne conservent que la dernière version du code. Les modifications sont apportées au serveur puis répercutées chez tous les clients.
* Distribués : Les clients conservent toutes les modifications du projet et se les transmettent entre eux. Il n'y a en théorie pas besoin de serveur mais, pour plus de facilité, il est courant d'en utiliser un comme point de rencontre entre les clients.

SVN fait partie de la première catégorie tandis que Git fait partie de la deuxième catégorie. Dans la suite, nous nous intéresserons donc au deuxième logiciel.

#### Git et GitHub, un combo gagnant du logiciel libre, mais pas du code propriétaire
Ce modèle est donc l'idéal pour les développeurs du monde libre : Chacun peut apporter sa pierre à l'édifice. Il existe donc des sites web collaboratifs, basés sur Git, tels que GitHub. De gros projets sont hébergés par ce dernier comme Symfony, Ruby on Rails ou encore jQuery.

Malheureusement, [à l'image d'OVH](https://www.ovh.com/fr/a1136.interview-github-octave-klaba-ovh), il est parfois nécessaire de ne pas laisser ses sources à la portée de tout le monde, notamment lors de l'écriture de programmes propriétaires. Dans ce cas de figure, il est possible d'utiliser GitLab qui remplit la même tâche que le GitHub mais en privé et propose une Community Edition (CE) et une Enterprise Edition (EE). Nous allons donc nous intéresser à son installation.

### Installation de GitLab : Passons aux choses sérieuses
﻿
#### Paquets et dépendances

    # Ajout des repos nécessaires (à exécuter en temps que root)
    echo 'deb http://http.debian.net/debian wheezy-backports main' > /etc/apt/sources.list.d/wheezy-backports.list

    # Mise à jour (à exécuter en temps que root)
    apt-get update -y
    apt-get upgrade -y

    # Installation de sudo et ajout de votre nom d'utilisateur au groupe
    apt-get install sudo -y
    adduser nom_utilisateur sudo

    # Il est nécessaire de se déconnecter et de se reconnecter en
    # tant qu'utilisateur pour que l'ajout au sudo soit pris en compte

    # Installation de vim en tant qu'éditeur
    # Si vous n'êtes pas familier de vim, vous pouvez toujours utiliser votre editeur préféré !
    sudo apt-get install -y vim
    sudo update-alternatives --set editor /usr/bin/vim.basic

    # Installation des paquets requis (pour compiler Ruby et les extensions natives en gemmes Ruby):
    sudo apt-get install -y build-essential zlib1g-dev libyaml-dev libssl-dev libgdbm-dev libreadline-dev libncurses5-dev libffi-dev curl openssh-server redis-server checkinstall libxml2-dev libcurl4-openssl-dev libicu-dev logrotate python-docutils pkg-config cmake libkrb5-dev libxslt1-dev nodejs

    # Installation de Gitservice
    sudo apt-get install -y git-core

    # Installation de postfix pour que GitLab puisse envoyer des mails
    sudo apt-get install -y postfix

#### Ruby

    # Suppression de ruby1.8 s'il est présent
    sudo apt-get remove ruby1.8

    # Installation du nouveau ruby
    mkdir /tmp/ruby && cd /tmp/ruby
    curl -L --progress http://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.5.tar.gz | tar xz
    cd ruby-2.1.5
    ./configure --disable-install-rdoc
    make
    sudo make install

    # Installation de Bundler Gem
    sudo gem install bundler --no-ri --no-rdoc

#### Utilisateurs

    # Création d'un utilisateur pour le GitLab
    sudo adduser --disabled-login --gecos 'GitLab' git

#### Base MySQL

    # Installation de MySQL et de la librairie de développement (compatibilité avec gem)
    sudo apt-get install -y mysql-server mysql-client libmysqlclient-dev

    # Connexion à la base de donnée MySQL
    mysql -u root -p

    # Création de l'utilisateur gitlab
    # Remplacez 'yourPassword' par un mot de passe de votre choix et rapellez vous en : Nous allons en avoir besoin
    mysql> CREATE USER 'gitlab'@'localhost' IDENTIFIED BY 'yourPassword';

    # Création de la table de gitlab
    mysql> CREATE DATABASE IF NOT EXISTS `gitlabhq_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;

    # Gestion des droits de la table nouvellement créée
    mysql> GRANT SELECT, LOCK TABLES, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON `gitlabhq_production`.* TO 'gitlab'@'localhost';

    # Quitte la console mysql
    mysql> \q

    # Verification du bon fonctionnement des modifications
    sudo -u git -H mysql -u gitlab -p -D gitlabhq_production

    # Si vous vous connectez sans problème, quittez la console mysql
    mysql> \q

#### Redis

    # Installation du serveur redis
    sudo apt-get install redis-server

    # Configure le serveur pour utiliser les sockets
    sudo cp /etc/redis/redis.conf /etc/redis/redis.conf.orig

    # Désactive l'écoute de Redis sur TCP en paramétrant 'port' à 0
    sed 's/^port .\*/port 0/' /etc/redis/redis.conf.orig | sudo tee /etc/redis/redis.conf

    # Active le socket Redis pour le chemin par défaut sous Debian/Ubuntu
    echo 'unixsocket /var/run/redis/redis.sock' | sudo tee -a /etc/redis/redis.conf

    # Donne la permission d'accès au socket pour tous les membres du groupe redis
    echo 'unixsocketperm 770' | sudo tee -a /etc/redis/redis.conf

    # Création du dossier qui contiendra le socket
    mkdir /var/run/redis
    chown redis:redis /var/run/redis
    chmod 755 /var/run/redis
    # Rend le dossier qui contient le socket persistant, si possible
    if [ -d /etc/tmpfiles.d ]; then
      echo 'd  /var/run/redis  0755  redis  redis  10d  -' | sudo tee -a /etc/tmpfiles.d/redis.conf
    fi

    # Active les changement de redis.conf
    sudo service redis-server restart

    # Ajout de git au groupe Redis
    sudo usermod -aG redis git

#### GitLab

    # Plaçons nous dans le répertoire de GitLab créé
    cd /home/git

    # Clone de GitLab
    sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-ce.git -b 7-9-stable gitlab

##### Configuration de Gitlab

    # Plaçons nous dans le dossier de GitLab
    cd /home/git/gitlab

    # Copie de l'example de configuration de gitlab
    sudo -u git -H cp config/gitlab.yml.example config/gitlab.yml

    # Changement du fichier de configuration :
    # Changez la valeur de 'host' par l'adresse votre futur gitlab (exemple: 'gitlab.votresite.com')
    # Changez la valeur de 'email_from' par l'adresse mail au format de votre adresse gitlab (exemple: 'gitlab@votresite.com')
    sudo -u git -H editor config/gitlab.yml

    # Donne les droits d'écriture à Git aux dossiers log et tmp
    sudo chown -R git log/
    sudo chown -R git tmp/
    sudo chmod -R u+rwX,go-w log/
    sudo chmod -R u+rwX tmp/

    # Création des dossiers pour les satellites
    sudo -u git -H mkdir /home/git/gitlab-satellites
    sudo chmod u+rwx,g=rx,o-rwx /home/git/gitlab-satellites

    # Donne les droits d'écriture aux dossiers tmp/pids, tmp/sockets et public/uploads
    sudo chmod -R u+rwX tmp/pids/
    sudo chmod -R u+rwX tmp/sockets/
    sudo chmod -R u+rwX  public/uploads

    # Copie de la configuration unicorn
    sudo -u git -H cp config/unicorn.rb.example config/unicorn.rb

    # Affiche le nombre de coeurs du processeur
    nproc

    # Autorisez le mode cluster si vous attendez une grosse charge sur serveur
    # Exemple : Changez la quantité de processus à 3 pour un serveur de 2G de RAM
    # Corrigez le nombre de processus au minimum au nombre de coeurs
    sudo -u git -H editor config/unicorn.rb

    # Copie l'exemple de configuration de Rack attack
    sudo -u git -H cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb

    # Configuration des réglages globaux de git pour l'utilisateur git, utile lors de l'édition via le portail web
    # Changez user.email en fonction de ce que vous avez mis dans gitlab.yml
    sudo -u git -H git config --global user.name "GitLab"
    sudo -u git -H git config --global user.email "example@example.com"
    sudo -u git -H git config --global core.autocrlf input

    # Copie de la configuration de Redis
    sudo -u git -H cp config/resque.yml.example config/resque.yml

    # Changez le chemin socket si vous n'utilisez pas la configuration de base (si vous installez Redis pour la première fois au cours de ce tutoriel, pas besoin).
    sudo -u git -H editor config/resque.yml

#### Configuration de la base de données GitLab

    # Copie de la configuration mysql
    sudo -u git cp config/database.yml.mysql config/database.yml

    # Mettez à jour le nom d'utilisateur et le mot de passe dans config/database.yml
    # Vous avez juste besoin d'adapter les réglages de production (première partie)
    # Si vous avez bien suivi ce tutoriel, faites comme suit :
    # Changez la valeur de username (actuellement 'git') par 'gitlab'
    # Changez la valeur de password (actuellement 'secure password') par la valeur du mot de passe que vous avez donné précédemment à la table MySQL
    # Vous pouvez garder les double-quotes autour du mot de passe
    sudo -u git -H editor config/database.yml

    # Rend la configuration de la base de donnée lisible par git seulement (protégeant le mot de passe au passage)
    sudo -u git -H chmod o-rwx config/database.yml

#### Installation des gems

    # Installation des gems pour la lecture MySQL (l'option dit "sans ... postgres")
    sudo -u git -H bundle install --deployment --without development test postgres aws

#### Installation du GitLab Shell

    # Lance l'installation de gitlab-shell (remplacez 'REDIS_URL' si besoin)
    sudo -u git -H bundle exec rake gitlab:shell:install REDIS_URL=unix:/var/run/redis/redis.sock RAILS_ENV=production

    # Par défaut, la configuration gitlab-shell est générée depuis la configuration principale de Gitlab
    # Vous pouvez la lire (et la modifier) en tapant la ligne suivante:
    sudo -u git -H editor /home/git/gitlab-shell/config.yml

#### Initialisation de la base de données et activation des options avancées

    # Lancement de l'installation de la base de données
    sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production

    # Entrez 'yes' lorsque cela vous sera demandé : Ce ne fera que créer des tables
    # Une fois le script terminé, notez bien les identifiants après 'Administrator account created:'

#### Installation du script d'Initialisation

    # Copie du script d'initialisation
    sudo cp lib/support/init.d/gitlab /etc/init.d/gitlab

    # Modification pour que le script soit enclenché lors du démarrage du serveur
    sudo update-rc.d gitlab defaults 21

#### Post-Installation

    # Mise en place des logs
    sudo cp lib/support/logrotate/gitlab /etc/logrotate.d/gitlab

    # Vérification de la configuration du GitLab
    sudo -u git -H bundle exec rake gitlab:env:info RAILS_ENV=production

    # Lancement de GitLab
    sudo /etc/init.d/gitlab restart

    # En plus : Compilation des fichiers statiques pour accélérer GitLab (cela peut prendre un moment)
    sudo -u git -H bundle exec rake assets:precompile RAILS_ENV=production

#### Configuration d'Apache

Il ne reste plus que la configuration d’Apache et vous pourrez profiter de votre installation.
Nous allons donc créer un fichier « gitlab.nomDeVotreSite.com » dans le dossier `/etc/apache2/sites-available/` avec la configuration suivante (pensez à bien changez l’adresse de votre site !) :

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

Ensuite il ne vous reste plus qu’à activer le site et relancer la configuration apache :

    # Activation du site
    sudo a2ensite gitlab.nomDeVotreSite.com

    # Activation des modules utilisés dans la conf de gitlab.nomDeVotreSite.com
    sudo a2enmod proxy_http
    sudo a2enmod proxy
    sudo a2enmod rewrite

    # Relancez la configuration Apache
    sudo service apache2 restart

Voilà ! Maintenant, il ne vous reste plus qu'à vous connecter avec les identifiants récupérés précédemment sur la page 'gitlab.votreSite.com' et le tour est joué ! (N'oubliez pas de changer votre mot de passe).

Sources :
[gitlab.com (section 'installation')](https://github.com/gitlabhq/gitlabhq/blob/master/doc/install/installation.md), [ezerdeniz.fr](http://eserdeniz.fr/articles/view/4/installer-gitlab-sous-debian-7-avec-nginx-et-mysql)
