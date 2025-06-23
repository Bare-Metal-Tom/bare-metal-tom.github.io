## The Problem with Passwords-

It can be very easy to set up the ssh service on one of your servers and then connect to it using the command -

    ssh username@10.10.10.1
    #change example username and IP address to match real username and IP
    #then enter "yes" to accept the fingerprint of the host your are connecting to
    #IF you trust this host. And then finally input your password

This CAN work well, especially if you are on trusted network. BUT you probably want to opt for a more secure authentication method if you are connecting to a server over an untrustworthy network such as the public internet or perhaps a large corporate network.

WHY though? Well, I might want to be able to access my server remotely from anywhere in the world. Meaning, SSH traffic is allowed to my server (inbound) from ANYWHERE in the world. It won't be long before a bad actor, human or bot, stumbles upon your server and tries to connect. If local authentication (basic username/password credentials stored in a database on your server) is the only barrier... Let the brute forcing begin. 

Brute Force attacks are when someone attempts to login to your computer by continually trying different username/password combinations until they find the one that matches a valid user on the computer. Most of the time, these attacks are automated using hacking tools like- Hydra, Medusa, Ncrack, etc.

This is where the benefit of public key authentication comes in- when enabled, there is no password prompt, instead the server (host you are connecting to) sends a cryptographic challenge instead of a password check. This challenge can only be solved if your device contains the matching private key.

From here their two optional practices you can implement to further harden your SSH remote server access security:

- Configure a passphrase- This not considered 2 factor authentication, but adds an extra layer of security just in case your private key gets stolen 
- Disable Password authentication- This ensures that the only accepted method of authentication is public key authentication

***NOTE: We will cover these practices in the tutorial below***

## STEP 1 --- Enable OpenSSH Client on the Windows Client Computer

- Open a Powershell terminal as an Administrator
- Run this cmdlet to install the OpenSSH Client-
	- ```Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0```
- The command should return-
	![[Pasted image 20250622180545.png]]


## STEP 2 --- Enable/Configure OpenSSH Server on the Windows 2022 Core Server

- Open a Powershell terminal as an Administrator
- Run this cmdlet to install the OpenSSH Client-
	- ```Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0```
- The command should return-
	![[Pasted image 20250622180545.png]]
- Run this cmdlet to start the sshd service and then make this service automatically start anytime the server machine boots 
	- ```Start-Service sshd```
	- ```Set-Service -Name sshd -StartupType 'Automatic'```
- Configure the default shell for OpenSSH so that you can use Powershell cmdlets when you connect from client machine
	- ```New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -PropertyType String -Force```

### STOP! --- If Having SSH With Basic Password Auth Is Your Goal, You Can Stop Here. For Public Key Auth, Continue

## STEP 3 --- Generate Your Client Keys On Client Computer

- Run this cmdlet to generate key files (ECDSA or ED25519 are recommended)
	- ```ssh-keygen -t ecdsa```
	- You will be prompted to select a file path for the key files (I leave this default)
	- Next you will prompted to create a passphrase for the key files
	- When the process completes you will see something called a Randomart Image
		- This image is an ASCII art grid that servers as a quick visual summary of the key's fingerprint
		- Example:
		- ![[Pasted image 20250622185031.png]]

## STEP 4 --- Manually Copy the Client Public Key From the Client Computer To the Server

- Open Window's File Explorer and navigate to the location of the public key (the default key location is C:\Users\<YourUsername>\.ssh\id_ecdsa.pub)
- Open the pub key file with notepad
- Copy all of the text, make sure there are no trailing whitespaces
- SSH into the server and execute the following cmdlets:
	- ```"<pasted_public_key_file_contents>" | Out-File -FilePath "C:\example.txt"```
	- This will create the file on the server if it doesn't exist
	- CAUTION- Will completely overwrite the file if it does already exist
	- Use the ```-Append``` flag to add to the file without overwriting

## STEP 5 --- Append Client Public Key To the Authorized Keys File On the Server

- ```Get-Content "<public_key_absolute_path>" | Add-Content "$env:ProgramData\ssh\administrators_authorized_keys"```
- Create the file administrators_authorized_keys if it doesn't exist (no file extension)

## STEP 6 --- Apply the Correct ACLs to the Authorized Key File Using One of the Server's Private Key Files

- ```get-acl "$env:programdata\ssh\ssh_host_ecdsa_key" | set-acl "$env:programdata\ssh\administrators_authorized_keys"```

## STEP 7 --- Try Out the Key-based Auth From Your Client Machine

- ```ssh username@<server_ip_address>```
- If the steps above were completed properly, you will not be prompted for a password. You will either connect automatically (if you didn't set a passphrase) OR you will be prompted for the passphrase you created, like this:
	- ```Enter passphrase for key 'C:\Users\Username\.ssh\id_ecdsa':```

## STEP 8 (OPTIONAL) --- Disable Password Auth

- Navigate into this folder -  ```"C:\ProgramData\ssh\"```
- Read and Modify this file- ```sshd_config```
- Ensure that these two lines are present in the config and not committed out-
	- ```PasswordAuthentication no```
	- ```PubkeyAuthentication yes```
- Recommended- make a backup file prior to modification-
	- ```Copy-Item C:\ProgramData\ssh\sshd_config C:\ProgramData\ssh\sshd_config.bak```
- After the config has been changed, restart the sshd service-
	- ```Restart-Service sshd```
- CAUTION- Only do this if you confirmed that public key encryption is working first
