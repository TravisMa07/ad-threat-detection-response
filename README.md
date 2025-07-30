# Overview
This project demonstrate threat detection and automated response utilizing **Active Directory** for user authentication, **Splunk(SIEM/IDS)** for event detection and logging, and **Shuffle with Slack(SOAR)** for automated alert and respone.

# Logical Diagram
![Logical Diagram](https://raw.githubusercontent.com/TravisMa07/active-directory-siem-soar-detection-response/refs/heads/main/SIEM-SOAR%20Integration%20for%20Unauthorized%20Active%20Directory%20Logins.drawio.png)

The logical diagram illustrates how communication and data flow are standardized across systems. The scenario simulates an adversary gaining unauthorized access to a domain-joined test machine. This triggers telemetry from the test machine to Splunk, which detects the unauthorized sucessful login and initiates two actions:

1. Sends an alert notification to Slack, a multi-purpose platform used for communication, threat detection, and incident response as part of this project’s workflow.

2. Sends a webhook to Shuffle, which triggers an automated SOAR playbook in response to the unauthorized login attempt. The playbook notifies a Security Personnel, who decides to either disable the compromised account or take no action. If the account is disabled, Shuffle & the Domain Controller executes the action, and a confirmation message is sent to Slack.

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










