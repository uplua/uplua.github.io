---
layout: post
title: "Fifth Week Report"
date: 2019-07-07 16:27:56 +0530
categories: jekyll update
---

### NodeMCU Wi-Fi with ESPlorer IDE

#### Introduction

* NodeMCU development Board is based on ESP8266 system on chip which combines the feature of Wi-Fi and microcontroller to satisfy the need of prototyping IoT applications within less time and with a few lines of Lua Scripts.

* Wi-Fi is wireless LAN technology used for short distance wireless networking applications. It is based on IEEE 802.11 standards.

* NodeMCU firmware provides Event driven APIs for network applications. NodeMCU Wi-Fi networking can be used to connect, fetch or upload data to internet.

![NodeMCU-WiFi](images/NodeMCU_WiFi_Mode.png)

#### NodeMCU has four Wi-Fi modes as

* Station (STA) Mode:
In this mode NodeMCU joins the existing networks. The NodeMCU connects the existing wi-fi router. This provides the internet access to the NodeMCU through the wi-fi router.

* Access Point (AP) Mode:
In this mode, NodeMCU creates its own network and others can join this network. Here, it has local IP address with which other devices can connect to it. NodeMCU assigns next available IPs to the other devices.

* Station + Access Point (BOTH) Mode:
This is the mode, where it creates its own network while at the same time being joined to another existing network.

* Wi-Fi OFF Mode:
In this mode, wi-fi remains OFF.

As shown in the figure above, NodeMCU (AP mode) creates wireless LAN to which other wi-fi enabled devices like PC/laptop, smart mobile phones, NodeMCU (STA mode) etc. can connect. NodeMCU (AP mode) assigns local IP address to each device connected to it.

#### NodeMCU Wi-Fi functions

* <strong><em>wifi.setmode() : </em></strong> 
    This function is used to configure the Wi-Fi mode to use. NodeMCU can run in one of four below Wi-Fi modes:
    * Station mode
    * Access point (AP) mode
    * Station + AP mode
    * WiFi off
 When using the combined Station + AP mode, the same channel will be used for both networks as the radio can only listen to a single channel.

<strong><em>Note : </em></strong> WiFi configuration will be retained until changed, even if the device is turned off.

<strong><em>Syntax : </em></strong> ``` wifi.setmode(mode[, save]) ```


<strong><em>Parameters : </em></strong>

1. Mode : <strong>wifi.STATION</strong>: station mode
          <strong>wifi.SOFTAP</strong>: Access Point mode, default IP:192.168.4.1
          <strong>wifi.STATIONAP</strong>: Both AP and station mode

2. Save : Choose whether to save wifi mode to flash or not. If true then configuration will be retained even after power cycle/reset (Default).


<strong><em>Returns : </em></strong> current mode after setup

<strong><em>Example : </em></strong>
``` $ wifi.setmode(wifi.STATION) ```




* <strong><em>wifi.sta.config() : </em></strong>
    This function sets the WiFi station configuration.

<strong><em>Syntax : </em></strong> ``` wifi.sta.config(station_config) ```
<strong><em>Parameters : </em></strong>

<strong><em>station_config</em></strong> : table containing configuration data for station

1. ``` ssid ``` : SSID string which is less than 32 bytes.
2. ``` pwd ``` : Password string which is 0-64. Empty string indicates an open WiFi access point.
3. ``` auto : true ```  to enable auto connect and connect to access point, hence with auto enable there's no need to call ``` wifi.sta.connect(). false ``` to disable auto connect and remain disconnected from access point. defaults to true.
4. ``` bssid ``` :  BSSID string that contains the MAC address of the access point (optional). You can set BSSID if you have multiple access points with the same SSID. ex "DE:A1:A6:52:F1:ED"
5. ``` save ``` : Save station configuration to flash. If true,then configuration will be retained even after power cycle/reset (Default). If false, then configuration will not be retained after power cycle/reset.
6. ``` Event callbacks ``` : [Refer to this link](https://www.electronicwings.com/nodemcu/nodemcu-wi-fi-with-esplorer-ide)

<strong><em>Returns : </em></strong>It returns ``` true ``` for success or ``` false ``` for failure.

<strong><em>Example : </em></strong>

```
station_cfg = {}
station_cfg.ssid = "AccessPointA"
station_cfg.pwd = "password"
wifi.sta.config(station_cfg)
```

* <strong><em>wifi.sta.connect() : </em></strong>
    This function is used to connect to the configured AP is station mode. This function needs to called, only if auto-connect was disabled in wifi.sta.config().

<strong><em>Syntax : </em></strong> ``` wifi.sta.connect([connected_cb]) ```
<strong><em>Parameters : </em></strong>

``` connected_cb ``` : Callback to execute when station is connected to an access point.Items returned in the table:

* SSID : ssid of access point
* BSSID : bssid of access point
* channel : The channel the access point is on. 

* <strong><em>wifi.sta.autoconnect() : </em></strong>
    This auto connects to AP in station mode.

<strong><em>Syntax : </em></strong> ``` wifi.sta.autoconnect(auto) ```

<strong><em>Parameters : </em></strong>

``` auto ``` : set 0  to disable auto connecting and 1 to enable auto connecting

<strong><em>Returns : </em></strong> null

<strong><em>Example : </em></strong> ``` wifi.sta.autoconnect(1) ```

* <strong><em>wifi.sta.suspend() : </em></strong>
This function is used to suspend Wifi to reduce current consumption.

<strong><em>Syntax : </em></strong> ``` wifi.suspend({duration[, suspend_cb, resume_cb, preserve_mode]}) ```

<strong><em>Parameters : </em></strong>

1. ``` duration ``` : Suspend duration in microseconds(μs). If a suspend duration is 0 specified, suspension will be indefinite (Range: 0 or 50000 - 268435454 μs (0:4:28.000454))
2. ``` suspend_cb ``` : Callback to execute when WiFi is suspended.
3. ``` resume_cb ``` : Callback to execute when WiFi wakes from suspension.
4. ``` preserve_mode ``` : preserve current WiFi mode through node sleep.

* <strong><em>wifi.sta.gethostname() : </em></strong>
    Use to get current station hostname.

<strong><em>Syntax : </em></strong> ``` wifi.sta.gethostname() ```

<strong><em>Returns : </em></strong> currently configured hostname

<strong><em>Example : </em></strong> ``` print("Current hostname is:\"..wifi.sta.gethostname().."\") ```

* <strong><em>wifi.sta.getip() : </em></strong>
    Use to get IP address, netmask, and gateway address in station mode.
<strong><em>Syntax : </em></strong> ``` wifi.sta.getip() ```

<strong><em>Returns : </em></strong>
It returns ``` IP address, netmask, gateway ``` address as string, e.g. "192.168.0.111".

<strong><em>Example : </em></strong> ``` print(wifi.sta.getip()) ```

* <strong><em>wifi.ap.config() : </em></strong>
    This function sets SSID and password in AP mode. Make the password at least 8 characters long! If you don't it will default to no password and not set the SSID! It will still work as an access point but use a default SSID like e.g. NODE_9997C3
<strong><em>Syntax : </em></strong> ``` wifi.ap.config(cfg) ```

<strong><em>Parameters : </em></strong>
1. ```cfg ``` : table to hold configuration
2. ```ssid ``` : wi-fi SSID chars 1-32
3. ```pwd ``` : wi-fi password chars 8-64
4. ```auth ``` : authentication method, one out of : ``` wifi.OPEN (default), wifi.WPA_PSK, wifi.WPA2_PSK, wifi.WPA_WPA2_PSK ``` 
5. ```channel ``` : channel number 1-14 default = 6
6. ```hidden ``` : false = not hidden, true = hidden, default = false
7. ```max ``` : maximum number of connections 1-4 default = 4
8. ``` beacon ``` : beacon interval time in range 100-60000, default = false
9. ``` save ``` : save configuration to flash.true configuration will be retained through power cycle (Default). false configuration will not be retained through power cycle.
10. ``` Event callbacks ``` : [refer to the link](https://www.electronicwings.com/nodemcu/nodemcu-wi-fi-with-esplorer-ide)

<strong><em>Returns :</em></strong> ``` true ``` if Success else ``` false ``` for Failure.
<strong><em>Example :</em></strong>
```
cfg = {}
cfg.ssid="myssid"
cfg.pwd="mypassword"
wifi.ap.config(cfg)
```
* <strong><em>wifi.ap.setip() : </em></strong>
This function sets IP address, netmask and gateway address in AP mode.

<strong><em>Syntax : </em></strong> ``` wifi.ap.setip(cfg) ```
<strong><em>Parameters : </em></strong>

 ``` cfg ``` : The table contain IP address, netmask, and gateway

<strong><em>Returns : </em></strong> ``` true ``` if successful, ``` false ``` otherwise
<strong><em>Example : </em></strong>

```
cfg = {
    ip="192.168.1.1",
    netmask="255.255.255.0",
    gateway="192.168.1.1"
}
wifi.ap.setip(cfg)
```
* <strong><em>wifi.ap.setmac() : </em></strong>
This function sets MAC address in AP mode.

<strong><em>Syntax : </em></strong> ``` wifi.ap.setmac(mac) ```

<strong><em>Parameters : </em></strong>

``` mac ``` : MAC address in byte string, for example "AC-1D-1C-B1-0B-22"
Returns: ```true``` if success, ```false``` otherwise

<strong><em>Example : </em></strong>
```
print(wifi.ap.setmac("AC-1D-1C-B1-0B-22"))
```

















