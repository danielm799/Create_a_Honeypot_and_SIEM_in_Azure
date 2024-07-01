# Create_a_Honeypot_and_SIEM_in_Azure
Another side project! I went on Azure to make a VM, turn it into a honeypot, and set up both logging and SIEM on Azure.

+ To start off, I made an account on Microsoft Azure and claim the $200 credit so I could do this project for free.
+ I went straight into making a VM. I made a new resource group called "Honeypot_Lab" to group up all of the Azure resources I was about to make.
+ In that same page, I made a VM called "The-HoneyPot". I gave it Windows 10 Pro and made an administrative account. I left it with port RDP 3389 open so I could connect to it later.
![making the honeyPot](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/dce4a03a-2531-4517-8a77-e5065525a2e8)

+ Still making the VM, I made a change the rules for the network security group so that it would allow any connection over any protocol. I named the rule: "This_is_intentional".
![make the bad rules](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/738d1a40-81a0-43dc-8ec5-788fc45c85a4)

+ I reviewed my settings and created the VM.
---
+ I needed to make sure I could get logs from the VM and read them in Azure. I created a new log analytics workspace called "respect-da-LAW". I added it to the Honeypot_Lab resource group.
![made respect da LAW](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/9464c422-b212-4a7c-8d23-4b568da28c16)

+ Enabled Microsoft Defender for Cloud for respect-da-LAW so that logs can be fetched for the log analytic workspace. I checked "on" for both Foundational CSPM and Servers, then went to data collection and selected "All Events" so that Azure will get all the data from the VM.
![set up defender plans](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/fe38b8e9-ae8c-4437-b43f-b94347a56f8b)

![select data collection plan](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/e1ff5d02-3ee7-478a-96f4-44e4da8bb95c)

+ Connected respect-da-LAW to The-HoneyPot.
+ Activated Microsoft Sentinel in Azure and added it to respect-da-LAW. This is the SIEM in Azure and was used to visualize the data from the logs.
![Screenshot 2024-06-30 212219](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/5c5c928e-eb82-41ae-bc10-a94dbd1a3ee8)

---
+ Copied the IP address of The-HoneyPot and RDP'd into it using my own PC. I entered the credentials I made when I made the VM, but before doing that, I accidentally used the wrong credentials.
![Remote desktop connection window](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/cac9de75-6de0-4264-b107-2fd8d96992b8)

![screenshot of VM](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/fb5b2cf1-db67-4c59-9fcd-c094401f15ac)


+ After opening the desktop of The-HoneyPot, I went straight to opening the Event Viewer. I saw my failed login attempt with the keyword "Audit Failure" and the task category "Logon".
+ Pinged the VM to see that the packet was not going through. I disabled the built-in Windows firewall so that The-HoneyPot could actually be a honeypot, able to be discoverable to the open internet. The ping ended up going through.
![turn off firewall in VM](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/349f57cf-b066-4608-8877-5a196ae34b83)

![ping vm from pc](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/820d447f-c44f-4593-94a7-41fa618ffbd6)


+ Downloaded a PS script [here](https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1). The script is meant run continuously, taking logs of failed logon attempts and using a unique API from [ipgeolocation](https://ipgeolocation.io/) to extract the location of the IP address that attempted to access The-HoneyPot and put them all in a text file called failed_rdp.log
  + The script will also place in some sample logs so that respect-da-LAW could see it.
![run the PS script, change API key from geolocation](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/8645b8c5-75a9-4f70-bb11-63331763a69e)

+ I made some additional failed logon attempts just to see it in both the script and Azure.
![better photo of the first failed login attempt](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/06bcc47f-2fa5-43d3-88d4-a35c562ecdc8)

---
+ In the Log Analytics workspace page, for respect-da-LAW, I went to Tables and made an MMA-based custom log. I fed failed_rdp.log to respect-da-LAW so it could know what to look for. For collection paths, I entered the path to the log as it is in The-HoneyPot VM. To finish, I named the custom log WHERE_ARE_YOU.
![create new table or log custom idk](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/84af1864-9a01-4c1d-bce5-ddb1cc337545)

![feed log](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/106d3450-76fd-4c53-8ffb-0c79384bb523)

![type path for log ](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/b610724c-d69d-4ef8-a56c-1311f0320e09)

![name the custom log](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/05aa6a65-9ad6-49c7-b22a-287f17b4fb3f)

+ After waiting for what felt like a very very long time, the logs from WHERE_ARE_YOU_CL started being read by respect-da-LAW so I could make queries
![query is finally working](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/506b4361-ab74-4450-99ad-655d6e632cab)

+ I ran the following query:
WHERE_ARE_YOU_CL
| extend 
    latitude = extract("latitude:([^,]+)", 1, RawData),
    longitude = extract("longitude:([^,]+)", 1, RawData),
    destinationhost = extract("destinationhost:([^,]+)", 1, RawData),
    username = extract("username:([^,]+)", 1, RawData),
    sourcehost = extract("sourcehost:([^,]+)", 1, RawData),
    state = extract("state:([^,]+)", 1, RawData),
    country = extract("country:([^,]+)", 1, RawData),
    label = extract("label:([^,]+)", 1, RawData),
    timestamp = extract("timestamp:([^,]+)", 1, RawData)
| where sourcehost != ''
| summarize event_count=count() by sourcehost, latitude, longitude, country, label, destinationhost, username
| project sourcehost, latitude, longitude, country, label, destinationhost, event_count, username
+ I could not figure out how to make custom fields in any other way than through a query, and I found that script online. I made some adjustments to include the username used in a failed logon attempt. I did this just to see the query results using fields based on the log.
![LAW log](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/03cbdb7d-7be3-4a55-865d-3e2b5dffc5de)
---
+ The plan at this point was to create a map to visualize where the logon attempts are coming from.
+ Went back to Microsoft Sentinel, go to Workbooks and made a new one.
![Screenshot 2024-06-30 213525](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/2ffe4627-0e00-4aef-828e-4c61b9207dbb)

+ Delete whatever widgets that was already there since all I wanted was a map.
+ I entered the following query:
WHERE_ARE_YOU_CL
| extend 
    latitude = extract("latitude:([^,]+)", 1, RawData),
    longitude = extract("longitude:([^,]+)", 1, RawData),
    destinationhost = extract("destinationhost:([^,]+)", 1, RawData),
    username = extract("username:([^,]+)", 1, RawData),
    sourcehost = extract("sourcehost:([^,]+)", 1, RawData),
    state = extract("state:([^,]+)", 1, RawData),
    country = extract("country:([^,]+)", 1, RawData),
    label = extract("label:([^,]+)", 1, RawData),
    timestamp = extract("timestamp:([^,]+)", 1, RawData)
| where sourcehost != ""
| where destinationhost != "samplehost"
| summarize event_count=count() by sourcehost, latitude, longitude, country, label, destinationhost, username
| project sourcehost, latitude, longitude, country, label, destinationhost, event_count, username
+ Made a small change to exclude the sample logs automatically made by the script.
+ Set visualization to Map. Latitude and longitude was already set to their correct fields, and "Size by" was set to event_count to visualize how many frequent a failed logon attempt is based on the location.
![making the map](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/592473f2-f6f3-4de1-af9b-922aaeab5b31)

+ Saved the map widget as "Found you!" and added it to the Honeypot_Lab resource group.
+ After waiting a few hours, I started getting some callers!
![got some more](https://github.com/danielm799/Create_a_Honeypot_and_SIEM_in_Azure/assets/33638895/7ecb198d-c8c9-4be1-b0f2-a724e66d5f3d)
