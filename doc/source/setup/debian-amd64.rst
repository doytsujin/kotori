.. include:: ../_resources.rst

.. _setup-debian-amd64:

############
Debian AMD64
############

.. contents::
   :local:
   :depth: 2

----

*****
Intro
*****
Install the whole stack on a Debian-based system. It is currently made of these free and open source software components:

- Mosquitto_, a MQTT message broker
- InfluxDB_, a time-series database
- Grafana_, a graph and dashboard builder for visualizing time series metrics
- :ref:`Kotori`, a data acquisition, graphing and telemetry toolkit


**************
Infrastructure
**************

Mosquitto
=========
::

    aptitude install mosquitto mosquitto-clients


InfluxDB
========
::

    wget https://s3.amazonaws.com/influxdb/influxdb_0.12.2-1_amd64.deb
    dpkg --install influxdb_0.10.2-1_amd64.deb

/etc/influxdb/influxdb.conf::

    [http]
      auth-enabled = true
      log-enabled = true

Configure rsyslog::

    cat /etc/rsyslog.d/influxdb.conf
    # redirect to application log
    if $programname contains 'influxd' then /var/log/influxdb/influxd.log

    # prevent bubbling up into daemon.log
    if $programname contains 'influxd' then stop

Restart rsyslog::

    service rsyslog restart

Start InfluxDB daemon::

    systemctl start influxdb
    tail -F /var/log/influxdb/influxd.log


Grafana
=======
Install package::

    aptitude install apt-transport-https curl
    curl https://packagecloud.io/gpg.key | apt-key add -
    echo 'deb https://packagecloud.io/grafana/stable/debian/ wheezy main' > /etc/apt/sources.list.d/grafana.list

    aptitude update
    aptitude install grafana


Configure::

    /etc/grafana/grafana.ini
    admin_password = XYZ


Enable system service::

    systemctl enable grafana-server
    systemctl is-enabled grafana-server

Start system service::

    systemctl start grafana-server
    tail -F /var/log/grafana/grafana.log


******
Kotori
******

Kotori package
==============

Prerequisites
-------------

Add GPG key for checking package signatures::

    wget -qO - https://packages.elmyra.de/elmyra/foss/debian/pubkey.txt | apt-key add -

Add https addon for apt::

    aptitude install apt-transport-https


Register with package repository
--------------------------------

Add source for "testing" distribution (e.g. append to /etc/apt/sources.list)::

    deb https://packages.elmyra.de/elmyra/foss/debian/ testing main

Reindex package database::

    aptitude update


Install package
---------------
::

    aptitude install kotori


.. seealso:: https://packages.elmyra.de/elmyra/foss/debian/README.txt

When adjusting the configuration in ``/etc/kotori``, please restart the service::

    systemctl restart kotori
    tail -F /var/log/kotori/*.log

For information beyond the package level, please visit :ref:`kotori-hacking`.


Daemon control
==============
Business as usual::

    systemctl start|stop|restart|status kotori
    systemctl enable|disable kotori


****************
All together now
****************

Check the status of all services::

    systemctl list-units influxdb* mosquitto.service grafana-server* kotori*
    systemctl status     influxdb* mosquitto.service grafana-server* kotori*


Count them::

    systemctl list-units influxdb* mosquitto.service grafana-server* kotori* | grep running | wc -l
    4

Watch the logs::

    tail -F /var/log/syslog /var/log/influxdb/*.log /var/log/mosquitto/* /var/log/grafana/*.log /var/log/kotori/*.log

