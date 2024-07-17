<h1>Azure Sentinel VM Scan</h1>



<h2>Description</h2>
In this project, I deployed a deliberately vulnerable Windows 10 VM on Azure to attract potential unauthorized access attempts. I utilized a PowerShell script to extract metadata from failed login attempts recorded in the Windows Event Viewer. This metadata was then sent to a third-party API to determine the geolocation of the intruders. The geolocation data was logged into a custom log within Azure's Log Analytics Workspace. Subsequently, I configured an Azure Sentinel workbook to visually display the global attack data on a world map, highlighting the location and intensity of the attacks.
<br />


<h2>Languages and Utilities Used</h2>

- <b>PowerShell</b> 
- <b>Sentinel</b> 

<h2>Environments Used </h2>

- <b>Windows 10</b>
- <b>Microsft Azure</b>

<h2>Project walk-through</h2>

<p align="center">
Before initiating this project, I deployed a Windows 10 VM in Azure and set up Azure Sentinel (now Microsoft Sentinel) to monitor the VM. During the VM creation process, I configured a new Network Interface Card (NIC) security group with specific settings as shown below and then removed the preconfigured security group: <br/>
<img src="https://i.imgur.com/TXQB63W.png" height="80%" width="90%" alt="Vuln Project"/>
<br />
<br />
To begin, I created a new Log Analytics workspace to ingest logs from the Windows 10 VM. This workspace will later allow me to build a custom rule set to process and analyze data from failed login events on the Windows 10 VM. To create this workspace I went to my Azure Portal > Lag Analytics workspace > Create > Enter the Basic information and then create:  <br/>
<img src="https://i.imgur.com/0E1DIvz.png" height="80%" width="90%" alt="Vuln Project"/>
<br/>
<br />
<img src="https://i.imgur.com/vhItCWJ.png" height="80%" width="90%" alt="Vuln Project"/>
<br />
<br />
I then needed to turn on Microsoft Defender for Cloud. By navigating to Microsoft Defender for Cloud and then Management > Environment settings > HoneypotSiem and turned-on Servers and turned off SQL servers, since I won’t be needing it and then saved it. Following that I went into the Data collections tab on the left and enabled All Events and saved it: <br/>
<img src="https://i.imgur.com/Msc77xa.png" height="80%" width="90%" alt="Vuln Project"/>
<br/>
<br />
<img src="https://i.imgur.com/2twF2eA.png" height="80%" width="90%" alt="Vuln Project"/>
<br/>
<br />
<img src="https://i.imgur.com/nrdJNr7.png" height="80%" width="90%" alt="Vuln Project"/>
<br />
<br />
I then went back to Log Analytics workspaces and went into my recently created workspace called HoneypotSiem. Then migrated to Classic > Virtual machines and clicked the VM I wanted it to connect to (Honeypot-VM). I then clicked connect at the top to establish the connection:  <br/>
<img src="https://i.imgur.com/0yKfc85.png" height="90%" width="90%" alt="Vuln Project"/>
<br/>
<br />
<img src="https://i.imgur.com/1jdTgQz.png" height="90%" width="90%" alt="Vuln Project"/>
<br/>
<br />
<img src="https://i.imgur.com/1olsaRU.png" height="90%" width="90%" alt="Vuln Project"/>
<br />
<br />
After connecting them I then logged into the Windows 10 VM via Remote Desktop Connection from my home PC. Once in the VM I turned off the Windows Defender Firewall Domain, Private, and Public profiles: <br/>
<img src="https://i.imgur.com/PUvGGLJ.png" height="80%" width="90%" alt="Vuln Project"/>
<br />
<br />
I then went into Event Viewer and selected the drop down “Windows Logs” and then selected Security. Note that I completed a failed login attempt before initially connecting to demonstrate a captured failed login attempt in the viewer:  <br/>
<img src="https://i.imgur.com/SlpbVda.png" height="80%" width="90%" alt="Vuln Project"/>
<br />
<br />
Using this log data, we can identify the IP addresses of users attempting to connect to the VM. I employed a PowerShell script to extract logs with an Event ID of 4625 (failed login events) and feed these IP addresses into a geolocation API called “ipgeolocation” to obtain detailed location information. The PowerShell script then retrieves the location data from the API and logs it into a custom log created in the Windows VM:  <br/>
<img src="https://i.imgur.com/FA6QsHe.png" height="80%" width="90%" alt="Vuln Project"/>
<br/>
<br />
<br/>
<br />
<img src="https://i.imgur.com/sn2tNOu.png" height="80%" width="90%" alt="Vuln Project"/>
<br />
<br/>
<br />
While the script is running it will add the location information into this custom log document for each new failed login attempt:  <br/>
<img src="https://i.imgur.com/u578IMK.png" height="80%" width="90%" alt="Vuln Project"/>
<br />
<br />
After running the PowerShell script I could see that it picked up two failed login attempts (highlighted in purple) that were both from me:  <br/>
<img src="https://i.imgur.com/ztIxEXs.png" height="80%" width="90%" alt="Vuln Project"/>
<br />
<br />
Next, I copied the contents of the custom log file on the Windows VM, then minimized the Window VM leaving everything running, and placed them into a notes file so I could use them as a sample log when creating the custom log in Azure. I then went to my Azure portal and created a custom log in Log Analytics workspace by going to Settings > Tables > Create > New custom log (MMA-based):  <br/>
<img src="https://i.imgur.com/xn2KYEs.png" height="80%" width="90%" alt="Vuln Project"/>
<br />
<br />
In this menu I used that notes file for the Sample log and then went to the Collection paths tab and put the path for the log data located on the Windows VM. Once done I then inputted a name for the log and then created it:  <br/>
<img src="https://i.imgur.com/edp5NW1.png" height="80%" width="90%" alt="Vuln Project"/>
<br/>
<br />
<img src="https://i.imgur.com/p6w1XNw.png" height="80%" width="90%" alt="Vuln Project"/>
<br/>
<br />
<img src="https://i.imgur.com/eTg5gHw.png" height="80%" width="90%" alt="Vuln Project"/>
<br />
<br />
I then navigated over to the Logs tab on the left side of the dashboard and in the area to input KQL queries I searched for the name of the custom log I just created. After finding the log and waiting some time for Log Analytics to sync with the Windows VM, I then selected Run. Displaying the logs PowerShell is capturing from the VM:  <br/>
<img src="https://i.imgur.com/sy11FSB.png" height="80%" width="90%" alt="Vuln Project"/>
<br/>
<br />
<img src="https://i.imgur.com/193FKzm.png" height="80%" width="90%" alt="Vuln Project"/>
<br />
<br />
Next, I wrote a KQL query that will essentially extract each field from the raw data, like the longitude, latitude, country. Allowing Sentinel to use those fields in a later step when I create a map widget:  <br/>
<img src="https://i.imgur.com/JMb36QP.png" height="80%" width="90%" alt="AD project"/>
<br />
<br />
After executing the query, I navigated to the Sentinel dashboard, accessed the Workbooks tab, and selected "Add Workbook":  <br/>
<img src="https://i.imgur.com/s34wu3Q.png" height="80%" width="90%" alt="Vuln Project"/>
<br />
<br />
Next, I entered the edit mode, removed the two preconfigured widgets, and used the "Add" drop-down menu to select "Add query." On that page, I added the previously created query and set the visualization type to "Map". Thus, creating the Sentinel map widget that will populate as more and more attackers attempt to gain access to the Windows 10 VM:  <br/>
<img src="https://i.imgur.com/iWapJt5.png" height="80%" width="90%" alt="Vuln Project"/>
<br/>
<br />
<img src="https://i.imgur.com/oJUlMf3.png" height="80%" width="90%" alt="Vuln Project"/>
<br />
<br />
After leaving the VM running for 24 hours I went back to the Sentinel widget to see that there were over nine thousand failed attempts logged from many parts of the world, most notably the Netherlands:  <br/>
<img src="https://i.imgur.com/a3CAdLx.png" height="80%" width="90%" alt="Vuln Project"/>
<br />
<h2>Conclusion</h2>
<br>
In conclusion, this project effectively demonstrated the use of Azure Sentinel to monitor and analyze unauthorized access attempts on a vulnerable Windows 10 VM. By visualizing the geolocation and intensity of these attacks, I provided valuable insights into global threat patterns.  <br/>
<br />
</p>
