Configuration
==================

Enabling proxy
----------------------


To enable Tigase Socks5 Proxy component for Tigase XMPP Server, you need to activate ``socks5`` component in Tigase XMPP Server configuration file (``etc/config.tdsl``). In simples solution it will work without ability to enforce any limits but will also work without a need of database to store informations about used bandwidth.

**Simple configuration.**

.. code:: text

   socks5 () {
       repository {
           default () {
               cls = 'dummy'
           }
       }
   }

**``remote-addresses``**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code:: text

   proxy {
       'remote-addresses' = '192.168.1.205,20.255.13.190'
   }

Comma seperated list of IP addresses that will be accessible VIA the Socks5 Proxy. This can be useful if you want to specify a specific router address to allow external traffic to transfer files using the proxy to users on an internal network.


Port settings
^^^^^^^^^^^^^^^

If socks5 is being used as a proxy, you may configure a specific ports for the proxy using the following line in config.tdsl:

.. code:: text

   proxy {
       'connections' {
           'ports' = [ 1080 ]
         }
   }

Enabling limits
^^^^^^^^^^^^^^^^^^

To enable limits you need to import schema files proper for your database and related to Tigase Socks5 Proxy component from ``database`` directory. To do this, refer to the previous section.

With that setup, it is possible to enable limits verifier by replacing entries related to Tigase Socks5 Proxy component configuration with following entries. This will use default database configured to use with Tigase XMPP Server.


``DummyVerifier``
~~~~~~~~~~~~~~~~~~~

-  Class Name: ``tigase.socks5.verifiers.DummyVerifier``

This accepts file transfers VIA SOCKS5 proxy from any user and does not check limitations against the database.

.. code:: text

   socks5 () {
       verifier (class: tigase.socks5.verifiers.DummyVerifier) {
       }
   }


``LimitsVerifier``
~~~~~~~~~~~~~~~~~~~~~~~~

-  Class Name: ``tigase.socks5.verifiers.LimitsVerifier``

Uses the database to store limits and record the amount of data transferred VIA the proxy.


Configuring limits


Following properties are possible to be set for ``LimitsVerifier``:

.. code:: text

   proxy {
       'verifier-class' = 'tigase.socks5.verifiers.LimitsVerifier'
       tigase.socks5.verifiers.LimitsVerifier {
           'transfer-update-quantization' = '1000'
           'instance-limit' = '3000'
       }
   }

Parameters for ``LimitsVerifier`` which will override the defaults. All of these limits are on a per calendar month basis. For example, a user is limited to 10MB for all transfers. If he transfers 8MB between the 1st and the 22nd, he only has 2MB left in his limit. On the 1st of the following month, his limit is reset to 10MB.

Available parameters:

-  ``transfer-update-quantization`` which value is used to quantitize value to check if value of transferred bytes should be updated in database or not. By default it is 1MB. (Low value can slow down file transfer while high value can allow to exceed quota)

-  ``global-limit`` - Transfer limit for all domains in MB per month.

-  ``instance-limit`` - Transfer limit for server instance in MB per month.

-  ``default-domain-limit`` - The Default transfer limit per domain in MB per month.

-  ``default-user-limit`` - The default transfer limit per user in MB per month.

-  ``default-file-limit`` - The default transfer limit per file in MB per month.

.. Note::

   Low values can slow down file transfers, while high values can allow for users to exceed quotas.


Individual Limits


Using the default database schema in table tig_socks5_users limits can be specified for individual users.

Value of the field *user_id* denotes the scope of the limitation:

-  *domain_name* defines limits for users which JIDs are within that domain;

-  *JID* of the user defines limit for this exact user.

Value of the limit bigger than 0 defines an exact value. If value is equal 0 limit is not override and more global limit is used. If value equals -1 proxy will forbid any transfer for this user. It there is no value for user in this table new row will be created during first transfer and limits for domain or global limits will be used.

Socks5 database is setup in this manner:

.. table:: Table 1. tig_socks5_users

   +-----+-----------------+------------------------------------------+------------+------------------------------------------+----------------+-------------------------+---------------------------+
   | uid | user_id         | sha1_user_id                             | domain     | sha1_domain                              | filesize_limit | transfer_limit_per_user | transfer_limit_per_domain |
   +=====+=================+==========================================+============+==========================================+================+=========================+===========================+
   | 1   | user@domain.com | c35f2956d804e01ef2dec392ef3adae36289123f | domain.com | e1000db219f3268b0f02735342fe8005fd5a257a | 0              | 3000                    | 0                         |
   +-----+-----------------+------------------------------------------+------------+------------------------------------------+----------------+-------------------------+---------------------------+
   | 2   | domain.com      | e1000db219f3268b0f02735342fe8005fd5a257a | domain.com | e1000db219f3268b0f02735342fe8005fd5a257a | 500            | 0                       | 0                         |
   +-----+-----------------+------------------------------------------+------------+------------------------------------------+----------------+-------------------------+---------------------------+

This example table shows that user@domain.com is limited to 3000MB per transfer whereas all users of domain.com are limited to a max file size of 500MB. This table will populate as users transfer files using the SOCKS5 proxy, once it begins population, you may edit it as necessary. A second database is setup tig_socks5_connections that records the connections and transmissions being made, however it does not need to be edited.


Using a separate database
-------------------------------

To use separate database with Tigase Socks5 Proxy component you need to configure new ``DataSource`` in ``dataSource`` section. Here we will use ``socks5-store`` as name of newly configured data source. Additionally you need to pass name of newly configured data source to ``dataSourceName`` property of ``default`` repository of Tigase Socks5 Proxy component.

.. code:: text

   dataSource {
       socks5-store () {
           uri = 'jdbc:db_server_type://server/socks5-database'
       }
   }

   socks5 () {
       repository {
           default () {
               dataSourceName = 'socks5-store'
           }
       }
       ....
   }

