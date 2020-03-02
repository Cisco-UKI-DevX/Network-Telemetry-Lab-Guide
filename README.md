# Network-Telemetry-Lab-Guide

## Streaming Telemetry with IOS-XE

Traditionally SNMP has been highly successful for monitoring enterprise networks, but it has limitations: unreliable transport, inconsistent encoding between versions, limited filtering and data retrieval options, as well as the impact to the CPU and memory of the running device when multiple Network Monitoring Solutions poll the device simultaneously. Model-Driven Telemetry addresses many of the shortfalls of legacy monitoring capabilities and provides an additional interface in which telemetry is now available to be published from.

With the availability of more programmatic interfaces and greater level of data available in controller based networking this starts to present even more opportunties for utilising telemetry data from the infrastructure. This lab will walk through how we can look to leverage some of that available data to create a simple network operations centre dashboard.

Since the IOS XE 16.6 release there has been support for model driven telemetry, which provides network operators with additional options for getting information from their network.

In this blog we'll use the TIG stack (Telegraf - to recieve data from the network device, InfluxDB - time series database and Grafana - Visualisation and dashboards) to collect, store and display the network and device data from our IOS-XE devices. The objective of this lab is to get you comfortable using TIG stack and Streaming Telemetry protocols. As you gain more expertise you'll be able to leverage these technologies to build more complex monitoring systems to gather data from a multitude of devices and systems and display in more useful ways.

![](https://github.com/sttrayno/Network-Telemetry-Lab-Guide/blob/master/images/tigstack.png?raw=true)

Before we get started with telemetry we'll need a network device to configure receive data from. One of the easiest test environments you'll find is on the Cisco DevNet Sandbox which has multiple options. These are completely free and can in some cases be accessed within seconds. https://developer.cisco.com/docs/sandbox/#!overview/all-networking-sandboxes

Most popular sandboxes include:

- IOS-XE (CSR) - Always-On
- IOS-CR - Always-On
- Multi IOS test environment (VIRL based) - Reservation required
- Cisco SD-WAN environment - Always-On
- Cisco DNA-C environment - Always-On

Please note you are free to use this with your own hardware or test environment. However the commands in this lab guide have been tested for the sandboxes they correspond to. For this lab guide we will be using the reservable IOS XE on CSR Recommended Code Sandbox which can be found on the Sandbox catalogue https://devnetsandbox.cisco.com/RM/Topology

![](https://github.com/sttrayno/Ansible-Lab-Guide/blob/master/images/sandbox-screen.png)

There is also a Network Streaming Telemetry sandbox which you can also find in the catalogue.

### Step 0 - Download code and configure TIG stack



 docker run -ti -p 3000:3000 -p 57500:57500 jeremycohoe/tig_mdt


