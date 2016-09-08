collectd-vmstat
===============

`collectd-vmstat` is a `vmstat` plugin for Collectd that allows you to
collect `vmstat` metrics.

This plugin is based on the SignalFx maintained fork of the
[signalfx/collectd-iostat-python](https://github.com/signalfx/collectd-iostat-python)
which was forked from the original project by [Denis Zhdanov](mailto:denis.zhdanov@gmail.com)
[deniszh/collectd-iostat-python](https://github.com/deniszh/collectd-iostat-python).
That plugin (and parts of this `README`) are a rewrite of the
[collectd-iostat](https://github.com/keirans/collectd-iostat) from Ruby to Python
using the
[collectd-python](http://collectd.org/documentation/manpages/collectd-python.5.shtml)
plugin.


Why?
----

Collectd standard [vmem plugin](https://collectd.org/wiki/index.php/Plugin:vmem)
looks at `/proc/vmstat` but does not report some of the information found in
the `vmstat` tool.  For example `vmstat` reports `r` (the number of processes waiting for run time)
and `b` (the number of processes in uninterruptible sleep).


How?
----

`collectd-vmstat` functions by calling `vmstat` with some predefined
intervals and push that data to Collectd using Collectd `python` plugin.

Collectd can be then configured to write the collected data into many output
formats that are supported by it's write plugins.


Setup
-----

Deploy the Collectd Python plugin into a suitable plugin directory for your
Collectd instance.

Configure Collectd's `python` plugin to execute the `iostat` plugin using a
stanza similar to the following:


```
<LoadPlugin python>
    Globals true
</LoadPlugin>

<Plugin python>
    ModulePath "/usr/share/collectd/collectd-vmstat"
    Import "collectd_vmstat"

    <Module collectd_vmstat>
        Path "/usr/bin/vmstat"
        Interval 2
        Verbose false
        NiceNames true
        # Include Nice Names
        Include ["process.waiting", "process.uninterruptible_sleep", "memory.swap", "memory.free", "memory.buffer", "memory.cache", "memory.inactive", "memory.active", "swap.in_per_second", "swap.out_per_second", "blocks.received_per_second", "blocks.sent_per_second", "system.interrupts_per_second", "system.context_switches_per_second", "cpu.user_time", "cpu.system_time", "cpu.idle", "cpu.wait", "cpu.stolen"]
        # Include Not So Nice Names
        Include ["r", "b", "swpd", "free", "buff", "cache", "inact", "active", "si", "so", "bi", "bo", "in", "cs", "us", "sy", "id", "wa", "st"]
    </Module>
</Plugin>
```

If you would like to use more legible metric names (e.g.
`requests_merged_per_second-read` instead of `rrqm_s`), you have to set
`NiceNames` to `true` and add load the custom types database (see the
`iostat_types.db` file) by adding the following into the Collectd config file:

```
# The default Collectd types database
TypesDB "/usr/share/collectd/types.db"
# The custom types database
TypesDB "/usr/share/collectd/collectd-vmstat/vmstat_types.db"
```

Once functioning, the `vmstat` data should then be visible via your various
output plugins. Please note, that you need to restart collectd service after
plugin installation, as usual.

In the case of Graphite, Collectd should be writing data in the
`hostname_domain_tld.collectd_iostat_python.DEVICE.column-name` style namespaces.
Symbols like `/`, `-` and `%` in metric names (but not in device names) are
automatically replaced by underscores (i.e. `_`).

Please note that this plugin will take only last line of `vmstat` output, so big
`Count` numbers also have no sense, but `Count` needs to be more than `1` to get
actual and not historical data. And please make `Interval * Count <<
Collectd.INTERVAL` (4 seconds by default). The deafult value of `Count=2` and
`Interval=2` works quite well.


Technical notes
---------------

For parsing `vmstat` output, I'm using [deniszh/collectd-iostat-python]
[jakamkon's](https://bitbucket.org/jakamkon)
[python-iostat](https://bitbucket.org/jakamkon/python-iostat) Python module, but
as an internal part of the script instead of a separate module because of couple
of fixes I have made (using Kbytes instead of blocks, adding -N to `iostat` for
LVM endpoint resolving, migration to `subprocess` module as replacement of
deprecated `popen3`, `objectification` etc).


Compatibility
-------------

Plugin was tested on Ubuntu 12.04/14.04 (Collectd 5.2/5.3/5.4, Python 2.7) and
CentOS (Collectd 5.4 / Python 2.6). Please note that if running Python 2.6 or
older (i.e. on CentOS and its derivatives) we trying to restore `SIGCHLD` signal
handler to mitigate a known [bug](http://bugs.python.org/issue1731717) which
according to the Collectd's
[documentation](https://collectd.org/documentation/manpages/collectd-python.5.shtml#configuration)
breaks the `exec` plugin, unfortunately.


TODO
----


Additional reading
------------------

* [signalfx/collectd-iostat-python](https://github.com/signalfx/collectd-iostat-python)
* [deniszh/collectd-iostat-python](https://github.com/deniszh/collectd-iostat-python)
* [man iostat(1)](http://linux.die.net/man/1/iostat)
* [Custom Collectd Plug-ins for Linux](http://support.rightscale.com/12-Guides/RightScale_101/08-Management_Tools/Monitoring_System/Writing_custom_collectd_plugins/Custom_Collectd_Plug-ins_for_Linux)
* [python-iostat](https://bitbucket.org/jakamkon/python-iostat)
* [collectd-iostat](https://github.com/keirans/collectd-iostat)
* [Graphite @ The Architecture of Open Source Applications](http://www.aosabook.org/en/graphite.html)

License
-------

[MIT](http://mit-license.org/)


Support
-------

Maintained by SignalFx
[support@signalfx.com](mailto:support@signalfx.com)
