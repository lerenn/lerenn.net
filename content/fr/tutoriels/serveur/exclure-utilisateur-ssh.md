+++
Categories = ["Serveur"]
Date = "2014-12-31"
Tags = ["Serveur", "SSH", "Sécurité"]
Title = "Exclure un utilisateur connecté en SSH"
Description = "Exclusion d'un utilisateur connecté en SSH"
+++

Peut être vous êtes vous déjà demandé "Si quelqu'un entre sur mon serveur, comment puis-je m'en débarasser ?". Si vous êtes dans ce cas, voici un petit script qui pourrait vous être utile.
<!--more-->

    # Récuperation des personnes connectées
    echo 'Voici les personnes connectées :'
    who -u -H

    # Récupération du PID
    echo 'Entrez le PID de l'utilisateur concerné (5e colonne) :'
    read

    # Exclusion de l'utilisateur
    let 'pidUser = $REPLY+2'
    kill -9 $pidUser

Vous n'aurez qu'à lancer ce script, une fois ce dernier enregistré dans un fichier. Vous verrez alors les utilisateurs connectés, leur PID et l'heure de connexion. Le script vous demandera ensuite le PID de l'utilisateur à exclure et le travail est terminé ! Vous pouvez tester en ouvrant 2 sessions.

Voici quelques explications :

* La commande `who -u -h` permet d'afficher les utilisateurs connectés. L'option `u` affiche plus d'informations sur les utilisateurs connectés, dont le PID. L'option `h` quand à elle précise le rôle des colonnes de la sortie de la commande.
* `read` permet de récupérer ce qui a été tapé par l'utilisateur (en l'occurence le PID)
* `let` permet une opération : Ici on additionne 2 au résultat du read soit `$REPLY"` Je ne sais pas encore pourquoi il faut ajouter 2 pour éliminer l'accès).
* `kill -9 $pidUser` permet de tuer le processus SSH de l'utilisateur visé.

Une dernière chose : N'oubliez pas de sécuriser votre fichier avec les commandes suivantes :

    chmod 700 nomFichier
    chown root:root nomFichier

Cela évitera que tout le monde puisse le lire, l'exécuter, etc. Ainsi, seul le root pourra l'exécuter.
