.. Copyright (C) 2018 Wazuh, Inc.

.. _setup_puppet_agent:

Installing Puppet agent
=======================

In this section we are going to install a Puppet agent. The Puppet agents are connected to the Puppet server and execute the manifests that the server indicates to them. We'll have a Puppet agent on every computer we want to run a manifest on. 

Installation on CentOS/RHEL/Fedora
------------------------------------

1 - Install the Puppet yum repository. See this `index <https://yum.puppetlabs.com/>`_ to find the correct rpm file needed to install the puppet repository for your Linux distribution. For example, to install Puppet 5 for CentOS 7 or RHEL 7, do the following:

.. code-block:: console

   # rpm -ivh https://yum.puppetlabs.com/puppet5/puppet5-release-el-7.noarch.rpm

.. code-block:: console

2 - Install the "puppet-agent" package. 

   # yum -y install puppet-agent

.. note:: 

  For a correct installation we recommend the use of Puppet versions equal or greater than 5. 

Installation on Debian/Ubuntu
------------------------------

1 - Install ``curl``, ``apt-transport-https`` and ``lsb-release``:

.. code-block:: console

	# apt-get update
	# apt-get install curl apt-transport-https lsb-release

2 - Get the appropriate Puppet apt repository. See https://apt.puppetlabs.com to find the correct deb file to install the puppet repository for your Linux distribution. For example, to install Puppet 5 for Ubuntu 16 xenial, do the following:

.. code-block:: console

  # wget https://apt.puppetlabs.com/puppet5-release-xenial.deb
  # dpkg -i puppet5-release-xenial.deb
  # apt update

3 - Install the "puppet-agent" package.

.. code-block:: console

  # apt-get install -y puppet-agent

.. note:: For a correct installation we recommend the use of Puppet versions equal or greater than 5. 


.. note:: The releases supported by the manifest to install Wazuh are as follows: 

      Ubuntu: **precise | trusty | vivid | wily | xenial | yakketi**

      Debian: **jessie | wheezy | stretch | sid**
  


Configuration
^^^^^^^^^^^^^

1 - Add the server value to the ``[main]`` section of the node’s ``/etc/puppetlabs/puppet/puppet.conf`` file, replacing ``puppet.example.com`` with your Puppet Server’s FQDN::

   [main]
   server = puppet.example.com

2 - Enable the Puppet service:

.. code-block:: console

   # /opt/puppetlabs/bin/puppet resource service puppet ensure=running enable=true
