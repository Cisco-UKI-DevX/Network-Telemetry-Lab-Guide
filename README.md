# Streaming-Telemetry-Lab-Guide

## DNA-C Network Operations Centre Custom Dashboard

`This lab is based from a blog by Adam Radford which walks through creating your own dashboard for DNA-C we'll walk through this process in a little more detail here https://blogs.cisco.com/developer/dna-center-noc-dashboard`

With the availability of more programmatic interfaces and greater level of data available in controller based networking this starts to present even more opportunties for utilising telemetry data from the infrastructure. This lab will walk through how we can look to leverage some of that available data to create a simple network operations centre dashboard.

In this blog we'll use the TIG stack (Telegraf - the agent which connects to our custom python script, InfluxDB - time series database and Grafana - Visualisation and dashboards) to collect, store and display the network and device data from DNA-C.

![](https://alln-extcloud-storage.cisco.com/ciscoblogs/5d78aeb3710f9.png)

## Step 0 - Download code and configure TIG stack

