---
layout: post
title: "Final Report"
date: 2019-08-26 12:16:56 +0530
categories: jekyll update
---

### The Original Project

Originally, the project was aimed at the following:

* Providing a Proof of Concept in lua using a real device such as NodeMCU/RaspberryPi interfaced with the temperature sensor to send data over to the Google Cloud IoT Core.

![image](images/i1.png)

* Doing the reverse, sending the data to the device using the Google Cloud IoT Core.

![image2](images/i2.png)

*  Implement a bootloader code for devices. New Code is executed in the NodeMCU by the bootloader every time it is sent via some command automatically or manually.

![image3](images/i3.png)

* Develop Lua Versions to the APIs in Google Cloud IoT Core: cloudiot and cloudiotdevice.


### Problems with original project 

The project is being narrowed down to the following:

* Develop a set of programs in lua on a device ( might not be NodeMCU but an ubuntu machine ), which would connect the device to a remote MQTT broker and publish data to it ( i.e. temperature readings ).

* The broker in turn would send the data to the Google Cloud IoT Core via Google Cloud pub/sub service.

Reasons to narrow down the project:

* There are two ways to connect to Google Cloud IoT Core, namely the HTTP bridge and the MQTT bridge.

* The HTTP bridge requires a Bearer JWT token authorization, but that is not supported by NodeMCU libraries, and developing such huge libraries is currently out of the scope of this project.

* The MQTT bridge requires to be authenticated via a TLS v1.2 handshaking and cacerts file authentication with roots.pem , which is currently not supported by NodeMCU and developing a library for the same is also out of the scope of this project

* Connect to the broker from a NodeMCU device,  sending and receiving data through Google Cloud IoT Core.

Due to the above reasons we are limiting the project to develop a proof of concept via a MQTT broker.

### Current Proposal for implementation

#### Phase 1

Lua program, running on Lua interpreter, sends an integer value (temperature reading simulation on a device) to IoT Core and this data can be checked there through Big Query.

![images5](images/i5.png)

The code in Lua for this can be found [here](https://github.com/uplua/lua_to_google_iot_core_examples/blob/master/post_data_to_iot_core.lua). 

For Authorization at device level, a JWT is needed as the bearer token for authorization of REST api requests ( GET, POST, PUT, DELETE etc ) for google cloud platform.

JWT generation which is compatible with lua is a separate problem. But, as of now we are using a Python script to generate the JWT ( Json Web Token ).

JWT can be generated from a Python program and the code can be found [here](https://github.com/uplua/JsonWebToken/tree/master). This code makes use of LuaSocket library.

JWT script for google cloud iot core is a future project that  lablua can look to propose as a solid idea for google summer of code or other such opportunities, here is a link to the repo.

With slight changes, the Lua program will send data from a predefined loop table.

The code in Lua for this can be found [here](https://github.com/uplua/lua_to_google_iot_core_examples/blob/master/post_multiple_requests.lua).

#### Phase 2

Lua program, running on NodeMCU, through an MQTT broker Node-RED , sends an integer value (temperature reading simulation on a device) to IoT Core and this data can be checked there through Big Query.

![image4](images/i7.png)

To do this, an IDE like ESPlorer ( refer to the picture shown below ) is used :

![images6](images/i6.png)

The code in Lua can be found [here](https://github.com/uplua/lua_to_google_iot_core_examples/tree/master/mqtt_example)

With slight changes, the Lua program will send data from a predefined in the [github](https://github.com/uplua) respository.

The code in Lua for this can be found [here](https://github.com/uplua/lua_to_google_iot_core_examples/blob/master/mqtt_example/mqtt.lua)

The Complete source code can be found [here](https://github.com/uplua).

#### Further Improvements

##### Phase 3

* Through an external program interface, it will be possible to send, via IoT Core, data to a Lua program running on the Lua Interpreter, which will display the values received on the interpreter console.

* [TO DO] : To fetch data from google cloud iot core , the use of rest api is required but since small devices aren’t powerful enough to comply with large rest API interfaces of google, so this part is further extension of the project.

* With slight changes, a looping Lua program may change the frequency for sending data from a table, based on the value received from the external program interface.

* [TO DO] : To fetch data from google cloud iot core , the use of rest api is required but since small devices aren’t powerful enough to comply with large rest API interfaces of google, so this part is further extension of the project.

##### Phase 4

* Through an external program interface, it will be possible to send, via IoT Core and MQTT Broker, data to a Lua program running on  NodeMCU.
To do this, an IDE like ESPlorer will be used.
Received values will be displayed in IDE console

* With slight changes, a looping Lua program running in NodeMCU may change the frequency for sending data from a table, based on the value received from the external program interface.

* [TO DO] There a very limited supporting libraries in nodemcu that can make this happen, so it is out of the scope of a gsoc project as of now, please have a look at similar library in [javascript](https://github.com/mqttjs/MQTT.js).

