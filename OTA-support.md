# OTA support

Zigate embedds an OTA server.

## Upgrade process overview

Basically the OTA upgrade process with Zigate is the following
- Plugin indicates to Zigate that there is an OTA image available (0500 message). Currently only 1 image can be notified.
- Optionally, device is notified there is an image for him thru 0505 message.
- Device requests if there is any OTA image for him (Query Next Image Request/Cmd 01)
- Zigate answers there is one (if plugin has "loaded" the firmware with 0500) => Query Next Image Response/Cmd 02
- Device sends block image requests as many as necessary to collect the full firmware (8501 message)
- Plugin receives 8501 message and answers to each request sending FW content thru 0502 message.
- Device signals transfer is completed with or without error (8503 message)

## Limitations

- Only 1 image can be "loaded" in Zigate

## Commands summary

| Command                   | Code   | Description                                    |
|--|---|---|
| LOAD_NEW_IMAGE            | 0x0500 | Load headers to ZiGate. Use this command first |
| BLOCK_REQUEST             | 0x8501 | ZiGate will receive this command when device asks OTA firmware |
| BLOCK_SEND                | 0x0502 | This is used to transfer firmware to device when it sends request 0x8501. |
| UPGRADE_END_REQUEST       | 0x8503 | Device will send this when it has received last part of firmware |
| UPGRADE_END_RESPONSE      | 0x0504 |  |
| IMAGE_NOTIFY              | 0x0505 | Notify desired device that ota is available |
| SEND_WAIT_FOR_DATA_PARAMS | 0x0506 | Can be used to delay/pause OTA update |

## Commands detail

| 0500 command                  | Nb bytes   | Description |
|---|-----|-----|
| addrMode | 1 | 02 |
| addr | 2 | Assuming addrMode=02 => addr=0000 |
| header | ? | Header from OTA file |

| 0502 command                  | Nb bytes   | Description |
|---|-----|-----|
| addrMode | 1 | |
| addr | 2 | Assuming addrMode=02 |
| srcEp | 1 | |
| dstEp | 1 | |
| sqn | 1 | |
| status | 1 | |
| imgOffset | 4 | |
| imgVersion | 4 | |
| imgType | 2 | |
| manufCode | 2 | |
| dataSize | 1 | |
| data | dataSize | |

| 0505 command                  | Nb bytes   | Description |
|---|-----|-----|
| addrMode | 1 | |
| addr | 2 | Assuming addrMode=02 |
| srcEp | 1 | |
| dstEp | 1 | |
| status | 1 | |
| imgVersion | 4 | |
| queryJitter | 1 | |

| 8503 command                  | Nb bytes   | Description |
|---|-----|-----|
| sequence | 1 | |
| endpoint | 1 | |
| cluster | 2 | |
| address_mode | 1 | |
| addr | 2 | Assuming addrMode=02 |
| file_version | 4 | |
| image_type | 2 | |
| manufacture_code | 2 | |
| status | 1 | |
