# swaystatus

A lightweight yet feature-rich status bar for i3bar or swaybar.

![screenshot]

It is written completely in C/C++ to make it as lightweight as possible and specifically, to avoid creating new processes every second as in bash script.

It uses libraries like `libasound` and glibc function `getifaddrs` to retrieve volume information as opposed to using `amixer` and `ip addr`.

For battery, backlight, load, and meminfo, it reads directly from `/sys/class/power_supply`,
`/sys/class/backlight`, `/proc/loadavg` and `/proc/meminfo`.

On my x86-64 computer, it compiles to a single binary that less than `60K` using `clang-11`.

## Runtime Dependency
 - libasound.so.2
 - libnm.so.0
 - libjson-c.so.5 (the same json library as sway and swaybar)

The only dependency that are not portable among system is libnm, which requires NetworkManager
to be installed, thus it might be removed in the future.

## Build

Install `clang` (must have lto support), `lld`, `make` and `pkg-config`, then run

```
git clone --recurse-submodules https://github.com/NobodyXu/swaystatus
cd swaystatus/src
make -j $(nproc)
```

To install, run `sudo make install`, which by default will install a single binary `swaystatus` to `/usr/local/bin`.

## Usage

### swaybar usage:

```
swaystatus: Usage: swaystatus [options] configuration_filename

  --help                    Show help message and exit
  --interval=unsigned_msec  Specify update interval in milliseconds, must be an unsignerinteger.
```

### Config file format

    {
        "name": {
            "format": "Hello, {variable_name}",
            "color": "##RRGGBBA",
            "background: "##RRGGBBA",
            "border": "##RRGGBBA",
            "border_top": 1,
            "border_bottom": 1,
            "border_left": 1,
            "border_right": 1,
            "min_width": 1,
            "align": "center",
            "separator": true,
            "separator_block_width": 9,
            "markup": "none"
        },
    }

All property present for "name" above are optional.
<br>For volume, you can also set property "mix_name" and "card".

The following values are valid name:

 - brightness
 - volume
 - battery
 - network_interface
 - load
 - memory_usage
 - time

If you want to disable a certain feature, say brightness,
then add the following to your configuration:

    {
        "brightness": false,
    }

#### Battery format variables:

 - `name`
 - `present`
 - `technology`
 - `model_name`
 - `manufacturer`
 - `serial_number`
 - `status`
 - `cycle_count`
 - `voltage_min_design`
 - `voltage_now`
 - `charge_full_design`
 - `charge_full`
 - `charge_now`
 - `capacity`
 - `capacity_level`
 - `is_charging`        (Check section "Conditional Variable" for usage)
 - `is_discharging`
 - `is_not_charging`
 - `is_full`

#### Memory Usage variables:

 - `MemTotal`
 - `MemFree`
 - `MemAvailable`
 - `Buffers`
 - `Cached`
 - `SwapCached`
 - `Active`
 - `Inactive`
 - `Mlocked`
 - `SwapTotal`
 - `SwapFree`
 - `Dirty`
 - `Writeback`
 - `AnonPages`
 - `Mapped`
 - `Shmem`

The unit (supports 'BKMGTPEZY') of the variables printed can be specified.
<br>For example, '{MemTotal:K}' will print MemTotal in KiloBytes.

#### Volume variables:

 - `volume`

#### Load variables:

 - `loadavg_1m`
 - `loadavg_5m`
 - `loadavg_15m`
 - `running_kthreads_cnt`
 - `total_kthreads_cnt`
 - `last_created_process_pid`

#### Brightness variables:

NOTE that these variables are evaluated per backlight_device.

 - `backlight_device`
 - `brightness`
 - `has_multiple_backlight_devices` (this is a Conditional Variable)

#### Network Interface variables;

 - `is_connected`
 - `is_not_connected`
 - `per_interface_fmt_str`:
    The specification for this variable will be formatted once for each interface.

    It contains subvariables that can be used inside:
     - `HAS_UAPI_DEF_IF_NET_DEVICE_FLAGS_LOWER_UP_DORMANT_ECHO`
     - `has_broadcast_support`
     - `is_pointopoint`
     - `has_no_arp_support`
     - `is_in_promisc_mode`
     - `is_in_notrailers_mode`
     - `is_master`
     - `is_slave`
     - `has_multicast_support`
     - `has_portsel_support`
     - `is_automedia_active`
     - `is_dhcp`
	 - `rx_packets`
	 - `tx_packets`
	 - `rx_bytes`
	 - `tx_bytes`
	 - `rx_errors`
	 - `tx_errors`
	 - `rx_dropped`
	 - `tx_dropped`
	 - `multicast`
	 - `collisions`
	 - `rx_length_errors`
	 - `rx_over_errors`
	 - `rx_crc_errors`
	 - `rx_frame_errors`
	 - `rx_fifo_errors`
	 - `rx_missed_errors`
	 - `tx_aborted_errors`
	 - `tx_carrier_errors`
	 - `tx_fifo_errors`
	 - `tx_heartbeat_errors`
	 - `tx_window_errors`
	 - `rx_compressed`
	 - `tx_compressed`
     - `ipv4_addrs`
     - `ipv6_addrs`
    Limit of number of ip address can be done via `{ipv4_addrs:1}` and `{ipv6_addrs:1}`

    Optionally if `HAS_UAPI_DEF_IF_NET_DEVICE_FLAGS_LOWER_UP_DORMANT_ECHO`, 
    the following variables are also defined:
     - `is_lower_up`
     - `is_dormant`
     - `is_echo_device`

To limit number of ip addresses in output, please use `{ipv4_config:1}`.

#### Format string for time:

Format string for time is parsed by strftime instead of fmtlib, so the format is
specified in [`strftime`] instead.

For format string other than time, check [fmt - Format String Syntax] for more
information on format specification.Conditional Variable:

#### Conditional Variables

Conditional variables are used to selectively print strings.

For example, setting "format" in "battery" to "{is_charging:Charging}" will print "Charging" only
when the battery is charging.

All variables start with "is" and "has" are conditional variables.

#### Recursive Conditional Variable:

In additional to printing strings conditionally, conditional variables can also be used to
print other variables conditionally.

For example, "{is_charging:{level}%}" will print "98%" when charging, where
"98" is the actual level of battery.

Check [`example-config.json`] for the example configuration.

### Use `swaybar` in `sway`

```
bar {
    status_command swaystatus
}
```

[screenshot]: https://raw.githubusercontent.com/NobodyXu/swaystatus/main/screenshot.png
[`strftime`]: https://man7.org/linux/man-pages/man3/strftime.3.html
[fmt - Format String Syntax]: https://fmt.dev/latest/syntax.html
[`example-config.json`]: https://github.com/NobodyXu/swaystatus/blob/main/example-config.json
