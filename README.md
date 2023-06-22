# Azure Monitor Lab


## Introduction
In this lab we will:


 - Configure a virtual machine such that telemetry and logs can be collected.
 - Show what telemetry and logs can be collected.
 - Show how the data can be used and queried.


#
<details>
<summary>
  
### Step 1: Creating a Resource Group and VM with Powershell 

</summary>  
<br/>
First we will create the resource group and VM for this lab. From the Azure Portal, begin a PowerShell session in the cloud shell. 
Once the PowerShell session has started, Create the resource group with the following command:
  
  ```
  New-AzResourceGroup -Name AzureMonitorLab -Location 'EastUS'
```
  
  We can now create a VM with the following command:
  
  ```
New-AzVm -ResourceGroupName "AzureMonitorLab" -Name "myVM" -Location 'EastUS' -VirtualNetworkName "myVnet" -SubnetName "mySubnet" -SecurityGroupName   "myNetworkSecurityGroup" -PublicIpAddressName "myPublicIpAddress" -PublicIpSku Standard -OpenPorts 80,3389 -Size Standard_DS1_v2
```

  After being prompted for credentials, create a username and password. 

  </details>
  
  
  #
<details>
<summary>
  
### Step 2: Create a Log Analytics Workspace 
</summary>  
<br/>
  
From the Azure Portal, we can search for ‘Log Analytics workspaces’. On the ‘Log Analytics workspaces’ blade click ‘+ Create’

  ![image](https://github.com/romhaki/Azure-Monitor/assets/136436650/2a3d0ae1-050f-4718-b523-0bccaf14882c)

  The workspace configuration is as follows:

  ![image](https://github.com/romhaki/Azure-Monitor/assets/136436650/094cfd15-4a8c-42bf-9ae9-338a6b1e9496)

  
  </details>
  

  #
<details>
<summary>
  
### Step 3: Enable the Log Analytics virtual machine extension  
</summary>  
<br/>
Now, we will enable the Log Analytics virtual machine extension. This will install a Log Analytics agent on the virtual machine, collecting data from the virtual machine and transferring it to the Log Analytics workspace we created. 

From the portal, we can navigate to the Log Analytics blade and in the ‘Overview’ section connect a VM as a data source: 

  ![image](https://github.com/romhaki/Azure-Monitor/assets/136436650/8049ae17-f5e5-47b1-babb-6b4761d3f2a1)

  On the following list, select the virtual machine being used for this lab, and click ‘Connect’. This may take a few minutes. 

  ![image](https://github.com/romhaki/Azure-Monitor/assets/136436650/55c5c88d-ed20-4699-af24-4afad01d4210)

  
</details>



  #
<details>
<summary>
  
### Step 4: Collect virtual machine event and performance data  
</summary>  
<br/>
Now, we will configure the collection of the Windows System log and common performance counters. First, navigate to the Log Analytics workspace that was previously created. Navigate to the ‘Legacy agents management’ under ‘Settings’. Click ‘Add windows event log’ in the ‘Windows event logs’ section. 
  

  ![image](https://github.com/romhaki/Azure-Monitor/assets/136436650/322ee2fe-ecbb-4eb1-8753-a6efcbbe0ccb)

  
  Type ‘System’ under ‘Log Name’. This will add event logs to the workspace. Also ensure that ‘Error’ and ‘Warning’ are selected. 
  
  
  ![image](https://github.com/romhaki/Azure-Monitor/assets/136436650/57e5fa56-2485-41b6-8c87-371ff3711813)

  Now navigate to the ‘Windows performance counters’ section, and click ‘Add performance counter’. The following performance counters will be added to monitor specific aspects of the virtual machine's performance:
- Memory(*)\Available Memory Mbytes
- Process(*)\% Processor Time
- Event Tracing for Windows\Total Memory Usage — Non-Paged Pool
- Event Tracing for Windows\Total Memory Usage — Paged Pool

  ![image](https://github.com/romhaki/Azure-Monitor/assets/136436650/e7baca7c-cd59-4423-aefe-40cafe7aade0)

  Once the performance counters are added, click ‘Apply’ at the bottom of the page. 

  
 </details>
  
  
  
  
  
  
  
  
  
  
  
  
  
  #
<details>
<summary>
  
### Step 5: View and query collected data  
</summary>  
<br/>
Now, we will run a log search on the data collection. In the Logs Analytics workspace we created, select ‘Logs’ in the General section. We can now use preset queries or write our own to show desired information. The predefined  ‘Memory and CPU usage’ is utilized by searching for it as follows: 
  

![image](https://github.com/romhaki/Azure-Monitor/assets/136436650/2babec70-7133-4da7-931d-dcda8c2d522a)

  
  The query is as follows:

  ```
// Memory and CPU usage 
// Chart all computers' used memory and CPU, over the last hour. 
Perf
| where TimeGenerated > ago(1h)
| where (CounterName == "% Processor Time" and InstanceName == "_Total") or CounterName == "% Used Memory"
| project TimeGenerated, CounterName, CounterValue
| summarize avg(CounterValue) by CounterName, bin(TimeGenerated, 1m)
| render timechart
```

  
  Optionally, we could generate additional load on the Azure VM to collect more data:
Navigate to the Azure VM blade. In the Operations section, select "Run command." On the "RunPowerShellScript" blade, run the following script:


  ```
cmd
:loop
dir c:\ /s > SWAP
goto loop
```

The script continuously lists all files and directories on the C: drive, saving the output to a file called "SWAP". It then repeats this process indefinitely, creating an endless loop. This will create additional load causing changes in the VM performance, which we can now visualize within the Log Analytics Workspace. 

  
 </details>
  
  
