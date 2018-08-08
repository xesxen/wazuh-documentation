.. Copyright (C) 2018 Wazuh, Inc.

Integration
===========

#. `How it works`_
#. `Getting started (Agent)`_
#. `Getting started (Manager)`_
#. `Adding triggers`_

How it works
------------

This integration utilizes the Kaspersky Endpoint Security to detect malicious content on the system. This integration functions as described bellow:

1. The manager receives an alert from an agent.

2. The alert is then procesed by `analysid` daemon.

3. If the rule matches with the KES group, an active response is generated.

4. The agent receives the active response an launches a KES scan by running the ``kaspersky.sh`` script.


Setup
-----

Getting started (Agent)
^^^^^^^^^^^^^^^^^^^^^^^

In order for this integration to function, the first thing that must be completed is the configuration of the Kaspersky Endpoint Security integration as shown below:

1. Download KES package:

Go to the link https://support.kaspersky.com/kes10linux#downloads, click on **Files** and download the **.deb** package or the **.rpm** package based on your distribution.

2. Install the KES package:

On a debian based distribution:

.. code-block:: console

    # dpkg -i kesl-10.1.0-<build number>_i386.deb 
    # dpkg -i kesl_10.1.0-<build number>_amd64.deb

On a rpm based distribution:

.. code-block:: console

    # rpm -i kesl-10.1.0-<build number>.i386.rpm
    # rpm -i kesl-10.1.0-<build number>.x86_64.rpm

3. Run the KES setup:

.. code-block:: console

    # /opt/kaspersky/kesl/bin/kesl-setup.pl

After this is complete, your **agent** can start to launch scans.


Getting started (Manager)
^^^^^^^^^^^^^^^^^^^^^^^^^

For the Wazuh manager there are already rules and decoders for Kaspersky Endpoint Security. There are located in the following directories:

.. code-block:: console

    /var/ossec/etc/rules/kes_rules.xml
    /var/ossec/etc/rules/kes-triggers_rules.xml
    /var/ossec/etc/decoders/kes_decoders.xml

The file ``/var/ossec/etc/shared/ar.conf`` must have the predefined active response commands for Kaspersky Endpoint Security.


.. code-block:: console
    
    .
    .
    .
    kaspersky-full0 - kaspersky.sh - 0
    kaspersky-boot0 - kaspersky.sh - 0
    kaspersky-memory0 - kaspersky.sh - 0
    kaspersky-realtime0 - kaspersky.sh - 0

Adding triggers
^^^^^^^^^^^^^^^

Now in order to launch an active response, we must modify some Wazuh rules. In this example we are going to modify the rule 5104.

Open the file ``/var/ossec/ruleset/rules/0020-syslog_rules.xml`` and search for the rule 5104:

.. code-block:: xml

    <rule id="5104" level="8">
        <if_sid>5100</if_sid>
        <regex>Promiscuous mode enabled|</regex>
        <regex>device \S+ entered promiscuous mode</regex>
        <description>Interface entered in promiscuous(sniffing) mode.</description>
        <group>promisc,pci_dss_10.6.1,pci_dss_11.4,gpg13_4.13,gdpr_IV_35.7.d,kaspersky_full_scan,kaspersky_realtime</group>
    </rule>

In the following table we can see a definition of the rule groups:

+-----------------------+----------------------------+---------------------+------------------+
| Rule group            | Scan type                  | Wrapper command     | KES CLI          |
+-----------------------+----------------------------+---------------------+------------------+
| kaspersky_full_scan   | Full scan                  | -\\-full_scan       | -\\-start-task 2 |
+-----------------------+----------------------------+---------------------+------------------+
| kaspersky_memory_scan | Memory scan                | -\\-memory_scan     | -\\-start-task 5 |
+-----------------------+----------------------------+---------------------+------------------+
| kaspersky_boot_scan   | Boot scan                  | -\\-boot_scan       | -\\-start-task 4 |
+-----------------------+----------------------------+---------------------+------------------+
| kaspersky_realtime    | Enable realtime protection | -\\-enable_realtime | -\\-start-task 1 |
+-----------------------+----------------------------+---------------------+------------------+

When your rules modifications are done, define some active response commands for triggering the script. Edit the ``/var/ossec/etc/ossec.conf`` file and add the following:

.. code-block:: xml

    <command>
        <name>kaspersky-full</name>
        <executable>kaspersky.sh</executable>
        <extra_args>--full_scan</extra_args>
    </command>

    <active-response>
        <disabled>no</disabled>
        <command>kaspersky-full</command>
        <location>local</location>
        <rules_group>kaspersky_full_scan</rules_group>
    </active-response>

This will launch a full scan on the agent when a rule with the group ``kaspersky_full_scan`` is triggered. 