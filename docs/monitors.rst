Monitor Configuration
=====================

Monitors are defined in (by default) :file:`monitors.ini`. The monitor is named
by its ``[section]`` heading. If you create a ``[defaults]`` section, the
values are used as defaults for all the other monitors. Each monitor's
configuration will override the values from the default.

.. contents::

Common options
--------------

These options are common to all monitor types.

.. confval:: type

   :type: string
   :required: true

    the type of the monitor; one of those in the list below.

.. confval:: runon

   :type: string
   :required: false
   :default: none

    a hostname on which the monitor should run. If not set, always runs. You
    can use this to share one config file among many hosts. (The value which is
    compared to is that returned by Python's :code:`socket.gethostname()`.)

.. confval:: depend

   :type: comma-separated list of string
   :required: false
   :default: none

   the monitors on which this one depends. This monitor will run after those, unless one of them fails or is skipped, in which case this one will also skip. A skip does not trigger an alerter.

.. confval:: tolerance

    :type: integer
    :required: false
    :default: 1

    the number of times a monitor can fail before it enters the failed state. Handy for things which intermittently fail, such as unreliable links. See also the :ref:`limit-option` on Alerters.

.. confval:: urgent

    :type: boolean
    :required: false
    :default: true

    if this monitor is "urgent" or not. Non-urgent monitors do not trigger urgent alerters (e.g. BulkSMS)

.. _gap:

.. confval:: gap

    :type: integer
    :required: false
    :default: 0

    the number of seconds this monitor should allow to pass before polling. Use it to make a monitor poll only once an hour (``3600``), for example. Setting this value lower than the ``interval`` will have no effect, and the monitor will run every loop like normal.

    Some monitors default to a higher value when it doesn't make sense to run their check too frequently because the underlying data will not change that often or quickly, such as :ref:`pkgaudit<pkgaudit>`. You can override their default to a lower value as required.

    .. hint:: Monitors which are in the failed state will poll every loop, regardless of this setting, in order to detect recovery as quickly as possible

.. confval:: remote_alert

    :type: boolean
    :required: false
    :default: false

    set to true to have this monitor's alerting handled by a remote instance instead of the local one. If you're using the remote feature, this is a good candidate to put in the ``[defaults]``.

.. confval:: recover_command

    :type: string
    :required: false
    :default: none

    a command to execute once when this monitor enters the failed state. For example, it could attempt to restart a service.

.. confval:: recovered_command

    :type: string
    :required: false
    :default: none

    a command to execute once when this monitor returns to the OK state. For example, it could restart a service which was affected by the failure of what this monitor checks.

.. confval:: notify

    :type: boolean
    :required: false
    :default: true

    if this monitor should alert at all.

.. confval:: group

    :type: string
    :required: false
    :default: ``default``

    the group the monitor belongs to. Alerters and Loggers will only fire for monitors which appear in their groups.

.. confval:: failure_doc

    :type: string
    :required: false
    :default: none

    information to include in alerts on failure (e.g. a URL to a runbook)


Monitors
--------

.. note:: The ``type`` of the monitor is the first word in its heading.

apcupsd - APC UPS status
^^^^^^^^^^^^^^^^^^^^^^^^

Uses an existing and configured ``apcupsd`` installation to check the UPS status. Any status other than ``ONLINE`` is a failure.

.. confval:: path

    :type: string
    :required: false
    :default: none

    the path to the :file:`apcaccess` binary. On Windows, defaults to :file:`C:\\apcupsd\\bin`. On other platforms, looks in ``$PATH``.

arlo_camera - Arlo camera battery level
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Checks Arlo camera battery level is high enough.

.. confval:: username

    :type: string
    :required: true

    Arlo username

.. confval:: password

    :type: string
    :required: true

    Arlo password

.. confval:: device_name

    :type: string
    :required: true

    the device to check (e.g. ``Front Camera``)

.. confval:: base_station_id

    :type: integer
    :required: false
    :default: ``0``

    the number of your base station. Only required if you have more than one. It's an array index, but figuring out which is which is an exercise left to the reader.

command - run an external command
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Run a command, and optionally verify its output. If the command exits non-zero, this monitor fails.

.. confval:: command

    :type: string
    :required: true

    the command to run.

.. confval:: result_regexp

    :type: string (regular expression)
    :required: false
    :default: none

    if supplied, the output of the command must match else the monitor fails.

.. confval:: result_max

    :type: integer
    :required: false

    if supplied, the output of the command is evaluated as an integer and if greater than this, the monitor fails. If the output cannot be converted to an integer, the monitor fails.

compound - combine monitors
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Combine (logical-and) multiple monitors. By default, if any monitor in the list is OK, this monitor is OK. If they all fail, this monitor fails. To change this limit use the ``min_fail`` setting.

.. warning:: Do not specify the other monitors in this monitor's ``depends`` setting. The dependency handling for compound monitors is a special case and done for you.

.. confval:: monitors

    :type: comma-separated list of string
    :required: true

    the monitors to combine

.. confval:: min_fail

    :type: integer
    :required: false
    :default: the number of monitors in the list

    the number of monitors from the list which should be failed for this monitor to fail. The default is that all the monitors must fail.

diskspace - free disk space
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Checks the free space on the given partition/drive.

.. confval:: partition

    :type: string
    :required: true

    the partition/drive to check. On Windows, give the drive letter (e.g. :file:`C:`). Otherwise, give the mountpoint (e.g. :file:`/usr`).

.. confval:: limit

    :type: :ref:`bytes<config-bytes>`
    :required: true

    the minimum allowed amount of free space.

dns - resolve record
^^^^^^^^^^^^^^^^^^^^

Attempts to resolve the DNS record, and optionally checks the result. Requires ``dig`` to be installed and on the PATH.

.. confval:: record

    :type: string
    :required: true

    the DNS name to resolve

.. confval:: record_type

    :type: string
    :required: false
    :default: ``A``

    the type of record to request

.. confval:: desired_val

    :type: string
    :required: false

    if not given, this monitor simply checks the record resolves.

    Give the special value ``NXDOMAIN`` to check the record **does not** resolve.

    If you need to check a multivalue response (e.g. MX records), format them like this (note the leading spaces on the continuation lines):

    .. code-block:: ini

        desired_val=10 a.mx.domain.com
          20 b.mx.domain.com
          30 c.mx.domain.com

.. confval:: server

    :type: string
    :required: false

    the server to send the request to. If not given, uses the system default.

.. _fail:

fail - alawys fails
^^^^^^^^^^^^^^^^^^^

This monitor fails 5 times in a row, then succeeds once. Use for testing. See the :ref:`null<null>` monitor for the inverse.

filestat - file size and age
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Examines a file's size and age. If neither of the age/size values are given, simply checks the file exists.

.. confval:: filename

    :type: string
    :required: true

    the path of the file to monitor.

.. confval:: maxage

    :type: integer
    :required: false

    the maximum allowed age of the file in seconds. If not given, not checked.

.. confval:: minsize

    :type: :ref:`bytes<config-bytes>`
    :required: false

    the minimum allowed size of the file in bytes. If not given, not checked.

hass_sensor - Home Automation Sensors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This monitor checks for the existence of a home automation sensor.

.. confval:: url

    :type: string
    :required: true

    API URL for the monitor

.. confval:: sensor

    :type: string
    :required: true

    the name of the sensor

.. confval:: token

    :type: string
    :required: true

    API token for the sensor

host - ping a host
^^^^^^^^^^^^^^^^^^

Check a host is pingable.

.. tip:: This monitor relies on executing the ``ping`` command provided by your OS. It has known issues on non-English locales on Windows. You should use the :ref:`ping<ping>` monitor instead. The only reason to use this one is that it does not require SimpleMonitor to run as root.

.. confval:: host

    :type: string
    :required: true

    the hostname/IP to ping

.. confval:: ping_regexp

    :type: regexp
    :required: false
    :default: automatic

    the regexp which matches a successful ping. You may need to set this to use this monitor in a non-English locale.

.. confval:: time_regexp

    :type: regexp
    :required: false
    :default: automatic

    the regexp which matches the ping time in the output. Must set a match group named ``ms``. You may need to set this as above.

http - fetch and verify a URL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Attempts to fetch a URL and makes sure the HTTP return code is (by default) 200/OK. Can also match the content of the page to a regular expression.

.. confval:: url

    :type: string
    :required: true

    the URL to open

.. confval:: regexp

    :type: regexp
    :required: false
    :default: none

    the regexp to look for in the body of the response

.. confval:: allowed_codes

    :type: comma-separated list of integer
    :required: false
    :default: `200`

    a list of acceptable HTTP status codes

.. confval:: verify_hostname

    :type: boolean
    :required: false
    :default: true

    set to false to disable SSL hostname verification (e.g. with self-signed certificates)

.. confval:: timeout

    :type: integer
    :required: false
    :default: ``5``

    the timeout in seconds for the HTTP request to complete

.. confval:: headers

    :type: JSON map as string
    :required: false
    :default: ``{}``

    JSON map of HTTP header names and values to add to the request

loadavg - load average
^^^^^^^^^^^^^^^^^^^^^^

Check the load average on the host.

.. confval:: which

    :type: integer
    :required: false
    :default: ``1``

    the load average to monitor. ``0`` = 1min, ``1`` = 5min, ``2`` = 15min

.. confval:: max

    :type: float
    :required: false
    :default: ``1.00``

    the maximum acceptable load average

memory - free memory percent
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Check free memory percentage.

.. confval:: percent_free

    :type: int
    :required: true

    the minimum percent of available (as per psutils’ definition) memory

.. _null:

null - always passes
^^^^^^^^^^^^^^^^^^^^

Monitor which always passes. Use for testing. See the :ref:`fail<fail>` monitor for the inverse.

This monitor has no additional parameters.

.. _ping:

ping - ping a host
^^^^^^^^^^^^^^^^^^

Pings a host to make sure it’s up. Uses a Python ping module instead of calling out to an external app, but needs to be run as root.

.. confval:: host

   :type: string
   :required: true

   the hostname or IP to ping

.. confval:: timeout

    :type: int
    :required: false
    :default: ``5``

    the timeout for the ping in seconds

.. _pkgaudit:

pkgaudit - FreeBSD pkg audit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Fails if ``pkg audit`` reports any vulnerable packages installed.

.. confval:: path

    :type: string
    :required: false
    :default: :file:`/usr/local/sbin/pkg`

    the path to the ``pkg`` binary

portaudit - FreeBSD port audit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Fails if ``portaudit`` reports any vulnerable ports installed.

.. confval:: path

    :type: string
    :required: false
    :default: :file:`/usr/local/sbin/portaudit`

    the path to the ``portaudit`` binary

process - running process
^^^^^^^^^^^^^^^^^^^^^^^^^

Check for a running process.

.. confval:: process_name

    :type: string
    :required: true

    the process name to check for

.. confval:: min_count

    :type: integer
    :required: false
    :default: ``1``

    the minimum number of matching processes

.. confval:: max_count

    :type: integer
    :required: false
    :default: infinity

    the maximum number of matching processes

.. confval:: username

    :type: string
    :required: false
    :default: any user

    limit matches to processes owned by this user.

rc - FreeBSD rc service
^^^^^^^^^^^^^^^^^^^^^^^

Checks a FreeBSD-style service is running, by running its rc script (in /usr/local/etc/rc.d) with the status command.

.. tip:: You may want the :ref:`unix_service<unix_service>` monitor for a more generic check.

.. confval:: service

    :type: string
    :required: true

    the name of the service to check. Should be the name of the rc.d script in :file:`/usr/local/etc/rc.d`. Any trailing ``.sh`` is optional and added if needed.

.. confval:: path

    :type: string
    :required: false
    :default: :file:`/usr/local/etc/rc.d`

    the path of the folder containing the rc script.

.. confval:: return_code

    :type: integer
    :required: false
    :default: ``0``

    the required return code from the script

ring - Ring doorbell battery
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Check the battery level of a Ring doorbell.

.. confval:: device_name

    :type: string
    :required: true

    the name of the Ring Doorbell to monitor.

.. confval:: minimum_battery

    :type: integer
    :required: false
    :default: ``25``

    the minimum battery percent allowed.

.. confval:: username

    :type: string
    :required: true

    your Ring username (e.g. email address). Accounts using MFA are not supported. You can create a separate user for API access.

.. confval:: password

    :type: string
    :required: true

    your Ring password.

.. warning:: Do not commit credentials to source control!

service - Windows Service
^^^^^^^^^^^^^^^^^^^^^^^^^

Checks a Windows service to make sure it's in the correct state.

.. confval:: service

    :type: string
    :required: true

    the short name of the service to monitor (this is the "Service Name" on the General tab of the service Properties in the Services MMC snap-in).

.. confval:: want_state

    :type: string
    :required: false
    :default: ``RUNNING``

    the required status for the service. One of:

    * ``RUNNING``
    * ``STOPPED``
    * ``PAUSED``
    * ``START_PENDING``
    * ``PAUSE_PENDING``
    * ``CONTINUE_PENDING``
    * ``STOP_PENDING``

.. tip:: version 1.9 and earlier had a **host** parameter, which is no longer used.

svc - daemontools service
^^^^^^^^^^^^^^^^^^^^^^^^^

Checks a daemontools ``supervise``-managed service is running.

.. confval:: path

    :type: string
    :required: true

    the path to the service's directory (e.g. :file:`/var/service/something`)

systemd-unit - systemd unit check
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Monitors a systemd unit status, via dbus. You may want the :ref:`unix_service<unix_service>` monitor instead if you just want to ensure a service is running.

.. confval:: name

    :type: string
    :required: true

    the name of the unit to monitor

.. confval:: load_states

    :type: comma-separated list of string
    :required: false
    :default: ``loaded``

    desired load states for the unit

.. confval:: active_states

    :type: comma-separated list of string
    :required: false
    :default: ``active,reloading``

    desired active states for the unit

.. confval:: sub_states

    :type: comma-separated list of string
    :required: false
    :default: none

    desired sub states for the service

tcp - open TCP port
^^^^^^^^^^^^^^^^^^^

Checks a TCP port is connectible. Doesn't care what happens after the connection is opened.

.. confval:: host

    :type: string
    :required: true

    the name/IP of the host to connect to

.. confval:: port

    :type: integer
    :required: true

    the port number to connect to.

tls_expiry - TLS cert expiration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Checks an SSL/TLS certificate is not due to expire/has expired. No support for SNI, and does not verify the certificate has the right hostname, chain, etc.

.. note:: This monitor's :ref:`gap<gap>` defaults to 12 hours.

.. warning:: Due to a limitation of the underlying Python modules in use, this does not currently support TLS 1.3.

.. confval:: host

    :type: string
    :required: true

    the hostname to connect to

.. confval:: port

    :type: integer
    :required: false
    :default: ``443``

    the port number to connect on

.. confval:: min_days

    :type: integer
    :required: false
    :default: ``7``

    the minimum allowable number of days until expiry

unifi_failover - USG failover WAN status
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Checks a Unifi Security Gateway for failover WAN status. Connects via SSH; the USG must be in your :file:`known_hosts` file. Requires the specified interface to have the carrier up, a gateway, and not be in the ``failover`` state.

.. confval:: router_address

    :type: string
    :required: true

    the address of the USG

.. confval:: router_username

    :type: string
    :required: true

    the username to log in as

.. confval:: router_password

    :type: string
    :required: conditional

    the password to log in with. Required if not using ``ssh_key``.

.. confval:: ssh_key

    :type: string
    :required: conditional

    path to the SSH private key to log in with. Required if not using ``router_password``.

.. confval:: check_interface

    :type: string
    :required: false
    :default: ``eth2``

    the interface which should be ready for failover.

unifi_watchdog - USG failover watchdog
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Checks a Unifi Security Gateway to make sure the failover WAN is healthy. Connects via SSH; the USG must be in your :file:`known_hosts` file. Requires the specified interface to have status ``Running`` and the ping target to be ``REACHABLE``.


.. confval:: router_address

    :type: string
    :required: true

    the address of the USG

.. confval:: router_username

    :type: string
    :required: true

    the username to log in as

.. confval:: router_password

    :type: string
    :required: conditional

    the password to log in with. Required if not using ``ssh_key``.

.. confval:: ssh_key

    :type: string
    :required: conditional

    path to the SSH private key to log in with. Required if not using ``router_password``.

.. confval:: primary_interface

    :type: string
    :required: false
    :default: ``pppoe0``

    the primary WAN interface

.. confval:: secondary_interface

    :type: string
    :required: false
    :default: ``eth2``

    the secondary (failover) WAN interface

.. _unix_service:

unix_service - generic UNIX service
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Generic UNIX service check, by running ``service ... status``.

.. confval:: service

    :type: string
    :required: true

    the name of the service to check

.. confval:: state

    :type: string
    :required: false
    :default: ``running``

    the state of the service; either ``running`` (status command exits 0) or ``stopped`` (status command exits 1).