# Command processing in ZiGate

## Introduction

The purpose of this document is to describe how the command flow with Zigate works (from the Serial link, but also from the application level).
It is not intended to describe the API and so for that please refer to [ZiGate Commands](https://zigate.fr/documentation/commandes-zigate/).

This document has been developped on our understanding of the NXP IoT ControlBridge by using NXP Forums, some NXP documentation but also by experimenting, enhancing the Control Bridge to reach to the current ZiGate firmware.

## Notions

* Host : This is the command initiator ( most likely a plugin communicating with the coordinator via USB, IP)
* Coordinator: ZiGate
* Node: any object ( router or end device)

## Serial line control flow

| Connection   | Zigate Model | Control flow |
| ------------ | ------------ | ------------ |
| USB          | ZiGate V1    | No flow control (hardware or software) is used |
| DIN          | ZiGate V1    | RTS/CTS HW flow control |
| PI           | ZiGate V1    | No flow control (hardware or software) is used |
| USB          | ZiGate +/V2  | RTS/CTS HW flow control |
| DIN          | ZiGate +/V2  | RTS/CTS HW flow control |
| PI           | ZiGate +/V2  | No flow control (hardware or software) is used |

RECOMMANDATION: Activate HW flow control when supported by Zigate.

## Sequence

![comman flow diagram](command-flow.png)

All commands generate a synchronous response code followed by any asynchronous responses as they become available. There is no sequence number associated with each command/response – the user must ensure that commands are issued sequentially.

| Step | Direction | Message |
| ---- | --------- | ------- |
| 1    | Host -> controller | the host is sending a command for a particular node. Unicast mode |
| 2    | controller -> Host | the controller check the command allocate the needed resources ( nPDU, aPDU ...) and delegate the send, and send a confirmation to the host that the command has been processed to be sent. |
| 3    | controller -> Node | the controller is sending the command to the node. It has up to 7s (and can do retry) to send.
| 4    | controller -> Host | Optional 0x8012, the command has left the controller and reach (at least the first hop, or its final destination) |
| 5    | Node -> Controller | the Node send an APS Ack confirming the command has been received. |
| 6    | controller -> Host | optional, the controller is sending the confirmation of the command received by the node |
|      | controller -> Host | optional, The message has not been successfully send (or the coordinator didn't received the Ack on time) and then a NACK message provided.0x8702  |
| 7    | Node -> Controller | if applicable the Node is sending its response |
| 8    | controller -> Host | Optional data messages as requested |

During the all sequence, the SQN provided by the controller with the 0x8000 status message will be used to reconciliate the up coming messages like ( 0x8012, 0x8011, 0x8702 and the response data).

ATTENTION:
The NXP stack might generate 0x8011 messages in relation with direct exchanges between node and controller (not visible to the hosts)

## Sending unicast command with Ack or not

When sending an unicat command ( which is dedicated to one Node ), you can send this command and expect in return some ack on the transit of the command from the controller to the Node.
For instance you can be informed that the command has reached the first hope (after the controller), and you can also be notified when the command has been fully received by the Node.

Sending an unicast command with ACK mode, allow the NXP stack to do enough retry to send the command in case of interferences ... until we reach a TimeOut of 7s and then a NACK is issued and notified back to the host by the Coordinator.

The mecanism to be use to trigger Ack/noAck is based on the address mode used to send the unicat message

| Address Mode | Impact |
| ------------ | ------ |
| 0x02         | Short address with Ack mode ( you will receive 0x8012 and 0x8011 or 0x8702 ) |
| 0x07         | Short address with no Ack ( you won't be informed at all on the progress of the message) |
| 0x03         | IEEE address with Ack mode ( you will receive 0x8012 and 0x8011 or 0x8702 ) |
| 0x08         | IEEE address with no Ack ( you won't be informed at all on the progress of the message) |

Address modes 0x01, 0x04 are not relevant to unicast messages and are not discussed here.

We highly recommend to use Ack mode when sending a message to a node ( like and end device) which is not listening all the time, to make sure that ZiGate will make enought retry ( up to timeout of 7s ) to get the message received by the node.

ATTENTION:

* When sending command with ACK do not send any other command until you get Ack ( 0x8011 ) or a NACK ( 0x8702 )
* when sending command without ACK, do not send any other command until you get 8012 which acknowledge the fact that the command has been reach the next hop.

* When data is sent and an acknowledgement is required from the receiver, a timeout of approximately 1600 ms is applied to the acknowledgement - if no acknowledgement is received by the sender within this timeout period, the data is automatically re-sent. Up to 3 more re-tries can subsequently be performed, totalling just over 3 seconds before the data transfer is finally abandoned.In the case of data sent to a sleeping End Device, the acknowledgement is generated by the End Device after collecting the data from its parent. Thus, if the data is not collected within the acknowledgement timeout period, the data will be re-sent to the End Device (via its parent).    Note that if the buffered data is collected by the End Device after the final re-try by the sender but before the data is discarded by the parent (between approximately 3 and 7 seconds after the initial transmission), the acknowledgement that is eventually generated by the End Device will be ignored by the sender, since the transaction has already timed out and terminated.

### Exemple 1: Get the ZiGate firmware version

| Direction         | Message | Description |
| ---------         | ------- | ----------- |
| Host -> Controller | 0x0010  | Command 0x0010 Get Firmware Version |
| Controller -> Host | 0x8000  | Status |
| Controller -> Host | 0x8010  | Firmware version as the response |

### Exemple 2: Get Attribute 0x0000 of Cluster 0x0006 of a particular Node

| Direction | Message        | Description |
| --------- | -------        | ----------- |
| Host -> Controller | 0x0100 | Read attribute request    |
| Controller -> Host | 0x8000 | Command accepted          |
| Controller -> Host | 0x8012 | Cmd received by next hope |
| Controller -> Host | 0x8011 | Node has received cmd     |
| Controller -> Host | 0x8100 | Read attribute response   |

## SQN Type (applicable with firmware > 31d)

You can refer to [table: zigate command](zigate-commands.md) to check the command layer ( ZDP, ZCL )
When sending a command, a 0x8000 status message will be given and will provide you enough information to track the up-coming messages related to this command.

| 0-1 | 2-6      | 6-10       | 10-12   |  12-14 | 14-16   | 16-20       | 20-22    | 22-24   |
| --  | -------- | ---------- | ------- | ------ | ------- | ----------- | -------- | ------- |
| 01  | 0x8000   | Msg Lenght | Msg Crc | Status | SQN APP | Packet Type | Type Sqn | Sqn Aps |

## Key Message Type

| Message type | SQN used |
| ------------ | -------- |
| 0x8012       | APS      |
| 0x8011       | APS      |
| 0x8702       | APS      |

Note:

0x8012 and 0x8702 have the same API, 0x8012 acknowledge the command has reach the next hop, while 0x8702 inform a failure

## Other message Type

| Message Type    | Description                    |
| ------------    | -----------                    |
| 0x8035          | Internal ZiGate Message        |
| 0x9999          | NXP Extended Error Code        |
| 0x0012          | Indicates a load of PDM (reboot Zigate)  |
| 0x8701          | Async message, Route Discovery |
| 0x9996 - 0x9998 | Debug message                  |

## PDUs  & APS ack

number of PDUs and APS ack are configured at compile time.

* nPDU are network PDU
* aPDU are buffer to contain instances of a cluster
* Max APS ack determine The maximum number of simultaneous APSDE data requests with APS acknowledgements

You can refer to the ZigBee  PRO Sack User Guide ( Chapter 10) for more information

## References

* <https://www.nxp.com/docs/en/application-note/JN-AN-1216.zip>
* [NXP ZigBee PRO Stack User Guide](https://www.nxp.com/docs/en/user-guide/JN-UG-3101.pdf)
