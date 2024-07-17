How-to setup ssh connectivity for up to 100 sessions on an IoT Device via IoT-Hub and [Azure Device Streams](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-device-streams-overview). 


## create an IoT device

You can use this service today for IoT Hubs in regions that support this preview feature [IoT-Hub regions](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-device-streams-overview#regional-availability). Go to your IoT-Hub in a supporting region and create a new device. If your device is already registered, you can use this existing registration. If you are working with an IoT-Edge device, you need to create an additional registration, with a new device id as a simple device. 

Copy the Primary Connection String and save it somewhere, you will need it later.

## installation on a Linux device

Proceed with the following steps on your Linux device 

```bash
git clone -b DeviceStreaming-PersistentProxy https://github.com/syed/azure-iot-sdk-c.git

cd azure-iot-sdk-c
git submodule update --init

mkdir cmake
cd cmake

# in case OpenSSL lib is missing. 
apt-get install libssl-dev 

# in case curl lib is missing. 
apt-get install libcurl4-openssl-dev

cmake ..  

#in case of "uuid/uuid.h: No such file or directory"
apt-get install uuid-dev

make -j 1 

cd ~/azure-iot-sdk-c/iothub_client/samples/iothub_client_c2d_streaming_proxy_sample
vi iothub_client_c2d_streaming_proxy_sample.c 

```

go to line 70 (in vi you can do this by pres `:70`) and edit 
```cpp 
static const char* connectionString = "[device connection string]";
```
and update this with your Primary Connection String you have noted in section ["create an IoT device"](#create-an-IoT-device)

```bash
cd ~/azure-iot-sdk-c/cmake/iothub_client/samples/iothub_client_c2d_streaming_proxy_sample
make -j 1 
./iothub_client_c2d_streaming_proxy_sample
```

now you are able to connect with the [client](#setup-the-client-proxy) for test purpose

## if you like to run it as a service

```bash
mkdir ~/bin 
cp ./iothub_client_c2d_streaming_proxy_sample ~/bin/iot-ssh-client
sudo vi /etc/systemd/system/iot-ssh-client.service
```

insert into `/etc/systemd/system/iot-ssh-client.service`


	[Unit]
	Description=IoT-ssh-Client
	
	[Service]
	ExecStart=/home/<youruserid>/bin/iot-ssh-client
	
	[Install]
	WantedBy=multi-user.target

and run the following steps to install the daemon

```bash
sudo systemctl daemon-reload
sudo systemctl start iot-ssh-client.service
sudo systemctl enable  iot-ssh-client.service
systemctl status iot-ssh-client.service
```

## setup the client proxy

To connect to your IoT device via port 22, you need to run a service on your local machine or a machine inside your local network. Microsoft have provided some example code for node.js and C#. With this, you connect using the Service SDK via the IoT Hub to your device. 

This section describes how to do this with node.js. As a first step download and install the latest version of https://nodejs.org, or install nodejs with your Linux installer. 

Download the [device streams service package](https://github.com/syed/ssh-via-device-streams/blob/main/device-streams-service/device-streams-service.zip) and extract it to a location of your choice.

Go to your IoT Hub in the Azure portal and choose in the left menu "Shared access policies" in the "Security settings" section. Then choose "Manage shared access policies" and copy the primary connection string for the "service" Policy Name.

### for Windows 

create a small script like `open_proxy.cmd`

	SET IOTHUB_CONNECTION_STRING=<your service primary connection string> 
	SET STREAMING_TARGET_DEVICE=<the device id from the device you like to connect> 
	SET PROXY_PORT=<choose a port, what ever you like e.g. 2225> 

	node <path2proxy.js>\proxy.js

### for Linux 

create a small script like `open_proxy.sh`

	#!/bin/bash 
	
	export IOTHUB_CONNECTION_STRING="<your service primary connection string>"
	export STREAMING_TARGET_DEVICE="<the device id from the device you like to connect>"
	export PROXY_PORT="<choose a port, what ever you like e.g. 2225>" 

	node <path2proxy.js>/proxy.js

When you start your `open_proxy.cmd` or `open_proxy.sh` this will open the PROXY_PORT on this machine which you can then access with putty, or whichever ssh client you like. 

If you using Linux, you may also run the local proxy as a [service](#if-you-like-to-run-it-as-a-service).











