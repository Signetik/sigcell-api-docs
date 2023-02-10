---
title: Signetik IoT API Reference

toc_footers:
  - Signetik IoT API 1.1.0
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - led

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Signetik IoT API
---

# Introduction

This document describes the API used by Signetik's wireless products. This document is subject to change and is versioned. Software releases of Signetik's IoT products refer to which version of the API is implemented.

## Changes and Additions
Signetik's APIs are complex and powerful. Signetik is always expanding and improving these modes and working with customers to add the features they need, within their schedule. If a feature is not applicable to Signetik's vision, then we work with customers on custom modifications by them or Signetik.


## Terminology
TBI: To Be Implemented

## Modes of Operation
Signetik's wireless modules support Sensor Master mode, Sensor Slave mode, and Modem modes. A short introduction to each mode follows.

### Sensor Master Mode
This mode allows the developer to connect sensor chips, such as I2C chips, to Signetik products, either as a prototype by wiring into one of the SigSense add-ons, or by creating a custom SigSense add-on board to put various sensor or control circuitry, onto a PCB.
Sensor interface configuration is provided by modifying Signetik's core application software.

Signetik's module pins are automatically configured using the Zephyr board definition. In Master Mode, UART0 is reserved for console input/output and UART1 is reserved for direct modem interaction.

### Sensor Slave Mode
This mode allows the developer to easily add Signetik products to an existing system, without having to manage the low-level reporting and connectivity. By sending simple ASCII “reports” to Signetik products, along with some configuration details, the system can offload sensor data and allow the device to manage the reporting. This document focus on the API for Sensor Slave Mode.

### Modem Modes
Modem modes are focused on pass-through use of the modems with some enhancements provided by Signetik.


# General
## Command Structure
### Format

```
+set,min_connection_delay:15,max_connection_delay:60\r\n
+rsp,ok\r\n
```

All commands start with "+" and end with "\r" and/or "\n". . If these characters are to be sent within the message, they should be escaped with the backslash "\" in the form of "\+" and "\\".

<aside class="success">
NOTE: when SigCell receives either "\r" or "\n", the command is immediately processed. Furthermore "\r" and "\n" characters are ignored until a non "\r"/"\n" character is received.
</aside>

Commands are of the form:  <cmd>,<arg> where <arg> is one or more key value pairs of the form <key>:<value> and multiple pairs are separated by the comma.
All commands and responses must contain less than 1024 characters, including the starting "+" and the final "\r" or "\n" sequence.

`+<cmd>\r\n`

Responses are of the form: <rsp>,<result> where <rsp> is the string "rsp" and <result> is context dependent.

### Format

`
+<rsp>\r\n`


# Convention
In the following API descriptions, the following characters are used:

 Character | Description
:-----:| ------
`<x>` | An item "x" which is further described.
`[x]` | An optional sequence "x"
`*` | Signifies the previous sequence can be repeated 0 to many times.

# Error Codes
Error | Meaning
:--: | -------
1 | invalid parameter value
2 | invalid parameter name
3 | push queue is full
4 | invalid access (read/write)
5 | invalid report

# Commands
## Set
```ascii
+set,min_connection_delay:15,max_connection_delay:60\r\n
+rsp,min_connection_delay:15,max_connection_delay:60\r\n
+set,min_connection_delay:150000,max_connection_delay:60\r\n
+rsp,min_connection_delay:!1,max_connection_delay:60\r\n
```
The set command allows one or more parameters to be set.
### Command

`+set,<key>:<value>[,<key>:<value>]*\r\n`

### Response
#### Success

`+rsp,<key>:<result>[,<key>:<result>]*\r\n`

where <result> is the newly set value for parameter <key>.

#### Failure

`+rsp,<key>:!<result>[,<key>:!<result>]*\r\n`

where <result> is the error code for failing to set parameter <key>.

<aside class="success">
NOTE: Some keys may return success and some may return an error, within one command. If one command sets multiple parameters, the operation will continue with setting others parameters even if setting a parameter fails.
</aside>

## Get
```
+get,min_connection_delay,max_connection_delay\r\n
+rsp,min_connection_delay:15,max_connection_delay:60\r\n
+get,min_connection_dela,max_connection_delay\r\n
+rsp,min_connection_dela:!2,max_connection_delay:60\r\n
```

The get command allows the retrieval of one or more parameters.
### Command
`+get,<key>[,<key>]*\r\n`
### Response
#### Success
`+rsp,<key>:<result>[,<key>:<result>]*\r\n`
where <result> is the current value for each key.
#### Failure
`+rsp,<key>:!<result>[,<key>:!<result>]*\r\n`
where <result> is the error code for each key.

<aside class="success">
NOTE: Some keys may return success and some may return an error, within one command.
</aside>

## Push
```
+set,queue1:THA
+push,THA,temp:25,humid:55,Ax:1,Ay:2,Az:3\r\n
+rsp,result:0,mark:37\r\n
+push,THA1,temp:25,humid:55,Ax:1,Ay:2,Az:3\r\n
+rsp,result:2\r\n
```
> NOTE: Push command failed due to a typo in report_name leading to not matching with any of the configured report_name


The push command allows sensor data to be pushed to the server-faced outgoing queue.
### Command
`+set,queue1:<queue name>`
`+push,<queue name>,<key>:<value>[,<key>:<value>]*\r\n`

For protocol "coap", the queue name must be pre-configured using SigNet webUI/API or via "+set" commands so that it can be matched up to the proper report type. Furthermore, the key/value pairs must match the report key/value pairs, previously configured at the SigNet.
Pushed data will be ignored/dropped by SigNet if <queue name> does not match one of the configured report types.
Pushed data will be ignored/dropped by SigNet if all the <key> names and values are not present and/or the corresponding <value> is not of the configured data type.
The <key>:<value> pairs can be present in any order.
For protocols that do not send key/value pairs, such as LoRaWAN (lora), the payload is sent using a single key/value where the key is "base64" and the value is base64 encoded data.
### Response
#### Success
`+rsp,result:<result>,mark:<id>\r\n`

where
<result> is 0 on success and non-zero on error
<id> is the unique id of the report being sent
#### Failure
`+rsp,result:<result>\r\n`

where <result> is the error code for the push operation.

<aside class="success">
NOTE: a success in the push command simply means the data was queued for sending. The mark value is used to check on the status of the push. See "push_mark" in the section Keys below.
</aside>
<aside class="success">
NOTE: various "notify" messages will be sent as the message works its way through the system, to the recipient.
</aside>
<aside class="success">
NOTE: push commands are put into a queue to be sent out at a later time. The push command will return an error if the queue is full
</aside>

## Pull
```
+notify,command:relay_command,enable:1\r\n
```

SigCell will asynchronously write to the terminal any received data or messages, as they come in. These will not interrupt command processing but may interrupt the serial visual display if operating manually with echo enabled.
Asynchronous receive commands contain the necessary information to allow decoding and processing.
### Command
None
### Response
`+notify,command:<command name>,<key>:<value>[,<key>:<value>]*\r\n`

where <command> is the command name being sent from the server, key is the name of the various keys being sent, and value is the value of the corresponding key..

# Configuration
## devid_gen
Read-only. The unique 64-bit device ID for this device. This can be used to set devid below.
devid
The unique 64-bit device ID for this device. This should be set by the user.

## server_address
The server IP address or DNS name to connect to. This key is not supported for SigLR or SigBLE.
default: iot.signetik.com

## server_port
The server port to connect to. This key is not supported for SigLR or SigBLE
range: 1 to 65535
default: 5715

## user
Allows setting of the user for protocols requiring login
Range: String
Default: Empty

## pw
Allows setting of the password for protocols requiring login
Range: String
Default: Empty

## connretry
This key allows for setting the time in seconds between retries in connecting to the network.
range: 15 to 3600
default: 300

## uart_echo
The received UART characters are typically echoed back to the terminal, as they are received. Furthermore, when an asynchronous event is received the current command buffer is re-echoed back to the terminal to allow for easy continuation of the interrupted command. This is desirable for human issuing of commands but not for devices. The echo can be turned off using the Set command.
Range: 0 to 1, where 0 is off and 1 is on.
Default: 1

## tls
Connections which support TLS will be enabled with TLS. UDP connections will enable DTLS.
Range: 0 to 1, where 0 is off and 1 is on.
Default: 1

## proto
Allows for choosing the protocol.
Choices: coap, mqtt, http, lora
Default: coap for SigCell and lora for SigLR
enabled
Enables protocol connectivity
Range: 0 to 1, where 0 is off and 1 is on
Default: 0

## cacert
Allows setting of the CA certificate
Range: String in double quotes. CR/LF are allowed inside the string
Default: Empty
Example:
+set,cacert:"-----BEGIN CERTIFICATE-----
MIIDdzCCAl+gAw…
...xMKQ3liZX
-----END CERTIFICATE-----"

## privcert
Allows setting of the private certificate
Range: String in double quotes. CR/LF are allowed inside the string
Default: Empty

## privkey
Allows setting of the private key
Range: String in double quotes. CR/LF are allowed inside the string
Default: Empty

## sectag
Allows setting of the security tag. The security tag is used by protocols in selecting which certificates to use. sectag = 0 means no certificates used.
Range: 0 to 255
Default: 0

# Status
## firmware
This key is used to retrieve the firmware version.
range text

## battery
This key is used to retrieve the battery voltage level.
range: 0 to 55
<aside class="success">
NOTE: returned value is 10 times the battery voltage.
</aside>

## connected
This key is used to indicate the connected state of the device on the network. It is not an indication of a connection to the server. SigCell will connect to the tower and enter PSM or eDRX mode. SigFi will connect to the wireless AP. SigLR will join the LoRaWAN network, and SigBLE will pair to another BLE device. This key indicates the state of these connections.
range: 0 to 1
0 indicates no connection, waiting to retry
1  indicates a successful connection

## queue_mark (TBI)
Returns the highest mark which has been queued.

## push_mark (TBI)
Returns the highest mark which has been successfully pushed to the server.

# Control
## save
This key is used to save certain variables to flash memory
range: 1
1 is used to save data to flash. A negative response, indicates failure.

## push_interval (TBI)
The nominal reporting interval in seconds. In order to preserve battery life and reduce network overhead, the device connects and pushes to the server in a buffered manner.  The device will not connect or push to the server  at a rate faster than this value. If no reports are queued to send, then the actual interval of the pushes will be extended. If this value is set to 0, then reports are pushed as they are received.
Range: 0 to 3600
Default: 120
In order to enforce a push interval larger than 3600 seconds, the controlling device should simply send push commands at the desired slow rate.

# Notifications
## Startup Notification
+notify,event:init,result:0

## Server Connection
+notify,event:mqtt,connected:1

## Push to Server
+notify,<proto>:push,mark:1

## Server Response (MQTT)
+notify,mqtt:pub,id:4667

## Downlink Message (LoRa)
+notify,lora:rx:base64:<base64 encoded payload>

# FOTA / DFU
## fotahost
Host to connect to for socket connection. This can be a name or IP address.
Range: String
Default: fota.signetik.com

## fotahostname
Host name used in request. This must be the DNS name of the server
Range: String
Default: fota.signetik.com

## fotafile
Filename and path of the file to download.
Range: String
Default: sigcell/v1.0.3.bin

## fotasubfile
Filename and path of the file to download for side application.
Range: String
Default: sigcell/v1.0.3.bin

## fotastart
```
+notify,fota:95
+notify,fota:complete
```

Initiate FOTA transfer
Range: 1
Default: 0

<aside class="success">
NOTE: once this entry is written to, FOTA begins. The file is downloaded via HTTP and if a valid firmware (BIN) is downloaded, then it will be applied. When it is complete, the device can be restarted and the new firmware will become active.
</aside>
<aside class="success">
NOTE: +notify,fota:<percent> will show for each chunk to indicate percent complete
</aside>

## fotasubstart
```
+notify,fota:95
+notify,fotasub:data,data:RVJJRlkgRkFJTC4gZXhwZWN0ZWQ6IDB4JTA4eCwgYWN0dWFsOiAweCUwOHgAAAAAZm90YV9mbGFzaF9ibG9jawAAAAAxMjMAAAAAAAAAAAAAwAAAvFADAAEAAAAAwAAAAAIAALxQAwACAAAAAMAAAACgBwC8UAMAAwAAAADCAAAAngcAvFADAAQAAAAAwgAAAMAAALxQAwAFAAAAAIIBAADeBgC8UAMABgAAAABgCAAAoAcAvFADAAcAAABOUkZfRkxBU0hfRFJWX05BTUUAAHBvd2VyAAAASQIDAHkAAwCZAQMAAAAAAF0AAwAAAAAAAAAAAAABAAQUUQMAAAAAAAAAAAAEAQgMHFEDAENMT0NLAAAAaGZjbGsAAABsZmNsawAAAGNsb2NrX2NvbnRyb2wAAABzeXNfY2xvY2sAAABUZXJtaW5hbAAAAABSVFQAU0VHR0VSAAByMC9hMTogIDB4JTA4eCAgcjEvYTI6ICAweCUwOHggIHIyL2EzOiAgMHglMDh4AAByMy9hNDogIDB4JTA4eCByMTIvaXA6ICAweCUwOHggcjE0L2xyOiAgMHglMDh4AAAgeHBzcjogIDB4JTA4eAAAc1slMmRdOiAgMHglMDh4ICBzWyUyZF06ICAweCUwOHggIHNbJTJkXTogIDB4JTA4eCAgc1slMmRdOiAgMHglMDh4AABmcHNjcjogIDB4==
+notify,fota:complete
```

Initiate FOTASUB transfer
Range: 1
Default: 0

<aside class="success">
NOTE: once this entry is written to, FOTASUB begins. The file is downloaded via HTTP and presented to the serial interface as notification of base64 data.
</aside>
<aside class="success">
NOTE: +notify,fota:<percent> will show for each chunk to indicate percent complete
</aside>

<aside class="success">
NOTE: Chunks are presently 4096 bytes, before base64 encoding, but are subject to change in the future. It is best that the user parses data in base64 fragments. I.E integer multiples of 4 bytes. In the future, if the chunk size changes, it will default to 4096 but allow the user to change if needed.
</aside>

## fotastate
FOTA state
Range: 0-2, where 0 is IDLE, 1 is starting, and 2 is in progress
Default: 0

<aside class="success">
NOTE: You should not start any FOTA or FOTASUB while fotastate or fotasubstate are non-zero
</aside>

## fotasubstate
FOTA sub state
Range: 0-2, where 0 is IDLE, 1 is starting, and 2 is in progress
Default: 0

<aside class="success">
NOTE: You should not start any FOTA or FOTASUB while fotastate or fotasubstate are non-zero
</aside>

## dlcount
FOTASUB download count. This variable is read only and shows the current chunk that is sent
Range: 0-65535
Default: 0

## dlchunks
FOTASUB download chunks. This allows the user to choose the number of chunks to download.
Range: 0-65535, where 0 is ALL and non-zero is partial downloads
Default: 0

<aside class="success">
NOTE: Chunks are presently 4096 bytes but subject to change in the future.
</aside>

## dlretcount
FOTA current retry count for FOTA and FOTASUB. Read only. This indicates which retry is being performed
Range: 0-65535
Default: 5

## dlretries
FOTA total retries for FOTA and FOTASUB
Range: 0-65535
Default: 5

## dloffset
> FOTA

```
+set,fotahost:fota.signetik.com
+set,fotahostname:fota.signetik.com
+set,fotafile:sigcell/v1.0.4.bin
+set,fotastart:1
```

> FOTASUB

```
+set,fotahost:fota.signetik.com
+set,fotahostname:fota.signetik.com
+set,fotasubfile:sigcell/v1.0.4.bin
+set,fotasubstart:1
```

> FOTASUB partial

```
+set,fotahost:fota.signetik.com
+set,fotahostname:fota.signetik.com
+set,dlchunks:1
+set,dloffset:4096
+set,fotasubfile:sigcell/v1.0.4.bin
+set,fotasubstart:1
```

FOTA byte offset for start of FOTASUB download
Range: 0-(232-1)
Default: 0

# Connections
In some cases, it is desirable to control the socket connections on SigCell and SigFi. These commands allow the opening, reading, writing, and closing of sockets.
## UDP
### udp_open
Command
+udp_open,host:<ip>,port:<port>\r\n
Response
+rsp,result:<result>,socket:<socket>\r\n
<result> will be zero on success and non-zero on error.
<socket> integer: [1, 255] will be returned upon success

<aside class="success">
NOTE: This value of <socket> should be used in subsequent udp_send, udp_close commands
</aside>

### udp_send
Command
+udp_send,socket:<socket>,data:”<data>”\r\n
Response
+rsp,result:<result>
where <result> is zero on success and non-zero on failure.

### udp_receive
UDP receive packets are displayed as they arrive, as follows.
Response
+notify,event:udp_receive,data:<data>\r\n

### udp_close
Command
+udp_close,socket:<socket>
Response
+rsp,result:<result>
where <result> is zero on success and non-zero on failure.

## TCP
### tcp_open
Command
+tcp_open,host:<ip>,port:<port>\r\n
Response
+rsp,result:<result>,socket:<socket>\r\n
<result> will be zero on success and non-zero on error.
<socket> integer: [1, 255] will be returned upon success

<aside class="success">
NOTE: This value of <socket> should be used in subsequent tcp_send, tcp_close commands
</aside>

### tcp_send
Command
+tcp_send,socket:<socket>,data:”<data>”\r\n
Response
+rsp,result:<result>
where <result> is zero on success and non-zero on failure.

### tcp_receive
TCP received packets are displayed as they arrive, as follows.
Response
+notify,event:tcp_receive,data:<data>\r\n

### tcp_close
Command
+tcp_close,socket:<socket>
Response
+rsp,result:<result>
where <result> is zero on success and non-zero on failure.

## HTTP
```
+set,uri:devices/1234AAAB/
+set,keepalive:1
```

HTTP can be used after opening a socket with either tcp_open or udp_open, and setting up the following parameters:
+set,server_address:www.signetik.iot

<aside class="success">
NOTE: The server address should match the host address used to open a connection.
</aside>

### http_get
Command
+http_get,socket:<socket>[,opt_headers:”<optional headers>\r\n”]\r\n
Response
+rsp,result:<result>\r\n
where <result> is zero on success and non-zero on failure.

### http_put
Command
+http_put,socket:<socket>[,opt_headers:”<optional headers>\r\n”],data:”<data>”\r\n
Response
+rsp,result:<result>
where <result> is zero on success and non-zero on failure.

### http_post
Command
+http_post,socket:<socket>[,opt_headers:”<optional headers>\r\n”],data:”<data>”\r\n
Response
+rsp,result:<result>
where <result> is zero on success and non-zero on failure.

### http_delete
Command
+http_delete,socket:<socket>[,opt_headers:”<optional headers>\r\n”]\r\n
Response
+rsp,result:<result>\r\n
where <result> is zero on success and non-zero on failure.

### http_patch
Command
+http_patch,socket:<socket>[,opt_headers:”<optional headers>\r\n”],data:”<data>”\r\n
Response
+rsp,result:<result>
where <result> is zero on success and non-zero on failure.

## SSL/TLS
TLS is enabled by setting the ‘sectag’ variable "+set,sectag:1". If the value of sectag is greater than 0, TLS will be enabled for any TCP connection. Once enabled, socket connections will use ‘sectag’ to load the certificate. Running "+set,sectag:0" will disable TLS for TCP connections.
In order to set the certificates to use TLS, the modem must be in a non-connected state. Setting the ‘connect’ variable to 0 will accomplish this, “+set,connect:0”. Then the ‘privkey’, ‘cacert’, and/or ‘privcert’ variables can be set. After setting ‘cacert’, running “+get,cacert” should report the same certificate that was set. In rare situations, the certificate will be truncated due to the way low level functionality is handled. The ‘privkey’ and ‘privcert’ variables cannot be read for security.

## MQTT
> Following are example settings for use on Azure IoT MQTT.

```
+set,proto:mqtt
+set,devid:1234AAAB
+set,server_address:signetik.azure-devices.net
+set,server_port:8883
+set,user:signetik.azure-devices.net/1234AAAB
+set,pwSharedAccessSignature sr=signetik.azure-devices.net%2Fdevices%2F1234AAAB&sig=6tN5M…….....9Kg%3D&se=1598116935
+set,subtopic:devices/1234AAAB/messages/devicebound/#
+set,subtopicmethod:$iothub/methods/POST/#
+set,pubtopic:devices/1234AAAB/messages/events/
+set,report1,test
```

> Once the settings are properly set, the connection should be enabled.

```
+set,enabled:1
```

> Then wait for the connection

```
+notify,event:mqtt,connected:1
```

> Now data can be pushed to a queue named by queue1-queue5.

```
+push,queue1,temp:1,humid:2
```

> The response indicates the "mark" of the data.

```
+rsp,result:0,mark:2
```

> Once this data is pushed to the queue, it will eventually be removed from the queue and sent to the server, with the following notifications.

```
+notify,mqtt:push,mark:2,id:4667
```

> The server responds that the data was received.

```
+notify,mqtt:pub,id:4667
```

> All data received from the server will show up as follows.

```
+notify,mqtt:sub,payload:<data>
```

In some cases, it is desirable to use MQTT directly from the device. These commands are supported on SigCell and SigFi.

### subtopic
Allows setting of the subscribe topic
Range: String
Default: Empty

### subtopicmethod
Allows setting of the subscribe topic for Azure IoT method invocation
Range: String
Default: Empty

### pubtopic
Allows setting of the publish topic to be used by push commands
Range: String
Default: Empty

### qos
Allows setting of the qos for all messages
Range: 0-2
Default: 1

### keepalive
Allows setting of the keep alive interval for pinging the broker
Range: 0-65535
Default: 60

## HTTP REST

> Following are example settings for use on HTTP REST.

```
+set,proto:rest
+set,devid:1234AAAB
+set,host:www.signetik.iot
+set,port:8080
+set,apikey:xyz1234
+set,uri:devices/1234AAAB/
+set,report1,test
```

> Once the settings are properly set, the connection should be enabled.

```
+set,enabled:1
```

> Although an actual connection is not enabled to a server, the connection must be enabled to be used.

```
+notify,event:rest,connected:1
```

> Now data can be pushed to a queue named by report1-report5.

```
+push,test,temp:1,humid:2
```

> The response indicates the "mark" of the data.

```
+rsp,result:0,mark:2
```
> Once this data is pushed to the queue, it will eventually be removed from the queue and sent to the server, with the following notifications.

```
+notify,rest:post,mark:2
```

> The server responds that the data was received.

```
+notify,rest:pub,id:4667
```

> All data received is received from the server as responses to push commands. They will show up as follows.

```
+notify,rest:rsp,payload:<data>
```

In some cases, it is desirable to use REST directly from the device. These commands are supported on SigCell and SigFi.

The examples to the right show how to use the REST connection directly.

# SigCell
## nbiot
Allows setting of NBIoT as default connection type
Range: 1
Default: 0
If value is 0, then LTE-M is attempted with NB IoT fallback. If value is 1, then NB IoT is attempted with LTE-M fallback. This system must be restarted for this to go into effect.

## mfirmware
This key is used to retrieve the modem firmware version.
range text

## cell_rsrp
Returns the signal strength in dbm.
Range: -140 to 0
NOTE: signal strength is typically -140 to -44. A value of 0 indicates unknown signal strength.

rsrp | meaning
:--: | ----
> -84 | Excellent
-85 to -102 | Good
-103 to -111 | Fair
< -111 | Poor


## cell_connected (TBI)
This key is used to indicate the connected state of the device on the network. It is not an indication of a connection to the server. SigCell will connect to the tower and enter PSM or eDRX mode. This key indicates the state of these connections.
range: 0 to 2

value | meaning
:--: | ----
0 | no connection, waiting to retry
1 | a connection is being attempted
2 | a successful connection
3 | a succesful bootstrap connection (connected for FOTA check, but not ready for communication.)

## imei
This key is used to read the IMEI of the device.

## imsi
This key is used to read the IMSI of the device.

## iccid
This key is used to read the ICCID of the device.

## time
This key is used to read the network time of the device.

# SigFi
TBD

# SigLRN
```
+set,enabled:1
+notify,lora:join,status:start
+notify,lora:join,status:complete
+push,base64:"dGVzdA=="
+notify,lora:rx,base64:<base64 encoded payload>
```

## auth
"abp" or "otaa"

## appeui (OTAA)
Allows setting of the AppEUI

## deveui (OTAA/ABP)
Allows setting of the DevEUI

## devaddr (ABP)
32-bit device address (ABP only)
## appkey (OTAA)
Application key

## nwkskey (ABP)
Network session key

## appskey (ABP)
Application session key (ABP only)

NOTE: EUI and keys should be set before enabling LoRa protocol.
NOTE: when "enabling" LoRa proto (see enable and proto variables), the device will automatically join the network for OTAA. Once joined, or if ABP is selected, `push` can be used to send binary messages.
NOTE: binary payloads in ascii uart mode can be sent using base64 encoding.
Downlink messages will show up using +notify messages. Join status will also show up as +notify messages.

Format
`+notify,lora:join,status:<state>`

state | meaning
:--: | -----
start | join process started
send | join send request
completed | join process completed successfully
timeout | failed to join
rejected | rejected by join server

# SigBLE
TBD

# GPS
Control Commands
## gpsenable
Enable/Disable GPS receiver
range: 1, 0
default: 0
Eg: +set, gpsenable:0 to enable gps

gps
Get GPS NMEA sentences
Eg: +get, gps


