# Network-Telemetry-Lab-Guide

## Streaming Telemetry with IOS-XE

## DNA-C Network Operations Centre Custom Dashboard

`This lab is based from a blog by Adam Radford which walks through creating your own dashboard for DNA-C we'll walk through this process in a little more detail here https://blogs.cisco.com/developer/dna-center-noc-dashboard`

With the availability of more programmatic interfaces and greater level of data available in controller based networking this starts to present even more opportunties for utilising telemetry data from the infrastructure. This lab will walk through how we can look to leverage some of that available data to create a simple network operations centre dashboard.

Since the IOS XE 16.6 release there has been support for model driven telemetry, which provides network operators with additional options for getting information from their network.

Traditionally SNMP has been highly successful for monitoring enterprise networks, but it has limitations: unreliable transport, inconsistent encoding between versions, limited filtering and data retrieval options, as well as the impact to the CPU and memory of the running device when multiple Network Monitoring Solutions poll the device simultaneously. Model-Driven Telemetry addresses many of the shortfalls of legacy monitoring capabilities and provides an additional interface in which telemetry is now available to be published from.

In this blog we'll use the TIG stack (Telegraf - the agent which connects to our custom python script, InfluxDB - time series database and Grafana - Visualisation and dashboards) to collect, store and display the network and device data from DNA-C.

![](https://alln-extcloud-storage.cisco.com/ciscoblogs/5d78aeb3710f9.png)

### Step 0 - Download code and configure TIG stack

