## led
The LED API includes commands to set operating parameters individually or with a single command that can take effect immediately or be queued by the LED task to process in order of reception. It also includes the ability to configure internal LED lookup tables for specific color/blink/intensity combinations which may be referenced by index. 
### LED Control API
- leds
- led_red
- led_green
- led_blue
- led_intensity
- led_blink_count
- led_blink_ontime
- led_blink_offtime
- led_blink_start
- ledctl
- ledmsg
- led_userlut
- active_ledlut
 
### leds

> This key controls the led enable. Range 0:1

NOTE: A value of 0 will disable the LED circuitry and turn off the LED.

### led_red

> This key is used to set the LED level. Range: 0-255.  The LED's red will be set to this brightness level.

NOTE: This message will cause ledctl to be auto-set to 1.

NOTE: A value of 0 will turn off this color. Setting all colors to 0 will turn off the LED.

### led_green

> This key is used to set the LED level. Range: 0-255.  The LED's green will be set to this brightness level.

NOTE: This message will cause ledctl to be auto-set to 1.

NOTE: A value of 0 will turn off this color. Setting all colors to 0 will turn off the LED.

### led_blue

> This key is used to set the LED level. Range: 0-255.  The LED's blue will be set to this brightness level.

NOTE: This message will cause ledctl to be auto-set to 1.

NOTE: A value of 0 will turn off this color. Setting all colors to 0 will turn off the LED.

### led_color

> This key is used to load values from a predefined color table into the led_red, led_green and led_blue component variables. Range text

 > - off
 > - white
 > - red
 > - green
 > - blue
 > - orange
 > - yellow
 > - purple
 
NOTE: This message will cause ledctl to be auto-set to 1.
 
### led_intensity

> This key is used to scale the default color intensity in 1% increments. Range 0-100


### led_blinks

> This key is used to set the number of on/off blinks for a requested sequence. Range 0-65535

NOTE: Setting blinks to 0 will permit indefinite blinking until turned off through a new command.


### led_blink_ontime

> This key is used to set the duration in milliseconds for active portion of the blink cycle. Range 0-65535

NOTE: Blink duration resolution will be set by the target processor and is typically 100ms. Durations of finer resolution will be truncated.


### led_blink_offtime

> This key is used to set the duration in milliseconds for off portion of the blink cycle. Range 0-65535

NOTE: Blink duration resolution will be set by the target processor and is typically 100ms. Durations of finer resolution will be truncated.


### led_blink_start

> This key is used to set the operating mode of the blink request. Range 0-2

Mode | Description
-------|--------
0 | Blink disabled
1 | Blink for minimum of led_blinks setting and then continuously until stopped by a command.
2 | Blink for led_blinks setting and then turn off LED.

### ledctl

```
+set,ledctl:1,color:custom,r:255,g:0,b:255,i:75,blink:4,on:500,off:1500,start:1
```
​
> The above command sets the ledctl to active and configures the LED controller to a custom color mixing red and blue at 75% intensity for a minimum of 4 blinks of 500ms ON and 1500ms OFF and returns
​
```ascii
+rsp,1,color:custom,r:255,g:0,b:255,i:75,blink:4,on:500,off:1500,start:1
```

This endpoint command sets the various led control keys in a single command. Most parameters are optional and those not included in the command will allow the controller to use the existing values for those settings.

### command

`+set,ledctl:<parameters>`

### +set Parameters

Parameter | Required | Description
--------- | -------- | -----------
ledctl | Yes |Enable for led controller
color | No | Predefined color name or "custom"
r | No | Red component value for custom color
g | No | Green component value for custom color
i | No | Intensity percentage for LED.
blink | No | Number of blinks if blink enabled
on | No | Duration of ON portion of blink (ms)
off| No | Duration of OFF portion of blink (ms)
start|No| Blink mode
lut |No | Selects active LED LUT to use when ledctl disabled.

<aside class="success">

</aside>

### ledmsg
This is a special version of the ledctl message in that it is received by a task queue and is processed in order of reception.  Unlike ledctl, it does not directly update the various led settings described above but passes values directly to the internal LED controller.

```
+set,ledmsg:1,color:blue,i:100,blink:4,on:500,off:1500,start:2
```
​
> The above command sets the ledctl to active and configures the LED controller to a predefined color called blue at 100% intensity for a fixed 4 blinks of 500ms ON and 1500ms OFF and returns
​
```ascii
+rsp,1,color:blue,r:0,g:0,b:0,i:100,blink:4,on:500,off:1500,start:2
```

This endpoint command sets the various led control keys in a single command. Most parameters are optional and those not included in the command will allow the controller to use the existing values for those settings.

### command

`+set,ledmsg:<parameters>`

### +set Parameters

Parameter | Required | Description
--------- | -------- | -----------
ledmsg | Yes |Enable for led controller
color | Yes | Predefined color name or "custom"
r | No | Red component value for custom color
g | No | Green component value for custom color
i | No | Intensity percentage for LED.
blink | No | Number of blinks if blink enabled
on | No | Duration of ON portion of blink (ms)
off| No | Duration of OFF portion of blink (ms)
start|No| Blink mode
lut |No | Selects active LED LUT to use when ledctl disabled.

<aside class="success">

</aside>


### led_userlut1, (led_userlut2)

This command allows the customization of internal LED LUT entries to use in place of the default factory LED lookup table.

```
+set,led_userlut1:3,color:orange,i:100,blink:4,on:1000,off:2000,mode:1
```
​
> The above command configures an entry using the predefined color orange of 100% intensity, for a minimum of 4 blinks with an ON duration of 1000ms and OFF duration of 2000ms, storing it at USER_LUT_1[3] and returns
​
```ascii
+rsp,3,color:orange,i:100,blink:4,on:1000,off:2000,start:1
```

Most parameters are optional and those not included in the command will allow the controller to use the existing values for those settings.

### command

`+set,led_userlut1:<parameters>`

### +set Parameters

Parameter | Required | Description
--------- | -------- | -----------
led_userlut1 | Yes |LED LUT identifier and array index
color | Yes | Predefined color name or "custom"
r | No | Red component value for custom color
g | No | Green component value for custom color
i | No | Intensity percentage for LED.
blink | No | Number of blinks if blink enabled
on | No | Duration of ON portion of blink (ms)
off| No | Duration of OFF portion of blink (ms)
start|No| Blink mode

<aside class="success">

</aside>
