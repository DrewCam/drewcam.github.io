---
title: Threat Hunt - Volt Typhoon
draft: false 
date: 2023-10-02
description: Threat hunting Volt Typhoon
categories:
  - Threat Hunting
tags:
  - Threat Hunting
  - Volt Typhoon

---

# Threat Hunt - Volt Typhoon

## Hypothesis

Based on the advisory from the [ACSC](https://www.cyber.gov.au/about-us/advisories/prc-state-sponsored-cyber-actor-living-off-the-land-to-evade-detection), [WA Cyber Security Unit (DGOV Technical)](https://soc.cyber.wa.gov.au/advisories/20230525001-State-Sponsored-Cyber-Actor-Living-off-the-Land-to-Evade-Detection/?h=volt) and observations detailed by [Microsoft](https://www.cyber.gov.au/about-us/advisories/prc-state-sponsored-cyber-actor-living-off-the-land-to-evade-detection) on the Volt Typhoon's methods, the hypothesis for this threat hunt revolves around the likelihood that the Volt Typhoon threat group is actively exploiting critical infrastructure systems within our monitored environments by employing living-off-the-land techniques. These techniques include the creation of domain controller installation media, establishment of internal proxies, and use of custom FRP executables to avoid detection and maintain persistence.

<!-- more -->

### Assumptions

1. Volt Typhoon could use known living-off-the-land binary (LOLBin) commands and procedures, as indicated by past behaviours targeting similar entities in the U.S.
2. There is a potential that despite no detections by current signature-based tools, these adversaries might be operating undetected due to their sophisticated use of native tools and processes.
3. The custom SHA256 hashes associated with FRP executables represent Volt Typhoon's custom tools, previously identified in engagements and potentially updated to evade signature-based detection.

### Investigation Goals

- **Verify the absence of known Volt Typhoon tactics:** Use historical and current logs to ensure that there have been no executions of suspicious command lines or process creations that align with Volt Typhoon's modus operandi.
- **Scrutinize for anomalies or outliers:** Given the group's known reliance on LOLBins and stealth tactics, any unusual use of system tools or commands related to domain controller configuration, proxy settings, and known malicious hashes should be closely examined.
- **Proactive threat identification:** Even if Volt Typhoon's specific signatures are not found, it is critical to identify deviations from baseline behaviours that might indicate similar adversary TTPs (Tactics, Techniques, and Procedures).

### Hypothesized Activities to Monitor

- **Creation of Domain Controller Installation Media:** Look for the execution of `ntdsutil` with commands related to creating or exporting domain controller databases, a technique possibly used for lateral movement or establishing persistence.
- **Establishment of Internal Proxies:** The use of `netsh portproxy` commands might indicate attempts to reroute internal traffic or exfiltrate data discreetly.
- **Execution of Custom FRP Executables:** Monitoring for any execution or deployment of known bad SHA256 hashed executables will help identify potentially malicious activities, even if those files have been slightly modified.

## Links

- [People's Republic of China State-Sponsored Cyber Actor Living off the Land to Evade Detection | Cyber.gov.au](https://www.cyber.gov.au/about-us/advisories/prc-state-sponsored-cyber-actor-living-off-the-land-to-evade-detection)
- [Volt Typhoon targets US critical infrastructure with living-off-the-land techniques | Microsoft Security Blog](https://www.microsoft.com/en-us/security/blog/2023/05/24/volt-typhoon-targets-us-critical-infrastructure-with-living-off-the-land-techniques/)
- [Detection Guidance for Volt Typhoon - 20230525001 - WA Cyber Security Unit (DGOV Technical)](https://soc.cyber.wa.gov.au/advisories/20230525001-State-Sponsored-Cyber-Actor-Living-off-the-Land-to-Evade-Detection/?h=volt)

## Details

Find commands creating domain controller installation media

```kusto
DeviceProcessEvents
| where ProcessCommandLine has_all ("ntdsutil", "create full", "pro")
```

Find commands establishing internal proxies

```kusto
DeviceProcessEvents
| where ProcessCommandLine has_all ("portproxy", "netsh", "wmic", "process call create", "v4tov4")
```

Find detections of custom FRP executables

```kusto
AlertEvidence
| where SHA256 in 
    ('baeffeb5fdef2f42a752c65c2d2a52e84fb57efc906d981f89dd518c314e231c', 
    'b4f7c5e3f14fb57be8b5f020377b993618b6e3532a4e1eb1eae9976d4130cc74', 
    '4b0c4170601d6e922cf23b1caf096bba2fade3dfcf92f0ab895a5f0b9a310349', 
    'c0fc29a52ec3202f71f6378d9f7f9a8a3a10eb19acb8765152d758aded98c76d', 
    'd6ab36cb58c6c8c3527e788fc9239d8dcc97468b6999cf9ccd8a815c8b4a80af', 
    '9dd101caee49c692e5df193b236f8d52a07a2030eed9bd858ed3aaccb406401a', 
    '450437d49a7e5530c6fb04df2e56c3ab1553ada3712fab02bd1eeb1f1adbc267', 
    '93ce3b6d2a18829c0212542751b309dacbdc8c1d950611efe2319aa715f3a066', 
    '7939f67375e6b14dfa45ec70356e91823d12f28bbd84278992b99e0d2c12ace5', 
    '389a497f27e1dd7484325e8e02bbdf656d53d5cf2601514e9b8d8974befddf61', 
    'c4b185dbca490a7f93bc96eefb9a597684fdf532d5a04aa4d9b4d4b1552c283b', 
    'e453e6efc5a002709057d8648dbe9998a49b9a12291dee390bb61c98a58b6e95', 
    '6036390a2c81301a23c9452288e39cb34e577483d121711b6ba6230b29a3c9ff', 
    'cd69e8a25a07318b153e01bba74a1ae60f8fc28eb3d56078f448461400baa984', 
    '17506c2246551d401c43726bdaec800f8d41595d01311cf38a19140ad32da2f4', 
    '8fa3e8fdbaa6ab5a9c44720de4514f19182adc0c9c6001c19cf159b79c0ae9c2', 
    'd17317e1d5716b09cee904b8463a203dc6900d78ee2053276cc948e4f41c8295', 
    '472ccfb865c81704562ea95870f60c08ef00bcd2ca1d7f09352398c05be5d05d', 
    '3e9fc13fab3f8d8120bd01604ee50ff65a40121955a4150a6d2c007d34807642')
```

[CreateDCInstallationMedia](https://github.com/Azure/Azure-Sentinel/blob/master/Solutions/Windows%20Security%20Events/Hunting%20Queries/CreateDCInstallationMedia.yaml)
detect attempts to create installation media from domain controllers either remotely or locally using a commandline tool called ntdsutil These media are intended to be used in the installation of new domain controllers

```kusto
(union isfuzzy=true 
    (SecurityEvent
    | where EventID == 4688
    | where CommandLine has_all ("ntdsutil", "ac i ntds", "create full")
    | project
        TimeGenerated,
        Computer,
        Account,
        Process,
        ProcessId,
        NewProcessName,
        NewProcessId,
        CommandLine,
        ParentProcessName,
        _ResourceId,
        SourceComputerId,
        SubjectLogonId,
        SubjectUserSid
    ),
    (WindowsEvent
    | where EventID == 4688 
    | extend CommandLine = tostring(EventData.CommandLine)
    | where CommandLine has_all ("ntdsutil", "ac i ntds", "create full")
    | extend
        NewProcessName = tostring(EventData.NewProcessName),
        NewProcessId = tostring(EventData.NewProcessId)
    | extend
        Process=tostring(split(NewProcessName, '\\')[-1]),
        ProcessId = tostring(EventData.ProcessId)
    | extend Account =  strcat(EventData.SubjectDomainName, "\\", EventData.SubjectUserName)
    | extend ParentProcessName = tostring(EventData.ParentProcessName)
    | extend
        SubjectUserName = tostring(EventData.SubjectUserName),
        SubjectDomainName = tostring(EventData.SubjectDomainName),
        SubjectLogonId = tostring(EventData.SubjectLogonId)
    | project
        TimeGenerated,
        Computer,
        Account,
        Process,
        ProcessId,
        NewProcessName,
        NewProcessId,
        CommandLine,
        ParentProcessName,
        _ResourceId,
        SubjectUserName,
        SubjectDomainName,
        SubjectLogonId
    )
)
```

[InternalProxies](https://github.com/Azure/Azure-Sentinel/blob/master/Solutions/Windows%20Security%20Events/Hunting%20Queries/InternalProxies.yaml) This hunting query helps to detect attempts to create proxies on compromised systems using the built-in netsh portproxy command. VoltTyphoon has been seen creating these proxies on compromised hosts to manage command and control communications.

```kusto
  (union isfuzzy=true 
    (SecurityEvent
    | where EventID == 4688
    | where CommandLine has_all ("portproxy", "netsh", "wmic", "process call create", "v4tov4", "listenport=50100")
    | project
        TimeGenerated,
        Computer,
        Account,
        Process,
        ProcessId,
        NewProcessName,
        NewProcessId,
        CommandLine,
        ParentProcessName,
        _ResourceId,
        SourceComputerId,
        SubjectLogonId,
        SubjectUserSid
    ),
    (WindowsEvent
    | where EventID == 4688 
    | extend CommandLine = tostring(EventData.CommandLine)
    | where CommandLine has_all ("portproxy", "netsh", "wmic", "process call create", "v4tov4", "listenport=50100")
    | extend
        NewProcessName = tostring(EventData.NewProcessName),
        NewProcessId = tostring(EventData.NewProcessId)
    | extend
        Process=tostring(split(NewProcessName, '\\')[-1]),
        ProcessId = tostring(EventData.ProcessId)
    | extend Account =  strcat(EventData.SubjectDomainName, "\\", EventData.SubjectUserName)
    | extend ParentProcessName = tostring(EventData.ParentProcessName) 
    | extend
        SubjectUserName = tostring(EventData.SubjectUserName),
        SubjectDomainName = tostring(EventData.SubjectDomainName),
        SubjectLogonId = tostring(EventData.SubjectLogonId)
    | project
        TimeGenerated,
        Computer,
        Account,
        Process,
        ProcessId,
        NewProcessName,
        NewProcessId,
        CommandLine,
        ParentProcessName,
        _ResourceId,
        SubjectUserName,
        SubjectDomainName,
        SubjectLogonId
    ) 
)
```

[SuspectedLSASSDump](https://github.com/Azure/Azure-Sentinel/blob/master/Solutions/Windows%20Security%20Events/Hunting%20Queries/SuspectedLSASSDump.yaml) Looks for evidence of the LSASS process being dumped either using Procdump or comsvcs.dll. Often used by attackers to access credentials stored on a system.

```kusto
  SecurityEvent 
  | where EventID == 4688
  | where CommandLine has_all ("procdump", "lsass") or CommandLine has_all ("rundll32", "comsvcs", "MiniDump")
  | extend timestamp = TimeGenerated, AccountCustomEntity = Account, HostCustomEntity = Computer
```

[PotentialImpacketExecution](https://github.com/Azure/Azure-Sentinel/blob/master/Solutions/Attacker%20Tools%20Threat%20Protection%20Essentials/Hunting%20Queries/PotentialImpacketExecution.yaml) This hunting query identifies execution of Impacket tool. Impacket is a popular tool used by attackers for remote service execution, Kerberos manipulation and Windows credential dumping.

```kusto
(union isfuzzy=true
    (SecurityEvent
    | where EventID == '5145'
    | where RelativeTargetName has 'SYSTEM32' and RelativeTargetName endswith @".tmp"
    | where ShareName has "\\\\*\\ADMIN$"
    ),
    (WindowsEvent
    | where EventID == '5145' 
    | extend RelativeTargetName= tostring(EventData.RelativeTargetName)
    | extend ShareName= tostring(EventData.ShareName)
    | where RelativeTargetName has 'SYSTEM32' and RelativeTargetName endswith @".tmp"
    | where ShareName has "\\\\*\\ADMIN$"
    | extend Account =  strcat(tostring(EventData.SubjectDomainName), "\\", tostring(EventData.SubjectUserName))
    )
)
| extend timestamp = TimeGenerated 
| extend NTDomain = split(Account, '\\', 0)[0], UserName = split(Account, '\\', 1)[0]
| extend
    HostName = split(Computer, '.', 0)[0],
    DnsDomain = strcat_array(array_slice(split(Computer, '.'), 1, -1), '.')
| extend Account_0_Name = UserName
| extend Account_0_NTDomain = NTDomain
| extend Host_0_HostName = HostName
| extend Host_0_DnsDomain = DnsDomain
```

[CreateDCInstallationMedia](https://github.com/Azure/Azure-Sentinel/blob/master/Solutions/Windows%20Security%20Events/Hunting%20Queries/CreateDCInstallationMedia.yaml) This hunting query helps to detect attempts to create installation media from domain controllers, either remotely or locally using a commandline tool called ntdsutil. These media are intended to be used in the installation of new domain controllers.

```kusto
(union isfuzzy=true 
    (SecurityEvent
    | where EventID == 4688
    | where CommandLine has_all ("ntdsutil", "ac i ntds", "create full")
    | project
        TimeGenerated,
        Computer,
        Account,
        Process,
        ProcessId,
        NewProcessName,
        NewProcessId,
        CommandLine,
        ParentProcessName,
        _ResourceId,
        SourceComputerId,
        SubjectLogonId,
        SubjectUserSid
    ),
    (WindowsEvent
    | where EventID == 4688 
    | extend CommandLine = tostring(EventData.CommandLine)
    | where CommandLine has_all ("ntdsutil", "ac i ntds", "create full")
    | extend
        NewProcessName = tostring(EventData.NewProcessName),
        NewProcessId = tostring(EventData.NewProcessId)
    | extend
        Process=tostring(split(NewProcessName, '\\')[-1]),
        ProcessId = tostring(EventData.ProcessId)
    | extend Account =  strcat(EventData.SubjectDomainName, "\\", EventData.SubjectUserName)
    | extend ParentProcessName = tostring(EventData.ParentProcessName) 
    | extend
        SubjectUserName = tostring(EventData.SubjectUserName),
        SubjectDomainName = tostring(EventData.SubjectDomainName),
        SubjectLogonId = tostring(EventData.SubjectLogonId)
    | project
        TimeGenerated,
        Computer,
        Account,
        Process,
        ProcessId,
        NewProcessName,
        NewProcessId,
        CommandLine,
        ParentProcessName,
        _ResourceId,
        SubjectUserName,
        SubjectDomainName,
        SubjectLogonId
    ) 
)
```
