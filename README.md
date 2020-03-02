*This lab is part of a series of guides from the [Network Automation and Tooling workshop series](https://github.com/sttrayno/Network-Automation-Tooling)*

# Network Telemetry Lab Guide

## Streaming Telemetry with IOS-XE

Traditionally SNMP and syslog have been our primary way of monitoring our network devices, however these aren't perfect, unreliable transport, inconsistent formats, tough to filter and often issues around historical data retrieval, as well as the impact to the device resources when multiple Network Monitoring Solutions poll the device simultaneously. The concept of Model-Driven Telemetry looks to address many of these limitations of legacy monitoring  and provides an additional types of data which we can now look to leverage with more off the self tools.

With the availability of more programmatic interfaces and greater level of data available in controller based networking this starts to present even more opportunties for utilising telemetry data from the infrastructure. This lab will walk through how we can look to leverage some of that available data to create a simple network operations centre dashboard.

Since the IOS XE 16.6 release there has been support for model driven telemetry, which provides network operators with additional options for getting information from their network.

In this blog we'll use the TIG stack (Telegraf - to recieve data from the network device, InfluxDB - time series database and Grafana - Visualisation and dashboards) to collect, store and display the network and device data from our IOS-XE devices. The objective of this lab is to get you comfortable using TIG stack and Streaming Telemetry protocols. As you gain more expertise you'll be able to leverage these technologies to build more complex monitoring systems to gather data from a multitude of devices and systems and display in more useful ways. Please note this is just one set of tools and other tools both opensource and enterprise exist including ELK stack and Splunk, In further posts I'll look to compare and constrast these solutions.

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

There is also a Model Driven Telemetry sandbox which you can also find in the catalogue, feel free to reserve that as many of the steps in this lab guide are already covered and the Docker containers are prespun up.

### Step 0 - Configure and Install Docker

For this lab we need to use Docker to host containers needed to run the TIG stack and yang-explorer which allows us to explore the streaming telemetry models available for our network device. If you already have docker installed you can proceed to Step 1 and start to pull down the containers required. If you do not have docker installed you can consult the docker documentation [here](https://docs.docker.com/install/)

Alternatively if you don't want to install docker on your own laptop, you can use Model Driven Telemetry sandbox already available in the DevNet sandbox which has a developer box with both of these containers already configured and running which will save you a number of these steps. For completeness we'll walk through all the steps here.

### Step 1 - Setup our TIG stack

First thing we need to do is configure our TIG stack on our local machine. Thankfully, Jeremy Cohoe has created a fantastic docker container with all the needed components preinstalled. You can pull the container from the Docker hub with the following shell command.

```docker pull jeremycohoe/tig_mdt```

Let that pull down the required image from Docker hub then run the following command to start the container. 

```docker run -ti -p 3000:3000 -p 57500:57500 jeremycohoe/tig_mdt```

### Step 2 - Setup our Yang explorer

Secondly we also need

```docker pull robertcsapo/yang-explorer```

Again, let that pull down the required image from Docker hub then run the following command to start the container. 

```docker run -it --rm -p 8088:8088 robertcsapo/yang-explorer```

Verify you can access the explorer at http://localhost:8088 then leave it aside for now, we'll come back to this later to look for the valid topics that we can receive data from.

Note: Yang explorer is only supported on Linux and Mac systems so if you are running windows it may make more sense to use the developer box within your DevNet sandbox reservation.

### Step 3 - Configure IOS-XE device for streaming telemetry and verify

Configuration of telemetry on IOS-XE can be done either by tradition CLI with the `telemetry ietf subscription` commands, or programmatically with XML. In this guide we'll use the traditional CLI to configure a couple of models which are available on the device.

Please note the receiver IP address will depend on the IP address of your device, as in this guide we're connected to the sandbox via a VPN we can find our IP address from the anyconnect statistics menu, alternatively if you're using the DevBox the IP address will be shown from within your sandbox environment.

![](https://github.com/sttrayno/Network-Telemetry-Lab-Guide/blob/master/images/ip-check.gif?raw=true)

For ease of use now we're going to use some predefined xpath's to collect data from, however should you wish to use your own device you can and use the yang explorer to find the xpath of the valid operational models.

To configure the ietf subscriptions for this lab use the below configs

```
   telemetry ietf subscription 101
    encoding encode-kvgpb
    filter xpath /process-cpu-ios-xe-oper:cpu-usage/cpu-utilization
    stream yang-push
    update-policy periodic 500
    receiver ip address 192.168.63.1 57500 protocol grpc-tcp


   telemetry ietf subscription 102
    encoding encode-kvgpb
    filter xpath /interfaces-ios-xe-oper:interfaces/interface/statistics
    stream yang-push
    update-policy periodic 500
    receiver ip address 192.168.63.1 57500 protocol grpc-tcp
  
   telemetry ietf subscription 103
    encoding encode-kvgpb
    filter xpath /memory-ios-xe-oper:memory-statistics/memory-statistic
    stream yang-push
    update-policy periodic 500
    receiver ip address 192.168.63.1 57500 protocol grpc-tcp

   telemetry ietf subscription 104
    encoding encode-kvgpb
    filter xpath /interfaces-ios-xe-oper:interfaces/interface
    stream yang-push
    update-policy periodic 500
    receiver ip address 192.168.63.1 57500 protocol grpc-tcp
```
    
When the 4 ietf subscriptions have been configured as above. You can verify the status of them with the command

```
csrv1000#show telemetry ietf subscription all
  Telemetry subscription brief

  ID               Type        State       Filter type
  --------------------------------------------------------
  101              Configured  Valid       xpath

csrv1000#
csrv1000#show telemetry ietf subscription 101 detail
Telemetry subscription detail:

  Subscription ID: 101
  Type: Configured
  State: Valid
  Stream: yang-push
  Filter:
    Filter type: xpath
    XPath: /process-cpu-ios-xe-oper:cpu-usage/cpu-utilization
  Update policy:
    Update Trigger: periodic
    Period: 500
  Encoding: encode-kvgpb
  Source VRF:
  Source Address:
  Notes:

  Receivers:
    Address                                    Port     Protocol         Protocol Profile
    -----------------------------------------------------------------------------------------
    192.168.63.1                               57500    grpc-tcp


csrv1000#
csrv1000#show telemetry ietf subscription 101 receiver
Telemetry subscription receivers detail:

  Subscription ID: 101
  Address: 192.168.63.1
  Port: 57500
  Protocol: grpc-tcp
  Profile:
  State: Connected
  Explanation:
```

Now that your data has been configured to push out at the required intervals you might want to just leave the sandbox running for a little while to allow the influxDB to populate so we have some interesting data to visualise.

### Step 4 - Configure Grafana dashboards

Now we should have our network device connected to our TIG stack pushing the telemetry data at the configured intervals, all thats now left to do is start to configure our dashboards so we can create some nice visualisations.

To log into your Grafana, navigate to http://localhost:3000/ and login with the username/password combination of admin/Cisco123, if you're configuring your TIG stack manually you will have set the username/password in the setup process/

When you're logged into Grafana you can then select `New Dashboard` from the left hand menu and create graphs as we do in the graphics below. You should be subscribed to most of the topics available in the drop downs, just experiment and explore to find the ones you wish. One thing you need to make sure is that the source is configured to the network device address, in our case 10.10.20.30.

![](https://github.com/sttrayno/Network-Telemetry-Lab-Guide/blob/master/images/grafana-1.gif?raw=true)

Once you've created one graph, you should then be able to add another panel from the 'add panel' button in the top right to create more graphs.

![](https://github.com/sttrayno/Network-Telemetry-Lab-Guide/blob/master/images/grafana-2.gif?raw=true)

A few sample configs for the graphs found in the "examples" folder, you can be guaranteed if you've followed this lab guide and the configs are correct these graphs should provide relevant data.

![](https://github.com/sttrayno/Network-Telemetry-Lab-Guide/blob/master/images/grafana-4.gif?raw=true)

Note: You may wish to adjust the time range at the top right to a smaller interval such as 15 minutes, rather than the default 6 hours to avoid having gaps in your graphs right now.

Once you have the required number of graphs for you dashboard - in our case 4. You can then save your dashboard and view it.

![](https://github.com/sttrayno/Network-Telemetry-Lab-Guide/blob/master/examples/Screenshot%202020-03-02%20at%2016.38.07.png?raw=true)

Congratulations, you've created your first network telemetry dashboard!
