# Live Windows Examination 

## Objective

The Live Windows Examination lab is aimed to apply different PowerShell commands to review a Windows system for signs of a possible compromise. Many commands used will also create a baseline for Windows systems. Use different techniques to identify the IOCs 

### Skills Learned


- Show running programs.
- Show listening ports. 
- Show what accounts exist on a system. 
- Show what groups exist, and which accounts are in each group. 
- Show what network shares are available.
- Identify registry keys associated with Auto Start Extensibility Points.
- Idenitfy interesting files. 

### Tools Used


- Windows Powershell

## Steps 
1. Show running processes by invoking Get-Process command. This will show several columns of information including:

   - Handles: A count of handles (open files, sockets, and pipe resources)
   - NPM(K): The amount of non-paged memory the process is using in kilobytes.
   - PM(K): Amount of paged memory the process is using in kilobytes.
   - WS(K): Process working set size (total amount of memory allocated to a process) in kilobytes.
   - CPU(s): amount of processor time that the process has used on all processors, in seconds.
   - Id: unique identifier for a process so that the system can reference it by the numeric value, also known as PID
   - ProcessName: process name often the executable name.
  
  ![Screenshot 2025-01-14 100921](https://github.com/user-attachments/assets/88c4b249-9887-4925-b496-88b6b95c3455)

2. Return information about the process lsass

   
   ![Screenshot 2025-01-14 102150](https://github.com/user-attachments/assets/3959f5a5-f3d7-4353-b28c-354315135b22)
    - To obtain more information about the process, add a pipeline character then Select-Object -Property *  
![Screenshot 2025-01-20 165207](https://github.com/user-attachments/assets/96d97d3d-a642-4065-914c-54d2d1a73650)

3. To identify Indicators of Compromise (IOC), we will filter on additional attributes which are common to attacker Tactics, Techniques, and Procedures (TTPs).

   
![Screenshot 2025-01-20 165318](https://github.com/user-attachments/assets/b3dcfedc-6c8b-49e3-86ae-c254e0a51ad7)

4. To filter further we will use the Where-Object command which will allow us to filter results to match specific fields. The following will display list of processes where process name is explorer.

   
![Screenshot 2025-01-20 165729](https://github.com/user-attachments/assets/7b85d2ce-5436-4246-a080-d9cc86810986)


5. An example of a possible indicator of compromise would be processes running from temporary directories. We will filter the results to show any path where the word temp is present.


![Screenshot 2025-01-20 165838](https://github.com/user-attachments/assets/27a4ae38-6300-4eae-93af-80e62fdf4b9b)

The result would be a suspicious process: calcache. 

Network Enumeration
1. Run the Get-NetTCPConnection command to get list of network connections for the host. This will show network listeners and connections on the host.


![Screenshot 2025-01-20 165945](https://github.com/user-attachments/assets/f9458ec1-97ee-49e7-90a5-2cc55cc2cbea)


2. Now display only the LocalAdress, LocalPort, State, and OwningProcess fields.

   
![Screenshot 2025-01-20 170055](https://github.com/user-attachments/assets/8cccd69c-8fe4-4117-a6d9-e2b995cc03a8)

- LocalAddress: The IP address that wil accept connections for this listener or is used for the outbound connection.
- LocalPort: The local port number used for the connection; if the state is Listen, then this is the listening TCP port number.
- State: The state of network connection.
- OwningProcess: The process ID of the process that made the connection or is listening for connections.

3. Port 4444 is commonly associated with Metasploit framework. Lets further investigate the process ID that was listening on port 4444 , which is 1216. Use Get-Process to display the process details.


   

![Screenshot 2025-01-20 170400](https://github.com/user-attachments/assets/c289139b-bd5f-4b0b-bfd3-65985ab0e87b)


4. The calcache process again is what returns. Previously we saw it used a temporary directory. Let's stop the process now using Stop-Process

   
![Screenshot 2025-01-20 170503](https://github.com/user-attachments/assets/26c420d4-358e-4949-b08e-bd5dba013787)

We have successfully stopped calcache as the Get-Process no longer works. 

# Now investigate how attacker launched: Registry Startup Keys

Windows Run or RunOnce registry keys are commonly used to start a process automaticallly by holding a registry value. They are known as Autostart extensibility Points (ASEPs), which are used to start a process automatically when the system boots or a user logs in. 
- Get-ChildItem command will enumerate registry keys, and Get-ItemProperty will enumerate registry values in specific key.

  1. Use Get-ChildItem to enumerate Powershell drive HKCU
     
![Screenshot 2025-01-20 170714](https://github.com/user-attachments/assets/f58458f2-edb1-48b9-a63f-84da7a160c0a)

- Output shows Name column (registry keys), and Property column (registry values)

  2. We will now check the four ASEP registry keys for processes by using Get-ItemProperty
![Screenshot 2025-01-20 170829](https://github.com/user-attachments/assets/cbed2447-20c1-402b-9bd8-8645224658f8)
***Screenshot not available: after running
   - Get-ItemProperty "HKCU: Software\Microsoft\Windows\CurrentVersion\Run"
  We see that calcache is listed as a registry value, this corresponds to the process identified earlier.

3. To remove the Calcache value from the Run Key : Remove-ItemProperty

   
![Screenshot 2025-01-20 171307](https://github.com/user-attachments/assets/7a7c0708-ab93-476a-90cb-93dbd42043b7)

Now we have removed the ASEP value, then confirmed it by checking Get-ItemProperty.  Now we will remove the calcache.exe program


![Screenshot 2025-01-20 171502](https://github.com/user-attachments/assets/bd1a334d-14b0-40d4-a4d7-af1c35643b90)

the calcahce.exe program is now removed. 

##Differential Analysis
Now we will capture a baseline of information to use in our assessment, then comapre the current environment to the known-good baseline. 
Earlier we created three baseline files for services, scheduled tasks, and local users. 

1. Examine the files using Get-ChildItem


![Screenshot 2025-01-20 171753](https://github.com/user-attachments/assets/9b5d7993-0e1c-4141-bbce-06d7c38e665b)

These are the commands used to save the files to a baseline:
- Get-Service | Select-Object -ExpandProperty Name | Out-File "baseline/services.txt"
- Get-ScheduledTask | Select-Object -ExpandProperty TaskName | Out-File "baseline/scheduledtasks.txt"
- Get-LocalUser | Select-Object -ExpandProperty Name | Out-File "baseline/localusers.txt"

-ExpandProperty parameter will provide normal header output observed when you use -Property

2. Now save the files to the current directory:
   
![Screenshot 2025-01-20 172030](https://github.com/user-attachments/assets/0431bc3e-f87f-4d8b-b978-248e551b30d9)

#Services Differential Analysis
First lets compare the service list. Start by inspecting first ten lines using Get-Content

![Screenshot 2025-01-20 172112](https://github.com/user-attachments/assets/23f753a2-056d-4380-ad61-8e655548ff15)

2. Save the file contents in a variable called $servicesnow  & baseline into $servicesbaseline
   
![Screenshot 2025-01-22 082631](https://github.com/user-attachments/assets/056e010b-96db-4e3c-b247-7b8b7a2542e0)

3. Now we will compare the variables using Compare-Object

   
![Screenshot 2025-01-22 083529](https://github.com/user-attachments/assets/bc9bf372-57cd-4f6c-bd6b-290e3342a71a)

In this output we see new services: Microsoft eDynamics Service. This might be suspicious but we are not sure yet. 

4. Now repeat the same process comparing the Users.
   
![Screenshot 2025-01-22 084313](https://github.com/user-attachments/assets/1e63c7d4-6aa2-4b55-9f23-0273f138e925)

The new username that was added is dynamics. 

5. Now lets use differential analysis on the scheduled tasks.
   - Scheduled Tasks are jobs that run on Windows at a specific time and with a specific frequency. Trigger can be clock time or it can be based on other attribtues including obsreved events in the Windows Event Log.

     
![Screenshot 2025-01-22 090739](https://github.com/user-attachments/assets/b40a54a5-21af-4347-a6e5-ddd5cffd87c3)

The added scheduled task is Microsoft eDynamics. 

6. Lets take a look at this scheduled task  using Export-ScheduledTask

   
![Screenshot 2025-01-22 090933](https://github.com/user-attachments/assets/a846c918-79e6-45b6-9236-1ae7dfeaefb1)

Towards the bottom there is an element Actions. The scheduled task launches a command C:\Windows\dynamics.exe and it uses the sc.exe (Service Control) utility to start hte service whenever the task is executed on the host. 

##Removing Microsoft eDynamics 

1. Stop the Dynamics service using Stop-Service

   ![Screenshot 2025-01-22 091358](https://github.com/user-attachments/assets/4d13fa3b-6775-4a69-b645-b7a70a49c86a)

2. Stop the Dynamics process using Get-Process, then Stop-Process in a pipeline

   
![Screenshot 2025-01-22 091455](https://github.com/user-attachments/assets/9bb520c2-d8e5-4f27-8e87-238e7133ee9c)

3. Remove the dynamics. exe directory from the C:\Windows directory

   
![Screenshot 2025-01-22 091548](https://github.com/user-attachments/assets/4030480e-8f3d-41e4-bf99-31564cd3dc2b)

4. Delete the service using the sc utility

   ![Screenshot 2025-01-22 091703](https://github.com/user-attachments/assets/c8abe4a0-b30f-4098-91d3-2096614dffae)
