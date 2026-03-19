<h1>Day 05 – Linux Troubleshooting Drill: CPU, Memory, and Logs</h1>

<h2>Task</h2>

Today’s goal is to run a focused troubleshooting drill. so lets start.

So in this article , I am gonna pick the service : ssh and I will drill it down.

1) Environment basics (2 commands)

   Terminal> uname -a
   <img width="1151" height="75" alt="image" src="https://github.com/user-attachments/assets/62d82afd-b387-4caa-8de9-d26052ee1d91" />

   Shows system kernal details. 
   Terminal> cat /etc/os-release/
   <img width="761" height="233" alt="image" src="https://github.com/user-attachments/assets/e9329f9e-0e7c-47ff-9436-cf200601d9b0" />

   Shows os related information.
2) CPU & Memory

   Terminal> top -b -n 1 | head -10
   <img width="810" height="205" alt="image" src="https://github.com/user-attachments/assets/3640f32f-bafc-48a0-83fb-6ad93f99a63e" />

   -b : non interative mode , batch mode
   -n 1 : run once, if not mentioned then it would run infinitely in background

   The CPU usage in percentage shows 95.1 % idle , no load at all.

   Terminal> free -h
   <img width="745" height="79" alt="image" src="https://github.com/user-attachments/assets/d4d5f038-e10a-4e28-9876-05101f08a6a9" />

   This command shows the Memory usage ( RAM )
   The logs shows total: 62GB of Mem, used 12GB and 808MB (we dont have to rely on this), buff/cache shows 49 GB can be free anytime
   So available is 49 BG , no spike inMem observed.

3) Disk & IO

   Terminal> du -sh /var/log/
   <img width="462" height="48" alt="image" src="https://github.com/user-attachments/assets/05fe7a9c-4b42-4d13-a916-683dc85f1c8f" />

   This command shows the disk getting used by the mentioned folder/dir, -s summary (leave subfolders/files), -h (human readable MB GB).

   Terminal> df -h
   <img width="745" height="594" alt="image" src="https://github.com/user-attachments/assets/ebaee8cd-ce7a-42c0-9fa5-c240f78be340" />

   This command shows Disk usage (hard drive / SSD)
   THis command is showing 945G Total disk, used 233 G, free 672G I think this is more than enough.

   remember : df = total disk usage, du = folder size.

4) Network

   Terminal> ss -tulnp | grep ssh
   <img width="1076" height="61" alt="image" src="https://github.com/user-attachments/assets/9118a476-7e1d-444a-a944-a43feb6828b5" />

   This command with ss means it shows the network connections / listening ports, and -tulnp those are flag :) yes.
   -t TCP connections, -u UDP connections, -l listening ports only, -n shows number (no DNS resolution, -p show process(like sshd))

   and the command shows the ssh is running on port 22 :)

   Terminal> curl -I http://www.google.com
   <img width="1904" height="389" alt="image" src="https://github.com/user-attachments/assets/c81c750b-1dfc-432b-8633-e7f61a0d62de" />

   This command "curl" is used to send an HTTP request to any url, -I means only the header part nothing else.
   first tried http://localhost, but got nothing since there was nothing running like nginx or any service so expected.
   then I tried google.com and worked and got the response as header from the HTTP request from the curl command.

5) Logs

   Terminal> tail -20 /var/log/syslog
   <img width="1907" height="418" alt="image" src="https://github.com/user-attachments/assets/50014c96-33ba-4ef4-8045-23b6674626c5" />

   This command shows last 20 lines of log from the file /var/log/syslog, and found that few apps are running and generating logs as well
   nothing serious or nothing is breaking.

   Terminal> journalctl -u ssh -n 20
   <img width="1106" height="397" alt="image" src="https://github.com/user-attachments/assets/b65a77b5-b9b7-4e48-843a-67db37f379d4" />

   This command will show the logs in the service ssh, last 20 lines, and found that there were multiple login attempt failed
   I checked using who command which shows that the user who is currently logged in and it was showing me, so no one logged
   but whoes IP it is ? it is an Internal IP and someone is accessing it internally, maybe another VM or automation script. for futher
   discussion


 My Findings:
 
  System is stable  
  No high CPU or memory usage  
  SSH service is running fine  
  No critical errors in logs

If this worsens (like it happened in journalctl ssh)

1. Restart ssh service using systemctl restart ssh  
2. Check detailed logs using journalctl -xe  
3. Monitor CPU spikes using top continuously
4. check who is logged in using who command
5. check the IP which is tryna login by doing nslookup IP-address - this will show the hostname.
6. ask the team for their help immediately.

Ciao adios :)
