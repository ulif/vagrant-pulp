Setup vagrant based dev env via fedora 23
=========================================

Get, Check and Install `vagrant` Image
--------------------------------------

We use the virtualbox variant here. Please note that there is also a
`libvirt` variant available which might work better on a Fedora host.

The short way:

    $ vagrant box add fedora/23-cloud-base

downloads, checks and installs the box image from hashicorp. If you go
this way, you're done and can proceed with next section.

The long way:

Get Fedora 23 Cloud-Base Box (and checksum file):

    $ wget https://download.fedoraproject.org/pub/fedora/linux/releases/23/Cloud/x86_64/Images/Fedora-Cloud-Base-Vagrant-23-20151030.x86_64.vagrant-virtualbox.box
    $ wget https://download.fedoraproject.org/pub/fedora/linux/releases/23/Cloud/x86_64/Images/Fedora-Cloud_Images-x86_64-23-CHECKSUM

Import GPG keys of Fedora project if you haven't already:

    $ curl https://getfedora.org/static/fedora.gpg | gpg --import

Check the checksum file:

    $ gpg --verify Fedora-Cloud_Images-x86_64-23-CHECKSUM

to ensure the list of checksums was not tampered with.

Check the checksum:

    $ sha256sum -c Fedora-Cloud_Images-x86_64-23-CHECKSUM
    Fedora-Cloud-Base-Vagrant-23-20151030.x86_64.vagrant-virtualbox.box: OK

We need the "OK" for our local image only. Complaints about missing
other files can be ignored.

After we made sure the checksum is okay, we make the box available locally:

    $ vagrant box add --name "fedora/23-cloud-base" Fedora-Cloud-Base-Vagrant-23-20151030.x86_64.vagrant-virtualbox.box

