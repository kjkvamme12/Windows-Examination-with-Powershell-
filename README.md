# Live Windows Examination 

## Objective

The Live Windows Examination lab is aimed to apply different PowerShell commands to review a Windows system for signs of a possible compromise. Many commands used will also create a baseline for Windows systems. 

### Skills Learned
[Bullet Points - Remove this afterwards]

- Show running programs.
- Show listening ports. 
- Show what accounts exist on a system. 
- Show what groups exist, and which accounts are in each group. 
- Show what network shares are available.
- Identify registry keys associated with Auto Start Extensibility Points.
- Idenitfy interesting files. 

### Tools Used
[Bullet Points - Remove this afterwards]

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

1. 
