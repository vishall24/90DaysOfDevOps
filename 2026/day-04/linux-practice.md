<h1>Day 04 – Linux Practice: Processes and Services</h1>

<h3>Task: Today’s goal is to practice Linux fundamentals with real commands.</h3>

Section 1: Process checks (run 2 commands):

 Terminal> ps aux
 : <img width="1221" height="380" alt="Screenshot 2026-03-19 at 12 48 54 AM" src="https://github.com/user-attachments/assets/7bf9b4f0-021e-4320-b6df-29dabc4830fc" />


 The command ps aux prints all the running processes and their CPU usage and Memory usage as well
 
 Terminal> ps aux | awk '{print $2, $11}'  :: this command will astract PID and the service name.
 
 -------------------------------------------------------------------------------------------------------------------------------------------
 
 Terminal> top 
 : ![ScreenRecording2026-03-19at12 57 30AM-ezgif com-video-to-gif-converter](https://github.com/user-attachments/assets/7c65475f-8431-42f9-8172-da091d773add)

 the top command will show the real-time CPU and MEM usage which can be further sorted by | (pipeline operator) and arguments

 <h2>Section 2: Service checks (systemd)</h2>

  Terminal> systemctl status ssh
  <img width="1110" height="405" alt="image" src="https://github.com/user-attachments/assets/cccc0bbe-a754-48fe-95ea-bab0b66eaa8b" />
  The systemctl status command is used to see the status of the service whether its active or inactive or stopped.
  
  Terminal> systemctl list-unit --type=service | head -10
  <img width="1094" height="210" alt="image" src="https://github.com/user-attachments/assets/c6a47ac3-2752-4e48-9b8e-198d841022de" />
  Here this command list all the units those are currently running with the type service. if we dont give "--type=service" then?
  well then, it will print not just services but mounts, sockets, timers etc.. , and what if we dont give list-unit ?
  then, the teminal will show the details about systemctl command but here we want the units running its like command without any args
  like npm after that no args.

  
<h3>Section 3: Log checks</h3>

  Terminal> journalctl -u ssh | tail -10
  <img width="1096" height="196" alt="image" src="https://github.com/user-attachments/assets/54c79fda-eeec-4cd6-bfa8-1ff2c9f9d90e" />

  This command will give the logs of ssh service ( history of events like login/failure/success ), -u stands for unit but which unit? 
  the unit which we have mentioned that is ssh, so the journalctl will show the logs for the service ssh , and shows last 10 lines of 
  logs, but, we could have used the systemctl right? :: the systemctl will only be used for managing service stop/start/restart,
  but to see logs or history of events use "journalctl".

  Terminal> tail -n 10 /var/log/syslog
  <img width="1891" height="222" alt="image" src="https://github.com/user-attachments/assets/e9f84198-bf85-4b48-8bab-8fa18b3f965d" />

  tail will show the logs from last 10 lines from location /var/log/syslog this file will show the system logs what's happening in the 
  system.

  The service ssh has been already inspected as mentioned in the above sections.

  Happy learning see you in next task
  
