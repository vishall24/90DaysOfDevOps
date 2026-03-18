<h1>Day 03 – Linux Commands Practice</h1>

Task:
  Today’s goal is to build your Linux command confidence.

I will be focusing on:

Process management
File system
Networking troubleshooting

List of commands:

<h2>File System Commands:</h2>

1) 'ls'                    - list all the files in the pwd, put "-l" for long listing (permissions, size , owner)

2) 'cd /file_path'         - to get into the file path mentioned, do cd .. to get one folder back, do cd to get into root dir.

3) 'mkdir folder_name'     - to make a folder, do rm -rf file_path to remove the mentioned dir.

4) 'cp SRC DEST'           - copies SRC to DEST

5) 'mv SRC DEST'           - move the files and dir to the destination, do "mv file.txt file2.txt" to rename the "file1.txt" to "file2.txt"

6) 'cat file.txt'          - display the content of any file mentioned

7) 'grep -r "you" .        - finds "you" in the pwd (.), -r represents : find in sub-folders as well.

8) 'chmod 755 script.sh'   - gives R/W/X permission to owner, R/X to Groups, R/X to others (4-->(R) , 2-->(W), 1-->(X)).

9) chown user:user file    - this changes the owner to user and group also to user.

10) df -h                  - Shows the disk usage, that means storage , not CPU not Memory(RAM).

11) du -sh folder/         - Shows total size of a folder

<h2>Process Management Commands:</h2>

12) ps aux                 - Lists all running processes with details like CPU / Memory(RAM) getting used.

13) top                    - top = real-time version of ps aux

14) kill -9 1234           - Force kills process with PID 1234, 15 Graceful stop (default), 9 Force kill (cannot be ignored), 2 Interrupt
                            (like Ctrl + C), 19 Pause process, 18 Resume process.


<h2>Networking Troubleshooting:</h2>

15) ping www.google.com    - checks the network connectivity to the mentioned host (just shows is the call happening to host)

16) ip addr                - shows the ip addresses and network interfaces.

17) curl -I https://ex.com - Fetches HTTP headers to test endpoint

18) dig google.com         - Gets DNS resolution details (get the IP address).

19) netstat -tulnp         - Shows open ports and listening services (tcp   0.0.0.0:80   LISTEN   1234/nginx) port 80 , on nginx.

20) traceroute google.com  - It shows how your request travels through the internet, Step by step, through different routers (hops)


