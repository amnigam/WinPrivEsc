# Windows Escalation Paths
---------------------------

This document captures in my preparation journey various Escalation Paths in Windows to get to Authority \ System.  
A couple of golden resources that you need to be aware of and should be referenced frequently 
- [PayloadAllThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)
- [Sushant's Blog](https://sushant747.gitbooks.io/total-oscp-guide/content/privilege_escalation_windows.html)

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

