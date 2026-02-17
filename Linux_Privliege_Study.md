# Linux security basic I learnt today (17/02/26)

### 1. adduser and switching
-adduser : sudo adduser [user id] : 'sudo adduser ilgyu' - I created the new user account 'ilgyu' in the system.
-Subtitute user : su [user ID] :  ' su -ilgyu'  - I switched the current account to 'ilgyu' which is I created.( The '-' option brings all environment settings).

### 2. Privilege Seperation and Security
- 'permission denied' verified that a regular user cannot read core system files (cat /etc/shadow)
- I switched to orgin user with prompt 'exit'

### 3. Granting administrative privileges 'sudo usermod -aG sudo [user id]'
- I gave the regular user the 'sudo' group key

### 4. Understanding system structure and permision.
- 'pwd'(print working Diretory) to checking my current location in the directory tree
- 'ls -l' to checking detailed file info and the status of locks ('rwx')
- 'rwx : r for Read, w for write, x for excute.
- ' Directory tree'  I understood that everything starts from root ('/') '/etc' is for configurations, and '/home' is for user room.
