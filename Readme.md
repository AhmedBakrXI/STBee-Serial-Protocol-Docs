# STBee Serial Protocol


# Table of Contents
<details>
<summary></summary>

- [1. Introduction](#1-introduction)
  * [1.1 Overview](#11-overview)
- [2. Command Layer Communication](#2-command-layer-communication)
  * [2.1 CRA Frame](#21-cra-frame)
  * [2.2 Command Format](#22-command-format)
    + [2.2.1 Unique Identifier](#221-unique-identifier)
    + [2.2.2 Reserved](#222-reserved)
    + [2.2.3 Acknowledge Flag (AF)](#223-acknowledge-flag--af-)
    + [2.2.4 Device or Network (DEV/NWK)](#224-device-or-network--dev-nwk-)
    + [2.2.5 Priority](#225-priority)
    + [2.2.6 Command](#226-command)
    + [2.2.7 Payload](#227-payload)
- [2.3 Response Format](#23-response-format)
    + [2.3.1 Unique Identifier](#231-unique-identifier)
    + [2.3.2 Status](#232-status)
    + [2.3.3 Response](#233-response)
  * [2.4 Diagrams](#24-diagrams)
    + [2.4.1 Layered diagram](#241-layered-diagram)
    + [2.4.2 Multiple Requests All Success Case](#242-multiple-requests-all-success-case)
    + [2.4.3 Multiple Requests Timeout Case](#243-multiple-requests-timeout-case)
    + [2.4.4 Single Command Duplicate ACK](#244-single-command-duplicate-ack)
    + [2.4.5 Single Command ACK MISS](#245-single-command-ack-miss)
    + [2.4.6 Single Corrupted Command](#246-single-corrupted-command)
    + [2.4.7 Single Command Corrupted Response](#247-single-command-corrupted-response)
    + [2.4.8 Client State Diagram](#248-client-state-diagram)
    + [2.4.9 Server State Diagram](#249-server-state-diagram)
  * [2.5 Entities](#25-entities)
- [3. Data Transfer Layer (DTL)](#3-data-transfer-layer--dtl-)
  * [3.1 DTL Packet Format](#31-dtl-packet-format)
    + [3.1.1 CRC Enable (CRC EN)](#311-crc-enable--crc-en-)
    + [3.1.2 Reserved](#312-reserved)
    + [3.1.3 Payload](#313-payload)
    + [3.1.4 CRC 16](#314-crc-16)
    + [3.1.5 SLIP Wrapper](#315-slip-wrapper)
  * [3.2 Entities](#32-entities)
- [4. Physical Layer](#4-physical-layer)
- [5. Services](#5-services)
  * [5.1 Network Services](#51-network-services)
    + [5.1.1 Network Capture](#511-network-capture)
    + [5.1.2 Node Discovery](#512-node-discovery)
    + [5.1.3 Link Query](#513-link-query)
- [6. Appendix](#6-appendix)
  * [ACK FRAME](#ack-frame)
  * [STATUS Byte](#status-byte)

</details>

# 1. Introduction

## 1.1 Overview
This protocol depends on Command-Response-Acknowledgment `(CRA)` messages and is designed to handle multiple requests at same time.

CRA messages are binary encoded to reduce the overhead of using strings.

There are two types of messages: error-checked messages and non error checked messages introduced in DTL section.

<img src="images/client-server-communication.png" style="background-color: white; border-radius: 5px;">

# 2. Command Layer Communication
## 2.1 CRA Frame
In this protocol standard there is three types of messages.

- Command Message: always sent from the client to request some action.

- Response Message: always sent from server after processing a request from client. Response is considered acknowledgment.

- Acknowledge Message: always sent from the client after processing to tell the server that response has arrived and processed correctly, so it can close the CRA call and discard response if needed.

<img src="out/content/core/seq-simple-cra/Simple CRA Flow.svg" style="display: block; margin: 0 auto">

## 2.2 Command Format
The following figure outlines the command format which is used for communication purposes. 

<img src="images/command-frame.png">

### 2.2.1 Unique Identifier
This field contains a unique 8-bit integer used to identify each CRA session in order to allow handling multiple requests at same time and this allows handling up to 256 request at same time.

### 2.2.2 Reserved
This field contains 4-bits that are reserved for future uses and can be used as flags if needed.

### 2.2.3 Acknowledge Flag (AF)
This flag is used to distinguish between commands and acknowledge frames. It's set to zero if command and one otherwise.

### 2.2.4 Device or Network (DEV/NWK)
This flag is used to distinguish between device and network commands.
Zero for device commands and one otherwise.

### 2.2.5 Priority
Contains two bits to support four levels of priority. Server should handle higher priority commands before lower ones.

### 2.2.6 Command
This field contains 8 bits to support up to 256 commands for network and same for device.

### 2.2.7 Payload
This field contains any parameters sent by application layer and has maximum size of 300 bytes.

# 2.3 Response Format
The following figure outlines the response format which is used for communication purposes. 

<img src="images/response-frame.png">

### 2.3.1 Unique Identifier
This field contains a unique 8-bit integer used to identify each CRA session in order to allow handling multiple requests at same time and this allows handling up to 256 request at same time.

It's generated by client.

### 2.3.2 Status
This field contains the status of response to support easy debugging and error handling.

### 2.3.3 Response
This field contains the response to the command sent.


## 2.4 Diagrams
### 2.4.1 Layered diagram
<img src="out/content/core/seq-single-cmd-layered/Single Command Execution - Success Case.svg">

### 2.4.2 Multiple Requests All Success Case
<img src="out/content/core/seq-multiple-cmds-hl/Multiple Commands.svg">

### 2.4.3 Multiple Requests Timeout Case
<img src="out/content/core/seq-multiple-cmds-timeout/Multiple Commands.svg">

### 2.4.4 Single Command Duplicate ACK
<img src="out/content/core/seq-single-cmd-dup-ok/Single Commands.svg">

### 2.4.5 Single Command ACK MISS
<img src="out/content/core/seq-single-resp-miss-ack/Single Commands.svg">

### 2.4.6 Single Corrupted Command
<img src="out/content/core/seq-send-single-corrupted-command/Single Commands.svg">

### 2.4.7 Single Command Corrupted Response
<img src="out/content/core/seq-single-resp-corrupted/Single Commands.svg">

### 2.4.8 Client State Diagram
<img src="out/content/client/state/Client State Diagram.svg">

### 2.4.9 Server State Diagram
<img src="out/content/server/state/Server State Diagram.svg">

## 2.5 Entities
<img src="images/command-entities.png">

# 3. Data Transfer Layer (DTL)

This layer provides two ways of sending messages: error-checked messages and non error checked messages.

In error-checked messages 16-bits FCS are added as footer.
On error found the error is reported to upper layer to be handled.

Slip encoding is performed as a last stage to wrap the packet and determine its length.


## 3.1 DTL Packet Format
The following figure outlines the dtl packet format.
<img src="images/dtl-packet.png" style="display: block; margin: 0 auto; background-color: white;">

### 3.1.1 CRC Enable (CRC EN)
On set to one then a 16-bit CRC is generated and attached to the packet footer, otherwise no CRC is generated.

### 3.1.2 Reserved
7 bits reserved for future use.

### 3.1.3 Payload
Contains the payload of the upper layer.

### 3.1.4 CRC 16
Optional 16-bits for CRC calculation.

### 3.1.5 SLIP Wrapper
This is used to wrap the packet before sending it to physical layer, this is important to determine the begin and end of the packet.

## 3.2 Entities
<img src="images/dtl-entities.png">

# 4. Physical Layer
This layer is responsible for sending packets over the physical wire using UART communication protocol.

Specifications:
- Baud Rate: 115200 bps
- One Stop Bit
- No Parity
- Data Bits: 8 bits
- Flow Control: RTS/CTS (if board doesn't support then use XON/XOFF)

# 5. Services
## 5.1 Network Services
### 5.1.1 Network Capture
This command should force the server to hold a capture of nodes and links.

Command Message
| Field | Content | Description |
| :----  | :------- | :----------- |
| AF | COMMAND_FRAME | Set frame as command frame |
| Command | NWK_CAPTURE | Network Capture Request |

Response Message
| Field | Content | Description |
| :----  | :------- | :----------- |
| Status | Status byte | see appendix |
| Response[0] | Capture Status | 0x00: START Capture <br> Else this indicates a problem |

### 5.1.2 Node Discovery
This command is used to discover a new node in the captured network, if all nodes are discovered then it returns `END_OF_CAPTURED_NODES`.


Command Message
| Field | Content | Description |
| :----  | :------- | :----------- |
| AF | COMMAND_FRAME | Set frame as command frame |
| Command | NWK_DISCOVER | Network Discovery Request |

Response Message
<div class="node-disc-table">

| Field | Content | Description |
| :----  | :------- | :----------- |
| Status | Status byte | see appendix |
| Response[0..1] | Network Address of Node or `END_OF_CAPTURED_NODES` | 16-bit integer |
| Response[2..11] | MAC Address of Node | 64-bit integer |
| Response[12] | Logical Type of Node | 0x00: Coordinator <br> 0x01: Router <br> 0x02: End Device |
| Response[13..14] | Parent Network Address of Node | 16-bit integer |

</div>


### 5.1.3 Link Query

Command Message
| Field | Content | Description |
| :----  | :------- | :----------- |
| AF | COMMAND_FRAME | Set frame as command frame |
| Command | LINK_DISCOVER | Link Discovery Request |

Response Message
<div class="node-disc-table">

| Field | Content | Description |
| :----  | :------- | :----------- |
| Status | Status byte | see appendix |
| Response[0..1] | Network Address of First Node or `END_OF_CAPTURED_LINKS` | 16-bit integer |
| Response[2..3] | Network Address of Second Node | 16-bit integer |
| Response[4] | Income Cost of Link | Ranges from 0 to 7 |
| Response[5] | Outcome Cost of Link | Ranges from 0 to 7 |

</div>


# 6. Appendix
## ACK FRAME
| Field | Content | Description |
| :---- | :------ | :---------- |
| AF    | ACK_FLAG_ON | Set to 1 |
| Command | ACK_FRAME_ACTION | The action that should be done after receiving the ACK frame |

## STATUS Byte

<style>
    .node-disc-table tr:nth-child(3) { background: gray;}
    .node-disc-table tr:nth-child(4) { background: gray;}
    .node-disc-table tr:nth-child(5) { background: gray;}
</style>


