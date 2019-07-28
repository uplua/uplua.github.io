---
layout: post
title: "Seventh Week Report"
date: 2019-07-21 23:19:56 +0530
categories: jekyll update
---

### Using Cloud IoT Core to connect a microcontroller (ESP32 / ESP8266) to the Google Cloud Platform

#### Google Cloud IoT Core

Connecting a single device to a remote service is quite an easy thing.
However, when you start having thousands of devices, you need a secure and cost-saving solution that helps you manage all your devices easily and dealing with all the received data.

Cloud IoT Core is a fully managed service that allows you to easily and securely connect large fleets of devices directly to the Google Cloud. By connecting microcontrollers to the cloud, you potentially transform these limited devices into hugely smart products.

We have to think of Cloud IoT Core as an entry point to the Google Cloud Platform for IoT devices.

It is composed of two main components, a device manager and a protocol bridge:

* The device manager helps establishing the identity of a device and provides the mechanism for authenticating a device when connecting. It also maintains a logical configuration of each device and can be used to remotely control any device from the cloud.
* The protocol bridge is the way for your device to access the Google Cloud through standard protocols such as MQTT and HTTP, so you can use your existing devices with minimal firmware changes.

The protocol bridge publishes all device telemetry to cloud Pub/Sub

![pipeline](images/espPipe.png)

Once the data is in Pub/Sub, It is in the Google Cloud, and you can consume it the way you want.

#### Connecting the microcontroller to Cloud IoT Core

Before using Cloud IoT core, we first have to create a Google Cloud project, and enable Cloud IoT Core and Pub/Sub APIs. This can be done either from the [web console](http://console.cloud.google.com/), or using the [gcloud](https://cloud.google.com/sdk/) command-line tool, which we will be using (If you have any issue, refer to the [quick start guide](https://cloud.google.com/iot/docs/quickstart))

```
export PROJECT_ID="luabigquery"

# Authorize gcloud to access the Cloud Platform with Google user credentials
gcloud auth login

# Create a new project
gcloud projects create $PROJECT_ID

# Set default project
gcloud config set project $PROJECT_ID

# Enable Cloud Pub/Sub
gcloud services enable pubsub.googleapis.com

# Open this URL to enable billing (required for Cloud IoT)
open "https://console.cloud.google.com/iot/api?project=$PROJECT_ID"

# Enable Cloud IoT
gcloud services enable cloudiot.googleapis.com

# Give permission to Cloud IoT Core to publish messages on Cloud Pub/Sub
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member=serviceAccount:cloud-iot@system.gserviceaccount.com \
  --role=roles/pubsub.publisher

```
Messages received from Cloud IoT Core will be automatically forwarded to a Pub/Sub topic. As you have guessed, we should create a Pub/Sub topic, and a subscription.
```
export PUBSUB_TOPIC="topic1"
export PUBSUB_SUBSCRIPTION="subs1"

gcloud pubsub topics create $PUBSUB_TOPIC
gcloud pubsub subscriptions create $PUBSUB_SUBSCRIPTION --topic $PUBSUB_TOPIC

```
It is now time to create a device registry. A registry is like a bucket for your IoT devices.
It allows you to group devices and set properties that they all share, such as connection protocol, data storage location, and Cloud Pub/Sub topics.

```
export REGISTRY_NAME="registry1"
# Choose your region: https://cloud.google.com/compute/docs/regions-zones/
# Valid cloud regions are [asia-east1, europe-west1, us-central1]
export REGION_NAME="us-central1"

# Create Cloud IoT registry specifying Cloud Pub/Sub topic name
gcloud iot registries create $REGISTRY_NAME \
  --region=$REGION_NAME \
  --event-notification-config=topic=$PUBSUB_TOPIC \
  --enable-mqtt-config --enable-http-config

```

We have enabled both the MQTT and HTTP bridge.
The HTTP bridge is only enabled here for the example. We actually don't need it for the project.

The registry has been created and is visible in the [web console](https://console.cloud.com/iot/registries):
![image_registry](images/registries.png)

Now that we have a device registry, we can register a device ( our esp32/esp8266 ) in that registry:

Devices will authenticate to the bridge using a Json Web Token (JWT).
Cloud IoT Core uses public key authentication, and supports the RSA and Elliptic Curve algorithms.
We will generate key pairs, and create a new device using newly generated public key:

```
export DEVICE_NAME="esp32"

# Generate an Eliptic Curve (EC) ES256 private / public key pair
openssl ecparam -genkey -name prime256v1 -noout -out ec_private.pem
openssl ec -in ec_private.pem -pubout -out ec_public.pem

# Create a new Cloud IoT device
gcloud iot devices create $DEVICE_NAME \
  --region=$REGION_NAME \
  --registry=$REGISTRY_NAME \
  --public-key="path=./ec_public.pem,type=es256"

```
Our device is created and visible in the web console:
![device_image_registry](images/devices.png)

#### Connecting to the HTTP/MQTT bridges

One of the advantages of Cloud IoT Core is that you can connect constraint devices such as MCUs using MQTT or HTTP.
This is very convenient, as size is limited on microcontrollers and often, you can’t afford to include or create a proprietary heavy library or SDK to your project.
Since MQTT and HTTP are industry standard protocols, there are great chances that those are already supported by your devices.
In this section, we will try both bridges from our computer, using curl( for the HTTP bridge) and mosquitto (for the MQTT bridge).

#### HTTP Bridge

To send telemetry events to the Cloud using the HTTP bridge, you have to send a POST request containing base64 encoded data to a given URL.
The request must contain an ```authorization``` header with a valid JWT generated using your device's private key
(A script in Lua is to be written for this). But temporarily we can use up the script written in [python](https://github.com/Nilhcem/esp32-cloud-iot-core-k8s/tree/master/04-generate-jwt/) for the same.

```
# Generate the JWT
$ ./main.py
b'<your-jwt-token>'

# Encode data in base64
$ echo "hello" | base64
aGVsbG8K

# Send the POST request
$ curl -X POST \
  -H 'authorization: Bearer <your-jwt-token>' \
  -H 'content-type: application/json' \
  -H 'cache-control: no-cache' \
  --data '{"binary_data": "aGVsbG8K"}' \
  'https://cloudiotdevice.googleapis.com/v1/projects/<your-project-id>/locations/<region-name>/registries/iotcore-registry/devices/<device-name>:publishEvent'
{}

```
Lua Script to POST data to Google Cloud IoT Core Pub/Sub topic using the LuaSocket Library through the Lua Interpreter.

```
  local http = require"socket.http"
  local ltn12 = require"ltn12"

  local reqbody = "aGVsbG8K"
  local respbody = {} -- for the response body

  local result, respcode, respheaders, respstatus = http.request {
      method = "POST",
      url = "https://cloudiotdevice.googleapis.com/v1/projects/luabigquery/locations/us-central1/registries/registry1/devices/esp32:publishEvent",
      source = ltn12.source.string(reqbody),
      headers = {
          ["content-type"] = "aplication/json",
          ["authorization"] = "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NiJ9.eyJpYXQiOjE1NjM3MzQxMTEsImV4cCI6MTU2MzczNzcxMSwiYXVkIjoibHVhYmlncXVlcnkifQ.0uhqGMVlByD3XI-EOuJdHqsnN3GQALO_CUXyNSFvst7OXFfHViBf4U7vHZ9CfsLK9FiU_ddt6uN9cQ8UXE6daw",
          ["cache-control"] = "no-cache"  
      },
      sink = ltn12.sink.table(respbody)
  }
  -- get body as string by concatenating table filled by sink
  respbody = table.concat(respbody)
```

#### MQTT Bridge

We will using Mosquitto to test the MQTT bridge on our computer before writing some microcontroller code.
This time, the JWT has to be passed in the password field; the username won’t be checked as all the information is in the JWT, so we can set the username to any value (such as "unused" for instance).

IoT core does all the encrypted communication through TLS1.2 or higher to ensure that all of your devices are secure when they’re talking to the cloud.
Make sure to specify the CA file, by downloading roots certificates and providing those to your MQTT client:

```
# First, download root certificates
$ curl https://pki.goog/roots.pem > roots.pem

# Use Mosquitto to publish a message to the MQTT bridge
mosquitto_pub \
--host mqtt.googleapis.com \
--port 8883 \
--id projects/luabigquery/locations/us-central1/registries/registry1/devices/esp32 \
--username unused \
--pw "<your-jwt-token>" \
--cafile ./roots.pem \
--tls-version tlsv1.2 \
--protocol-version mqttv311 \
--debug \
--qos 1 \
--topic /devices/<device-name>/events \
--message "Hello MQTT"

```
Again, you can pull the MQTT subscription to verify that it works:

```
$ gcloud pubsub subscriptions pull --auto-ack $PUBSUB_SUBSCRIPTION --limit=1
```
This will generate the following output log:

```
┌────────────┬─────────────────┬─────────────────────────────────────┐
│    DATA    │    MESSAGE_ID   │              ATTRIBUTES             │
├────────────┼─────────────────┼─────────────────────────────────────┤
│ Hello MQTT │ 410700146251251 │ deviceId=esp32                      │
│            │                 │ deviceNumId=3069493838977348        │
│            │                 │ deviceRegistryId=registry1          │
│            │                 │ deviceRegistryLocation=us-central11 │
│            │                 │ projectId=luabigquery               │
│            │                 │ subFolder=                          │
└────────────┴─────────────────┴─────────────────────────────────────┘

```
#### Connecting the microcontroller ( esp32/esp8266 ) to Cloud IoT Core

We will be using the MQTT bridge, which means that our microcontroller code will need an MQTT client, and a way to generate a JWT from a given private key.
We could write some code for that, but hopefully some very lightweight libraries and SDKs already exist to speed up and simplify the integration between a device and Cloud IoT Core.


