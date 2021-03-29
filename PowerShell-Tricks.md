# PowerShell Tricks & Tips 
---

This document captures various Powershell techniques that I have come across that might be useful when enumerating, privilege escalation or just general knowledge. 
This is a running document wherein I will keep uploading new stuff as I come across.

\
\
\

### Getting Help
---

```
Get-Help Command-Name
```

This will provide you with the help for a given powershell command and is very useful.
\
\
\

### Enumerating Members for a given command

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
 