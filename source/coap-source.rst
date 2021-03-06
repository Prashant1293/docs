
Kafka Connect CoAP Source
=========================

A Connector and Source to stream messages from a CoAP server and write them to a Kafka topic.

The Source supports:

1. DTLS secure clients.
2. Observable resources.

The Source Connector automatically converts the CoAP response into a Kafka Connect ``Struct`` to be store in Kafka as Avro or
Json dependent on the Converters used in Connect. The schema can found :ref:`here <coap_schemas>`.

The key of the ``Struct`` message sent to Kafka is made from the source defined in the message, the resource on the CoAP server
and the message id.

Prerequisites
-------------

- Confluent 3.3
- Java 1.8
- Scala 2.11

Setup
-----

Confluent Setup
~~~~~~~~~~~~~~~

Follow the instructions :ref:`here <install>`.

CoAP Setup
~~~~~~~~~~

The connector uses `Californium <https://github.com/eclipse/californium>`__ Java API under the hood. A resource is host at
``coap://californium.eclipse.org:5683``. Copper, a `FireFox <https://addons.mozilla.org/en-US/firefox/addon/copper-270430/>`__ browser
addon is available so you can browse the server and resources.. This is an observable non confirmable resource. You can
view messages via the browser addon by selecting the resource and clicking the ``observe`` button on the top menu.

Source Connector QuickStart
---------------------------

We you start the Confluent Platform, Kafka Connect is started in distributed mode (``confluent start``). 
In this mode a Rest Endpoint on port ``8083`` is exposed to accept connector configurations. 
We developed Command Line Interface to make interacting with the Connect Rest API easier. The CLI can be found in the Stream Reactor download under
the ``bin`` folder. Alternatively the Jar can be pulled from our GitHub
`releases <https://github.com/datamountaineer/kafka-connect-tools/releases>`__ page.

Starting the Connector
~~~~~~~~~~~~~~~~~~~~~~

Download, unpack and install the Stream Reactor and Confluent. Follow the instructions :ref:`here <install>` if you haven't already done so.
All paths in the quickstart are based in the location you installed the Stream Reactor.

Once the Connect has started we can now use the kafka-connect-tools :ref:`cli <kafka-connect-cli>` to post in our distributed properties file for MQTT.
If you are using the :ref:`dockers <dockers>` you will have to set the following environment variable to for the CLI to
connect to the Rest API of Kafka Connect of your container.

.. sourcecode:: bash

   export KAFKA_CONNECT_REST="http://myserver:myport"

.. sourcecode:: bash

    ➜  bin/connect-cli create coap-source < conf/coap-source.properties

    #Connector name=`coap-source`
    name = coap-source
    tasks = 1
    connector.class = com.datamountaineer.streamreactor.connect.coap.source.CoapSourceConnector
    connect.coap.uri = coap://californium.eclipse.org:5683
    connect.coap.kcql = INSERT INTO coap-topic SELECT * FROM obs-pumping-non
    #task ids: 0

The ``coap-source.properties`` file defines:

1.  The name of the source.
2.  The name number of tasks.
3.  The class containing the connector.
4.  The uri of the CoAP Server and port to connect to.
5.  :ref:`The KCQL routing querying. <kcql>`. This specifies the target topic and source resource on the CoAP server.

Use the Confluent CLI to view Connects logs.

.. sourcecode:: bash

    # Get the logs from Connect
    confluent log connect

    # Follow logs from Connect
    confluent log connect -f

We can use the CLI to check if the connector is up but you should be able to see this in logs as-well.

.. sourcecode:: bash

    #check for running connectors with the CLI
    ➜ bin/connect-cli ps
    coap-source

.. sourcecode:: bash

    INFO
        ____        __        __  ___                  __        _
       / __ \____ _/ /_____ _/  |/  /___  __  ______  / /_____ _(_)___  ___  ___  _____
      / / / / __ `/ __/ __ `/ /|_/ / __ \/ / / / __ \/ __/ __ `/ / __ \/ _ \/ _ \/ ___/
     / /_/ / /_/ / /_/ /_/ / /  / / /_/ / /_/ / / / / /_/ /_/ / / / / /  __/  __/ /
    /_____/\__,_/\__/\__,_/_/  /_/\____/\__,_/_/ /_/\__/\__,_/_/_/ /_/\___/\___/_/
           ______                 _____
          / ____/___  ____ _____ / ___/____  __  _______________
         / /   / __ \/ __ `/ __ \\__ \/ __ \/ / / / ___/ ___/ _ \ By Andrew Stevenson
        / /___/ /_/ / /_/ / /_/ /__/ / /_/ / /_/ / /  / /__/  __/
        \____/\____/\__,_/ .___/____/\____/\__,_/_/   \___/\___/
                        /_/ (com.datamountaineer.streamreactor.connect.coap.source.CoapSourceTask:54)
    [2017-01-09 20:42:44,830] INFO CoapConfig values:
        connect.coap.uri = coap://californium.eclipse.org:5683
        connect.coap.port = 0
        connect.coap.truststore.pass = [hidden]
        connect.coap.cert.chain.key = client
        connect.coap.keystore.path =
        connect.coap.kcql = INSERT INTO coap-topic SELECT * FROM obs-pumping-non
        connect.coap.truststore.path =
        connect.coap.certs = []
        connect.coap.keystore.pass = [hidden]
        connect.coap.host = localhost
     (com.datamountaineer.streamreactor.connect.coap.configs.CoapConfig:178)
    [2017-01-09 20:42:44,831] INFO Source task WorkerSourceTask{id=coap-source-0} finished initialization and start (org.apache.kafka.connect.runtime.WorkerSourceTask:138)
    [2017-01-09 20:42:45,927] INFO Discovered resources /.well-known/core (com.datamountaineer.streamreactor.connect.coap.source.CoapReader:60)
    [2017-01-09 20:42:45,927] INFO Discovered resources /large (com.datamountaineer.streamreactor.connect.coap.source.CoapReader:60)
    [2017-01-09 20:42:45,928] INFO Discovered resources /large-create (com.datamountaineer.streamreactor.connect.coap.source.CoapReader:60)
    [2017-01-09 20:42:45,928] INFO Discovered resources /large-post (com.datamountaineer.streamreactor.connect.coap.source.CoapReader:60)
    [2017-01-09 20:42:45,928] INFO Discovered resources /large-separate (com.datamountaineer.streamreactor.connect.coap.source.CoapReader:60)
    [2017-01-09 20:42:45,928] INFO Discovered resources /large-update (com.datamountaineer.streamreactor.connect.coap.source.CoapReader:60)

Check for records in Kafka
~~~~~~~~~~~~~~~~~~~~~~~~~~

Check for records in Kafka with the console consumer.

.. sourcecode:: bash

 ➜  bin/kafka-avro-console-consumer \
    --zookeeper localhost:2181 \
    --topic coap-topic \
    --from-beginning

.. sourcecode:: json

    {"message_id":{"int":4803},"type":{"string":"ACK"},"code":"4.04","raw_code":{"int":132},"rtt":{"long":35},"is_last":{"boolean":true},"is_notification":{"boolean":false},"source":{"string":"idvm-infk-mattern04.inf.ethz.ch:5683"},"destination":{"string":""},"timestamp":{"long":0},"token":{"string":"b24774e37c2314a4"},"is_duplicate":{"boolean":false},"is_confirmable":{"boolean":false},"is_rejected":{"boolean":false},"is_acknowledged":{"boolean":false},"is_canceled":{"boolean":false},"accept":{"int":-1},"block1":{"string":""},"block2":{"string":""},"content_format":{"int":-1},"etags":[],"location_path":{"string":""},"location_query":{"string":""},"max_age":{"long":60},"observe":null,"proxy_uri":null,"size_1":null,"size_2":null,"uri_host":null,"uri_port":null,"uri_path":{"string":""},"uri_query":{"string":""},"payload":{"string":""}}
    {"message_id":{"int":4804},"type":{"string":"ACK"},"code":"4.04","raw_code":{"int":132},"rtt":{"long":34},"is_last":{"boolean":true},"is_notification":{"boolean":false},"source":{"string":"idvm-infk-mattern04.inf.ethz.ch:5683"},"destination":{"string":""},"timestamp":{"long":0},"token":{"string":"b24774e37c2314a4"},"is_duplicate":{"boolean":false},"is_confirmable":{"boolean":false},"is_rejected":{"boolean":false},"is_acknowledged":{"boolean":false},"is_canceled":{"boolean":false},"accept":{"int":-1},"block1":{"string":""},"block2":{"string":""},"content_format":{"int":-1},"etags":[],"location_path":{"string":""},"location_query":{"string":""},"max_age":{"long":60},"observe":null,"proxy_uri":null,"size_1":null,"size_2":null,"uri_host":null,"uri_port":null,"uri_path":{"string":""},"uri_query":{"string":""},"payload":{"string":""}}
    {"message_id":{"int":4805},"type":{"string":"ACK"},"code":"4.04","raw_code":{"int":132},"rtt":{"long":35},"is_last":{"boolean":true},"is_notification":{"boolean":false},"source":{"string":"idvm-infk-mattern04.inf.ethz.ch:5683"},"destination":{"string":""},"timestamp":{"long":0},"token":{"string":"b24774e37c2314a4"},"is_duplicate":{"boolean":false},"is_confirmable":{"boolean":false},"is_rejected":{"boolean":false},"is_acknowledged":{"boolean":false},"is_canceled":{"boolean":false},"accept":{"int":-1},"block1":{"string":""},"block2":{"string":""},"content_format":{"int":-1},"etags":[],"location_path":{"string":""},"location_query":{"string":""},"max_age":{"long":60},"observe":null,"proxy_uri":null,"size_1":null,"size_2":null,"uri_host":null,"uri_port":null,"uri_path":{"string":""},"uri_query":{"string":""},"payload":{"string":""}}
    {"message_id":{"int":4806},"type":{"string":"ACK"},"code":"4.04","raw_code":{"int":132},"rtt":{"long":35},"is_last":{"boolean":true},"is_notification":{"boolean":false},"source":{"string":"idvm-infk-mattern04.inf.ethz.ch:5683"},"destination":{"string":""},"timestamp":{"long":0},"token":{"string":"b24774e37c2314a4"},"is_duplicate":{"boolean":false},"is_confirmable":{"boolean":false},"is_rejected":{"boolean":false},"is_acknowledged":{"boolean":false},"is_canceled":{"boolean":false},"accept":{"int":-1},"block1":{"string":""},"block2":{"string":""},"content_format":{"int":-1},"etags":[],"location_path":{"string":""},"location_query":{"string":""},"max_age":{"long":60},"observe":null,"proxy_uri":null,"size_1":null,"size_2":null,"uri_host":null,"uri_port":null,"uri_path":{"string":""},"uri_query":{"string":""},"payload":{"string":""}}

Features
--------

1.  Secure DTLS client connection.
2.  Supports Observable resources to stream changes on a resource to Kafka.
3.  Routing of data via KCQL to topics.
4.  Automatic conversion of CoAP Response messages to Connect Structs.

Kafka Connect Query Language
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**K** afka **C** onnect **Q** uery **L** anguage found here `GitHub repo <https://github.com/datamountaineer/kafka-connector-query-language>`__
allows for routing and mapping using a SQL like syntax, consolidating typically features in to one configuration option.

The CoAP Source supports the following:

.. sourcecode:: bash

    INSERT INTO <topic> SELECT * FROM <resource>

No selection of fields on the CoAP message is support. All the message attributes are mapped to predefined ``Struct`` representing
the CoAP response message.

DTLS Secure connections
^^^^^^^^^^^^^^^^^^^^^^^

The Connector use the  `Californium <https://github.com/eclipse/californium>`__ Java API and for secure connections use the
Scandium security module provided by Californium. Scandium (Sc) is an implementation of Datagram Transport Layer Security 1.2,
also known as `RFC 6347 <https://tools.ietf.org/html/rfc6347>`__.

Please refer to the Californium `certification <https://github.com/eclipse/californium/tree/master/demo-certs>`__ repo page for
more information.

The connector supports:

1.  SSL trust and key stores
2.  Public/Private PEM keys and PSK client/identity
3.  PSK Client Identity

The Sink will attempt secure connections in the following order if the URI schema of ``connect.coap.uri`` set to secure, i.e.``coaps``.
If ``connect.coap.username`` is set PSK client identity authentication is used, if additional ``connect.coap.private.key.path``
Public/Private keys authentication will also be attempt. Otherwise SSL trust and key store.

.. sourcecode:: bash

     `openssl pkcs8 -in privatekey.pem -topk8 -nocrypt -out privatekey-pkcs8.pem`

 Only cipher suites TLS_PSK_WITH_AES_128_CCM_8 and TLS_PSK_WITH_AES_128_CBC_SHA256 are currently supported.

.. warning::

    The keystore, truststore, public and private files must be available on the local disk of the worker task.

Loading specific certificates can be achieved by providing a comma separated list for the ``connect.coap.certs`` configuration option.
The certificate chain can be set by the ``connect.coap.cert.chain.key`` configuration option.

Configurations
--------------

``connect.coap.uri``

Uri of the CoAP server.

* Data Type : string
* Importance: high
* Optional  : no

``connect.coap.kcql``

The KCQL statement to select and route resources to topics.

* Data Type : string
* Importance: high
* Optional  : no

``connect.coap.port``

The port the DTLS connector will bind to on the Connector host.

* Data Type : int
* Importance: medium
* Optional  : yes
* Default   : 0

``connect.coap.host``

The hostname the DTLS connector will bind to on the Connector host.

* Data Type : string
* Importance: medium
* Optional  : yes
* Default   : localhost

``connect.coap.username``

CoAP PSK identity.

* Data Type : string
* Importance: medium
* Optional  : yes

``connect.coap.password``

CoAP PSK secret.

* Data Type : password
* Importance: medium
* Optional  : yes

``connect.coap.public.key.file``

Path to the public key file for use in with PSK credentials.

* Data Type : string
* Importance: medium
* Optional  : yes

``connect.coap.private.key.file``

 Path to the private key file for use in with PSK credentials in PKCS8 rather than PKCS1
 Use open SSL to convert.

.. sourcecode:: bash

     `openssl pkcs8 -in privatekey.pem -topk8 -nocrypt -out privatekey-pkcs8.pem`

 Only cipher suites TLS_PSK_WITH_AES_128_CCM_8 and TLS_PSK_WITH_AES_128_CBC_SHA256 are currently supported.

* Data Type : string
* Importance: medium
* Optional  : yes

``connect.coap.keystore.pass``

The password of the key store

* Data Type : string
* Importance: medium
* Optional  : yes
* Default   : rootPass

``connect.coap.keystore.path``

The path to the keystore.

* Data Type : string
* Importance: medium
* Optional  : yes
* Default   :

``connect.coap.truststore.pass``

The password of the trust store

* Data Type : string
* Importance: medium
* Optional  : yes
* Default   : rootPass

``connect.coap.truststore.path``

The path to the truststore.

* Data Type : string
* Importance: medium
* Optional  : yes
* Default   :

``connect.coap.certs``

The certificates to load from the trust store.

* Data Type : list
* Importance: medium
* Optional  : yes
* Default   :

``connect.coap.cert.chain.key``

The key to use to get the certificate chain.

* Data Type : string
* Importance: medium
* Optional  : yes
* Default   : client

``connect.coap.batch.size``

The number of events to take from the internal queue to batch together to send to Kafka.

* Data Type : init
* Importance: medium
* Optional  : yes
* Default   : 100

``connect.progress.enabled``

Enables the output for how many records have been processed.

* Type: boolean
* Importance: medium
* Optional: yes
* Default : false

.. _coap_schemas:

Schema Evolution
----------------

The schema is fixed.

The following schema is used for the key:

    +-----------------+---------------------------+
    | Name            | Type                      |
    +-----------------+---------------------------+
    | source          | Optional string           |
    +-----------------+---------------------------+
    | source_resource | Optional String           |
    +-----------------+---------------------------+
    | message_id      | Optional int32            |
    +-----------------+---------------------------+


The following schema is used for the payload:

    +-----------------+---------------------------+
    | Name            | Type                      |
    +-----------------+---------------------------+
    | message_id      | Optional int32            |
    +-----------------+---------------------------+
    | type            | Optional String           |
    +-----------------+---------------------------+
    | code            | Optional String           |
    +-----------------+---------------------------+
    | raw_code        | Optional int32            |
    +-----------------+---------------------------+
    | rtt             | Optional int64            |
    +-----------------+---------------------------+
    | is_last         | Optional boolean          |
    +-----------------+---------------------------+
    | is_notification | Optional boolean          |
    +-----------------+---------------------------+
    | source          | Optional String           |
    +-----------------+---------------------------+
    | destination     | Optional String           |
    +-----------------+---------------------------+
    | timestamp       | Optional int64            |
    +-----------------+---------------------------+
    | token           | Optional String           |
    +-----------------+---------------------------+
    | is_duplicate    | Optional boolean          |
    +-----------------+---------------------------+
    | is_confirmable  | Optional boolean          |
    +-----------------+---------------------------+
    | is_rejected     | Optional boolean          |
    +-----------------+---------------------------+
    | is_acknowledged | Optional boolean          |
    +-----------------+---------------------------+
    | is_canceled     | Optional boolean          |
    +-----------------+---------------------------+
    | accept          | Optional int32            |
    +-----------------+---------------------------+
    | block1          | Optional String           |
    +-----------------+---------------------------+
    | block2          | Optional String           |
    +-----------------+---------------------------+
    | content_format  | Optional int32            |
    +-----------------+---------------------------+
    | etags           | Array of Optional Strings |
    +-----------------+---------------------------+
    | location_path   | Optional String           |
    +-----------------+---------------------------+
    | location_query  | Optional String           |
    +-----------------+---------------------------+
    | max_age         | Optional int64            |
    +-----------------+---------------------------+
    | observe         | Optional int32            |
    +-----------------+---------------------------+
    | proxy_uri       | Optional String           |
    +-----------------+---------------------------+
    | size_1          | Optional String           |
    +-----------------+---------------------------+
    | size_2          | Optional String           |
    +-----------------+---------------------------+
    | uri_host        | Optional String           |
    +-----------------+---------------------------+
    | uri_port        | Optional int32            |
    +-----------------+---------------------------+
    | uri_path        | Optional String           |
    +-----------------+---------------------------+
    | uri_query       | Optional String           |
    +-----------------+---------------------------+
    | payload         | Optional String           |
    +-----------------+---------------------------+


Deployment Guidelines
---------------------

Distributed Mode
~~~~~~~~~~~~~~~~

Connect, in production should be run in distributed mode. 

1.  Install the Confluent Platform on each server that will form your Connect Cluster.
2.  Create a folder on the server called ``plugins/streamreactor/libs``.
3.  Copy into the folder created in step 2 the required connector jars from the stream reactor download.
4.  Edit ``connect-avro-distributed.properties`` in the ``etc/schema-registry`` folder where you installed Confluent
    and uncomment the ``plugin.path`` option. Set it to the path you deployed the stream reactor connector jars
    in step 2.
5.  Start Connect, ``bin/connect-distributed etc/schema-registry/connect-avro-distributed.properties``

Connect Workers are long running processes so set an ``init.d`` or ``systemctl`` service accordingly.

Connector configurations can then be push to any of the workers in the Cluster via the CLI or curl, if using the CLI 
remember to set the location of the Connect worker you are pushing to as it defaults to localhost.

.. sourcecode:: bash

    export KAFKA_CONNECT_REST="http://myserver:myport"

Kubernetes
~~~~~~~~~~

Helm Charts are provided at our `repo <https://datamountaineer.github.io/helm-charts/>`__, add the repo to your Helm instance and install. We recommend using the Landscaper
to manage Helm Values since typically each Connector instance has it's own deployment.

Add the Helm charts to your Helm instance:

.. sourcecode:: bash

    helm repo add datamountaineer https://datamountaineer.github.io/helm-charts/


TroubleShooting
---------------

Please review the :ref:`FAQs <faq>` and join our `slack channel <https://slackpass.io/datamountaineers>`_.