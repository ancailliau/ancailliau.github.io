---
title: Creating EVTX for malicious activity
tags: Sigma threat-hunting
---

[Previous post](/2022/08/16/Developing-Sigma-rules-with-chainsaw.html) explained my process for developing [Sigma](https://github.com/SigmaHQ/Sigma) rules to detect 
suspicious activity. A key element in developing such rules is the log file
itself. While incident response will provides you plenty of (large) event logs
with malicious activity, it might be time-consuming to read hundreds of
thousands log lines to find the one you are interested in detecting. Or you might
not even have the possiblity to get such event logs; For example, a new version
of [SysMon](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon) is released and you would like to test the new capabilities.

Webshell
are a very common plague in today attack and are used by almost all the 
documented threat actors. You can find plenty webshells freely available on 
GitHub. A few YARA rules to webshells are commonly available, and a few [Sigma rules](https://github.com/SigmaHQ/Sigma/search?q=webshell)
detects suspicious process spawned from common web server executables.

Let's assume for the example a webshell in C# (because that is my favorite 
language) that compiles and execute the code dynamically, like [one](https://d.uijn.nl/2019/04/18/yet-another-apt34-oilrig-leak-quick-analysis/) 
allegedly used by APT34 leaked in 2019, [SUPERNOVA](https://www.sentinelone.com/labs/solarwinds-understanding-detecting-the-supernova-webshell-trojan/) as used during the SolarWinds compromise or in [ShaPyShell](https://github.com/antonioCoco/SharPyShell) 
released a year ago. The webshell will take a source code as a parameter, will
compile the code, and execute the compiled code. A second attack we will have
a look at will drop an executable in the `C:\inetpub\wwwroot` folder. I will not provide such webshell,
there is no reason to lower the bar for any attacker.

First, I started with a clean Windows 10 machine, installed IIS and ASP.NET
components, and the webshell causing the activity I want to detect. For monitoring and detection, I'm focused on SysMon and wanted to
test the latest version that went out yesterday. I thereby installed SysMon 14
and used a [default configuration file](https://github.com/SwiftOnSecurity/sysmon-config)
with a small addition to trigger the latest capability of SysMon:

```xml
<RuleGroup name="" groupRelation="or">
    <FileBlockExecutable onmatch="include">
        <TargetFilename condition="contains all">C:\inetpub\wwwroot</TargetFilename>
    </FileBlockExecutable>
</RuleGroup>
```

Now, the testing harness is set up. We can simulate the attack and generate the
EVTX file. My process is 

1. Test the malicious activity to ensure that it works.
2. Open `eventvwr` and clear the SysMon log (or other log source I can use to detect the behavior)
3. Execute the malicious activity.
4. Refresh the `eventvwr` and export the relevant log file(s) as EVTX.

For the example, I generated the following 3 events. An event recording `w3wp.exe` writing a DLL,

```{xml}
<Event xmlns="http://schemas.microsoft.com/win/2004/08/events/event">
    <System>
        <Provider Name="Microsoft-Windows-Sysmon" Guid="{5770385f-c22a-43e0-bf4c-06f5698ffbd9}" /> 
        <EventID>11</EventID> 
        <Version>2</Version> 
        <Level>4</Level> 
        <Task>11</Task> 
        <Opcode>0</Opcode> 
        <Keywords>0x8000000000000000</Keywords> 
        <TimeCreated SystemTime="2022-08-17T14:36:03.5803606Z" /> 
        <EventRecordID>2246</EventRecordID> 
        <Correlation /> 
        <Execution ProcessID="3000" ThreadID="1232" /> 
        <Channel>Microsoft-Windows-Sysmon/Operational</Channel> 
        <Computer>MalwareLab</Computer> 
        <Security UserID="S-1-5-18" /> 
    </System>
    <EventData>
        <Data Name="RuleName">DLL</Data> 
        <Data Name="UtcTime">2022-08-17 14:36:03.568</Data> 
        <Data Name="ProcessGuid">{0fd50764-f82f-62fc-4504-000000001600}</Data> 
        <Data Name="ProcessId">8124</Data> 
        <Data Name="Image">c:\windows\system32\inetsrv\w3wp.exe</Data> 
        <Data Name="TargetFilename">C:\Windows\Temp\qriyblj4.dll</Data> 
        <Data Name="CreationUtcTime">2022-08-17 14:36:03.568</Data> 
        <Data Name="User">IIS APPPOOL\DefaultAppPool</Data> 
    </EventData>
</Event>
```

An event recording `w3wp.exe` executing a compiler `csc.exe`,

```{xml}
<Event xmlns="http://schemas.microsoft.com/win/2004/08/events/event">
    <System>
        <Provider Name="Microsoft-Windows-Sysmon" Guid="{5770385f-c22a-43e0-bf4c-06f5698ffbd9}" /> 
        <EventID>1</EventID> 
        <Version>5</Version> 
        <Level>4</Level> 
        <Task>1</Task> 
        <Opcode>0</Opcode> 
        <Keywords>0x8000000000000000</Keywords> 
        <TimeCreated SystemTime="2022-08-17T14:36:03.6081662Z" /> 
        <EventRecordID>2248</EventRecordID> 
        <Correlation /> 
        <Execution ProcessID="3000" ThreadID="1232" /> 
        <Channel>Microsoft-Windows-Sysmon/Operational</Channel> 
        <Computer>MalwareLab</Computer> 
        <Security UserID="S-1-5-18" /> 
    </System>
    <EventData>
        <Data Name="RuleName">-</Data> 
        <Data Name="UtcTime">2022-08-17 14:36:03.601</Data> 
        <Data Name="ProcessGuid">{0fd50764-fcd3-62fc-9104-000000001600}</Data> 
        <Data Name="ProcessId">6300</Data> 
        <Data Name="Image">C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe</Data> 
        <Data Name="FileVersion">4.8.4084.0 built by: NET48REL1</Data> 
        <Data Name="Description">Visual C# Command Line Compiler</Data> 
        <Data Name="Product">Microsoft® .NET Framework</Data> 
        <Data Name="Company">Microsoft Corporation</Data> 
        <Data Name="OriginalFileName">csc.exe</Data> 
        <Data Name="CommandLine">"C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe" /noconfig /fullpaths @"C:\WINDOWS\TEMP\qriyblj4.cmdline"</Data> 
        <Data Name="CurrentDirectory">C:\inetpub\wwwroot\</Data> 
        <Data Name="User">IIS APPPOOL\DefaultAppPool</Data> 
        <Data Name="LogonGuid">{0fd50764-f82f-62fc-acea-9d0000000000}</Data> 
        <Data Name="LogonId">0x9deaac</Data> 
        <Data Name="TerminalSessionId">0</Data> 
        <Data Name="IntegrityLevel">High</Data> 
        <Data Name="Hashes">MD5=F65B029562077B648A6A5F6A1AA76A66,SHA256=4A6D0864E19C0368A47217C129B075DDDF61A6A262388F9D21045D82F3423ED7,IMPHASH=EE1E569AD02AA1F7AECA80AC0601D80D</Data> 
        <Data Name="ParentProcessGuid">{0fd50764-f82f-62fc-4504-000000001600}</Data> 
        <Data Name="ParentProcessId">8124</Data> 
        <Data Name="ParentImage">C:\Windows\System32\inetsrv\w3wp.exe</Data> 
        <Data Name="ParentCommandLine">c:\windows\system32\inetsrv\w3wp.exe -ap "DefaultAppPool" -v "v4.0" -l "webengine4.dll" -a \\.\pipe\iisipmda7b0f18-2ba4-4c04-ae39-359c977c2aa7 -h "C:\inetpub\temp\apppools\DefaultAppPool\DefaultAppPool.config" -w "" -m 0 -t 20 -ta 0</Data> 
        <Data Name="ParentUser">IIS APPPOOL\DefaultAppPool</Data> 
    </EventData>
</Event>
```

And an event recording `w3wp.exe` dropping an executable in the inetpub folder,

```{xml}
<Event xmlns="http://schemas.microsoft.com/win/2004/08/events/event">
    <System>
        <Provider Name="Microsoft-Windows-Sysmon" Guid="{5770385f-c22a-43e0-bf4c-06f5698ffbd9}" /> 
        <EventID>27</EventID> 
        <Version>5</Version> 
        <Level>4</Level> 
        <Task>27</Task> 
        <Opcode>0</Opcode> 
        <Keywords>0x8000000000000000</Keywords> 
        <TimeCreated SystemTime="2022-08-17T14:36:04.2708793Z" /> 
        <EventRecordID>2252</EventRecordID> 
        <Correlation /> 
        <Execution ProcessID="3000" ThreadID="1232" /> 
        <Channel>Microsoft-Windows-Sysmon/Operational</Channel> 
        <Computer>MalwareLab</Computer> 
        <Security UserID="S-1-5-18" /> 
    </System>
    <EventData>
        <Data Name="RuleName">-</Data> 
        <Data Name="UtcTime">2022-08-17 14:36:04.270</Data> 
        <Data Name="ProcessGuid">{0fd50764-f82f-62fc-4504-000000001600}</Data> 
        <Data Name="ProcessId">8124</Data> 
        <Data Name="User">IIS APPPOOL\DefaultAppPool</Data> 
        <Data Name="Image">c:\windows\system32\inetsrv\w3wp.exe</Data> 
        <Data Name="TargetFilename">C:\inetpub\wwwroot\MyAppCopy.exe</Data> 
        <Data Name="Hashes">MD5=870000F5100C6F8F407540C96200D5C4,SHA256=DA19CCE90EA7B0CB0A0FC754BB10C015ECCC73147EBB7712F2818A44D1663BC9,IMPHASH=698FA83DA166EDB916866EF085B426D9</Data> 
    </EventData>
</Event>
```

Based on these three events, we can write Sigma rules that detects the suspicious
activity.

* The first event is suspicious because it looks abnormal for IIS (`Image` ends with `w3wp.exe`) to write (`EventID: 11`) a DLL in the temp folder (`TargetFilename` starts with `C:\Windows\Temp\` and ends with `.dll`, avoid regular expression where possible).
* The second event is suspicious because it looks abnormal for IIS (`ParentImage` ends with `w3wp.exe`) to call (`EventID: 1`) spawn a C# compiler (`Image` ends with `csc.exe`).
* The third event is suspicious because it looks abnormal for IIS (`Image` ends with `w3wp.exe`) to drop an executable (`EventID: 27`).

Because the first two events are related to the dynamic execution of C# code
within IIS, I decided to group them within a single Sigma rule.

```yaml
title: Dynamic C# Webshell compilation by IIS
id: ebf2c122-d2fc-416c-a90c-698a4ba9161e
status: experimental
description: Detects dynamic compilation of C# code executed by IIS.
tags:
    - attack.execution
author: Antoine Cailliau
date: 2022/08/17
logsource:
    category: process_creation
    product: windows
detection:
    selection_csc:
        EventID: 1
        ParentImage|endswith: 'w3wp.exe'
        Image|endswith: 'csc.exe'
    selection_compiled_output:
        EventID: 11
        Image|endswith: 'w3wp.exe'
        TargetFilename|endswith: 'dll'
        TargetFilename|startswith: 'C:\Windows\Temp'
    condition: selection_csc or selection_compiled_output
level: medium
falsepositives:
    - Legitimate dynamic web applications
```

The third event is not related to that specific type of webshell and we created
a second Sigma rule.

```yaml
title: IIS writing executable file
id: 2f8504e9-b715-4a7f-bb7c-fa3100dc8246
status: experimental
description: Detects a blocked executable dropped by IIS
tags:
    - attack.execution
author: Antoine Cailliau
date: 2022/08/17
logsource:
    category: file_creation
    product: windows
detection:
    selection:
        EventID: 27
        Image|endswith: 'w3wp.exe'
    condition: selection
level: low
falsepositives:
    - Unknown
```

Now, we complete the process described [here](/2022/08/16/Developing-Sigma-rules-with-chainsaw.html) by testing the rules on many EVTX to avoid false positive and include more true positive.

I do not claim that these rules are fool proof and they are likely to require
some tuning depending on your environment. Knowing your baseline is key aspect
in detection, and could be part of a complete post series. 

