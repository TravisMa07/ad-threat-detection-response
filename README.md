# Overview
This project demonstrate threat detection and automated response utilizing **Active Directory** for user authentication, **Splunk(SIEM/IDS)** for event detection and logging, and **Shuffle with Slack(SOAR)** for automated alert and respone.

# Logical Diagram
![Logical Diagram](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/SIEM-SOAR%20Integration%20for%20Unauthorized%20Active%20Directory%20Logins.drawio.png)

The logical diagram illustrates how communication and data flow are standardized across systems. The scenario simulates an adversary gaining unauthorized access to a domain-joined test machine. This triggers telemetry from the test machine to Splunk, which detects the unauthorized sucessful login through RDP and initiates two actions:

1. Sends an alert notification to Slack, a multi-purpose platform used for communication, threat detection, and incident response as part of this project’s workflow.

2. Sends a webhook to Shuffle, which triggers an automated SOAR playbook in response to the unauthorized login. The playbook notifies a Security Personnel, who decides to either disable the compromised account or take no action. If the account is disabled, Shuffle & the Domain Controller executes the action, and a confirmation message is sent to Slack.

# Machine Configuration

The project consist of 4 virtual machines, 3 being on the cloud (Oracle Cloud Provider) and 1 being on-prem. The 3 virtual machine on the cloud consist of the Splunk Server, Domain Controller, Domain User/Test Machine. The last virtual machine is on-prem acting as an adversary machine attacking the Virtual Private Cloud (VPC). Lastly, there's an personal computer that is able to interact with all 4 virtual machine and the entire environment.

<br/>
<br/>

![Subnet Connectvity and Machines on VPC](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_VM_INSTANCES.png)
*Three virtual machines including Splunk, a test machine, and a domain controller are connected to the same subnet in the same virtual private cloud (VPC).*

<br/>
<br/>

![Ingress Rules](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_INGRESS_RULES.png)
*Ingress firewall rules allowing all source IP to send data to the respected ports.*
<br/>
*For an real enterprise network, **DO NOT ALLOW ALL SOURCE IP**. For this lab in particular, instead of using my public IP as the source IP, I allow all source IP as it's not a production environment and is not of importance to the scope of this lab.*

<br/>
<br/>

![Egress Rules](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_EGRESS_RULES.png)
*Egress firewall rules allowing all outbound traffic to be sent.*

# Active Directory Installation & Configuration

For the scope of this project, majority of the Active Directory Setup and Configuration won't be documented. Reference image can be found in the github repo under **ADSSDR_AD_SETUP#.png** for Active Directory Configuration.

**1. travis-ADDC01 Machine will be the Domain Controller.**

![AD1](https://github.com/TravisMa07/active-directory-siem-soar-detection-response/blob/main/ADSSDR_AD_SETUP1.png)
![AD2](https://github.com/TravisMa07/active-directory-siem-soar-detection-response/blob/main/ADSSDR_AD_SETUP2.png)

**2. Join the Test Machine into the new domain: travis.local. Test Machine will be login as a new user created under the Domain: Travis Ma (TMa).**
   
![AD3](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_AD_SETUP4.png)
![AD4](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_AD_SETUP5.png)

*If any issues arise during the Active Directory configuration and setup, ensure that the DNS server is properly configured, firewall ingress rules allow traffic on the appropriate ports, and Remote Desktop is enabled for new users (TMa). Additional troubleshooting solutions can be found on the Microsoft Forums.*

# Splunk Installation & Configuration

Splunk must be installed and configure on the Ubuntu Server where it will host Splunk and its Web Interface. Splunk Agents (Splunk Universal Forwarder) will be install on both the Test Machine and the Domain Controller Machine allowing telemetry to be sent from the VMs to the Splunk Server.

**1. Head over to Splunk website, create an account, then copy the wget link for Splunk.**

![Splunk1](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_SPLUNK2.png)

**2. Make to update and upgrade locally install packages (apt get update/upgrade). Then proceed to run the wget command to get splunk.deb downloaded on the local machine. After it's downloaded, install the packages.**

![Splunk2](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_SPLUNK3.png)
![Splunk3](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_SPLUNK4.png)

**3. Start splunk and head over to the web interface via [Public IP Address]:8000**

![Splunk4](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_SPLUNK5.png)
![Splunk5](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_SPLUNK6.png)

**4. Configure Splunk to recieve data ingestion via a netork port (9997) and creating an index to store that data.**

![Splunk6](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_SPLUNK9.png)
*This configuration enables Splunk to recieve data from external sources like the Universal Forwarders Agents on TCP port 9997.*

![Splunk7](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_SPLUNK8.png)
*travis-ad index stores incoming data.*

# Splunk Universal Forwarder Agents Installation
Splunk Universal Forwader Agents is to be install on the Test Machine and the Domain Controller VMs. Installation Process is the same on both machines.

**1. Head over to Splunk website and install Splunk Universal Forwarder.**

 ![SplunkA1](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_SPLUNKA1.png)

**2. Continue with the installation process and enter the Splunk Server Private IP to port 9997 as the Recieving Indexer.**

![SplunkA2](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_SPLUNKA2.png)

**3. Head over to the Splunk Forwarder directory on the VM. Copy the index.conf file from ./etc/system/default and paste it into ./etc/system/local**

![SplunkA3](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_SPLUNKA3.png)
![SplunkA4](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_SPLUNKA4.png)

**4. Open the index.conf file located in ./etc/system/local with NotePad and add the lines in the image below into the bottom of the file with the excpetion of changing index = "your personal index name"**

![SplunkA5](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_SPLUNKA5.png)
<br/>*Collect Windows Security Event Logs and forward them to the indexer where it will be stored in.*

**5. Head over to Services.msc and locate SplunkForwarder. Open properties and select "Log on as: Local System Account". Apply changes and restart the service.**

![SplunkA6](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_SPLUNKA5.5.png)
![SplunkA7](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_SPLUNKA6.png)
<br/>*Update property changes and index.conf configuration*

**6. Head over to Splunk Web Interface, go to Apps: Search and Reporting, then query "index=...".**

![SplunkA8](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_SPLUNKA7.png)
*This confirm the connection of the telemetry links between the Test Machine and Domain Controller Machine to the Splunk Server showing real-time Windows Security Event Logs for the purpose of finding Sucessful Unauthorized Logins.*

# Creating Alert for Sucessful Authentication
A sucessful Login on an Windows Environment generate a Window Security Log: Event ID 4624. Under this security log, it can generate 9 different "logon types". Events related to RDP, Event 4624 can generate types of 7 & 10. To simulate an unauthorized RDP login, an alert should be created with a focus on suspicious Source Network Address Values.

![Alert1](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_ALERT1.png)
![Alert2](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_ALERT2.png)

**1. Modify the query to:**
<br/>
"index=... EventCode=4624 (Logon_Type=7 OR Logon_Type=10) Source_Network_Address=* Source_Network_Address!="-" Source_Network_Address!=129."

**What this Query/Alert do?"**
This Splunk query is designed to detect potentially unauthorized RDP (Remote Desktop Protocol) login activity by examining successful Windows login events and filtering based on the source network address.
<br/>
Each component of the query performs a specific function:
   - **"index..."** <br/>
     Searches only for the designated index where Window Security Event Logs are stored.
     
   - **"EventCode=4624"** <br/>
     A specific Window Security log Event that indicates a Sucessful User Logon.
     
   - **"(Logon_Type=7 OR Logon_Type=10)"** <br/>
     Specifies the method of the logon attempts shown. Reference can be found in the chart above
     
   - "Source_Network_Address=*" <br/>
     Indicate a value MUST exist under this field name. Prevent logs where the Source Network Address is missing or Undefined.
     
   - **"Source_Network_Address!="-""** <br/>
     Filter NULL Source Network Address. Excludes events where the source address is the placeholder "-", which often appears when the address isn’t available.
     
   - **"Source_Network_Address!=129."** <br/>
     FILTER OUT PUBLIC IP ADDRESS COMING FROM OUR TRUSTED MACHINE (specific to your configuration). Any number that isn't part of your public ip address for your environment can be consider unauthorize.
      - *DiSCLAIMER: This doesn't involve other environment/circumstances such as Work at Home, Vacations, etc. As a result, the alert may generate false positives if legitimate users connect from outside the 129. IP range. Consider building an allowlist or adding additional filters if needed.*

**2. Test the Query**
<br/>
Initiate a Remote Desktop Protocol (RDP) from the Adversary Virtual Machine into the Test Machine or the Domain Controller Machine. Since the Adversary VM operates outside the trusted public IP space (129.x.x.x), this action should generate the appropriate Window Event Log.

**3. Create the Alert/Rules**

With the Query inputted, hit **Save As, then Alert**.
<br/>
**Title the Alert** to indicate for Unauthorized Succesful Login (travislocal-UnauthorizedSucessful-Login-RDP). <br/>
**Alert Type**: you can run in any specific interval. <br/>
**Cron Schedule** allow the alert to run at **specific intervals during a time range for precise control over time execution** <br/>
**Cron Expression:** "* * * * *" - This allow the Alert to run every minute for the time range (60 minute). <br/>
**Trigger Actions:** Add action to "Add to Triggered Alerts". This allow the alert to be on the Triggered Alerts list.<br/>

![Alert3](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/ADSSDR_ALERT3.png)

**Top right of the Splunk Web Interface, Hit Activity then Triggered Alerts. This is where the Unauthorized Succesful Login Alert will be stored when triggered.**







    
   
        





