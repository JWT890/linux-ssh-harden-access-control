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
<img width="153" height="136" alt="image" src="https://github.com/user-attachments/assets/98ab2afe-a083-4601-83f9-bdfd0aca63cd" />  
Enter in this for now.  
Then check to see if the server is ready by typing sudo fail2ban-server -t, then run sudo systemctl enable fail2ban then sudo systemctl start fail2ban. Then run sudo fail2ban-client status sshd to see if its up and running.  
To ensure the fail2ban alerts are being sent to your email, you will need to install postfix. Type sudo apt install postfix -y and then type sudo nano /etc/postfix/main.cf after installation.  
In the main.cf file, go all the way down and enter in this information:  
<img width="494" height="115" alt="image" src="https://github.com/user-attachments/assets/278c5ab7-96af-4e93-9f38-846550e0c8e5" />  
Then save it and type sudo nano /etc/postfix/sasl_password and add this [smtp.gmail.com]:587 jwtaylor659@gmail.com:app_password.  
Then save it and enter sudo chmod 600 /etc/postfix/sasl_passwd, then sudo postmap /etc/postfix/sasl_passwd, then sudo systemctl restart postfix.  
Then type echo "Test from PostFix" | mail -s "Test" email and press enter, then go check your email for the result:  
<img width="207" height="155" alt="image" src="https://github.com/user-attachments/assets/27fc54a1-01a1-4cc7-b485-60472718a801" />  
Then go to jail.local and add to [DEFAULT] the destination email, sender email, and the sendername along with action = %(action_mw)s and save it, then type sudo systemctl restart fail2ban. Then type sudo fail2ban-client status sshd and make sure to it looks like this:  
<img width="495" height="168" alt="image" src="https://github.com/user-attachments/assets/4cbb3544-08d3-4a20-b176-89dfadc8dff0" />  
To test this type sudo fail2ban-client set sshd banip 1.2.3.4 and press enter, then go check your email for the result:  
<img width="429" height="263" alt="image" src="https://github.com/user-attachments/assets/7ded2d64-cd9c-434c-8809-9c70bfe9eb2f" />  
<img width="635" height="700" alt="image" src="https://github.com/user-attachments/assets/a30f42bc-cf0c-4d03-ae9d-e37accca74bf" />  
Then to unban it type sudo fail2ban-client set sshd unbanip 1.2.3.4 and it will be unbanned.  
To add another layer of security, add another layer of SSH security sudo apt install libpam-google-authenticator, then after installation enter google-authenticator and press y to all the prompts and scan the QR code with a authenticator app like Duo and sign in with the cdoe from the app.  
As one way to test it, install hydra on a attacker VM and create a users.txt file with random info in it and a passwords.txt file with random passwords and enter this command hydra -L users.txt -P passwords.txt ssh://10.0.0.20 and press enter. Should see this:  
<img width="1078" height="173" alt="image" src="https://github.com/user-attachments/assets/a09ba07f-9e58-4a40-bc77-204f0a8bf0c9" />  
<img width="526" height="170" alt="image" src="https://github.com/user-attachments/assets/c5b3129c-b5a8-47f8-ba03-41c03273c8fc" />    
*Note, might go test a few more times and will document it*

# Metasploit test
To install Metasploit on WSL:  
sudo apt install -y curl gpg postgresql  
then curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall  
chmod 755 msfinstall  
sudo ./msfinstall this will take a few minutes    
sudo service postgresql start  
sudo msfdb init  *It might pop with the line Please run msfdb as a non-root user if you are doing it in WSL*  
Steps:  
sudo -u postgresql psql -c "CREATE USER $USER WITH PASSWORD 'password';"  
sudo -u postgresql psql -c "ALTER USER $USER CREATEDB;"  
sudo -c postgresql psql -c "GRANT ALL PRIVILEGES ON DATABASE msf_database to $USER;"  
Then run sudo msdb init, and then msfconsole to get it up.  

SSH Brute Force method  

To get it to run type use auxillary/scanner/ssh/ssh_login  
show options  
set RHOSTS 10.0.0.20  
set RPORT 2222  
set USERNAME jon *or whatever else*  
set PASS_FILE passwords.txt  
set THREADS 5  
set VERBOSE to true  
show options  
type run  
And the result:  
<img width="486" height="323" alt="image" src="https://github.com/user-attachments/assets/d7275de1-c30b-4334-9558-475f75b42516" />  
<img width="520" height="357" alt="image" src="https://github.com/user-attachments/assets/5ccf0c8a-8c10-4b33-92a0-9919b0fba1e7" />  
You will need to set Allow Users to something else, set KBDInteractive line to no, and maybe or two others.  

SSH Version Detection & Vulnerability Scanning  
use auxiliary/scanner/ssh/ssh_version  
set RHOSTS 10.0.0.20  
set RPORT 22  
run and result:  
<img width="547" height="24" alt="image" src="https://github.com/user-attachments/assets/9c31e969-63dd-4b57-9be0-4cab9dd04cc3" />  
In this project, I hardened SSH on Linux by enforcing key-based authentication, disabling root login, and restricting access to specific users or groups. I also secured the server by disabling weak ciphers and protocols, limiting login attempts, testing hardend status with Metasploit and optionally using tools like fail2ban to prevent brute-force attacks. The goal was to reduce the attack surface while following best practices and providing clear setup guidance.
















