---
id: tr064
label: TR-064
title: TR-064 - Bindings
type: binding
description: "This binding brings support for internet gateway devices that support the TR-064 protocol (e.g. the AVM FritzBox family of routers)."
since: 3x
install: auto
---

<!-- Attention authors: Do not edit directly. Please add your changes to the appropriate source repository -->

{% include base.html %}

# TR-064 Binding

This binding brings support for internet gateway devices that support the TR-064 protocol (e.g. the AVM FritzBox family of routers). 
It can be used to gather information from the device and/or re-configure it.
Even though textual configuration is possible, it is strongly recommended to use the Main User Interface for configuration.

## Supported Things

Two Bridge things are supported:
- `generic`: the internet gateway device itself (generic device)
- `fritzbox`: similar to `generic` with extensions for AVM FritzBox devices.

Two kind of Things are supported:
- `subDevice`: a sub-device of the Bridge thing (e.g. a WAN interface) 
- `subDeviceLan`: a special type of sub-device that supports MAC-detection

## Discovery

The gateway device needs to be added manually.
After that, sub-devices are detected automatically.

## Thing Configuration

All thing types have a `refresh` parameter.
It sets the refresh-interval in seconds for each device channel.
The default value is 60.

### `generic`, `fritzbox`

The `host` parameter is required to communicate with the device.
It can be a hostname or an IP address.

For accessing the device you need to supply credentials.
If you only configured password authentication for your device, the `user` parameter must be skipped and it will default to `dslf-config`.
The second credential parameter is `password`, which is mandatory.
For security reasons it is highly recommended to set both, username and password.

### `fritzbox`

The `fritzbox` devices can give additional informations in dedicated channels, controlled 
 by additional parameters (visible if Show Advanced is selected), w.r.t. to `generic` devices.
If the parameters are specified, the corresponding channels will be added to the device. 

One or more TAM (telephone answering machines) are supported by most fritzbox devices.
By setting the `tamIndices` parameter you can instruct the binding to add channels for these
 devices to the thing.
Values start with `0`.
This is an optional parameter and multiple values are allowed: add one value per line in the Main User Interface.

Most devices allow to configure call deflections.
If the `callDeflectionIndices` parameter is set, channels for the status of the pre-configured call deflections are added.
Values start with `0`, including the number of "Call Blocks" (two configured call-blocks -> first deflection is `2`).
This is an optional parameter and multiple values are allowed:  add one value per line in the Main User Interface.

Most devices support call lists.
The binding can retrieve these call lists and provide channels for the number of missed calls, inbound calls, outbound calls and rejected (blocked) calls,
for a given number of days. A channel is added to the Thing if such a number is set through the corresponding parameter
in the Main User Interface. 
The parameters are: `missedCallDays`, `rejectedCallDays`, `inboundCallDays`, `outboundCallDays` and `callListDays`.

Since FritzOS! 7.20 WAN access of local devices can be controlled by their IPs.
If the `wanBlockIPs` parameter is set, a channel for each IP is created to block/unblock WAN access for this IP.
Values need to be IPv4 addresses in the format `a.b.c.d`.
This is an optional parameter and multiple values are allowed:  add one value per line in the Main User Interface.

If the `PHONEBOOK` profile shall be used, it is necessary to retrieve the phonebooks from the FritzBox.
The `phonebookInterval` is uses to set the refresh cycle for phonebooks. It defaults to 600 seconds,
and it can be set to 0 if phoneooks are not to be used.


### `subdevice`, `subdeviceLan`

Additional informations (i.e. channels) are available in subdevices of the bridge.
Each subdevice is characterized by a unique `uuid` parameter: this is the UUID/UDN of the device. 
This is a mandatory parameter to be set in order to add the subdevice. Since the parameter value can only be determined
by examining the SCPD of the root device, the simplest way to obtain it is through auto-discovery.

Auto discovery may find several sub-devices, each one holding channels as described in the following.

The LAN sub-device, in particular, is also used for presence detection.
It therefore optionally contains
a channel for each MAC address (in a format 11:11:11:11:11:11, different than the old v1 version of this binding),
defined by the parameter `macOnline`.
This is an optional parameter and multiple values are allowed:  add one value per line in the Main User Interface.

## Channels

Channels are grouped according to the subdevice they belong to. 

### fritzbox Bridge channels

Advanced channels appear only if the corresponding parameters are set in the Thing definition.

| channel                    | item-type                 | advanced | description                                                    |
|----------------------------|---------------------------|:--------:|----------------------------------------------------------------|
| `callDeflectionEnable`     | `Switch`                  |     x    | Enable/Disable the call deflection setup with the given index. |
| `callList`                 | `String`                  |     x    | A string containing the call list as JSON (see below)          |    
| `deviceLog`                | `String`                  |     x    | A string containing the last log messages                      |
| `missedCalls`              | `Number`                  |          | Number of missed calls within the given number of days.        |
| `outboundCalls`            | `Number`                  |     x    | Number of outbound calls within the given number of days.      |
| `inboundCalls`             | `Number`                  |     x    | Number of inbound calls within the given number of days.       |
| `reboot`                   | `Switch`                  |          | Reboot                                                         |
| `rejectedCalls`            | `Number`                  |     x    | Number of rejected calls within the given number of days.      |
| `securityPort`             | `Number`                  |     x    | The port for connecting via HTTPS to the TR-064 service.       |
| `tamEnable`                | `Switch`                  |     x    | Enable/Disable the answering machine with the given index.     |
| `tamNewMessages`           | `Number`                  |     x    | The number of new messages of the given answering machine.     |
| `uptime`                   | `Number:Time`             |          | Uptime of the device                                           |

### LAN subdeviceLan channels

| channel                    | item-type                 | advanced | description                                                    |
|----------------------------|---------------------------|:--------:|----------------------------------------------------------------|
| `wifi24GHzEnable`          | `Switch`                  |          | Enable/Disable the 2.4 GHz WiFi device.                        |
| `wifi5GHzEnable`           | `Switch`                  |          | Enable/Disable the 5.0 GHz WiFi device.                        |
| `wifiGuestEnable`          | `Switch`                  |          | Enable/Disable the guest WiFi.                                 |
| `macOnline`                | `Switch`                  |     x    | Online status of the device with the given MAC                 |

### WANConnection subdevice channels

| channel                    | item-type                 | advanced | description                                                    |
|----------------------------|---------------------------|:--------:|----------------------------------------------------------------|
| `Uptime`                   | `Number:Time`             |          | Uptime                                                         |
| `pppUptime`                | `Number:Time`             |          | Uptime (if using PPP)                                          |
| `wanConnectionStatus`      | `String`                  |          | Connection Status                                              |
| `wanPppConnectionStatus`   | `String`                  |          | Connection Status (if using PPP)                               |
| `wanIpAddress`             | `String`                  |     x    | WAN IP Address                                                 |
| `wanPppIpAddress`          | `String`                  |     x    | WAN IP Address (if using PPP)                                  |

### WAN subdevice channels

| channel                    | item-type                 | advanced | description                                                    |
|----------------------------|---------------------------|:--------:|----------------------------------------------------------------|
| `dslCRCErrors`             | `Number:Dimensionless`    |     x    | DSL CRC Errors                                                 |
| `dslDownstreamMaxRate`     | `Number:DataTransferRate` |     x    | DSL Max Downstream Rate                                        |
| `dslDownstreamCurrRate`    | `Number:DataTransferRate` |     x    | DSL Curr. Downstream Rate                                      |
| `dslDownstreamNoiseMargin` | `Number:Dimensionless`    |     x    | DSL Downstream Noise Margin                                    |
| `dslDownstreamAttenuation` | `Number:Dimensionless`    |     x    | DSL Downstream Attenuation                                     |
| `dslEnable`                | `Switch`                  |          | DSL Enable                                                     |
| `dslFECErrors`             | `Number:Dimensionless`    |     x    | DSL FEC Errors                                                 |
| `dslHECErrors`             | `Number:Dimensionless`    |     x    | DSL HEC Errors                                                 |
| `dslStatus`                | `Switch`                  |          | DSL Status                                                     |
| `dslUpstreamMaxRate`       | `Number:DataTransferRate` |     x    | DSL Max Upstream Rate                                          |
| `dslUpstreamCurrRate`      | `Number:DataTransferRate` |     x    | DSL Curr. Upstream Rate                                        |
| `dslUpstreamNoiseMargin`   | `Number:Dimensionless`    |     x    | DSL Upstream Noise Margin                                      |
| `dslUpstreamAttenuation`   | `Number:Dimensionless`    |     x    | DSL Upstream Attenuation                                       |
| `wanAccessType`            | `String`                  |     x    | Access Type                                                    |
| `wanMaxDownstreamRate`     | `Number:DataTransferRate` |     x    | Max. Downstream Rate                                           |
| `wanMaxUpstreamRate`       | `Number:DataTransferRate` |     x    | Max. Upstream Rate                                             |
| `wanPhysicalLinkStatus`    | `String`                  |     x    | Link Status                                                    |
| `wanTotalBytesReceived`    | `Number:DataAmount`       |     x    | Total Bytes Received                                           |
| `wanTotalBytesSent`        | `Number:DataAmount`       |     x    | Total Bytes Sent                                               |


Parameters that accept lists (e.g. `macOnline`, `wanBlockIPs`) can contain comments.
Comments are separated from the value with a '#' (e.g. `192.168.0.77 # Daughter's iPhone`).
The full string is used for the channel label.

### Channel `callList`

Call lists are provided for one or more days (as configured) as JSON.
The JSON consists of an array of individual calls with the fields `date`, `type`, `localNumber`, `remoteNumber`, `duration`.
The call-types are the same as provided by the FritzBox, i.e. `1` (inbound), `2` (missed), `3` (outbound), `10` (rejected).
 
## `PHONEBOOK` Profile

The binding provides a profile for using the FritzBox phonebooks for resolving numbers to names.
The `PHONEBOOK` profile takes strings containing the number as input and provides strings with the caller's name, if found.

The parameter `thingUid` with the UID of the phonebook providing thing is a mandatory parameter.
If only a specific phonebook from the device should be used, this can be specified with the `phonebookName` parameter.
The default is to use all available phonebooks from the specified thing.
In case the format of the number in the phonebook and the format of the number from the channel are different (e.g. regarding country prefixes), the `matchCount` parameter can be used.
The configured `matchCount` is counted from the right end and denotes the number of matching characters needed to consider this number as matching.
A `matchCount` of `0` is considered as "match everything".

## Rule Action

The phonebooks of a `fritzbox` thing can be used to lookup a number from rules via a thing action:

`String name = phonebookLookup(String number, String phonebook, int matchCount)`

`phonebook` and `matchCount` are optional parameters.
You can omit one or both of these parameters.
The configured `matchCount` is counted from the right end and denotes the number of matching characters needed to consider this number as matching.
A `matchCount` of `0` is considered as "match everything" and is used as default if no other value is given.
The return value is either the phonebook entry (if found) or the input number.

Example (use all phonebooks, match 5 digits from right):

```
val tr064Actions = getActions("tr064","tr064:fritzbox:2a28aee1ee")
val result = tr064Actions.phonebookLookup("49157712341234", 5)
```
## A note on textual configuration

Textual configuration through a `.things` file is possible but, at present, strongly discouraged because it is significantly more error-prone
than the configuration through Main User Interface.

If an advanced user is really motivated to define a textual configuration, it is suggested to perform
an automatic scan through the user interface first in order to extract the required parameters (namely the different `uuid` of the
needed subdevices).

The definition of the bridge and of the subdevices things is the following
```
Bridge tr064:fritzbox:rootuid "Root label" @ "location" [ host="192.168.1.1", user="user", password="passwd",
                                                         phonebookInterval="0"]{
    Thing subdeviceLan LAN "label LAN"   [ uuid="uuid:xxxxxxxx-xxxx-xxxx-yyyy-xxxxxxxxxxxx",
                                                macOnline="XX:XX:XX:XX:XX:XX",  
                                                          "YY:YY:YY:YY:YY:YY"]  
    Thing subdeviceLan WAN "label WAN"               [ uuid="uuid:xxxxxxxx-xxxx-xxxx-zzzz-xxxxxxxxxxxx"]
    Thing subdeviceLan WANCon "label WANConnection"  [ uuid="uuid:xxxxxxxx-xxxx-xxxx-wwww-xxxxxxxxxxxx"]
    }
```

The channel are automatically generated and it is simpler to use the Main User Interface to copy the textual definition of the channel

```
Switch PresXX "[%s]" {channel="tr064:subdeviceLan:rootuid:LAN:macOnline_XX_3AXX_3AXX_3AXX_3AXX_3AXX"}
Switch PresYY "[%s]" {channel="tr064:subdeviceLan:rootuid:LAN:macOnline_YY_3AYY_3AYY_3AYY_3AYY_3AYY"}

```