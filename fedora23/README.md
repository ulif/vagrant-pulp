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


VirtualBox Guest Additions
--------------------------

The default Fedora 23 vagrant virtualbox cloud image comes without
VBGuestAdditions. These are needed for most communication with the
"outer-world", even for mounting simple paths. Luckily, there is a
vagrant plugin that installs suitable stuff in your virtualbox if
non-matching vbox guest additions (or none at all) were detected on
boot-up.

The `vbguest` plugin can be installed like this:

    $ vagrant plugin install vagrant-vbguest

and requires a

    $ vagrant reload

if you got your vagrant box running already. Otherwise it will check
at time of next `vargrant up` whether everything is allright with your
local box.


Setup Vagrant Machine (not: Box)
--------------------------------

You should now be ready for setting up your vagrant machine, i.e.
we will create a running machine from the box prepared before.

If you want to add SSH RSA keys for cloning data from your github
account, you can put them into the local `ssh/` dir. They should
be named `id_github_vagrant` and `id_github_vagrant.pub` for the
private/public key respectively.

Of course the key must be registered with github if you want to use
it inside the local machine.

Another requirement: we expect a directory ../../devel in the hosts
local filesystem. This will be mounted as /home/vagrant/devel/. If
you do not have such a directory, please change the `Vagrantfile`
accordingly.

If everything is prepared, start the machine:

    $ vagrant up

will take lots of time and prepare your machine. Afterwards you can
login into the machine:

    $ vagrant ssh

After first start the devel environment will not be ready.
