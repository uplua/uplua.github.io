---
layout: post
title: "Eighth Week Report"
date: 2019-07-28 13:47:56 +0530
categories: jekyll update
---

### Publish telemetry events through a mqtt broker

* Open a terminal window and authorize gcloud to access the google cloud platform with Google user credentials using the following command:
```
$ gcloud auth login
```

It will open up a window in your default browser and you just need to select the account with which you are working.

* Create a project or open an existing one, We already have one project, so I will use it:
```
$ gcloud config set project luabigquery
```
* Enable Cloud Pub/Sub:
```
$ gcloud services enable pubsub.googleapis.com
```

* Open this URL to enable billing 
```
$ open "https://console.cloud.google.com/iot/api?project=luabigquery"
```

* Enable Cloud IoT 
```
$ gcloud services enable cloudiot.googleapis.com
```

* Give permissions to Cloud IoT Core to publish messages on cloud Pub/Sub
```
gcloud projects add-iam-policy-binding luabigquery \
  --member=serviceAccount:cloud-iot@system.gserviceaccount.com \
  --role=roles/pubsub.publisher
```


Messages received from Cloud IoT Core will be automatically forwarded to a Pub/Sub topic. As you have guessed, we should create a Pub/Sub topic and a subscription:

```
export PUBSUB_TOPIC="lua12"
export PUBSUB_SUBSCRIPTION="lua22"

gcloud pubsub topics create $PUBSUB_TOPIC
gcloud pubsub subscriptions create $PUBSUB_SUBSCRIPTION --topic $PUBSUB_TOPIC
```
It is now time to create a device registry. A registry is like a bucket for your IoT devices. It allows you to group devices and set properties that they share, such as connection protocol, data storage location and Cloud Pub/Sub topics.

```
export REGISTRY_NAME="Nodes12"
export RESION_NAME="us-central1"

gcloud iot registries create $REGISTRY_NAME \
    --region=$REGION_NAME \
    --event-notification-config=topic=$PUBSUB_TOPIC \
    --enable-mqtt-config --enable-http-config
```

We have enabled both the MQTT and the HTTP Bridge.
The HTTP bridge is not required actually, but I enabled it in case we might develop something on REST API too.

Now, that we have a device registry, we can register a device in that registry:

Devices will authenticate to the bridge using a JWT ( JSON WEB TOKEN ). Cloud IoT Core uses public key authentication, and supports the RSA and Elliptic Curve algorithms. We will generate key pairs, and create a new device using the newly generated public key:
```
export DEVICE_NAME="esp32one"

#Generate an Elliptic Curve (EC) ES256 private / public key pair
openssl ecparam -genkey -name prime256v1 -noout -out ec_private.pem
openssl ec -in ec_private.pem -pubout -out ec_public.pem

#Create a new Cloud IoT device
gcloud iot devices create $DEVICE_NAME \
  --region=$REGION_NAME \
  --registry=$REGISTRY_NAME \
  --public-key="path=./ec_public.pem,type=es256"

```

Using the following python script, generate a JWT:

```
#!/usr/bin/env python

import datetime

import jwt

PROJECT_ID = 'luabigquery'
SSL_PRIVATE_KEY_FILEPATH = 'ec_private.pem'
SSL_ALGORITHM = 'ES256'


def create_jwt(project_id, private_key_file, algorithm):
    """Creates a JWT (https://jwt.io) to establish an MQTT connection.
        Args:
         project_id: The cloud project ID this device belongs to
         private_key_file: A path to a file containing either an RSA256 or
                 ES256 private key.
         algorithm: The encryption algorithm to use. Either 'RS256' or 'ES256'
        Returns:
            An MQTT generated from the given project_id and private key, which
            expires in 60 minutes. After 60 minutes, your client will be
            disconnected, and a new JWT will have to be generated.
        Raises:
            ValueError: If the private_key_file does not contain a known key.
        """

    token = {
        # The time that the token was issued at
        'iat': datetime.datetime.utcnow(),
        # The time the token expires.
        'exp': datetime.datetime.utcnow() + datetime.timedelta(minutes=60),
        # The audience field should always be set to the GCP project id.
        'aud': project_id
    }

    # Read the private key file.
    with open(private_key_file, 'r') as f:
        private_key = f.read()

    print('Creating JWT using {} from private key file {}'.format(algorithm, private_key_file))

    return jwt.encode(token, private_key, algorithm=algorithm)


print('JWT token:')
print(create_jwt(PROJECT_ID, SSL_PRIVATE_KEY_FILEPATH, SSL_ALGORITHM))
```
Now, We shall use mosquitto to test the MQTT bridge on our computer, before writing some microcontroller code.
We have to pass the JWT in the password field and 'unused' keyword in the username field.

```
# First, download root certificates
$ curl https://pki.goog/roots.pem > roots.pem

# Use Mosquitto to publish a message to the MQTT bridge
mosquitto_pub \
--host mqtt.googleapis.com \
--port 8883 \
--id projects/<your-project-id>/locations/<region-name>/registries/iotcore-registry/devices/<device-name> \
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

The log of above command would be:

```
┌───────────────┬─────────────────┬────────────────────────────────────┐
│      DATA     │    MESSAGE_ID   │             ATTRIBUTES             │
├───────────────┼─────────────────┼────────────────────────────────────┤
│ Hello lablua2 │ 643135129740493 │ deviceId=esp32one                  │
│               │                 │ deviceNumId=2698856991425448       │
│               │                 │ deviceRegistryId=Nodes12           │
│               │                 │ deviceRegistryLocation=us-central1 │
│               │                 │ projectId=luabigquery              │
│               │                 │ subFolder=                         │
└───────────────┴─────────────────┴────────────────────────────────────┘
```




