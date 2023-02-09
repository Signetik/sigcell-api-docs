# Future API
Pending

(Restore this when code implements this method as described here)
connected
This key is used to indicate the connected state of the device on the network. It is not an indication of a connection to the server. SigCell will connect to the tower and enter PSM or eDRX mode. SigFi will connect to the wireless AP. SigLR will join the LoRaWAN network, and SigBLE will pair to another BLE device. This key indicates the state of these connections.
range: 0 to 2
0 indicates no connection, waiting to retry
1 indicates a connection is being attempted
2 indicates a successful connection

## GPS
Keys
Control Commands
gpsenabled
Enable/Disable GPS receiver
range: 1, 0
default: 0
Report Commands
gpsinterval
Interval to query GPS data (seconds)
range: 1, 65535
default: 1
gpsreport:
	- What GPS data gets reported? (pos, date, time, datetime, all, gps)
		- pos (position)
		- date
		- time
		- datetime (date and time)
		- all (pos, date, and time)
		- gps (same as all)
+get,gpsreport
	- List GPS report data


### Data query commands

+get,gps
	- Get all GPS data (position, date, and time)
+get,gpspos
	- Get current GPS position (Lat, Lon, Alt)
+get,gpsdate
	- Get current GPS date (YYYY-MM-DD)
+get,gpstime
	- Get current GPS time (HH:MM:SS)
+get,gpsdatetime
	- Get current GPS date and time


### Advanced (NMEA) Data Query

+get,gga
	- Get Global Positioning System Fix Data.
+get,gll
	- Get Geographic Position, Latitude / Longitude and time.
+get,rmc
	- Get Recommended minimum specific GPS/Transit data.
+get,vtg
	- Get Track Made Good and Ground Speed.
+get,zda
	- Get Date & Time.

Alternate Form:
+get,nmea:[gga/gll/rmc/vtg/zda]

.SigLRN
auth
"abp" or "otaa"
Selects ABP (Authorization By Personalization) or OTAA (Over The Air Authorization)
default: “ABP”
+set,auth:OTAA
class
“A" or "C"
Selects Class A or Class C LoRaWAN operation
default: “C”
+set,class:a
adrenabled
Boolean for enabling LoRaWAN Adaptive Data Rate.  Enabled = 1, disabled = 0.
range: 1, 0
default: 0
+set,adrenabled:1
datarate
Integer value for fixed LoRaWAN data rates.  Valid only if Adaptive Data Rate is disabled.
Range 0 - 15
default: 1
+set,datarate:2
chanmask
Ten byte array specifying the LoRa channel mask assignment
+set,chanmask:0x00112233445566778899
deveui (OTAA/ABP)
Allows setting of the DevEUI
+set,deveui:0x0011223344556677

devaddr (ABP)
32-bit device address (ABP only)
+set,devaddr:0x11223344
nwkskey (ABP)
Network session key
+set,nwkskey:0x00112233445566778899AABBCCDDEEFF
appskey (ABP)
Application session key (ABP only)
+set,appskey:0x00112233445566778899AABBCCDDEEFF
appkey (OTAA)
Application key
+set,appkey:0x00112233445566778899AABBCCDDEEFF
appeui (OTAA)
Allows setting of the AppEUI
+set,appeui:0x0011223344556677

NOTE: EUI and keys should be set before enabling LoRa protocol.
NOTE: when "enabling" LoRa proto (see enable and proto variables), the device will automatically join the network for OTAA. Once joined, or if ABP is selected, `push` can be used to send binary messages.
NOTE: binary payloads in ascii uart mode can be sent using base64 encoding.
Downlink messages will show up using +notify messages. Join status will also show up as +notify messages.
+notify,lora:join,status:start
+notify,lora:join,status:send
+notify,lora:join,status:timeout
+notify,lora:join,status:rejected
+notify,lora:join,status:complete
+notify,lora:rx,base64:<base64 encoded payload>
LoRa transmit status messages will show us as +notify messages, some including the channel and the data rate of the transmission:
+notify,lora:tx,status:success,chan:0,dr:1
+notify,lora:tx,status:fail,chan:1,dr:1
+notify,lora:tx,status:send
+notify,lora:tx,status:complete
+notify,lora:tx,status:error
Example
+set,enabled:1
+push,base64:"dGVzdA=="

SigLRN Internally Controlled LED Operations, enabled by default.
Power up:  		Blue
Lora packet transmit: 	Purple for duration of transmission
Lora packet receive:	Green blink
Lora transmit error: 	Red blink of 1 second duration
SigLRN User LED Operation
leds
This key is used to control power to LEDS.  The individual Red, Green and Blue LED elements can be selected individually or in combination, and the Enable controls the power to the LED assembly.
The user LED combination will remain active until one of the internally controlled LED operations occurs.
range: 0 : 15   [xxxx3210]
Enable: Bit position 3:
1: Status LED is enabled.
0: Status LED is disabled for ultra low power mode.
RED: Bit position 2:
1: Red LED is enabled.
0: Red LED is disabled.
GREEN: Bit position 1:
1: Green LED is enabled.
0: Green LED is disabled.
BLUE: Bit position 0:
1: Blue LED is enabled.
0: Blue LED is disabled.
