<h1>Day 07 – Linux File System Hierarchy & Scenario-Based Practice</h1>

<h2>Today's goal is to understand where things live in Linux and practice troubleshooting.</h2>

  I will create notes covering:
  
  Linux File System Hierarchy (the most important directories)
  Practice solving real-world scenarios step by step

This file is divided into two sections: File System Hierarchy and Scenario Practice.

lets go with the first one : 

==========================================================================================================================================

<h3>(1)File System Hierarchy</h3>


1.1) / (root)
This dir contains everything in Linux, starting point of filesystem.

Terminal> ls -l /

Observation: saw directories like bin, etc, home
This dir is the first dir there is nothing before this this is the root rest all are branches and leafs.

---

1.2) /home
This dir contains user directories.

Terminal> ls -l /home

Observation: user folders present
I would use this to access user files, there can be more users. (ex: vishal/ sonith/ ...)

---

1.3) /etc
This dir stores configuration files, like system configs, service configs, user/account configs.

Terminal> ls -l /etc

Observation: files like hostname(stores system info), passwd(user account info)
I would use this to check/edit configs. 

---

1.4) /var/log
This fir contains logs. like system logs

Terminal> ls -l /var/log

Observation: syslog, auth.log
I would use this during debugging.

---

1.5) /tmp
This is an temporary files directory.

Terminal> ls -l /tmp

Observation: temp files
I would use this for temporary storage.

---

1.6) /root

THis is the Home directory of the root (admin) user. Not same as the "/" dir (root filesystem)
/ -> whole system.
/root -> home folder of root user.

Terminal> ls -l /root

Observation: contains files and configs specific to root user
I would use this when working as root or accessing admin-level files.


1.7) /bin

This dir contains essential command binaries (basic commands needed to run the system).

Terminal> ls -l /bin

Observation: saw commands like ls, cp, mv, cat
This dir is used for storing basic commands which are required for operating normal commands.

1.8) /usr/bin

This dir contains user-level command binaries (non-critical commands).

Terminal> ls -l /usr/bin

Observation: saw many commands like git, python, curl
This dir is used for storing most of the user-installed or system-installed applications and commands (not required for boot).

1.9) /opt

This dir contains optional or third-party applications.

Terminal> ls -l /opt

Observation: may see folders of custom apps (like /opt/google/, /opt/custom-app)
This dir is used for installing software that is not part of default system packages (manual installs or external tools).

=======================================================================================================================================

# Find the largest log file in /var/log
du -sh /var/log/* 2>/dev/null | sort -h | tail -5

Output: from the logs I can see that the folder /var/log/journal = 5.3 GB , has the highest size and consuming alot of space.

# Look at a config file in /etc
cat /etc/hostname

Output: svdi-u12q-x-036,  this is the host name from the machine I am executing commands.

# Check your home directory
ls -la ~

Output: list all the files including hidden files as well

=======================================================================================================================================

<h3>(2)Scenario-Based Practice</h3>

