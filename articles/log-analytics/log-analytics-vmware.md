---
title: VMware Monitoring solution in Log Analytics | Microsoft Docs
description: Learn about how the VMware Monitoring solution can help manage logs and monitor ESXi hosts.
services: log-analytics
documentationcenter: ''
author: bandersmsft
manager: carmonm
editor: ''
ms.assetid: 16516639-cc1e-465c-a22f-022f3be297f1
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/12/2017
ms.author: banders
---

# VMware Monitoring (Preview) solution in Log Analytics
The VMware Monitoring solution in Log Analytics is a solution that helps you create a centralized logging and monitoring approach for large VMware logs. This article describes how you can troubleshoot, capture, and manage the ESXi hosts in a single location using the solution. With the solution, you can see detailed data for all your ESXi hosts in a single location. You can see top event counts, status, and trends of VM and ESXi hosts provided through the ESXi host logs. You can troubleshoot by viewing and searching centralized ESXi host logs. And, you can create alerts based on log search queries.

The solution uses native syslog functionality of the ESXi host to push data to a target VM, which has OMS Agent. However, the solution doesn't write files into syslog within the target VM. The OMS agent opens port 1514 and listens to this. Once it receives the data, the OMS agent pushes the data into OMS.

## Installing and configuring the solution
Use the following information to install and configure the solution.

* Add the VMware Monitoring solution to your OMS workspace using the process described in [Add Log Analytics solutions from the Solutions Gallery](log-analytics-add-solutions.md).

#### Supported VMware ESXi hosts
vSphere ESXi Host 5.5 and 6.0

#### Prepare a Linux server
Create a Linux operating system VM to receive all syslog data from the ESXi hosts. The [OMS Linux Agent](log-analytics-linux-agents.md) is the collection point for all ESXi host syslog data. You can use multiple ESXi hosts to forward logs to a single Linux server, as in the following example.  

   ![syslog flow](./media/log-analytics-vmware/diagram.png)

### Configure syslog collection
1. Set up syslog forwarding for VSphere. For detailed information to help set up syslog forwarding, see [Configuring syslog on ESXi 5.x and 6.0 (2003322)](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2003322). Go to **ESXi Host Configuration** > **Software** > **Advanced Settings** > **Syslog**.
   ![vsphereconfig](./media/log-analytics-vmware/vsphere1.png)  
2. In the *Syslog.global.logHost* field, add your Linux server and the port number *1514*. For example, `tcp://hostname:1514` or `tcp://123.456.789.101:1514`
3. Open the ESXi host firewall for syslog. **ESXi Host Configuration** > **Software** > **Security Profile** > **Firewall** and open **Properties**.  

    ![vspherefw](./media/log-analytics-vmware/vsphere2.png)  

    ![vspherefwproperties](./media/log-analytics-vmware/vsphere3.png)  
4. Check the vSphere Console to verify that that syslog is properly set up. Confirm on the ESXI host that port **1514** is configured.
5. Download and install the OMS Agent for Linux on the Linux server. For more information, see the [Documentation for OMS Agent for Linux](https://github.com/Microsoft/OMS-Agent-for-Linux).
6. After the OMS Agent for Linux is installed, go to the  /etc/opt/microsoft/omsagent/sysconf/omsagent.d directory and copy the vmware_esxi.conf file to the /etc/opt/microsoft/omsagent/conf/omsagent.d directory and the change the owner/group and permissions of the file. For example:

    ```
    sudo cp /etc/opt/microsoft/omsagent/sysconf/omsagent.d/vmware_esxi.conf /etc/opt/microsoft/omsagent/conf/omsagent.d
   sudo chown omsagent:omiusers /etc/opt/microsoft/omsagent/conf/omsagent.d/vmware_esxi.conf
    ```
7. Restart the OMS Agent for Linux by running `sudo /opt/microsoft/omsagent/bin/service_control restart`.
8. Test the connectivity between the Linux server and the ESXi host by using the `nc` command on the ESXi Host. For example:

    ```
    [root@ESXiHost:~] nc -z 123.456.789.101 1514
    Connection to 123.456.789.101 1514 port [tcp/*] succeeded!
    ```

9. In the OMS Portal, perform a log search for `Type=VMware_CL`. When OMS collects the syslog data, it retains the syslog format. In the portal, some specific fields are captured, such as *Hostname* and *ProcessName*.  

    ![type](./media/log-analytics-vmware/type.png)  

    If your view log search results are similar to the image above, you're set to use the OMS VMware Monitoring solution dashboard.  

## VMware data collection details
The VMware Monitoring solution collects various performance metrics and log data from ESXi hosts using the OMS Agents for Linux that you have enabled.

The following table shows data collection methods and other details about how data is collected.

| platform | OMS Agent for Linux | SCOM agent | Azure Storage | SCOM required? | SCOM agent data sent via management group | collection frequency |
| --- | --- | --- | --- | --- | --- | --- |
| Linux |![Yes](./media/log-analytics-vmware/oms-bullet-green.png) |![No](./media/log-analytics-vmware/oms-bullet-red.png) |![No](./media/log-analytics-vmware/oms-bullet-red.png) |![No](./media/log-analytics-containers/oms-bullet-red.png) |![No](./media/log-analytics-vmware/oms-bullet-red.png) |every 3 minutes |

The following table show examples of data fields collected by the VMware Monitoring solution:

| field name | description |
| --- | --- |
| Device_s |VMware storage devices |
| ESXIFailure_s |failure types |
| EventTime_t |time when event occurred |
| HostName_s |ESXi host name |
| Operation_s |create VM or delete VM |
| ProcessName_s |event name |
| ResourceId_s |name of the VMware host |
| ResourceLocation_s |VMware |
| ResourceName_s |VMware |
| ResourceType_s |Hyper-V |
| SCSIStatus_s |VMware SCSI status |
| SyslogMessage_s |Syslog data |
| UserName_s |user who created or deleted VM |
| VMName_s |VM name |
| Computer |host computer |
| TimeGenerated |time the data was generated |
| DataCenter_s |VMware datacenter |
| StorageLatency_s |storage latency (ms) |

## VMware Monitoring solution overview
The VMware tile appears in the OMS portal. It provides a high-level view of any failures. When you click the tile, you go into a dashboard view.

![tile](./media/log-analytics-vmware/tile.png)

#### Navigate the dashboard view
In the **VMware** dashboard view, blades are organized by:

* Failure Status Count
* Top Host by Event Counts
* Top Event Counts
* Virtual Machine Activities
* ESXi Host Disk Events

![solution1](./media/log-analytics-vmware/solutionview1-1.png)

![solution2](./media/log-analytics-vmware/solutionview1-2.png)

Click any blade to open Log Analytics search pane that shows detailed information specific for the blade.

From here, you can edit the search query to modify it for something specific. For a tutorial on the basics of OMS search, check out the [OMS log search tutorial.](log-analytics-log-searches.md)

#### Find ESXi host events
A single ESXi host generates multiple logs, based on their processes. The VMware Monitoring solution centralizes them and summarizes the event counts. This centralized view helps you understand which ESXi host has a high volume of events and what events occur most frequently in your environment.

![event](./media/log-analytics-vmware/events.png)

You can drill further by clicking an ESXi host or an event type.

When you click an ESXi host name, you view information from that ESXi host. If you want to narrow results with the event type, add `“ProcessName_s=EVENT TYPE”` in your search query. You can select **ProcessName** in the search filter. That narrows the information for you.

![drill](./media/log-analytics-vmware/eventhostdrilldown.png)

#### Find high VM activities
A virtual machine can be created and deleted on any ESXi host. It's helpful for an administrator to identify how many VMs an ESXi host creates. That in-turn, helps to understand performance and capacity planning. Keeping track of VM activity events is crucial when managing your environment.

![drill](./media/log-analytics-vmware/vmactivities1.png)

If you want to see additional ESXi host VM creation data, click an ESXi host name.

![drill](./media/log-analytics-vmware/createvm.png)

#### Common search queries
The solution includes other useful queries that can help you manage your ESXi hosts, such as high storage space, storage latency, and path failure.

![queries](./media/log-analytics-vmware/queries.png)

#### Save queries
Saving search queries is a standard feature in OMS and can help you keep any queries that you’ve found useful. After you create a query that you find useful, save it by clicking the **Favorites**. A saved query lets you easily reuse it later from the [My Dashboard](log-analytics-dashboards.md) page where you can create your own custom dashboards.

![DockerDashboardView](./media/log-analytics-vmware/dockerdashboardview.png)

#### Create alerts from queries
After you’ve created your queries, you might want to use the queries to alert you when specific events occur. See [Alerts in Log Analytics](log-analytics-alerts.md) for information about how to create alerts. For examples of alerting queries and other query examples, see the [Monitor VMware using OMS Log Analytics](https://blogs.technet.microsoft.com/msoms/2016/06/15/monitor-vmware-using-oms-log-analytics) blog post.

## Frequently asked questions
### What do I need to do on the ESXi host setting? What impact will it have on my current environment?
The solution uses the native ESXi Host Syslog forwarding mechanism. You don't need any additional Microsoft software on the ESXi Host to capture the logs. It should have a low impact to your existing environment. However, you do need to set syslog forwarding, which is ESXI functionality.

### Do I need to restart my ESXi host?
No. This process does not require a restart. Sometimes, vSphere does not properly update the syslog. In such a case, log on to the ESXi host and reload the syslog. Again, you don't have to restart the host, so this process isn't disruptive to your environment.

### Can I increase or decrease the volume of log data sent to OMS?
Yes you can. You can use the ESXi Host Log Level settings in vSphere. Log collection is based on the *info* level. So, if you want to audit VM creation or deletion, you need to keep the *info* level on Hostd. For more information, see the [VMware Knowledge Base](https://kb.vmware.com/selfservice/microsites/search.do?&cmd=displayKC&externalId=1017658).

### Why is Hostd not providing data to OMS? My log setting is set to info.
There was an ESXi host bug for the syslog timestamp. For more information, see the [VMware Knowledge Base](https://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2111202). After you apply the workaround, Hostd should function normally.

### Can I have multiple ESXi hosts forwarding syslog data to a single VM with omsagent?
Yes. You can have multiple ESXi hosts forwarding to a single VM with omsagent.

### Why don't I see data flowing into OMS?
There can be multiple reasons:

* The ESXi host is not correctly pushing data to the VM running omsagent. To test, perform the following steps:

  1. To confirm, log on to the ESXi host using ssh and run the following command: `nc -z ipaddressofVM 1514`

      If this is not successful, vSphere settings in the Advanced Configuration are likely not correct. See [Configure syslog collection](#configure-syslog-collection) for information about how to set up the ESXi host for syslog forwarding.
  2. If syslog port connectivity is successful, but you don't still see any data, then reload the syslog on the ESXi host by using ssh to run the following command: ` esxcli system syslog reload`
* The VM with OMS Agent is not set correctly. To test this, perform the following steps:

  1. OMS listens to the port 1514 and pushes data into OMS. To verify that it is open, run the following command: `netstat -a | grep 1514`
  2. You should see port `1514/tcp` open. If you do not, verify that the omsagent is installed correctly. If you do not see the port information, then the syslog port is not open on the VM.

     1. Verify that the OMS Agent is running by using `ps -ef | grep oms`. If it is not running, start the process by running the command ` sudo /opt/microsoft/omsagent/bin/service_control start`
     2. Open the `/etc/opt/microsoft/omsagent/conf/omsagent.d/vmware_esxi.conf` file.

         Verify that the proper user and group setting is valid, similar to: `-rw-r--r-- 1 omsagent omiusers 677 Sep 20 16:46 vmware_esxi.conf`

         If the file does not exist or the user and group setting is wrong, take corrective action by [Preparing a Linux server](#prepare-a-linux-server).

## Next steps
* Use [Log Searches](log-analytics-log-searches.md) in Log Analytics to view detailed VMware host data.
* [Create your own dashboards](log-analytics-dashboards.md) showing VMware host data.
* [Create alerts](log-analytics-alerts.md) when specific VMware host events occur.
