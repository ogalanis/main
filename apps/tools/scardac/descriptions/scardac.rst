scardac scans an :term:`SDS waveform archive <SDS>` , e.g.,
created by :ref:`slarchive` or :ref:`scart` for
available :term:`miniSEED <miniSeed>` data. It will
collect information about

* Data extents - the absolute earliest and latest times data is available of a
  particular channel,
* Data segments - continuous data segments sharing the same quality and sampling
  rate attributes.

scardac is intended to be executed periodically, e.g., as a cronjob.

The data availability information is stored in the SeisComP database under the
root element :ref:`DataAvailability <api-datamodel-python>`. Access to the data
availability is provided by the :ref:`fdsnws` module via the services:

* :ref:`/fdsnws/station <sec-station>` (extent information only, see
  ``matchtimeseries`` and ``includeavailability`` request parameters).
* :ref:`/fdsnws/ext/availability <sec-avail>` (extent and segment information
  provided in different formats)


.. _scarcac_non-sds:

Non-SDS archives
----------------

scardac can be extended by plugins to scan non-SDS archives. For example the
*daccaps* plugin provided by :cite:t:`caps` allows scanning archives generated
by a CAPS server. Plugins are added to global module configuration, e.g.:

.. code-block:: properties

   plugin = xyz


.. _scarcac_workflow:

Workflow
--------

#. Read existing ``Extents`` from database
#. Scan the SDS archive for new channel IDs and create new ``Extents``
#. Subsequently process the ``Extents`` using ``threads`` number of parallel
   threads. For each ``Extent``:

   #. Find all available daily data files
   #. Sort the file list according date
   #. For each data file

      * remove ``DataSegments`` that do longer exists
      * update or create ``DataSegments`` that changed or are new
      * a segment is split if

        * the ``jitter`` (difference between previous records end time and
          current records start time) is exceeded
        * the quality or sampling rate changed

      * merge segment information into ``DataAttributeExtents`` (``Extents``
        sharing the same quality and sample rate information)
      * merge segment start and end time into overall ``Extent``


Examples
--------

#. Get command line help or execute scardac with default parameters and informative
   debug output:

   .. code-block:: sh

      scardac -h
      scardac --debug

#. Update the availability of waveform data files existing in the standard
   :term:`SDS` archive to the seiscomp database and create an XML file using
   :ref:`scxmldump`:

   .. code-block:: sh

      scardac -d mysql://sysop:sysop@localhost/seiscomp -a $SEISCOMP_ROOT/var/lib/archive --debug
      scxmldump -Yf -d mysql://sysop:sysop@localhost/seiscomp -o availability.xml

#. Update the availability of waveform data files existing in the standard
   :term:`SDS` archive to the seiscomp database. Use :ref:`fdsnws` to fetch a flat file containing a list
   of periods of available data from stations of the CX network sharing the same
   quality and sampling rate attributes:

   .. code-block:: sh

      scardac -d mysql://sysop:sysop@localhost/seiscomp -a $SEISCOMP_ROOT/var/lib/archive
      wget -O availability.txt 'http://localhost:8080/fdsnws/ext/availability/1/query?network=CX'

   .. note::

      The |scname| module :ref:`fdsnws` must be running for executing this
      example.
