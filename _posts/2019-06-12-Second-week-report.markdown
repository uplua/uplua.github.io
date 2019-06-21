---
layout: post
title: "Second Week Report"
date: 2019-06-12 17:26:56 +0530
categories: jekyll update
---

### Google Cloud IoT Core - Concepts study
#### A thorough study of Google Cloud IoT Core concepts is required in order to develop Lua based APIs for it.

![ Google IoT Core APIs ](images/API_google_iot_core.png)

The main components of Cloud IoT Core are the device manager and the protocol bridges:

   * A device manager for registering devices with the service, so you can then monitor and configure them
   * Two protocol bridges (MQTT and HTTP) that devices can use to connect to Google Cloud Platform
![cloud-iot-overview](images/cloud-iot-overview3.png)
#### So we will see the following IoT Core Concepts in brief
* IoT Core Protocols
* IoT Core Security and Authentication
* IoT Core Devices, Configuration and State

#### IoT Core Protocols
Cloud IoT Core Supports two Protocols:

1. MQTT : Standard publish/subscribe protocol that is frequently used and supported by embedded devices and is also common in machine to machine protocols.
2. HTTP : A 'connectionless' protocol. Devices don't retain a connection to cloud IoT Core, they send and receive responses

 The Choice of protocol can be made using the following comparative study of both the protocols
![ HTTPvsMQTT ](images/HTTPvsMQTT.png)

#### IoT Core Security
Cloud IoT offers the following architecture to ensure the device to device or device to core service or core service to device security:
![ picture ](images/auth-sequence1.png)

The further Authentication is managed via:
![ picture2 ](images/auth-sequence2.png)

* The JWT is signed by private key with the authentication flow.
* MQTT bridge and device see the 'JWT' as the password in the "MQTT connect".
* MQTT bridge verifies the JWT against the device's public key.
* MQTT bridge accepts the connection and closes it when JWT expires.

#### IoT Core Devices, Configuration and state

* Device Registration :
    1. In order for a device to connect it must be first registered in the device manager.
    2. Device manager lets you create and configure device registries and the devices within them.
    3. Device manager can be accessed through the cloud platform console, gcloud command line interface or the REST-style API.

Device registry is a container of devices

     A registry is identified in the cloudiot.googleapis.com by its full name as :projects/{project-id}/locations/{cloud-region}/registries/{registry-id}

     A Device registry is configured with one or more cloud pub/sub topics to which telemetary events are published for all devices in that registry.
     
     A Single topic can be used to collect data accross all regions.

* Devices :

    1. When a device is created within a registry, it is defined as a cloudIoT core resource.
    2. Each device is identified by its full resource name: projects/{project-id}/locations/{cloud-region}/registries/{registry-id}/devices/{device-id} OR projects/{project-id}/locations/{cloud-region}/registries/{registry-id}/devices/{device-numeric-id}.



* Device Identifiers :

    1. A device-id defined by the user.
    2. A server-generated device numeric-id created by cloud iot core.
    3. Full path of the device.

* Device Metadata :

    We can define metadata for a device, such as hardware thumbprint, serial number, manufacturer information, or any other such attribute.

* Device Configuration :

    1. With Cloud IoT Core we can control a device by sending a device configuration to it.
    2. Device configuration is a user defined blob of data sent from Cloud IoT Core to a device.
    3. MQTT bridge and HTTP bridge are two configuration versions.


* Device State :

    1. Device State information captures the current status of the device, not the current environment.
    2. Devices can describe their state with an arbitrary user-defined blob of data sent from the device to the cloud.

##### The Information that needs to be sent to or from a device should not be stored as a device metadata because device metadata stays in the cloud.
##### The information should be in a device configuration if we need to send it to a device, or in device state data if we need to report it back to cloud iot core.
##### Using the device configuration one can :
    
    1. Send a device's expected state as a configuration.
    2. send a command to a device.




### Initial Github Repository and weblog
The github repository for the initial release of APIs in lua can be found
[here](https://github.com/uplua/lua-docs-samples)

Build a basic weblogging site using jekyll and github pages is also up and
[running](https://uplua.github.io)
