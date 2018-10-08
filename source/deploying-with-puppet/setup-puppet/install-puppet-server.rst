.. Copyright (C) 2018 Wazuh, Inc.

.. _setup_puppet_server:

Installing Puppet Server
========================

In this section we are going to install the Puppet server. This server will be in charge of communicating with all the Puppet agents and executing the relevant manifests in each of them. We only need to have one Puppet server. 

Install NTP
-----------

Time must be set accurately on puppet master that will be acting as a certificate authority to sign the certificates coming from the client nodes. We will use NTP for this purpose.

1 - Install the NTP package and perform the time sync with upstream NTP servers.

a) CentOS/RHEL/Fedora

.. code-block:: console

  # sudo yum -y install ntp
  # sudo ntpdate pool.ntp.org

b) Debian/Ubuntu

.. code-block:: console

  # sudo apt-get install -y ntp ntpdate
  # sudo ntpdate -u 0.ubuntu.pool.ntp.org

.. note:: Ensure that all the nodes are in same time zone using.

Installation on CentOS/RHEL/Fedora
----------------------------------

1 - Install the Puppet yum repository. See this `index <https://yum.puppetlabs.com/>`_ to find the correct rpm file needed to install the puppet repository for your Linux distribution. For example, to install Puppet 5 for CentOS 7 or RHEL 7, do the following:

.. code-block:: console

   # rpm -ivh https://yum.puppetlabs.com/puppet5/puppet5-release-el-7.noarch.rpm

2 - Install the "puppetserver" package.

.. code-block:: console

   # yum -y install puppetserver

.. note:: 

  For a correct installation we recommend the use of Puppet versions equal or greater than 5. 


Installation on Debian/Ubuntu
-----------------------------

Install ``curl``, ``apt-transport-https`` and ``lsb-release``:

.. code-block:: console

	# apt-get update
	# apt-get install curl apt-transport-https lsb-release

1 - Get the appropriate Puppet apt repository. See https://apt.puppetlabs.com to find the correct deb file to install the Puppet 5 repository for your Linux distribution. For example, to install Puppet 5 for Ubuntu 16 xenial, do the following:

.. code-block:: console

  # wget https://apt.puppetlabs.com/puppet5-release-xenial.deb
  # dpkg -i puppet5-release-xenial.deb
  # apt update

2 - Install the "puppetserver" package.

.. code-block:: console

  # apt-get install -y puppetserver

.. note:: For a correct installation we recommend the use of Puppet versions equal or greater than 5. 


.. note:: The releases supported by the manifest to install Wazuh are as follows: 

      Ubuntu: **precise | trusty | vivid | wily | xenial | yakketi | artful | bionic**

      Debian: **jessie | wheezy | stretch | sid**

Configure Hosts
---------------

We must make sure that our Puppet server is accessible, we must have our hostname attached to our IP address. For example, on a CentOS 7 server.

.. code-block:: console

  [root@localhost centos]# hostname
  puppet-server
  [root@localhost centos]# cat /etc/hosts
  . . . 
  192.168.1.57  puppet-server



Memory Allocation
-----------------

By default, Puppet Server will be configured to use 2GB of RAM. However, if you want to experiment with Puppet Server on a VM, you can safely allocate as little as 512MB of memory. To change Puppet Server memory allocation, you can edit the following init config file.

  a) CentOS/RHEL/Fedora

    - ``/etc/sysconfig/puppetserver``

  b) Debian/Ubuntu

    - ``/etc/default/puppetserver`` 

Replace 2g with the amount of memory you want to allocate to Puppet Server. For example, to allocate 1GB of memory, use ``JAVA_ARGS="-Xms1g -Xmx1g"``; for 512MB, use ``JAVA_ARGS="-Xms512m -Xmx512m"``.

In our example:

.. code-block:: none

  [root@localhost centos]# cat /etc/sysconfig/puppetserver
  ###########################################
  # Init settings for puppetserver
  ###########################################

  # Location of your Java binary (version 7 or higher)
  JAVA_BIN="/usr/bin/java"

  # Modify this if you'd like to change the memory allocation, enable JMX, etc
  JAVA_ARGS="-Xms512m -Xmx512m -Djruby.logger.class=com.puppetlabs.jruby_utils.jruby.Slf4jLogger"

  # Modify this if you'd like TrapperKeeper specific arguments
  TK_ARGS=""

  # These normally shouldn't need to be edited if using OS packages
  USER="puppet"
  GROUP="puppet"
  INSTALL_DIR="/opt/puppetlabs/server/apps/puppetserver"
  CONFIG="/etc/puppetlabs/puppetserver/conf.d"

  # Bootstrap path
  BOOTSTRAP_CONFIG="/etc/puppetlabs/puppetserver/services.d/,/opt/puppetlabs/server/apps/puppetserver/config/services.d/"

  # SERVICE_STOP_RETRIES can be set here to alter the default stop timeout in
  # seconds.  For systemd, the shorter of this setting or 'TimeoutStopSec' in
  # the systemd.service definition will effectively be the timeout which is used.
  SERVICE_STOP_RETRIES=60

  # START_TIMEOUT can be set here to alter the default startup timeout in
  # seconds.  For systemd, the shorter of this setting or 'TimeoutStartSec'
  # in the service's systemd.service configuration file will effectively be the
  # timeout which is used.
  START_TIMEOUT=300


  # Maximum number of seconds that can expire for a service reload attempt before
  # the result of the attempt is interpreted as a failure.
  RELOAD_TIMEOUT=120


Puppet server configuration
---------------------------

1 - Edit the ``/etc/puppetlabs/puppet/puppet.conf`` file, adding this line to the ``[main]`` section (create the section if it does not exist), and replacing ``puppet.example.com`` with your own FQDN: ::

   dns_alt_names = puppet,puppet.example.com

For example:

.. code-block:: console

  [root@localhost centos]# cat /etc/puppetlabs/puppet/puppet.conf
  # This file can be used to override the default puppet settings.
  # See the following links for more details on what settings are available:
  # - https://puppet.com/docs/puppet/latest/config_important_settings.html
  # - https://puppet.com/docs/puppet/latest/config_about_settings.html
  # - https://puppet.com/docs/puppet/latest/config_file_main.html
  # - https://puppet.com/docs/puppet/latest/configuration.html
  [main]
  dns_alt_names = puppet,puppet-server
  [master]
  vardir = /opt/puppetlabs/server/data/puppetserver
  logdir = /var/log/puppetlabs/puppetserver
  rundir = /var/run/puppetlabs/puppetserver
  pidfile = /var/run/puppetlabs/puppetserver/puppetserver.pid
  codedir = /etc/puppetlabs/code


.. note:: If you find ``templatedir=$confdir/templates`` in the config file, delete that line.  It has been deprecated.

2 - Then, restart your Puppet Server to apply changes:

  a) For Systemd:

  .. code-block:: console

        # systemctl start puppetserver
        # systemctl enable puppetserver

  b) For SysV Init:

  .. code-block:: console

        # service puppetserver start
        # update-rc.d puppetserver

