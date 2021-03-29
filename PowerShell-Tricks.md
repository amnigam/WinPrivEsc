# PowerShell Tricks & Tips 
---

This document captures various Powershell techniques that I have come across that might be useful when enumerating, privilege escalation or just general knowledge. 
This is a running document wherein I will keep uploading new stuff as I come across.

\
\
\

### 1. Getting Help
---

```
Get-Help Command-Name
```

This will provide you with the help for a given powershell command and is very useful.  

\
\
\
\
\


### 2. Enumerating Members for a given command
---

```
PS C:\Users\Administrator> get-service | Get-Member -MemberType Property


   TypeName: System.ServiceProcess.ServiceController

Name                MemberType Definition
----                ---------- ----------
CanPauseAndContinue Property   bool CanPauseAndContinue {get;}
CanShutdown         Property   bool CanShutdown {get;}
CanStop             Property   bool CanStop {get;}
Container           Property   System.ComponentModel.IContainer Container {get;}
DependentServices   Property   System.ServiceProcess.ServiceController[] DependentServices {get;}
DisplayName         Property   string DisplayName {get;set;}
MachineName         Property   string MachineName {get;set;}
ServiceHandle       Property   System.Runtime.InteropServices.SafeHandle ServiceHandle {get;}
ServiceName         Property   string ServiceName {get;set;}
ServicesDependedOn  Property   System.ServiceProcess.ServiceController[] ServicesDependedOn {get;}
ServiceType         Property   System.ServiceProcess.ServiceType ServiceType {get;}
Site                Property   System.ComponentModel.ISite Site {get;set;}
StartType           Property   System.ServiceProcess.ServiceStartMode StartType {get;}
Status              Property   System.ServiceProcess.ServiceControllerStatus Status {get;}


PS C:\Users\Administrator>
```

In this command for example we were able to list down all the properties for various services. 
We can also list Methods by giving the following command

```
Get-Service | Get-Member -MemberType Method
```

\
\
\
\
\

### 3. Creating Objects with selected properties
---

Continuing with our previous example of listing Services - suppose we were interested only in the CanShutdown, StartType & Status properties for some service say WinRM.
In this scenario we can list these properties by using **"Select-Object"** and then specifying the properties that we want. 

```
PS C:\Users\Administrator> get-service -Name WinRm | Select-Object -Property CanShutdown,Status,StartType

CanShutdown  Status StartType
-----------  ------ ---------
       True Running Automatic
```

These are the values for the service WinRm. If you don't provide the name like WinRm then it will print the values for **each** of the services on that box. 


As an extension to this, we often come across situations wherein we would like to know all the details about a Service e.g. WinRm. In that scenario you can do it like this - by specifying a wildcard agains Property

```
PS C:\Users\Administrator> get-service -Name WinRM | Select-Object -Property *


Name                : WinRM
RequiredServices    : {RPCSS, HTTP}
CanPauseAndContinue : False
CanShutdown         : True
CanStop             : True
DisplayName         : Windows Remote Management (WS-Management)
DependentServices   : {}
MachineName         : .
ServiceName         : WinRM
ServicesDependedOn  : {RPCSS, HTTP}
ServiceHandle       : SafeServiceHandle
Status              : Running
ServiceType         : Win32ShareProcess
StartType           : Automatic
Site                :
Container           :
```

This is very useful!

\
\
\
\
\


### 4. Filtering Objects - using where-object

In this section the idea is to filter a very specific object from a list of objects returned. 
The General Format for this is as follows

> **Verb-Noun | Where-Object -Property PropertyName -operator Value**    
> **Verb-Noun | Where-Object {$_.PropertyName -operator Value}**

The second version uses the $_ operator to iterate through every object passed to the Where-Object cmdlet.

Where -operator is a list of the following operators:

    -Contains: if any item in the property value is an exact match for the specified value
    -EQ: if the property value is the same as the specified value
    -GT: if the property value is greater than the specified value

There are other values too and they can be referenced here
[MS Where-Object](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/where-object?view=powershell-7.1&viewFallbackFrom=powershell-6)


Continuing with our Services Example, suppose if we want to list all services that we can shutdown then we can do it like this.
```
PS C:\Users\Administrator> get-service | Where-Object -Property CanShutdown -eq Yes

Status   Name               DisplayName
------   ----               -----------
Running  AmazonSSMAgent     Amazon SSM Agent
Running  AWSLiteAgent       AWS Lite Guest Agent
Running  CertPropSvc        Certificate Propagation
Running  CryptSvc           Cryptographic Services
Running  Dhcp               DHCP Client
Running  DPS                Diagnostic Policy Service
Running  DsmSvc             Device Setup Manager
Running  EventLog           Windows Event Log
Running  FontCache          Windows Font Cache Service
Running  MSDTC              Distributed Transaction Coordinator
Running  PcaSvc             Program Compatibility Assistant Ser...
Running  PlugPlay           Plug and Play
Running  ProfSvc            User Profile Service
Running  Schedule           Task Scheduler
Running  StateRepository    State Repository Service
Running  TermService        Remote Desktop Services
Running  TrkWks             Distributed Link Tracking Client
Running  UmRdpService       Remote Desktop Services UserMode Po...
Running  VaultSvc           Credential Manager
Running  W32Time            Windows Time
Running  Wcmsvc             Windows Connection Manager
Running  WinDefend          Windows Defender Service
Running  Winmgmt            Windows Management Instrumentation
Running  WinRM              Windows Remote Management (WS-Manag...
```

This way we can list any property across the section of services on the box. 
The same command can also be run via the second method in the following manner

```
PS C:\Users\Administrator> get-service | Where-Object {$_.CanShutDown -eq 'Yes'}

Status   Name               DisplayName
------   ----               -----------
Running  AmazonSSMAgent     Amazon SSM Agent
Running  AWSLiteAgent       AWS Lite Guest Agent
Running  CertPropSvc        Certificate Propagation
Running  CryptSvc           Cryptographic Services
Running  Dhcp               DHCP Client
Running  DPS                Diagnostic Policy Service
Running  DsmSvc             Device Setup Manager
Running  EventLog           Windows Event Log
Running  FontCache          Windows Font Cache Service
Running  MSDTC              Distributed Transaction Coordinator
Running  PcaSvc             Program Compatibility Assistant Ser...
Running  PlugPlay           Plug and Play
Running  ProfSvc            User Profile Service
Running  Schedule           Task Scheduler
Running  StateRepository    State Repository Service
Running  TermService        Remote Desktop Services
Running  TrkWks             Distributed Link Tracking Client
Running  UmRdpService       Remote Desktop Services UserMode Po...
Running  VaultSvc           Credential Manager
Running  W32Time            Windows Time
Running  Wcmsvc             Windows Connection Manager
Running  WinDefend          Windows Defender Service
Running  Winmgmt            Windows Management Instrumentation
Running  WinRM              Windows Remote Management (WS-Manag...
```

This construct is not very clear in its applicability other than the fact that is applies the check to each object returned from the previous statement.
In our case here, the same was also performed by the previous command. However, we will keep this at the back of our mind.

\
\
\
\
\
\


