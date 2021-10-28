# Time Server

Time Server, how it works under the ZiGate environment

## Overview

By default, ZiGate implement a Time Server. In order to get the Time server correctly set two APIs are available.
Those are documented in the [ZiGate Commands](https://zigate.fr/documentation/commandes-zigate/).

| Message Type | Parameters | Description |
| ------------ | ---------- | ----------- |
|  0x0016      | <timestamp UTC: uint32_t> from EPOCH: 2000-01-01 00:00:00 | will set the time server is zigate to the date specified by the number of second since EPOCH ( 1st January 2000) |
|  0x0017      |  - | Get the time of now |

## Cluster 0x000a

By default when a Node is requesting the Controller for time , the node is using a Read Attribute Request on cluster 0x000a.

| Attribute | Description | Data Type | Note |*
| --------- | ----------- | --------- | ---- |
| 0x0000    | secondes since EPOCH ( 01/01/2000 )             | UTC ( 0xe2)    | provided by Zigate |
| 0x0001    | Time status                                     | bitmap8 (0x18) | not provided by ZiGate, plugin needs to respond |
| 0x0002    | Timezone                                        | uint32 (0x23)  | not provided by ZiGate, plugin needs to respond |
| 0x0007    | Local time  secondes since EPOCH ( 01/01/2000 ) | uint32 (0x23)  | not provided by ZiGate, plugin needs to respond |

The plugin will get the request via the Data Indication message 0x8002

Please do not that some devices might required the Local time related to EPOCH (01/01/1970)

## Specific to Legrand-Netatmo devices

During the pairing process, Legrand-Netatmo devices are doing a Read Attribute Request in broadcast on Cluster 0x0000 and Atttribute 0xf000

This attribut represent the number of working hours of the device.

As this is a broadcast from the device annoucing itself to the network , the controller has also to respond. The Zigate is responding, and so is measuring the number of working hours (in seconds) since the last start ( Power Off/On or Soft Reset).
