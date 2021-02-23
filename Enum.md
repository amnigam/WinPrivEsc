Before I begin I also want to highlight a wonderful resource relating to Windows PrivEsc -- PayloadAllThings
```
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#user-enumeration
```


List of commands for various types of Windows Enumeration

# System Enumeration
---------------------

1. Systeminfo

> **systeminfo**

``` 
Host Name:                 DEVEL   
OS Name:                   Microsoft Windows 7 Enterprise   
OS Version:                6.1.7600 N/A Build 7600   
OS Manufacturer:           Microsoft Corporation   
```

And so on the output continues. 

We can refine this research to bring out a few of the more important parameters from our results by

> **systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"**

And the following is going to be the output of this   
```
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"  
OS Name:                   Microsoft Windows 7 Enterprise   
OS Version:                6.1.7600 N/A Build 7600   
System Type:               X86-based PC  
```


2. Get Patching Information from WMIC

> **wmic qfe**

This gives us the "quick fix engineering" or hot fixes via the Windows Management Instrumentation service - the c stands for command line
We can further narrow down our search by specifying exactly what is it that we want

> **wmic qfe get Caption,Description,HotFixID,InstalledOn**

And an output along these lines will show up
```
Caption                                     Description      HotFixID   InstalledOn   
http://support.microsoft.com/?kbid=4601056  Update           KB4601056  2/11/2021  
http://support.microsoft.com/?kbid=4497165  Update           KB4497165  5/23/2020  
```

3. Listing Drives on the Box
For this we can utilize 2 commands

> **list drives**  
This one might or might not work.

The other command utilizes the WMIC service and can be used to pull down the Drives information. 

> **wmic logicaldisk get Caption,Description,ProviderName**  

And you will get an output like this one
```
C:\Users\ACER>wmic logicaldisk get Caption,Description,ProviderName  
Caption  Description       ProviderName  
C:       Local Fixed Disk  
D:       Local Fixed Disk  
```


# User Enumeration
-------------------

Listing of commands that can be utilized to enumerate a User on a given box.

1. **WHOAMI**  


2. To list all the privileges associated with a given user

> **whoami /priv**  

This will give us the privileges associated with the user that we are logged in with.

```
** PRIVILEGES INFORMATION **  

Privilege Name                Description                               State     
============================= ========================================= ========  
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled  
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled  
SeShutdownPrivilege           Shut down the system                      Disabled  
SeAuditPrivilege              Generate security audits                  Disabled  
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled   
SeUndockPrivilege             Remove computer from docking station      Disabled  
SeImpersonatePrivilege        Impersonate a client after authentication Enabled   
SeCreateGlobalPrivilege       Create global objects                     Enabled   
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled  
SeTimeZonePrivilege           Change the time zone                      Disabled  
```

3. List all Group Memberships for the logged on user

> **whoami /groups**

```
Group Name                           Type             SID          Attributes                                        
==================================== ================ ============ ==================================================
Mandatory Label\High Mandatory Level Label            S-1-16-12288                                                   
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\SERVICE                 Well-known group S-1-5-6      Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                        Well-known group S-1-2-1      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
BUILTIN\IIS_IUSRS                    Alias            S-1-5-32-568 Mandatory group, Enabled by default, Enabled group
LOCAL                                Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
                                     Unknown SID type S-1-5-82-0   Mandatory group, Enabled by default, Enabled group
```

4. To list **users** on a machine or a box we use
> **net user**

This will give us a listing of **all** users on a box or machine.

Example - in the following output for Devel box we see only 3 users

```
User accounts for \\

-------------------------------------------------------------------------------
Administrator            babis                    Guest                    
The command completed with one or more errors.
```

5. Getting information specifically about a User

Continuing from our previous example - suppose we want more information about "babis" user then we can do

> **net user babis**

This will give
```
c:\windows\system32\inetsrv>net user babis
net user babis
User name                    babis
Full Name                    
Comment                      
User's comment               
Country code                 000 (System Default)
Account active               Yes
Account expires              Never

Password last set            18/3/2017 1:15:19 ��
Password expires             Never
Password changeable          18/3/2017 1:15:19 ��
Password required            No
User may change password     Yes

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   18/3/2017 1:17:50 ��

Logon hours allowed          All

Local Group Memberships      *Users                
Global Group memberships     *None                 
The command completed successfully.
```

We can see a whole bunch of information along with the memberships for this user. And in this case he is just a simple Users (group) member. 


6. Getting Group Information

We can also list the groups available on the box with 

> **net localgroup**

However, this command might or might not work. 
In case if we know the groups that exist on a system e.g. by doing net user *username* on that box. We can then ascertain the nature of that group. 

> **net localgroup administrators**

```
C:\Users\ACER>net localgroup Administrators
Alias name     Administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
ACER
Administrator
The command completed successfully.
```

This way we can find out memberships of Users in key groups.


# Network Enumeration
----------------------

Network enumeration begins with standard IPCONFIG which will reveal the IP address along with DNS and other related parameters.

1. **IPCONFIG**

This gives basic information about network related items.


2. IPCONFIG /ALL

> **ipconfig /all**

This provides a whole host of information relating to network e.g. DHCP, DNS, Host details, Network IPs etc.
```
c:\windows\system32\inetsrv>ipconfig /all
ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : devel
   Primary Dns Suffix  . . . . . . . : 
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No

Ethernet adapter Local Area Connection 3:

   Connection-specific DNS Suffix  . : 
   Description . . . . . . . . . . . : vmxnet3 Ethernet Adapter #3
   Physical Address. . . . . . . . . : 00-11-22-33-44-55
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   IPv6 Address. . . . . . . . . . . : dead:beef::58c0:f1cf:abc6:bb9e(Preferred) 
   Temporary IPv6 Address. . . . . . : dead:beef::984:d99c:3ed7:3890(Preferred) 
   Link-local IPv6 Address . . . . . : fe80::58c0:f1cf:abc6:bb9e%15(Preferred) 
   IPv4 Address. . . . . . . . . . . : 10.10.10.5(Preferred) 
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : fe80::250:56ff:feb9:5d92%15
                                       10.10.10.2
   DNS Servers . . . . . . . . . . . : fec0:0:0:ffff::1%1
                                       fec0:0:0:ffff::2%1
                                       fec0:0:0:ffff::3%1
   NetBIOS over Tcpip. . . . . . . . : Enabled
```

This information is critical - sometimes the Domain Controller does DNS and by knowing the IP of DNS we can guess where the domain controller might be located.


3. ARP information

The ARP information can reveal other machines or boxes on the same network. 
We can get that information through 

> **arp -a**

Continuing with our example - we have the following
```
c:\windows\system32\inetsrv>arp -a
arp -a

Interface: 10.10.10.5 --- 0xf
  Internet Address      Physical Address      Type
  10.10.10.2            00-50-56-b9-5d-92     dynamic   
  10.10.10.255          ff-ff-ff-ff-ff-ff     static    
  224.0.0.22            01-00-5e-00-00-16     static    
  224.0.0.252           01-00-5e-00-00-fc     static
  ```

  As can be seen - 10.10.10.5 is our address -- 10.10.10.2 is the gateway or router that is doing DNS for us. Also 255 is the broadcast ID. 


  4. Route Information

  We can print the Route details for a given box with the following command

  > **route print**

  ```
  c:\windows\system32\inetsrv>route print
route print
===========================================================================
Interface List
 15...00 11 22 33 44 55 ......vmxnet3 Ethernet Adapter #3
  1...........................Software Loopback Interface 1
 11...00 00 00 00 00 00 00 e0 Microsoft ISATAP Adapter
 12...00 00 00 00 00 00 00 e0 Teredo Tunneling Pseudo-Interface
===========================================================================

IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0       10.10.10.2       10.10.10.5    261
       10.10.10.0    255.255.255.0         On-link        10.10.10.5    261
       10.10.10.5  255.255.255.255         On-link        10.10.10.5    261
     10.10.10.255  255.255.255.255         On-link        10.10.10.5    261
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    306
        127.0.0.1  255.255.255.255         On-link         127.0.0.1    306
  127.255.255.255  255.255.255.255         On-link         127.0.0.1    306
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    306
        224.0.0.0        240.0.0.0         On-link        10.10.10.5    261
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    306
  255.255.255.255  255.255.255.255         On-link        10.10.10.5    261
===========================================================================
Persistent Routes:
  Network Address          Netmask  Gateway Address  Metric
          0.0.0.0          0.0.0.0       10.10.10.2  Default 
          0.0.0.0          0.0.0.0       10.10.10.2  Default 
          0.0.0.0          0.0.0.0       10.10.10.2  Default 
===========================================================================
```


5. Open Ports and related information

Sometimes certain ports are open and are visible only from the inside. These ports might not be revealed in an NMAP scan and hence it is important to understand what kind of services are running internally and are not visible from outside. NETSTAT can help us in identifying these ports and related traffic.

> **netstat -ano**

```
c:\windows\system32\inetsrv>netstat -ano
netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:21             0.0.0.0:0              LISTENING       1444
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       712
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:5357           0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:49152          0.0.0.0:0              LISTENING       416
  TCP    0.0.0.0:49153          0.0.0.0:0              LISTENING       776
  TCP    0.0.0.0:49154          0.0.0.0:0              LISTENING       896
  TCP    0.0.0.0:49155          0.0.0.0:0              LISTENING       520
  TCP    0.0.0.0:49156          0.0.0.0:0              LISTENING       528
  TCP    10.10.10.5:139         0.0.0.0:0              LISTENING       4
  TCP    10.10.10.5:49161       10.10.14.2:4444        ESTABLISHED     2796
  ```

  I have only put in a short sample of the output. But you get the idea.




# Basic Password Hunting
---------------------------

Here we will do some basic password hunting to supplement the Tools that we will utilize as part of enumeration.

A lot of the content is available on the following link *https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_windows.html*   
It has some very nice notes that we can leverage as well. 

However here are some of the things that I found

1. Basic String search 

> **findstr /si password \*.txt**

This will search for string password in all .txt files.


2. Searching for any type of file through CMD

> **dir \*.txt /s /p**

This will search for all files of type .txt from the folder from where this command is executed. 




# Basic Firewall & AV Enumeration
---------------------------------

Again refer to PayloadAllThings & Sushant's guide for more info. I am just making some very basic notes.

1. Query Service Center for info on WinDefend

> **sc query windefend**

This will give you info on whether the windows defender is running or not. Here is the output of that command

```
c:\windows\system32\inetsrv>sc query windefend
sc query windefend

SERVICE_NAME: windefend 
        TYPE               : 20  WIN32_SHARE_PROCESS  
        STATE              : 4  RUNNING 
                                (STOPPABLE, NOT_PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

c:\windows\system32\inetsrv>
```


2. Query All Services on Service Center

We can query all services on Service center and then pick out the one that we want to query in depth later.

> **sc queryex type= service**

This will produce a humongous output - I am only showing the most basic snipped version 

```
c:\windows\system32\inetsrv>sc queryex type= service
sc queryex type= service

SERVICE_NAME: AppHostSvc
DISPLAY_NAME: Application Host Helper Service
        TYPE               : 20  WIN32_SHARE_PROCESS  
        STATE              : 4  RUNNING 
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
        PID                : 1344
        FLAGS              : 

SERVICE_NAME: AudioEndpointBuilder
DISPLAY_NAME: Windows Audio Endpoint Builder
```


3. Use NETSH for Firewall Checks

We can use the NETSH utility to do further enumeration. Go through the NETSH help utility carefully

> **netsh firewall show state**

This will give a state of the firewall on the box. Here is the output 

```
c:\windows\system32\inetsrv>netsh firewall show state
netsh firewall show state

Firewall status:
-------------------------------------------------------------------
Profile                           = Standard
Operational mode                  = Enable
Exception mode                    = Enable
Multicast/broadcast response mode = Enable
Notification mode                 = Enable
Group policy version              = Windows Firewall
Remote admin mode                 = Disable

Ports currently open on all network interfaces:
Port   Protocol  Version  Program
-------------------------------------------------------------------
No ports are currently open on all network interfaces.

IMPORTANT: Command executed successfully.
However, "netsh firewall" is deprecated;
use "netsh advfirewall firewall" instead.
For more information on using "netsh advfirewall firewall" commands
instead of "netsh firewall", see KB article 947709
at http://go.microsoft.com/fwlink/?linkid=121488 .
```



> **netsh advfirewall firewall dump** 

This is a more modern version of the NETSH command for the same purpose.


4. Use NETSH for firewall config

> **netsh firewall show config**

This will pull down the configuration for the firewall

```
c:\windows\system32\inetsrv>netsh firewall show config
netsh firewall show config

Domain profile configuration:
-------------------------------------------------------------------
Operational mode                  = Enable
Exception mode                    = Enable
Multicast/broadcast response mode = Enable
Notification mode                 = Enable

Allowed programs configuration for Domain profile:
Mode     Traffic direction    Name / Program
-------------------------------------------------------------------

Port configuration for Domain profile:
Port   Protocol  Mode    Traffic direction     Name
-------------------------------------------------------------------
```

I have snipped out the output to make it snappier. But there is more output relating to ports etc.


5. Dump all firewall rules

> **netsh advfirewall firewall show rule name=all > firewall-rules.txt**

This will take all firewall rules and copy it in a separate file (firewall-rules.txt)



# Final Words
--------------

There are certain services like 
1. NETSH
2. SC 
3. WMIC
4. NET USER

All of these services - they have their own individual help contexts that one can leverage to get more information. 
As an example

> **netsh firewall ?**

This will bring down its own help menu for firewall related commands.

```
c:\windows\system32\inetsrv>netsh firewall ?
netsh firewall ?

The following commands are available:

Commands in this context:
?              - Displays a list of commands.
add            - Adds firewall configuration.
delete         - Deletes firewall configuration.
dump           - Displays a configuration script.
help           - Displays a list of commands.
set            - Sets firewall configuration.
show           - Shows firewall configuration.

To view help for a command, type the command, followed by a space, and then
 type ?.
```

You can look at these further. 

Similarly for WMIC you can get information on similar lines. 

> **wmic /?** 

This will list all the aliases associated with this service. Then for each of those aliases you can further query the help menus. 


And we can query for Service Center as well.

> **sc ?**

This lists all service center options along with query & queryex (ex stands for extended)

```
DESCRIPTION:
        SC is a command line program used for communicating with the
        Service Control Manager and services.
USAGE:
        sc <server> [command] [service name] <option1> <option2>...


        The option <server> has the form "\\ServerName"
        Further help on commands can be obtained by typing: "sc [command]"
        Commands:
          query-----------Queries the status for a service, or
                          enumerates the status for types of services.
          queryex---------Queries the extended status for a service, or
                          enumerates the status for types of services.
          start-----------Starts a service.
          pause-----------Sends a PAUSE control request to a service.
          interrogate-----Sends an INTERROGATE control request to a service.
          continue--------Sends a CONTINUE control request to a service.
          stop------------Sends a STOP request to a service.
          config----------Changes the configuration of a service (persistent).
          description-----Changes the description of a service.
          failure---------Changes the actions taken by a service upon failure.
```

I have only taken a part of the output. 
Keep all of this in mind when you are doing any kind of enumeration. 

Even the NET USER and the NET command itself has a host of options.

> **net ?**

This reveals the true amount of options that we have which includes the USER category that we use so often.

```
C:\WINDOWS\system32>net ?
The syntax of this command is:

NET
    [ ACCOUNTS | COMPUTER | CONFIG | CONTINUE | FILE | GROUP | HELP |
      HELPMSG | LOCALGROUP | PAUSE | SESSION | SHARE | START |
      STATISTICS | STOP | TIME | USE | USER | VIEW ]
```






