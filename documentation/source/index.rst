*******
Tracing
*******

.. current-library:: tracing

This is a tracing library based on the `Dapper`_ work by Google.
It is also similar to Twitter's `Zipkin`_ and `HTrace`_ from Cloudera
which were also inspired by the `Dapper`_ work.

Tracing is different from simply recording metrics about a service
or logging text strings to a log file. Tracing is intended to capture
information about the units of work that are being performed, how they
relate to one another, and how long the pieces of the units of work
take to execute.

Each unit of work at the top level is captured by a trace which consists
of a tree of spans (:class:`<span>`). Each unit of work is tracked by
a span in the tree.

Each span can have key/value data associated with it via either
:func:`trace-add-data` or :func:`span-add-data`. This data typically
represents data associated with the computation or action. Examples
might include the user name making a request, the SQL being executed,
the table being queried, or the IP address associated with a request.

Each span can also have timestamped annotations provided via either
:func:`trace-annotate` or :func:`span-annotate`. This associates a
text description of an event with a timestamp and the span. This might
be used to indicate progress through a task, unusual events, or
anything interesting.

Other important details are discussed below, such as `Writers`_
and `Sampling`_.

.. _Dapper: http://research.google.com/pubs/pub36356.html
.. _Zipkin: http://twitter.github.io/zipkin/
.. _HTrace: https://github.com/cloudera/htrace/

The TRACING-CORE module
=======================

.. current-module:: tracing-core

.. contents::
   :local:

Tracing
-------

The tracing functions in this section represent the high level
interface to the tracing library and are what would typically
be used, rather than the span-specific functions.

There may be times though when using the lower level,
span-specific functions is appropriate, such as when you have
multiple units of work executing asynchronously. The asynchronous
tasks may find it easier to track their own spans separately.

.. function:: trace-push

   :signature: trace-push (description #key sampler) => (span?)

   :parameter description: An instance of :drm:`<string>`.
   :parameter #key sampler: An instance of :drm:`<function>`.
   :value span?: An instance of ``false-or(<span>)``.

   :description:

     Create a new :class:`<span>` and make it the current tracing
     span. If there is already a span, the new span will use the
     existing span as the parent.

   See also:

   * :func:`trace-pop`

.. function:: trace-add-data

   :signature: trace-add-data (key data) => ()

   :parameter key: An instance of :drm:`<string>`.
   :parameter data: An instance of :drm:`<string>`.

   :description:

     Adds key / value data to the current trace span (if any),
     using :gf:`span-add-data`.

   See also:

   * :gf:`span-add-data`

.. function:: trace-annotate

   :signature: trace-annotate (description) => ()

   :parameter description: An instance of :drm:`<string>`.

   :description:

     Adds an annotation to the current trace span (if any), using
     :gf:`span-annotate`.

   See also:

   * :gf:`span-annotate`

.. function:: trace-pop

   :signature: trace-pop (span?) => ()

   :parameter span?: An instance of ``false-or(<span>)``.

   :description:

     Stops the current span and pops it from the stack, returning
     the previous span to the current slot.

   See also:

   * :func:`trace-push`

.. macro:: with-tracing

   :macrocall:

     .. code-block:: dylan

       with-tracing ("Span description")
         trace-add-data("Table", "users");
         ...
       end with-tracing

Spans
-----

.. class:: <span>

   :superclasses: <object>

   :keyword description:
   :keyword parent-id:
   :keyword trace-id:

   :description:

     A span tracks a period of time associated with a computation
     or action, along with annotations and key / value data. Spans
     exist within a tree of spans all of which share the same
     ``trace-id``.

.. generic-function:: span-accumulated-time

   :signature: span-accumulated-time (span) => (time?)

   :parameter span: An instance of :class:`<span>`.
   :value time?: An instance of ``false-or(<duration>)``.

   :description:

     If the span has not yet been stopped, this returns ``#f``. Once
     the span has been stopped, the duration that the span was running
     will be returned.

.. generic-function:: span-add-data

   :signature: span-add-data (span key data) => ()

   :parameter span: An instance of :class:`<span>`.
   :parameter key: An instance of :drm:`<string>`.
   :parameter data: An instance of :drm:`<string>`.

   :description:

      Key / value pairs may be stored on a span to provide better
      context. This might include the query being executed, address
      or host information or whatever is relevant to the application
      being traced.

   See also:

   * :gf:`span-data`

.. generic-function:: span-annotate

   :signature: span-annotate (span description) => ()

   :parameter span: An instance of :class:`<span>`.
   :parameter description: An instance of :drm:`<string>`.

   :description:

      Annotations are to record an occurrence of an event
      during a span. They have a specific timestamp associated
      with them that is automatically set to the time when
      the annotation is created.

   See also:

   * :gf:`span-annotations`
   * :class:`<span-annotation>`
   * :gf:`annotation-description`
   * :gf:`annotation-timestamp`

.. generic-function:: span-annotations

   Returns the collection of :class:`<span-annotation>` associated with
   this span.

   :signature: span-annotations (span) => (annotations)

   :parameter span: An instance of :class:`<span>`.
   :value annotations: An instance of :drm:`<vector>`.

   See also:

   * :gf:`span-annotate`
   * :class:`<span-annotation>`
   * :gf:`annotation-description`
   * :gf:`annotation-timestamp`

.. generic-function:: span-data

   Returns the property list of data associated with this span.

   :signature: span-data (span) => (data)

   :parameter span: An instance of :class:`<span>`.
   :value data: An instance of :drm:`<vector>`.

   See also:

   * :gf:`span-add-data`

.. generic-function:: span-description

   Returns the description of the span.

   :signature: span-description (span) => (description)

   :parameter span: An instance of :class:`<span>`.
   :value description: An instance of :drm:`<string>`.

.. generic-function:: span-id

   Returns the unique ID associated with this span.

   :signature: span-id (span) => (id)

   :parameter span: An instance of :class:`<span>`.
   :value id: An instance of ``<object>``.

.. generic-function:: span-parent-id

   :signature: span-parent-id (span) => (id)

   :parameter span: An instance of :class:`<span>`.
   :value id: An instance of ``<object>``.

.. generic-function:: span-stop

   Stops a span and sends it to the current registered
   :class:`<span-writer>` instances.

   :signature: span-stop (span) => ()

   :parameter span: An instance of :class:`<span>`.

   See also:

   * :gf:`span-stopped?`
   * :func:`store-span`

.. generic-function:: span-stopped?

   Has the span been stopped yet?

   :signature: span-stopped? (span) => (well?)

   :parameter span: An instance of :class:`<span>`.
   :value #rest results: An instance of :drm:`<boolean>`.

   See also:

   * :gf:`span-stop`

.. generic-function:: span-trace-id

   Return the trace-id for a span.

   :signature: span-trace-id (span) => (id)

   :parameter span: An instance of :class:`<span>`.
   :value id: An instance of ``<object>``.

   :description:

     Returns the trace-id for a span. This ID is the same for all
     spans within a single trace.

Annotations
-----------

Annotations let you attach events that happened at a point in time
(noted by a timestamp) to a span.

.. class:: <span-annotation>

   :superclasses: <object>

   :keyword description:
   :keyword timestamp:

.. generic-function:: annotation-description

   Return the description of an annotation.

   :signature: annotation-description (annotation) => (description)

   :parameter annotation: An instance of :class:`<span-annotation>`.
   :value description: An instance of :drm:`<string>`.

.. generic-function:: annotation-timestamp

   Return the timestamp at which the annotation was created and attached.

   :signature: annotation-timestamp (annotation) => (timestamp)

   :parameter annotation: An instance of :class:`<span-annotation>`.
   :value timestamp: An instance of :class:`<timestamp>`.

Sampling
--------

Samplers allow for collecting a subset of the data, making the
usage of this tracing framework in a heavily loaded production
scenario more realistic.

Samplers are simply a function that returns a boolean value
indicating whether or not an actual trace should be generated
and recorded.

.. note:: In the future, the sampler will take arguments
   to let it make contextual decisions about sampling.

.. function:: $always-sample

   Alaways returns true, so that the trace is sampled.

   :signature: $always-sample () => (well?)

   :value well?: An instance of :drm:`<boolean>`.

.. function:: $if-tracing-sample

   Returns true if tracing is enabled, otherwise ``#f``.

   :signature: $if-tracing-sample () => (well?)

   :value well?: An instance of :drm:`<boolean>`.

   See also:

   * :func:`disable-tracing`
   * :func:`enable-tracing`
   * :func:`tracing-enabled?`

.. function:: $never-sample

   Always returns false, so that the trace isn't sampled.

   :signature: $never-sample () => (well?)

   :value well?: An instance of :drm:`<boolean>`.

.. function:: disable-tracing

   :signature: disable-tracing () => ()

   See also:

   * :func:`enable-tracing`
   * :func:`tracing-enabled?`

.. function:: enable-tracing

   :signature: enable-tracing () => ()

   See also:

   * :func:`disable-tracing`
   * :func:`tracing-enabled?`

.. function:: tracing-enabled?

   :signature: tracing-enabled? () => (well?)

   :value well?: An instance of :drm:`<boolean>`.

   See also:

   * :func:`disable-tracing`
   * :func:`enable-tracing`

Writers
-------

Spans are stored by using instances of :class:`<span-writer>` which
have been registered using :func:`register-span-writer`. Spans are
stored when they are stopped (:func:`trace-pop`, :func:`span-stop`).
Spans are also stored when they are finalized without having been
stopped previously. This finalization is only present to prevent
data from being lost and should not be a default mode of operation.

.. class:: <span-writer>

   :superclasses: <object>

   See also:

   * :func:`register-span-writer`
   * :func:`registered-span-writers`
   * :func:`unregister-span-writer`

.. function:: register-span-writer

   :signature: register-span-writer (span-writer) => ()

   :parameter span-writer: An instance of :class:`<span-writer>`.

   See also:

   * :class:`<span-writer>`
   * :func:`registered-span-writers`
   * :func:`unregister-span-writer`

.. function:: registered-span-writers

   :signature: registered-span-writers () => (span-writers)

   :value span-writers: An instance of ``<span-writer-vector>``.

   See also:

   * :class:`<span-writer>`
   * :func:`register-span-writer`
   * :func:`unregister-span-writer`

.. function:: store-span

   :signature: store-span (span) => ()

   :parameter span: An instance of :class:`<span>`.

   See also:

   * :func:`registered-span-writers`

.. function:: unregister-span-writer

   :signature: unregister-span-writer (span-writer) => ()

   :parameter span-writer: An instance of :class:`<span-writer>`.

   See also:

   * :class:`<span-writer>`
   * :func:`register-span-writer`
   * :func:`registered-span-writers`

Writer Implementation
---------------------

To add a new storage, subclass :class:`<span-writer>` and
implement the :gf:`span-writer-add-span` method. Then, call
:func:`register-span-writer` with an instance of your span
writer and all subsequent spans completed will be written to it.

.. generic-function:: span-writer-add-span

   :signature: span-writer-add-span (span span-writer) => ()

   :parameter span: An instance of :class:`<span>`.
   :parameter span-writer: An instance of :class:`<span-writer>`.

   :description:

      This method is specialized for each subclass of
      :class:`<span-writer>`. It is called whenever a span
      needs to be processed by a span writer.

Miscellaneous
-------------

.. function:: get-unique-id

   :signature: get-unique-id () => (id)

   :value id: An instance of ``<unique-id>``.

