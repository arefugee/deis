:title: SSL Endpoints
:description: Configure SSL termination for your Deis cluster


.. _ssl-endpoints:

SSL/TLS Endpoints
=================

SSL/TLS is the standard security technology for establishing an encrypted link
between a web server and a browser. This link ensures that all data passed between the web server
and browsers remain private and integral.

To enable SSL for your cluster and all apps running upon it, you can add an SSL key to your load
balancer. You must either provide an SSL certificate that was registered with a CA or provide your
own self-signed SSL certificate.


Generating an SSL Certificate
-----------------------------

To generate your own self-signed SSL certificate for testing purposes, you can run the following:

.. code-block:: console

    $ openssl genrsa -out server.key 2048
    $ openssl req -new -key server.key -out server.csr

This will create a private key and a Certificate Signing Request. This CSR is typically sent to a
CA such as Verisign, but in this example we will be using it to sign our own SSL certificate.

Though most fields are self-explanatory, pay close attention to the following:

+--------------+-------------------------------------------------------------------------+
| Field        | Description                                                             |
+==============+=========================================================================+
| Country Name | The two letter code, in ISO 3166-1 format, of the country in which your |
|              | organization is based.                                                  |
+--------------+-------------------------------------------------------------------------+
| Common Name  | This is the fully qualified domain name that you wish to secure. In     |
|              | most cases, this will be a wildcard subdomain.                          |
+--------------+-------------------------------------------------------------------------+

To generate a temporary certificate which is good for 365 days, issue the following command:

.. code-block:: console

    $ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

.. note::

    Some SSL vendors like RapidSSL will secure both the root domain and the www subdomain if you
    set the Common Name to www.example.com

    See your vendor's documentation for more information.


Installing the SSL Certificate
------------------------------

On most cloud-based load balancers, you can install a SSL certificate onto the load balancer
itself. This is the recommended way of enabling SSL onto a cluster, as any communication inbound to
the cluster will be encrypted while the internal components of Deis will still communicate over
HTTP. To enable SSL, you will need to open port 443 on the load balancer and forward it to port 80
on the routers. For EC2, you'll also need to add port 443 in the security group settings for your
load balancer.

See your vendor's specific instructions on installing SSL on your load balancer. For EC2, see their
documentation on `installing an SSL cert for load balancing`_. For Rackspace, see their
`Product FAQ`_.

Installing SSL on the Deis Routers
----------------------------------

You can also use the Deis routers to terminate SSL connections.
Use ``deisctl`` to install the certificate and private keys:

.. code-block:: console

    $ deisctl config router set sslKey=<path-to-key> sslCert=<path-to-cert>

If your certificate has intermediate certs that need to be presented as part of a
certificate chain, append the intermediate certs to the bottom of the sslCert value.

.. note::

    To secure all endpoints on the platform domain, you must use a wildcard certificate.


Redirecting traffic to HTTPS
----------------------------

Once your cluster is serving traffic over HTTPS, you can optionally instruct the router component
to forward all traffic on HTTP to HTTPS (application traffic and requests to the controller component).

This is achieved with ``deisctl``:

.. code-block:: console

    $ deisctl config router set enforceHTTPS=true


.. _`installing an SSL cert for load balancing`: http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/ssl-server-cert.html
.. _`Product FAQ`: http://www.rackspace.com/knowledge_center/product-faq/cloud-load-balancers
