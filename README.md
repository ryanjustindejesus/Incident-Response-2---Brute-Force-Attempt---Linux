<h1>Incident Response: Brute Force Attempt - Linux</h1>

- <b>This tutorial outlines the configuration of performing incident response for a brute force attempt on our Linux Syslog virtual machine using Microsoft Sentinel and Log Analytics Workspace</b>

<h2>Environments and Technologies Used</h2>

- <b>Microsoft Azure</b> 
- <b>Microsoft Sentinel</b>
- <b>Log Analytics Workspace</b>

<h2>Operating Systems</h2>

- <b>Windows 10</b>

<h2>Configuration Steps</h2>

![image](https://github.com/user-attachments/assets/59b3b430-cb82-4e24-97f2-923ca7bb11c5)
- <b>Navigate to Microsoft Sentinel and click a CUSTOM: BRUTE FORCE ATTEMPT LINUX SYSLOG incident</b>
- <b>Set Owner: Ryan Justin De Jesus, Status: Active, Severity: Medium</b>
- <b>Click view full details</b>

![image](https://github.com/user-attachments/assets/0280418f-8307-4008-b57c-f562d9ef10b6)
- <b>Click activity log and observe the activity log</b>

![image](https://github.com/user-attachments/assets/7faf20bd-4b4f-4a95-b7f2-696fd2ca9cd6)
- <b>Click investigation on the bottom left</b>

![image](https://github.com/user-attachments/assets/7d510b8c-b1cb-4d78-adfc-42e99432f910)
- <b>Investigate and determine the scope</b>

![image](https://github.com/user-attachments/assets/568cc113-e50b-4478-be0a-8dcc34ab49b0)
- <b>Click the IP Address and observe the related event</b>
- <b>This specific incident is related to 42+ events</b>

![image](https://github.com/user-attachments/assets/f7613b7a-0de2-4afa-989f-a61df32023ae)
- <b>More Information is presented in the Log Analytics Workspace from this query:</b>

``` 
let GetIPRelatedAlerts = (v_IP_Address: string) {
    SecurityAlert
    | summarize arg_max(TimeGenerated, *) by SystemAlertId
    | extend entities = todynamic(Entities)
    | mv-expand entities
    | project-rename entity=entities
    | where entity['Type'] == 'ip' and entity['Address'] =~ v_IP_Address
    | project-away entity
};
GetIPRelatedAlerts(@'183.81.169.238')
```

```
SecurityEvent
| where EventID == 4625
| where IpAddress == '183.81.169.238'
```
 
![image](https://github.com/user-attachments/assets/ebe15060-692a-47c0-be5e-7e9d389ee438)
- <b>Determine the legitimacy of the Incident (True Positive, False Positive, etc.)</b>
- <b>Based on the results, I will conclude this as a True Positive - Suspicious Activity due to the results containing multiple failed authentication attempts by the same user account or from the same IP address, further suggesting malicious intent.

## Incident Management Playbook 
- <b>Incident Description</b>
    - This incident involves observation of potential brute force attempts against a Linux Syslog.

- <b>Initial Response Actions</b>
    - Verify the authenticity of the alert or report.
    - Immediately isolate the machine and change the password of the affected user
    - Identify the origin of the attacks and determine if they are attacking or involved with anything else
    - Determine how and when the attack occurred
        - Are the NSGs not being locked down? If so, check other NSGs
    - Assess the potential impact of the incident.
        - What type of account was it? Permissions?

## Containment and Recovery

![image](https://github.com/user-attachments/assets/86ee1b28-092f-4490-b5b1-c0bc58562da9)
- <b>Lock down the NSG assigned to that VM/Subnet, either entirely, or to allow only necessary traffic</b>

![image](https://github.com/user-attachments/assets/0f43e8a9-e329-405e-9467-d57dbbd165ab)
- <b>Reset the affected user’s password</b>
