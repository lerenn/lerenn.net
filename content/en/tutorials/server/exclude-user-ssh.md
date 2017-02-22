+++
Categories = ["Server"]
Date = "2014-12-31"
Tags = ["Server", "SSH", "Security"]
Title = "Exclude a user connected by SSH"
Description = "Exclusion of a connected user by SSH."
+++

Maybe you’ve already wondered: “If someone access my server, how can I get rid of him ?”. If you’re in that case, here is a little script which could help you.
<!--more-->

    # Recuperation of connected users
    echo 'Here is connected users :'
    who -u -H

    # ID identification
    echo 'Enter the user PID concerned (5th column):'
    read

    # Exclude of the user
    let 'pidUser = $REPLY+2'
    kill -9 $pidUser

All you have to do is to launch the script, once this one saved in a file. You’ll see connected users, their PID and login time. Then, the script will ask you the PID of the user to exclude and the work’ll be done ! You can test by opening 2 sessions and eclude yourself (“What?!”).

Some explanations:

* The command `who -u -h` displays connected users. The option `u` displays more informations on connected users, by example, their PID. Option `h` will precise the columns role.
* `read` read what you’ve entered in the console.
* `let` allows a math operation : Here, we add 2 to the result of the read, `$REPLY` (But I don’t know why we have to add 2… But it works !)
* `kill -9 $pidUser`, what you’ve been waiting for : Get this guy out of here.

One last thing : Don’t forget to secure your file with the following lines.

    chmod 700 fileName
    chown root:root fileName

It will avoid anyone, except you, to read it, execute it,… This way, only root will be able to execute it.
