Python SDK Architecture
=======================

The Splunk SDK for Python is architected in a layered approach. Each
module is for the most part independent, and can be used independently.

Binding Layer
-------------

The ``binding`` layer (``splunklib.binding``) represents a very thin
abstraction over raw HTTP. While you are still free to make the raw HTTP
calls yourself, the ``binding`` layer gives you several useful pieces of
functionality:

-  It will construct proper URLs from endpoint fragments
   (e.g. ``search/jobs`` will become
   ``https://localhost:8089/servicesNS/-/-/search/jobs``). It will do
   this based on simple rules and the namespace information that is
   supplied to it during creation.
-  It can perform the ``login`` operation for you, remember your session
   key (and cookies if Splunk 6.2+ as of v1.4.0 of the SDK), and append
   the ``Authorization`` header or ``Cookie`` header to every request.
-  It will handle certificates for ``HTTPS`` access if specified.

The ``binding`` layer is divided into a few components:

``Context`` class
~~~~~~~~~~~~~~~~~

This is the main piece of functionality in the ``binding`` layer. It is
a utility class that will remember your username/password/session key
(and cookies if Splunk 6.2+ as of v1.4.0 of the SDK), as well as
construct URLs and make HTTP requests.

``HTTPError`` exception
~~~~~~~~~~~~~~~~~~~~~~~~

The Splunk SDK for Python exposes HTTP errors as HTTPError exceptions.
When we receive any response code that is greater or equal to ``400``,
we will throw an HTTPError exception. The exception will include a great
of useful information:

-  Response code
-  Error reason
-  Returned headers
-  Splunk-supplied error message
-  The actual response body

Custom HTTP request handlers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Splunk SDK for Python provides a default HTTP request handler. The
handler will, given a dictionary containing the HTTP method, URL,
headers and body, make an HTTP request to the specified URL.

However, you might want to write our own custom HTTP request handler.
Some reasons might be:

-  You want to have your program sit behind a proxy, and so need to
   configure the handler appropriately.
-  You might want to use eventing libraries like ``gevent`` or
   ``eventlet``.
-  You might want to add more logging for each HTTP request sent.

As such, when you create your an instance of your ``Context`` class, you
can supply your custom handler. It would look something like the
following:

.. code:: python

   def my_custom_handler(url, message, **kwargs):
       # do some custom HTTP handling here

   ctx = splunklib.binding.Context(..., handler=my_custom_handler)

We have written several examples of custom handlers, which you can find
here:
https://github.com/splunk/splunk-sdk-python/tree/master/examples/handlers

Client Layer
------------

What the ``binding`` layer is to HTTP the ``client``
(``splunklib.client``) layer is to the Splunk REST API. It provides a
\**stateless*, Pythonic approach to accessing the various endpoints
exposed by the REST API. The ``client`` layer sits on top of the
``binding`` layer, and uses its HTTP capabilities to easily access the
REST API.

Again, while you could easily rebuild the ``client`` layer with the
capbilities exposed in ``splunklib.binding``, the current implementation
has a few benefits:

-  Consistent access to Splunk resources. For example, whether you want
   to access ``apps`` or ``users``, the code and capabilities are mostly
   the same to list, add, remove and update.
-  Useful abstractions on top of the REST API. For example, you can
   easily use the ``receivers/simple`` endpoint by just getting the
   index you want to submit to:

   .. code:: python

      index = service.indexes["my_index"]
      index.submit("some event", sourcetype="myevent")

The ``client`` layer is divided into a few components:

``Service`` class
~~~~~~~~~~~~~~~~~

The ``Service`` class inherits from ``splunklib.binding.Context``, and
provides utility functions to access specific parts of the Splunk REST
API. For example, should you want to get a list of installed apps on
Splunk, the following code would be sufficient:

.. code:: python

   svc = splunklib.client.connect(...)
   print svc.apps.list()

Similarily, if you wanted to create a new index:

.. code:: python

   svc.indexes.create("my_index")

``Endpoint``, ``Entity`` and ``Collection`` classes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These sets of classes encapsulate functionality common to different
kinds of Splunk REST resources.

``Endpoint`` class
~~~~~~~~~~~~~~~~~~

The ``Endpoint`` class provides common functions relevant to all REST
endpoints. It “wraps” over the path, and provides methods to execute
``GET`` and ``POST`` HTTP requests against that path.

``Entity`` class
~~~~~~~~~~~~~~~~

The ``Entity`` class is a subclass of ``Endpoint``, and provides
functions that implement the Splunk “entity” protocol. This includes the
ability to read properties of the entity, update properties, read
metadata, and so on.

``Collection`` class
~~~~~~~~~~~~~~~~~~~~

Similar to the ``Endpoint`` class, the ``Collection`` class is subclass
of ``Endpoint``, and provides functions that implement the Splunk
“collection” protocol. This includes listing all entries in the
collection, getting a specific item, creating new items, and so on.

Specific endpoint extensions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Beyond the generic classes, the ``client`` layer also provides
endpoint-specific implementations and extensions for some REST
endpoints. For example, as noted above, we have added the ``submit``
function to the ``Index`` class (which itself inherits from
``Entity) in order to provide Pythonic access to the``\ receivers/simple\`
endpoint.

Results
-------

The ``splunklib.results`` module provides a Splunk-specific streaming
XML reader. It abstracts over the details of the Splunk XML responses,
and provides a Pythonic way to access the stream of data. The best way
to understand it is an example:

.. code:: python

   service = splunklib.client.connect(...)
   job = service.jobs.create("search index=myindex | stats count by sourcetype", exec_mode="blocking")
   reader = splunklib.results.ResultsReader(job.results())

   for kind, result in reader:
       if kind == results.RESULT:
           # This is a result
           sourcetype = result["sourcetype"]
           count = int(result["count"])
           print "%s: %d" % (sourcetype, count)

You can also use the ``ResultsReader`` class with the ``search/export``
endpoint, which will provide you with streaming output as results become
available.

Note that it is up to you whether you wish to use ``ResultsReader`` or
not. Neither the ``client`` or ``binding`` layers will make use of it
for you.

Data
----

**UNDONE**
