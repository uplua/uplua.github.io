---
layout: post
title: "Third Week Report"
date: 2019-06-21 14:26:56 +0530
categories: jekyll update
---

### Create a Cloud IoT device registry and register a device

Before starting, ensure these steps are done:

1. In the GCP console, go to the [ Manage Resources ](https://console.cloud.google.com/cloud-resource-manager?authuser=1&_ga=2.171265312.-1264177381.1544934535) page and select/create a project.
2. Make Sure Billing is enabled for your Google Cloud Platform project.
3. Enable the Cloud IoT core and Cloud Pub/Sub APIs.

Prerequesites:

1. [Install and initialise the Cloud SDK](https://cloud.google.com/sdk/docs/?authuser=1).
2. Setup a [Nodejs development environment](https://cloud.google.com/nodejs/docs/setup?authuser=1)
3. Alternatively, we can use [Google cloud shell](https://cloud.google.com/shell/docs/starting-cloud-shell?authuser=1) which comes with cloud SDK and Nodejs already installed.

#### Create a device registry

1. Go to the Google cloud IoT Core page in [GCP console](https://console.cloud.google.com/iot?authuser=1&_ga=2.147270327.-1264177381.1544934535).
2. Click Create Registry and enter a name e.g. 'my-registry' for the Registry ID.
3. Select your region from this [region-list](https://cloud.google.com/iot/docs/requirements?authuser=1#cloud_regions).
4. Select MQTT/HTTP or both as per your requirements for the protocol.
5. In the 'Default telemetary topic' dropdown list, select 'Create a topic'.
6. In the 'Create a topic' dialog, enter a name e.g. 'my-device-events' in the name field.
7. Click 'Create' in the 'Create a topic' dialog.
8. The 'Device state topic' and 'Certificate value' fields are optional, so you may leave them blank.
9. Click 'Create ' on the Cloud IoT Core page.

You've just created a device registry with a Cloud Pub/Sub topic for publishing device telemetary events.

#### Generate a device key pair

Open a terminal and run the following command to create an RS256 key:

```
$ openssl req -x509 -newkey rsa:2048 -keyout rsa_private.pem -nodes -out rsa_cert.pem -subj "/CN=unused"

```

In the following section, you'll add a device to the registry and associate the public key with the device.

#### Add a device to the registry

1. On the registries page, select 'my-registry'.
2. Select the 'Devices' tab and click 'Create a Device'.
3. Enter a name for Device ID e.g.'my-device'.
4. Select 'Allow' for 'Device communication'.
5. Add the public key information to the Authentication fields:
    * Copy the contents of rsa_cert.pem 
    * Select 'RS256_X509' for the 'Public Key Format'.
    * Paste the public key in the 'Public key value' box.
    * Click 'Add' to associate the 'RS256_X509' key with the device.
6. The 'Device Metadata' field is optional, leave it blank.
7. click 'Add'.

You've just added a device to your registry. The RS256_X509 key appears on the 'Device details' page for your device.

#### Run a nodejs sample to connect a virtual device and view telemetry

* Get the Cloud IoT Core Node.js samples from Github. The Cloud IoT Core Samples are in the IoT directory.

```
$ git clone https://github.com/GoogleCloudPlatform/nodejs-docs-samples

```
* In the cloned repository, navigate to the iot/mqtt_example directory. You'll complete the rest of these steps in this directory:

```
$ cd nodejs-docs-samples/iot/mqtt-example

```

* Copy the private key you created in the previous section to the current directory.

```
$ cp ../../../rsa_private.pem
```

* Install the Node.js dependencies:

```
$ npm install
```

* Run the following command to create a subscription to the registry's Pub/Sub topic, substituting your project ID:

```
$ gcloud pubsub subscriptions create \
     projects/PROJECT_ID/subscriptions/my-subscription \
     --topic=projects/PROJECT_ID/topics/my-device-events
```

* Run the following command to connect a virtual device to Cloud IoT Core using the MQTT bridge, substituting your project ID:

```
$ node cloudiot_mqtt_example_nodejs.js \
     mqttDeviceDemo \
     --projectId=PROJECT_ID \
     --cloudRegion=REGION \
     --registryId=my-registry \
     --deviceId=my-device \
     --privateKeyFile=rsa_private.pem \
     --numMessages=25 \
     ---algorithm=RS256
```
The output shows that the sample device is publishing messages to the telemetary topic. Twenty-five messages are published.

* Run the following command to read the messages published to the telemetary topic, substituting your project ID:

```
$ gcloud pubsub subscriptions pull --auto-ack \
    projects/PROJECT_ID/subscriptions/my-subscription

```
* Repeat the 'subscriptions pull' commmand to view additional messages.


> To see the official documentation from where this post is inspired, please check : [ quickstart google iot core ](https://cloud.google.com/iot/docs/quickstart?authuser=1)

### Create a linux ( Ubuntu 18.04 ) Virtual Machine ( VM ) in Google Cloud Platform

Prerequesites :

1. Ensure that you have created a project in Google cloud platform before starting this tutorial blog.

2. Make sure that billing is enabled for your Google Cloud Platform project.

#### Create a Virtual Machine ( VM ) instance 

1. In the GCP Console, go to the 'VM Instances' page.
2. Click 'Create Instance'.
3. In the 'Boot disk' section, click 'Change' to begin configuring your boot disk.
4. On the 'OS images' tab, choose 'Ubuntu 18.04'.
5. Click 'Select'.
6. In the 'Firewall' section, select 'Allow HTTP traffic'.
7. Click 'Create' to create the instance.

![ reference image ](images/gcp_vm_instance.png)

Allow a short time for the instance to start up. Once ready, it will be listed on the VM instances page with a green status icon.

#### Connect to your Instance

1. In the GCP Console, go to the 'VM Instances' page.
2. In the list of virtual instances, click 'SSH' in the row of the instance that you want to connect to.
![ refer ](images/establish-ssh-connection-1.png)

3. A Terminal window would open, which will let you access the VM instance that you have just created.

### Running a basic Apache web server

Prerequesites :

1. Create a Linux Instance using the above steps or follow this [guided blog](https://cloud.google.com/compute/docs/quickstart-linux?authuser=1).
2. While creating your VM instance, scroll to the Firewalls section and check the 'Allow HTTP Traffic' box. Checking this box enables the External IP address.

#### Install Apache

1. Use Debian package manager to install apache2 package.
```
$ sudo apt-get update && sudo apt-get install apache2 -y
```
2. Overwrite the Apache Web server default webpage with the following command:
```
$ echo '<!doctype html><html><body><h1>Hello World!</h1></body></html>' | sudo tee /var/www/html/index.html
```

#### Test your server

Test that your instance is serving traffic on its external IP:
1. Go to VM Instances page in the Google Cloud Platform Console.
2. Copy the external IP for your instance under the 'EXTERNAL_IP' column.
3. In a browser, navigate to 'http://[EXTERNAL_IP]'.

You should now see the "Hello World!" page.

#### Create a subscription to the registry's Pub/Sub topic, substituting your project ID:

```
$ gcloud pubsub subscriptions create \
    projects/PROJECT_ID/subscriptions/my-subscription \
    --topic=projects/PROJECT_ID/topics/my-device-events

```

Run the following command to read the messages published to the telemetry topic, substituting your project ID:

```
$ gcloud pubsub subscriptions pull --auto-ack \
    projects/PROJECT_ID/subscriptions/my-subscription
```

Repeat the subscriptions pull command to view additional messages.




 

