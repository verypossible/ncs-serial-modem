.. _SM_AT_GNSS:

GNSS AT commands
****************

.. contents::
   :local:
   :depth: 1

This page describes GNSS-related AT commands.

Control GNSS #XGNSS
===================

The ``#XGNSS`` command controls the GNSS.

Set command
-----------

The set command allows you to start and stop the GNSS module.

The `nRF Cloud A-GNSS`_ service can be used with the module to reduce the time it takes for the GNSS module to estimate device position.

.. only:: not nrf91m1

   Both the `nRF Cloud A-GNSS`_ service and the `nRF Cloud P-GPS`_ service can be used with the module, either together or independently of each other.
   Using them reduces the time it takes for the GNSS module to estimate device position.

   To use either of them, the device must be connected to nRF Cloud (using the :ref:`#XNRFCLOUD <SM_AT_NRFCLOUD>` AT command) when starting the GNSS module.
   In addition, the following Kconfig options must be enabled:

   * ``CONFIG_NRF_CLOUD_AGNSS`` to use A-GNSS.
   * ``CONFIG_NRF_CLOUD_PGPS`` to use P-GPS.
     This also requires applying the :file:`pgps-nrf9151.overlay` devicetree overlay to both the application and the ``mcuboot`` image (using the absolute path for the latter).

   If both assistive services were enabled during compilation, you cannot choose to use only one of them at run time.

Syntax
~~~~~~

::

   AT#XGNSS=<op>,<cloud_assistance>,<interval>[,<timeout>]

The parameters and their defined values are the following:

<op>
   * ``0`` - Stop the GNSS module.
     In this case, no other parameter is allowed.
   * ``1`` - Start the GNSS module.

<cloud_assistance>
   * ``0`` - Disabled.
   * ``1`` - Enabled.

<interval>
   Integer.
   The GNSS fix interval in seconds.
   It can have one of the following values:

   * ``0`` - Single-fix navigation mode.
   * ``1`` - Continuous navigation mode.
     The fix interval is set to 1 second.
   * Ranging from ``10`` to ``65535`` - Periodic navigation mode.
     The fix interval is set to the specified value.

<timeout>
   Integer.
   The maximum time in seconds that the GNSS receiver is allowed to run while trying to produce a valid Position, Velocity, and Time (PVT) estimate.
   Can only be specified in single-fix and periodic navigation modes, and must be omitted in continuous navigation mode.
   If omitted in single-fix or periodic navigation modes, the default value is 60 seconds.

   * ``0`` - The GNSS receiver runs indefinitely until a valid PVT estimate is produced.
   * Any positive integer - The GNSS receiver is turned off after the specified time is up, even if a valid PVT estimate was not produced.

In periodic navigation mode, the ``<interval>`` and ``<timeout>`` parameters are temporarily ignored during the first fix.

.. note::

   When ``<cloud_assistance>`` is disabled, no request is made to nRF Cloud for assistance data.
   However, if it has been previously enabled and used, such data may remain locally available and will be used if still valid.

.. note::

   Make sure that the GNSS antenna is configured properly.
   You may need to issue the ``AT%XMAGPIO`` and ``AT%XCOEX0`` AT commands to configure the Low-Noise Amplifier (LNA), depending on the hardware (HW).

   If you want to use the nRF9151 DK with an onboard antenna, use the ``AT%XCOEX0=1,1,1565,1586`` command.

.. tip::

   When the LTE link is enabled, make sure to have either Power Saving Mode (PSM) or extended Discontinuous Reception (eDRX) enabled to give the GNSS receiver the time it needs to acquire fixes.

.. only:: not nrf91m1

   .. tip::

      The |SM| application logs NMEA and PVT data when trying to acquire fixes, which can help with troubleshooting.
      These logs are enabled if the ``CONFIG_SM_LOG_LEVEL_DBG`` Kconfig option is set and logs have been enabled with ``AT#XLOG``.

   .. note::

      See the documentation for the `nRF Cloud A-GNSS`_ and `nRF Cloud P-GPS`_ libraries for information on how to best configure and use A-GNSS and P-GPS, respectively.

   .. note::

      When using P-GPS, make sure that the value of the :ref:`CONFIG_SM_PGPS_INJECT_FIX_DATA <CONFIG_SM_PGPS_INJECT_FIX_DATA>` Kconfig option matches your use case.
      It is enabled by default but should be disabled if the device is expected to move distances longer than a few dozen kilometers between fix attempts.

   .. note::

      When using P-GPS, apply the :file:`pgps-nrf9151.overlay` devicetree overlay to both the application and the ``mcuboot`` image to provide a dedicated flash partition for P-GPS prediction data storage and to ensure MCUboot operates with the same partition layout.
      The absolute path to the overlay file must be provided when passing it to the ``mcuboot`` image using :makevar:`mcuboot_EXTRA_DTC_OVERLAY_FILE`.
      See :ref:`sm_config_files` for more information.

As an alternative to GNSS-based positioning, see :ref:`#XNRFCLOUDPOS <SM_AT_NRFCLOUDPOS>` for cellular and Wi-Fi® positioning.

Unsolicited notification
~~~~~~~~~~~~~~~~~~~~~~~~

``#XGNSSPOS`` notification indicates that a new position fix has been acquired.

.. code-block::

   #XGNSSPOS: <latitude>,<longitude>,<altitude>,<accuracy>,<speed>,<heading>,<datetime>

The parameters and their defined values are the following:

<latitude>
   Float.
   The latitude in degrees.

<longitude>
   Float.
   The longitude in degrees.

<altitude>
   Float.
   The altitude above the WGS-84 ellipsoid in meters.

<accuracy>
   Float.
   The accuracy (2D 1-sigma) in meters.

<speed>
   Float.
   The horizontal speed in meters per second.

<heading>
   Float.
   The heading of the movement of the user in degrees.

<datetime>
   String.
   The UTC date-time.

``#XGNSS`` notification indicates GNSS status.

.. sm_gps_status_notif_start

.. code-block::

   #XGNSS: <gnss_service>,<gnss_status>

The parameters and their defined values are the following:

<gnss_service>
   Integer.
   When it has the value ``1``, it means that GNSS is supported in ``%XSYSTEMMODE`` and activated in ``+CFUN``.

<gnss_status>
   * ``0`` - GNSS is stopped.
   * ``1`` - GNSS is started.
   * ``2`` - GNSS wakes up in periodic mode.
   * ``3`` - GNSS enters sleep because of timeout.
   * ``4`` - GNSS enters sleep because a fix is acquired.

.. sm_gps_status_notif_end

Example
~~~~~~~

Get GNSS fix in GNSS only mode:

::

  AT%XSYSTEMMODE=0,0,1,0

  OK
  AT+CFUN=31

  OK
  AT#XGNSS=1,0,0,0

  #XGNSS: 1,1

  OK

  #XGNSSPOS: 35.457576,139.625090,121.473785,22.199919,0.442868,0.000000,"2021-06-02 06:25:48"

  #XGNSS: 1,4

Set PSM and eDRX configurations, use GNSS assistance from nRF Cloud and get GNSS fixes in continuous navigation mode:

::

  AT%XSYSTEMMODE=1,0,1,0

  OK
  AT+CPSMS=1,,,"00000001","00000011"

  OK
  AT+CEDRXS=2,4,"0011"

  OK
  AT+CFUN=1

  OK

  +CEDRXP: 4,"0011","0011","0011"

  AT#XNRFCLOUD=1

  OK
  #XNRFCLOUD: 1,0
  AT#XGNSS=1,1,1

  #XGNSS: 1,1

  OK

  #XGNSSPOS: 35.457417,139.625211,162.850952,15.621976,1.418092,0.000000,"2021-06-02 05:21:31"

  #XGNSSPOS: 35.457435,139.625348,176.104797,14.245458,1.598184,69.148659,"2021-06-02 05:21:32"

  #XGNSSPOS: 35.457417,139.625415,179.132980,13.318132,1.235241,69.148659,"2021-06-02 05:21:33"

  #XGNSSPOS: 35.457410,139.625469,181.223541,12.667312,0.803951,69.148659,"2021-06-02 05:21:34"
  AT#XGNSS=0
  OK

  #XGNSS: 1,0

Set PSM and eDRX configurations, use GNSS assistance from nRF Cloud and get GNSS fixes in periodic navigation mode every 10 seconds:

::

  AT%XSYSTEMMODE=1,0,1,0

  OK
  AT+CPSMS=1,,,"00000001","00000011"

  OK
  AT+CEDRXS=2,4,"0011"

  OK
  AT+CFUN=1

  OK
  AT#XNRFCLOUD=1

  OK
  #XNRFCLOUD: 1,0
  AT#XGNSS=1,1,10

  #XGNSS: 1,1

  OK

  #XGNSS: 1,2

  #XGNSSPOS: 35.457243,139.625435,149.005020,28.184258,10.431827,281.446014,"2021-06-24 04:35:52"

  #XGNSS: 1,4

  #XGNSS: 1,2

  #XGNSSPOS: 35.457189,139.625602,176.811203,43.015198,0.601837,281.446014,"2021-06-24 04:36:32"

  #XGNSS: 1,4

  #XGNSS: 1,2

  #XGNSSPOS: 35.457498,139.625422,168.243591,31.753956,0.191195,281.446014,"2021-06-24 04:37:12"

  #XGNSS: 1,4

  #XGNSS: 1,2

  #XGNSSPOS: 35.457524,139.624667,100.745979,25.324850,6.347160,94.699837,"2021-06-24 04:37:52"

  #XGNSS: 1,4
  AT#XGNSS=0

  #XGNSS: 1,0

  OK

Read command
------------

The read command allows you to check GNSS support and its status.

Syntax
~~~~~~

::

   AT#XGNSS?

Response syntax
~~~~~~~~~~~~~~~

.. include:: at_gnss.rst
   :start-after: sm_gps_status_notif_start
   :end-before: sm_gps_status_notif_end

Example
~~~~~~~

::

  AT#XGNSS?

  #XGNSS: 1,1

  OK

Test command
------------

The test command tests the existence of the command and provides information about the type of its subparameters.

Syntax
~~~~~~

::

   AT#XGNSS=?

Example
~~~~~~~

::

  AT#XGNSS=?

  #XGNSS: (0,1),(0,1),<interval>,<timeout>

  OK

GNSS dynamics mode #XGNSSDYN
============================

The ``#XGNSSDYN`` command selects the GNSS dynamics mode, which tunes positioning for how the device moves.
The default is stationary.

The modem accepts a dynamics mode only while GNSS is running, and stores it in non-volatile memory.
The selected mode is applied when GNSS is first started after boot, and again when it is changed.

Set command
-----------

The set command allows you to select the dynamics mode.
If GNSS is running, the mode is applied immediately.
Otherwise, it is applied when GNSS is next started with ``#XGNSS``.

Syntax
~~~~~~

::

   AT#XGNSSDYN=<mode>

The parameters and their defined values are the following:

<mode>
   * ``0`` - General purpose.
   * ``1`` - Stationary (default).
   * ``2`` - Pedestrian.
   * ``3`` - Automotive.

Example
~~~~~~~

::

  AT#XGNSSDYN=2
  OK

Read command
------------

The read command allows you to check the selected mode, and whether the modem has accepted it since boot.

Syntax
~~~~~~

::

   AT#XGNSSDYN?

Response syntax
~~~~~~~~~~~~~~~

::

   #XGNSSDYN: <mode>,<applied>

<mode>
   The selected mode, as in the set command.

<applied>
   * ``0`` - Not yet applied. It is applied when GNSS is next started.
   * ``1`` - Applied.

Example
~~~~~~~

::

  AT#XGNSSDYN?

  #XGNSSDYN: 1,1

  OK

Test command
------------

The test command tests the existence of the command and provides information about the type of its subparameters.

Syntax
~~~~~~

::

   AT#XGNSSDYN=?

Example
~~~~~~~

::

  AT#XGNSSDYN=?

  #XGNSSDYN: (0,1,2,3)

  OK

GNSS time #XGNSSTIME
====================

The ``#XGNSSTIME`` command gives GNSS the current time from the host.

GNSS loses the GPS system time when the modem resets or powers off, although its stored almanacs and ephemerides survive.
Without the time, it cannot use them to predict which satellites are visible, and searches as in a cold start.
Injecting the time before starting GNSS lets it use the stored data.

The command requires GNSS to be enabled in the system mode and the functional mode, for example after ``AT+CFUN=31``.

Set command
-----------

The set command injects the given UTC time into GNSS, unless GNSS already knows the time.
Send it before starting GNSS with ``#XGNSS``.

Syntax
~~~~~~

::

   AT#XGNSSTIME=<utc_s>[,<ms>]

The parameters and their defined values are the following:

<utc_s>
   Integer.
   UTC time in seconds since 1970-01-01 (Unix time).
   It must not be earlier than the GPS epoch, 1980-01-06.

<ms>
   Integer.
   Milliseconds part of the time, ``0`` to ``999``.
   The default is ``0``.

Response syntax
~~~~~~~~~~~~~~~

::

   #XGNSSTIME: <injected>

<injected>
   * ``0`` - Not injected, because GNSS already knows the time.
   * ``1`` - Injected.

Example
~~~~~~~

::

  AT+CFUN=31
  OK
  AT#XGNSSTIME=1795046400
  #XGNSSTIME: 1
  OK
  AT#XGNSS=1,0,0,0
  OK

Read command
------------

The read command allows you to check whether GNSS knows the time.
It cannot return the time itself.

Syntax
~~~~~~

::

   AT#XGNSSTIME?

Response syntax
~~~~~~~~~~~~~~~

::

   #XGNSSTIME: <known>

<known>
   * ``0`` - GNSS does not know the time.
   * ``1`` - GNSS knows the time.

Example
~~~~~~~

::

  AT#XGNSSTIME?

  #XGNSSTIME: 1

  OK

Test command
------------

The test command tests the existence of the command and provides information about the type of its subparameters.

Syntax
~~~~~~

::

   AT#XGNSSTIME=?

Example
~~~~~~~

::

  AT#XGNSSTIME=?

  #XGNSSTIME: <utc_s>,<ms>

  OK

GNSS PVT status #XGNSSPVT
=========================

The ``#XGNSSPVT`` command reports the latest PVT (position, velocity, and time) frame from GNSS: a summary, and the satellites it is tracking.
It is meant for debugging GNSS reception, for example to see how many satellites are tracked and their signal strength while waiting for a fix.

.. note::
   This is a debug feature.

Set command
-----------

The set command allows you to subscribe to PVT frames as unsolicited notifications.
The notifications are sent to the channel the command was issued on.
The setting is kept when GNSS is stopped and started again.

Syntax
~~~~~~

::

   AT#XGNSSPVT=<mode>[,<every>]

The parameters and their defined values are the following:

<mode>
   * ``0`` - No notifications (default).
   * ``1`` - ``#XGNSSPVT`` summary only.
   * ``2`` - ``#XGNSSPVT`` summary followed by one ``#XGNSSSV`` line per tracked satellite.

<every>
   Integer.
   Send a notification on every ``<every>``-th PVT frame, counted from the start of GNSS.
   The default is ``1``, which is every frame.
   In continuous navigation mode, GNSS produces a PVT frame every second.

Unsolicited notification
~~~~~~~~~~~~~~~~~~~~~~~~

The notifications have the same syntax as the read command response.

Example
~~~~~~~

::

  AT#XGNSSPVT=1,10
  OK
  AT#XGNSS=1,0,0,0
  OK

  #XGNSSPVT: 10,1,0,10003,4,0

  #XGNSSPVT: 20,1,0,20005,6,0

  #XGNSSPVT: 30,0,1,30002,7,5

Read command
------------

The read command allows you to read the latest PVT frame, a summary followed by one line per tracked satellite.

Syntax
~~~~~~

::

   AT#XGNSSPVT?

Response syntax
~~~~~~~~~~~~~~~

::

   #XGNSSPVT: <count>,<age_ms>,<flags>,<exec_ms>,<tracked>,<used>
   #XGNSSSV: <sv>,<signal>,<cn0>,<elevation>,<azimuth>,<sv_flags>
   ...

If GNSS has produced no PVT frame since it was started, the response is ``#XGNSSPVT: 0``, without the other fields.

<count>
   Integer.
   Number of PVT frames since GNSS was started.

<age_ms>
   Integer.
   Milliseconds since the frame was received.

<flags>
   Integer.
   A bitwise ``OR`` of the following bitmasks:

   * ``0x01`` - Fix valid.
   * ``0x02`` - Leap second valid.
   * ``0x04`` - GNSS slept between PVT frames (power saving).
   * ``0x08`` - GNSS missed its deadline because of LTE activity.
   * ``0x10`` - Not enough time windows were given to GNSS because of LTE activity.
   * ``0x20`` - Velocity valid.
   * ``0x40`` - GNSS is downloading ephemerides or almanacs from the satellites.

<exec_ms>
   Integer.
   Cumulative GNSS execution time since it was started, in milliseconds.

<tracked>
   Integer.
   Number of satellites being tracked.

<used>
   Integer.
   Number of tracked satellites used in the fix.

<sv>
   Integer.
   Satellite ID.

<signal>
   * ``1`` - GPS L1 C/A.
   * ``3`` - QZSS L1 C/A.
   * ``4`` - Galileo E1.

<cn0>
   Integer.
   Carrier-to-noise density ratio, in units of 0.1 dB-Hz.

<elevation>
   Integer.
   Satellite elevation angle in degrees.

<azimuth>
   Integer.
   Satellite azimuth angle in degrees.

<sv_flags>
   Integer.
   A bitwise ``OR`` of the following bitmasks:

   * ``0x02`` - Used in the fix.
   * ``0x08`` - Unhealthy.

Example
~~~~~~~

::

  AT#XGNSSPVT?

  #XGNSSPVT: 42,312,1,42008,2,2

  #XGNSSSV: 5,1,412,63,112,2

  #XGNSSSV: 12,1,385,41,287,2

  OK

Test command
------------

The test command tests the existence of the command and provides information about the type of its subparameters.

Syntax
~~~~~~

::

   AT#XGNSSPVT=?

Example
~~~~~~~

::

  AT#XGNSSPVT=?

  #XGNSSPVT: (0,1,2),<every>

  #XGNSSPVT: <count>,<age_ms>,<flags>,<exec_ms>,<tracked>,<used>

  #XGNSSSV: <sv>,<signal>,<cn0>,<elevation>,<azimuth>,<flags>

  OK

Delete GNSS data #XGNSSDEL
==========================

The ``#XGNSSDEL`` command deletes GNSS data from non-volatile memory.
This command should be issued when GNSS is activated but not started yet.

.. note::
   This is a debug feature, and is not supposed to be used in production code.

Set command
-----------

The set command allows you to delete cached GNSS data.
Using this command does not trigger A-GNSS nor P-GPS data request events.
The execution of this command may delay the full functionality of A-GNSS and P-GPS until the next periodic data request has been received.

Syntax
~~~~~~

::

   AT#XGNSSDEL=<mask>

The parameters and their defined values are the following:

<mask>
   Integer.
   A bitwise ``OR`` of the following bitmasks:

   * ``0x001`` - Ephemerides.
   * ``0x002`` - Almanacs (excluding leap second and ionospheric correction).
   * ``0x004`` - Ionospheric correction parameters.
   * ``0x008`` - Last good fix (the last position).
   * ``0x010`` - GPS time-of-week (TOW).
   * ``0x020`` - GPS week number.
   * ``0x040`` - Leap second (UTC parameters).
   * ``0x100`` - Precision estimate of GPS time-of-week (TOW).
   * ``0x400`` - Extended Kalman Filter (EKF) state (last position/velocity/time solution).

For example, the integer value with all of the above bits enabled will be ``1407`` (``0x57f``).

Example
~~~~~~~

::

  AT%XSYSTEMMODE=0,0,1,0
  OK
  AT+CFUN=31
  OK
  AT#XGNSSDEL=1407
  OK
  AT+CFUN=0
  OK

Read command
------------

The read command is not supported.

Test command
------------

The test command tests the existence of the command and provides information about the type of its subparameters.

Syntax
~~~~~~

::

   AT#XGNSSDEL=?

Example
~~~~~~~

::

  AT#XGNSSDEL=?

  #XGNSSDEL: <mask>

  OK
