Set up a net of pulp servers
============================

This setup requires the `vagrant-hosts` plugin installed:

    $ vagrant plugin install vagrant-hosts

has to be run once.

Initialize the hosts:

    $ vagrant up

Check that ansible is able to work with the environment:

    $ ansible -i ansible/hosts -m ping pulp1.example.org
    $ ansible -i ansible/hosts -m ping pulp2.example.org
    $ ansible -i ansible/hosts -m ping pulp3.example.org
    $ ansible -i ansible/hosts -m ping all
    $ ansible -i ansible/hosts -m shell -a "uname -a" all
