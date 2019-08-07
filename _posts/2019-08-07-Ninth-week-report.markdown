---
layout: post
title: "Ninth Week Report"
date: 2019-08-07 22:32:56 +0530
categories: jekyll update
---

### Setting up a connection between the NodeMCU and Google Cloud IoT Core using a broker

Why?

Since, Google Cloud IoT Core supports only TLS v1.2 and NodeMCU firmware supports TLS upto v1.1 only. So,We have to push data from the NodeMCU to a broker and then use that broker to publish the same data to Google Cloud IoT Core.

Which MQTT broker to use?
We can use any broker as per our convenience but due to fine compatibility 
of Cloud MQTT with NodeMCU, we will be using Cloud MQTT broker instance.

#### Set up a Cloud MQTT broker instance

* Go to the website of [CloudMQTT](https://www.cloudmqtt.com)
* Register with an account.
* Now create a new instance i.e. temperature_sensor_01 with default free plan 'cat'.
* Navigate to the details of the new instance created. It should look like this:

![CloudMQTT](images/cloudmqtt.png)

#### Device side ( NodeMCU ) code to send data to CloudMQTT

* Open ESPlorer IDE with the MQTT package alongwith default packages flashed.
* Write init.lua to connect to a local Wi-Fi hotspot.

```
--init.lua
station_cfg={}
station_cfg.ssid="Gaurav"  -- Enter SSID here
station_cfg.pwd="kumar123"  -- Enter password here


wifi.setmode(wifi.STATION)  -- set wi-fi mode to station
wifi.sta.config(station_cfg)-- set ssid&pwd to config
wifi.sta.connect(1)         -- connect to router

ip = wifi.sta.getip()       -- IP address of the connected access point 

print(ip)
```

Use the following code to send data to cloudmqtt:

```
-- initiate the mqtt client and set keepalive timer to 120sec
mqtt = mqtt.Client("client_id", 120, "username", "password")

mqtt:on("connect", function(con) print ("connected") end)
mqtt:on("offline", function(con) print ("offline") end)

-- on receive message
mqtt:on("message", function(conn, topic, data)
  print(topic .. ":" )
  if data ~= nil then
    print(data)
  end
end)

mqtt:connect("hostname", port, 0, function(conn)
  print("connected")
  -- subscribe topic with qos = 0
  mqtt:subscribe("my_topic",0, function(conn)
    -- publish a message with data = my_message, QoS = 0, retain = 0
    mqtt:publish("my_topic","my_message",0,0, function(conn)
      print("sent")
    end)
  end)
end)

```
