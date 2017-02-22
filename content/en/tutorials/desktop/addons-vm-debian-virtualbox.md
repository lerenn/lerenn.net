+++
Categories = [ "Desktop" ]
Date = "2015-03-23"
Tags = [ "Server", "Debian", "VirtualBox", "Additions" ]
Title = "Install Guest Additions on a Debian VM"
+++
This is a short article with few instructions to install guest additions on a
Debian. You will be able to access some tools very useful (adaptated display,
bidirectional copy-past, etc.).
<!--more-->

    # As root, update your system
    apt-get update -y && apt-get upgrade -y

    # Install necessary packages
    apt-get install build-essential module-assistant

    # System configuration for kernel modules creation :
    m-a prepare

    # Do "Insert Guest Additions CD image" in the menu "Devices" of your VM
    # Then, mount the CD with this command :
    mount /media/cdrom

    # Launch installation, with the following command and follow instructions :
    sh /media/cdrom/VBoxLinuxAdditions.run
