#Setting up a 6loWPAN gateway

##Introduction

##Requirements

1. One Zolertia [Remote](https://github.com/Zolertia/Resources/wiki/RE-Mote) or [Firefly](https://github.com/Zolertia/Resources/wiki/Firefly) mote which will provide the 802.15.4 interface to the gateway.
2. Tools for compiling and flashing Contiki code. This  step is required for flashing the edge-router and is optional for flashing motes of the wireless network. Step-by-step instructions on installing on various platforms can be found [here]().
3. Apache Maven in order to compile californium-proxy. Detailed steps on installing can be found [here](https://maven.apache.org/install.html).

##Border Router
Follow these steps to get a IPv6 edge router Up&Running:

1. cd to Contiki border-router directory:

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
4. Change to Contiki tools directory and make *tunslip6*:
	
	```shell
	cd contiki/tools
	make tunslip6
	```
5. Create an IPv6 network interface:
	
	```shell
	sudo ./tunslip6 -s /dev/ttyUSB0 -t tun0 fd00::1/64
	```
	notice that the device path may vary depending on your platform and connected port.
	
6. The interface with name **tun0** must be set. Check with running:
	
	```shell
	sudo ifconfig
	``` 
7. By default the router also implements a web-server. It can be accessed through the IPv6 address of the device (advertized when initializing the interface).


##Proxy server

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

##NodeRED

##Documentation
Californium
6loWPAN
CoAP
Apache Maven [documentation](https://maven.apache.org/guides/index.html)

##TODO
Picture of border-router web-server
Proxy server screenshots