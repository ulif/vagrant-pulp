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

Provision all hosts:

    $ ansible-playbook -i ansible/hosts ansible/server-playbook.yml
    $ ansible-playbook -i ansible/hosts ansible/consumer-playbook.yml


First Steps with Pulp
=====================

`pulp1` is our "commanding" host. We ssh into it:

    $ vagrant ssh pulp1

You can check the server status

    (pulp1) $ pulp-admin status

Should tell you lots of stuff. You can, however, yet not do restricted
operations, like listing users. In fact you have to authenticate
before.


Authenticate by Password or Cert
--------------------------------

At the very beginning there is only one user to work as pulp-admin,
the user "admin" with password "admin".

You can authenticate by password all the time when doing `pulp-admin`
commands:

    (pulp1) $ pulp-admin -u admin auth user list
    Enter password:

will work (with "admin" ad password), but it is tedious in the long
run. Therfore you can request a session certificate to authenticate
you automatically with each command. Use `login` to create it:

    (pulp1) $ pulp-admin login -u admin

Again, use "admin" as password. This should get you an SSH x509 cert
for accessing the pulp admin stuff in `~/.pulp/user-cert.pem`. The
certificate will hold for a limited time span (1 week).

From now on you can skip "-u admin" until the certificate expires.

You can end the session with:

    (pulp1) $ pulp-admin logout
    Session certificate successfully removed.

In the following sections we assume you are logged in.


User Management
---------------

See what users are there:

    (pulp1) $ pulp-admin auth user list --details

You can create/update/delete users:

    (pulp1) $ pulp-admin auth user create --login test-user
    (pulp1) $ pulp-admin auth user update --login test-user --name "Test User" -p
    (pulp1) $ pulp-admin auth user delete --login test-user


Permissions
-----------

Resources are essentially REST-API paths. They can be listed, granted,
revoked:

    (pulp1) $ pulp-admin auth permission list   --resource /
    (pulp1) $ pulp-admin auth permission grant  --resource /v2/repositories/ --login test-user -o create -o update -o read
    (pulp1) $ pulp-admin auth permission revoke --resource /v2/repositories/ --login test-user -o create -o update -o read


Roles
-----

Roles are there to group users and permissions. They can be listed,
created, deleted and applied to users and permissions.

    (pulp1) $ pulp-admin auth role list   --details
    (pulp1) $ pulp-admin auth role create --role-id consumer-admin
    (pulp1) $ pulp-admin auth role delete --role-id consumer-admin

Users can be added/removed from roles.

    (pulp1) $ pulp-admin auth role user add    --role-id super-users --login test-user
    (pulp1) $ pulp-admin auth role user remove --role-id super-users --login test-user

The same applies to permissions. Users with a certain role will
inherit all the role's permissions.

    (pulp1) $ pulp-admin auth permission grant  --resource / --role-id test-user -o read
    (pulp1) $ pulp-admin auth permission revoke --resource / --role-id test-user -o read


Repositories
------------

You can create a sample RPM repository like this:

    (pulp1) $ pulp-admin rpm repo create --repo-id=zoo --feed=https://repos.fedorapeople.org/repos/pulp/pulp/demo_repos/zoo/

All repos on a host can be listed by:

    (pulp1) $ pulp-admin rpm repo list

searched by:

    (pulp1) $ pulp-admin rpm repo search

Metadata can be changed by:

    (pulp1) $ pulp-admin rpm repo update --repo-id=zoo

A created repository can be deleted by:

    (pulp1) $ pulp-admin rpm repo delete --repo-id=zoo



Interacting with Consumers
==========================


Consumer (Un-)Registration
--------------------------

Consumers must register themselves first, using
`pulp-consumer`. `pulp-consumer` always requires root (`sudo`)
permissions:

    (pulp2) $ sudo pulp-consumer -u admin register --consumer-id my-consumer

This creates an x509 cert which will be valid for 10 years and stored
in `/etc/pki/pulp/consumer/consumer-cert.pem`.

Unregistration works accordingly:

    (pulp2) $ sudo pulp-consumer unregister


Binding to Repos
----------------

A registered consumer can bind to repositories (not need for 'sudo'
any more). The respective repository must be available at the server,
of course.

    (pulp2) $ pulp-consumer rpm bind --repo-id=zoo


Working with Nodes
==================

Preparing the Parent Node
-------------------------

Parent nodes can provide "repositories" for child nodes. To do so, a
parent node must enable repositories first.

    (pulp1) $ pulp-admin rpm repo create --repo-id=zoo --feed=https://repos.fedorapeople.org/repos/pulp/pulp/demo_repos/zoo/
    (pulp1) $ pulp-admin node repo enable --repo-id zoo

Enabling is not enough. Admins additionally must publish repos:

    (pulp1) $ pulp-admin node repo publish --repo-id zoo

Admins can afterwards list repos:

    (pulp1) $ pulp-admin node repo list --all

To retract repos, we have to unregister them:

    (pulp1) $ pulp-admin node repo disable --repo-id zoo


Handling Child Nodes
--------------------

Once, a parent host is ready (see above), consumers on child nodes can
bind to parent nodes:

    (pulp2) $ sudo pulp-consumer node activate

Once activated, a child node can be bound to repos on parent. This can
be done from both sides, from the parent side:

    (pulp1) $ pulp-admin node bind --node-id c2 --repo-id zoo

or from the child side:

    (pulp2) $ pulp-consumer node bind --repo-id zoo

Syncing a child node can be triggered from the parent side:

    (pulp1) $ pulp-admin node sync run --node-id c2

