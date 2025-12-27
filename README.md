# linux-ssh-harden-access-control
This project walks through how I took a regular SSH (remote login) setup on a server and made it much more secure, step-by-step. I started by turning off direct login as the “root” super-user and requiring secure keys instead of passwords, then limited exactly which users were allowed to connect. To protect against hackers trying lots of passwords, I added fail2ban so repeated failures get blocked automatically. I also tightened the firewall so only approved IP addresses can connect, removed old/weak encryption settings, and watched the security logs to see what was really happening in the background. As optional extras, I explored adding multi-factor authentication and using a dedicated “jump box” for admins. The repo includes my cleaned-up SSH config, a before-and-after checklist, testing results, and easy-to-understand explanations of why each change matters.  

The first step is to go the sshd_config file, type sudo nano /etc/ssh/sshd_config and do the following:  
Add  
Port 22  
Protocol 2  
AddressFamily inet  
PermitRootLogin no  
PubkeyAuthentication yes  
PassowrdAuthentication no  
PermitEmptyPassword no  
ChallengeResponseAuthentication no  
Expect .ssh/authorized_keys2 to be disregarded by default in future.  
AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2  
<img width="614" height="500" alt="image" src="https://github.com/user-attachments/assets/610b4b64-e090-4bb1-9d3f-10662f6d9cb0" />  
Make sure for allowed users that have you the name of the user account that you want ssh in with listed  
<img width="949" height="728" alt="image" src="https://github.com/user-attachments/assets/b56c2ef9-36de-4678-9d70-67d92a62231f" />  
<img width="847" height="783" alt="image" src="https://github.com/user-attachments/assets/1461d2c6-8771-473e-b48e-ec56c5b1e4d2" />  
Then after making the neccessary changes and checking for spelling errors from the screenshots, type sudo systemctl restart ssh to restart ssh.  
Then generate the keygen on the host and client computers by typing ssh-keygen -t ed25519 -a 100 on a client VM or in this case WSL or Windows Subsystem For Linux.  
On the WSL Linux server, run ssh-keygen -t ed25519 to create a ssh-key to get into the hardened Linux server, then run the command ssh-copy-id -i /home/jon/.ssh/id_ed25519.pub user@x.x.x.x.   
Or run cat /home/user/.ssh/id_ed25519.pub and copy and paste the ssh key into the main server by typing cat ~./ssh/id_ed25519.pub | ssh user@x.x.x.x "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys".  
After that its time to install fail2ban, type sudo apt install fail2ban to install it, then type sudo /etc/fail2ban/jail.local to enter the fail2ban info needed to harden the server more.  


