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
| PI           | ZiGate V1    | ???? |
| USB          | ZiGate +/V2  | RTS/CTS HW flow control |
| DIN          | ZiGate +/V2  | RTS/CTS HW flow control |
| PI           | ZiGate +/V2  | ???? |

## Sequence

All commands generate a synchronous response code followed by any asynchronous responses as they become available. There is no sequence number associated with each command/response – the user must ensure that commands are issued sequentially.

| Direction | Message |
| --------- | ------- |
| Host -> Controler | Command |
| Controler -> Host | Status ( message type 0x8000 ) providing Status, SQN_APP, SQN_Type |
| Controler -> Host | Optional 0x8012, The controler acknowledges the fact that the command has been received by at least the next hope |
| Controler -> Host | optional, The node acknowledges each command with an “ACK” message. 0x8011 |
| Controler -> Host | optional, The message has not been successfully send (or the coordinator didn't received the Ack on time) and then a NACK message provided.0x80702  |
| Controler -> Host | Optional data messages as requested |

## Sending unicast command with Ack or not


When sending an unicat command ( which is dedicated to one Node ), you can send this command and expect in return some ack on the transit of the command from the controler to the Node.
For instance you can be informed that the command has reached the first hope (after the controler), and you can also be notified when the command has been fully received by the Node.

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

### Exemple 1: Get the ZiGate firmware version

| Direction         | Message | Description |
| ---------         | ------- | ----------- |
| Host -> Controler | 0x0009  | Command 0x0009 Get Firmware Version |
| Controler -> Host | 0x8000  | Status |
| Controler -> Host | 0x8009  | Firmware version as the response |

### Exemple 2: Get Attribute 0x0000 of Cluster 0x0006 of a particular Node

| Direction | Message        | Description |
| --------- | -------        | ----------- |
| Host -> Controler | 0x0100 |             |
| Controler -> Host | 0x8000 |             |
| Controler -> Host | 0x8012 |             |
| Controler -> Host | 0x8011 |             |
| Controler -> Host | 0x8100 |             |

## SQN Type

| Type | Description |
| ---- | ----------- |
| APS  |             |
| APP  |             |

## Key Message Type

| Message type |
| ------------ |
| 0x8000       |
| 0x8012       |
| 0x8011       |
| 0x8701       |
| 0x8702       |

## Other message Type

| Message Type    | Description                    |
| ------------    | -----------                    |
| 0x8035          |                                |
| 0x9999          |                                |
| 0x0012          |                                |
| 0x8701          | Async message, Route Discovery |
| 0x9996 - 0x9999 |                                |

## References

* <https://www.nxp.com/docs/en/application-note/JN-AN-1216.zip>
