List of commands for various types of Windows Enumeration

# System Enumeration
---------------------

1. Systeminfo
> c:\windows\system32\inetsrv>systeminfo     
> Host Name:                 DEVEL   
> OS Name:                   Microsoft Windows 7 Enterprise   
> OS Version:                6.1.7600 N/A Build 7600   
> OS Manufacturer:           Microsoft Corporation   

And so on the output continues. 

We can refine this research to bring out a few of the more important parameters from our results by

> c:\windows\system32\inetsrv>systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"

And the following is going to be the output of this   
> systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"  
> OS Name:                   Microsoft Windows 7 Enterprise   
> OS Version:                6.1.7600 N/A Build 7600   
> System Type:               X86-based PC  





