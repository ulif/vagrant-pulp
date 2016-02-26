vagrant-pulp
============

Vagrant configurations for pulp envs.

The vagrant installs done here happened on an Ubuntu 14.04 host. To
make use of the images the stock vagrant package was not
sufficient. Instead a version (1.8.1) from

  https://www.vagrantup.com/downloads.html

was fetched and installed (Debian 64-bit, using `dpkg -i`).

Each directory in this project provides a different vagrant
configuration creating pulp developer envs.

All repositories expect a `devel` dir in `../../`, i.e. one directory
level above from this directory and two directory levels above each
vagrant dir.

`centos71`
  CentOS 7.1-based pulp env.

`fedora23`
  Fedora 23 cloud base image-based pulp env.

See the respective directories to learn more about the specific
builds.
