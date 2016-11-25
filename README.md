#Setting up a 6loWPAN gateway

##Introduction
A step-by-step guide on setting up a 6loWPAN gateway on a host machine. The gateway uses various tools in order to provide a wide spectrum of functionalities. Beyond the standard 802.15.4 interface, the gateway offers proxying between HTTP and CoAP protocols and communication between IPv4 and IPv6 space. Moreover, it implements a node-RED flow which enables pushing data to *relayr.* cloud and of course allows the user to build custom flows, engaging with different web-services. 

##Requirements

1. A gateway host machine with access to the Internet, like RPi.
2. One Zolertia [Remote](https://github.com/Zolertia/Resources/wiki/RE-Mote) or [Firefly](https://github.com/Zolertia/Resources/wiki/Firefly) mote which will provide the 802.15.4 interface to the gateway.
3. Tools for compiling and flashing Contiki code. This  step is required for flashing the edge-router and is optional for flashing motes of the wireless network. Step-by-step instructions on installing on various platforms can be found [here](https://github.com/Zolertia/Resources/wiki/Toolchain-and-tools).
4. Apache Maven in order to compile californium-proxy. Detailed steps on installing can be found [here](https://maven.apache.org/install.html).

##Deployment
###Border Router
Follow these steps to get a IPv6 edge router Up&Running:

1. 	```cd``` to Contiki ```border-router/``` directory:

	```shell
	cd contiki/examples/ipv6/rpl-border-router
	``` 
2. Compile the border-router for your device:

	```shell
	make TARGET=zoul border-router
	```
3. Plug the board and flash the code:

	```shell
	make TARGET=zoul border-router.upload
	```
4. Change to Contiki ```tools/``` directory and make *tunslip6*:
	
	```shell
	cd contiki/tools
	make tunslip6
	```
5. Create an IPv6 network interface:
	
	```shell
	sudo ./tunslip6 -s /dev/ttyUSB0 -t tun0 fd00::1/64
	```
	notice that the device path may vary depending on your platform and connected port.
	
6. A network interface with name **tun0** must be set. Check with running:
	
	```shell
	sudo ifconfig
	``` 
7. By default the router also implements a web-server. It can be accessed through the IPv6 address of the device (advertized when initializing the interface).


###Proxy server

In order to get the *cf-proxy* on the gateway, go through the following:

1. Use Maven to build Californium. Change to the project's root directory and run:

	```shell
	mvn clean install
	```
2. All the executable jar files can be found in *californium/demo-apps*. We need the *cf-proxy*:

	```shell
	cd californium/demo-apps/cf-proxy/target/
	```
3. Run the executable:

	```shell
	java -jar cf-proxy-1.1.0-SNAPSHOT.jar
	```
4. The proxy server will start running in the current shell. Information about the server's listening ports will be there also. Default ports are *8080* for HTTP and *5683* for CoAP.
5. When first run, the proxy creates three property files. In **Californium.properties** you can change various properties (eg listening ports).

###NodeRED

Node-RED documentation [page](http://nodered.org/docs/) contains an Installation guide for various devices. Depending on your chosen host device, follow the instructions found [here](http://nodered.org/docs/getting-started/installation). When ready you should be able to [run](http://nodered.org/docs/getting-started/running) node_RED.

##Making requests
Notice that in order to accept requests from a remote host (outside the local network) you need to forward the specific listening port of the application. Also most of the times setting a static IP address for your gateway-device is convenient.  
*Port forwarding is device, platform and network specific and not a topic of this README.*
###HTTP requests
HTTP requests can be made both locally and remotely. Using an HTTP request tool (or simply from your browser if it is a GET request) access:```http://localhost:8080```.

The format for requesting a device resource is:

```
http://<host_address>:<http_port>/proxy/<device_address>:<device_port>/<resource_uri>/
```
An example request to a remote IPv6 CoAP device:

```
http://90.90.0.100:8080/proxy/[fd00::212:4b00:60d:3fee]:5683/toggleLED/
```
 
###CoAP requests
Same is applied for CoAP requests. Using a CoAP client (like Firefox's [Copper Cu](https://addons.mozilla.org/el/firefox/addon/copper-270430/)) you can make a request to the host and proxy it to HTTP or CoAP. Each case uses a different resource as seen below, but both use the **Proxy-Uri** header for storing the uri of the desired device.

1. Proxy a CoAP request to a CoAP resource:  

	```shell
	coap://<host_address>:<http_port>/coap2coap/
	```
2. Proxy a CoAP request to an HTTP resource:  

	```shell
	coap://<host_address>:<coap_port>/coap2http/
	```

Example: GET a resource from an IPv6 CoAP device through the gateway-host with address ```90.90.0.100``` :

1. Set the header ```Proxy-Uri = coap://[fd00::212:4b00:60d:3fee]:5683/hello```.
2. Set the request type to *GET*.
3. Make the request to ```coap://90.90.0.100:5683/coap2coap/```


###Push data to the *relayr.* cloud
For pushing data to the *relayr.* cloud we will use the Node-RED flow located in this repository. The flow parses a formatted message and explicitly creates an API call to the cloud, using cURL.  

In order to create the flow:

1. Run ```node-red```, open a browser and visit ```localhost:1880```.
2. Go on the upper-right corner, *Options* ‣ *Import* ‣ *Clipboard* , and copy the code from the flow file.
3. This creates a CoAP server listening to port *8181*. An incoming message will go through the parser, which in turn will create the attributes of the cURL call.

The format of such call is:  

```shell
curl -X POST -H "Content-Type: application/json" -H "Authorization: <authorization_token>" -d "{values in JSON format}" "https://api.relayr.io/devices/<device_id>/data"
```
The CoAP client must send a message with the following format:

```
"<device_id>,<authorization_token>,<resource_meaning>,<resource_value>"
```
Notice that the Node-RED CoAP server is listening to port *8181* and not to default *5683*.

##Documentation
Info and repositories of the tools described can be found on the following links:  

* [Californium](http://www.eclipse.org/californium/).    
* Apache Maven [documentation](https://maven.apache.org/guides/index.html).  
* Node-RED [official page](https://nodered.org).  
* *relayr.* [API calls](http://docs.relayr.io/api/#Send-device-reading) 

##TODO
1. Screenshot of border-router web-server.  
2. Proxy server screenshots.  
3. Auto set after boot using bash script. 
4. Change the contiki-tools-installation link with recent.