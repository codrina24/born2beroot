
*This project has been created as part of the 42 curriculum by coholbur.*

# born2beroot
Debian VM server configuration


## Description

Born2beROOT is a system administration project that consists of setting up a secure Linux server using a Debian virtual machine.
The objective is to learn the server essentials, including virtualization, security, and user management, by configuring a minimal system (without a graphical interface).

## Instructions

After completing the server setup according to the requirements, no compilation is necessary. The repository must contain only the `signature.txt` file with the server’s disk signature.

## Kwnoledge Overview

  ### **Encrypted partitions using LVM**   
Implemented via LUKS (Linux Unified Key Setup), represent an important step for successful defense-in-depth. By encrypting the partitions, as the name suggests, the data becomes unreadable([ciphertext](https://en.wikipedia.org/wiki/Ciphertext)) without the decryption passphrase. In worst-case scenarios—if the server is stolen or the hard drive is replaced without being properly wiped—without this encryption, the files could be easily accessed.

The architecture of the system follows this flow:

`Physical Disk -> Partition -> LUKS Container -> LVM Physical Volume (PV) -> Volume Group (VG) -> Logical Volumes (LV) -> Filesystem`

The importance of encrypted partitions lies in the fact that this security applies to all logical volumes ( `/`, `/home`, `/var`), including metadata (filesystem structure, directory names, and file sizes are hidden), ensuring complete protection of the system.

Special mention goes to **Swap**, which acts as a temporary buffer for data when RAM is full. Swap can contain sensitive information, so if it is unprotected, it is at risk of being accessed. Encrypted LVM ensures that the Swap Logical Volume is also encrypted. 

  The well-known born2beroot project requires a strict partitioning scheme, and using LVM allows resizing partitions like `/var` or `/tmp` as needed, while LUKS encryption keeps all data, including system files and swap, secure.
  
   Check disk command:
   ```bash
     lsblk
   ```
  ### **Secure SSH configuration**
Is the primary defense mechanism against unauthorized **remote** access.

#### ***1. Check SSH service***

Check if the SSH service is active and identify the default port (usually 22) where it is running. Use the following command as root:
```bash
      systemctl status ssh
 ```
#### ***2. Change SSH port***

Configure SSH to use a non-standard port, such as 4242 instead of the default 22. This step could reduce random automated attacks that target the default port.    

Technical impact:

  - decreases the number of failed login attempts in the logs, simplifying threat monitoring     
  - helps prevent generic brute-force attacks from automated scripts that scan port 22.

#### ***3. Disable root login***

Set `PermitRootLogin no` in the SSH configuration to prevent direct root login.

Technical impact:
  - it forces a two-step authentication process. An attacker must first compromise a standard user's credentials and then bypass the `sudo` policy to gain administrative control.
  - creates an audit trail in `/var/log/auth.log` tracking which user escalated privileges and improving accountability and security monitoring.

 ### **Strong password policy**
 
 #### ***1. Install password package and configure rules***
 
 Enforce password complexity by implement rules for length and character types using `libpam-pwquality`.

 Install the package with:
 
 ```bash
apt install libpam-pwquality
```
Configure  `minlen`, `ucredit`, `dcredit`, etc., in `/etc/pam.d/common-password`.

Technical impact:
- increases the time and computational effort required to crack passwords
- protects against brute-force and dictionary attacks targeting `/etc/shadow`.

  #### ***2.Limit Password Aging***
  
Configure password expiration and reuse rules to reduce the risk of compromised passwords by editing `/etc/login.defs`.

Technical impact:
  - `PASS_MAX_DAYS` – ensures that even if a password is intercepted, it becomes useless after a set period
  - `PASS_MIN_DAYS` – prevents users from immediately reusing old passwords

  #### ***3.Ensure strong passwords for all users***
  
Both root and sudo should have high-complexity passwords, and the rules implemented should be generally applicable across all accounts.

Access the previous file `/etc/pam.d/common-password` and make sure the rule `enforce_for_root` is integrated. This applies the password rules to the root account as well.

Then, update passwords for all users to apply the new rules using this command:

```bash
passwd <username>
```

Technical impact:

  - prevents attackers from performing a “sudo brute-force” attack after gaining access to a standard account
  - protects the integrity of the Trusted Computing Base (TCB), ensuring that administrative privileges are not easily compromised.


 ### **Restricted and logged sudo usage**
Valorizes the Principle of Least Privilege (PoLP) and maintains accountability for administrative actions.
Use the following bash command for configuration :
```bash
  visudo
```
  ####  ***Path security***
  
  - define a strict PATH for sudo commands  `/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`

  ####  ***TTY***
  
  - `requiretty` force sudo to be run from a real TTY, blocking scripts or background processes from executing sudo non-interactively.
    
  ####  ***Authentication limits***
  
  - `badpass_message` and `passwd_tries` maintain security by limiting and controlling failed sudo authentication attempts.

  ####  ***Input and output login***
  - while standard sudo logging records who ran which command, enabling `log_input` and `log_output` records everything that happens during the sudo session — including what the user types and what appears on the screen

 ###  **Active firewall**

 ###  **System monitoring script using bash and cron**.

## Resources

- **Peer-to-peer learning** – The most valuable resource; this project was developed with guidance and collaboration from other 42 students.
- **Linux man pages** – Used to understand system commands, configuration files, and their options.
- **Debian documentation** – Consulted for official guidelines on system installation, security, and package management.
- **Tutorials and articles** – Used to clarify concepts related to system administration, security policies, and virtualization, and to understand the correct order in which to implement them.  

### AI Assistance

AI tools were used to help:
-  understand system administration concepts and the design of scripts

All configurations and scripts were implemented manually, and no AI was used to directly complete the project tasks.
