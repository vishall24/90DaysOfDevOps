<h1>Day 02 – Linux Architecture, Processes, and systemd</h1>

<h3>Task:</h3>
  
  <h3> - The core components of Linux (kernel, user space, init/systemd)</h3>
    
  Let's discuss about the Architecture of linux and understand how linux works,


  back in 1991, Linus torvalds This guy : <img width="192" height="191" alt="image" src="https://github.com/user-attachments/assets/5f0d7b7b-64df-43ff-81a1-3d946586fc1a" /> chill guy right? he is, he just need an internet connect, and a table & chair 
  and boom Git & linux in front of you
  


  he created a system an OS called as " Linux " and made it open-source (the code will be accessable by anyone), but why?
  because he was frustated with the limitations of existing operating systems, particularly MINIX and MS-DOS, and wanted a free,
  powerful, Unix-like system for his new 80386 PC.

  What benefits it gives over other OS?
  open-source, Unix-like operating system to provide an affordable, accessible alternative to expensive proprietary systems like Unix.
  It gives you freedom to change any file modify anything, go into any file.

  now let's discuss about the Architecture of Linux ::

  Remember :: ASK

  A - application
  S - Shell
  K - kernal

  what are these?

  <img width="423" height="400" alt="image" src="https://github.com/user-attachments/assets/d2ca9eb5-3e09-455e-b5f9-f0869326ab8c" />

  the Image Above explains how the I/O operation gets to you from the hardware

  Hardware (CPU, Memory...) --> Kernal( Communicates with Hardware ) --> shell ( Takes commands from user/app and sends to kernal)
  --> Application (The actual application anything like chrome / whatsapp / terminal / UI).

  - And How processes are created and managed in Linux ?
  = So, when you press on your power on button on your laptop or desktop, " the BIOS " power on the chip inside the motherboard, and loads
    the hardware then the hardware loads the GNU GRUB and GNU GRUB then loads Linux kernal.

    and when this linux kernal loads it also creates a process with PID ( 1 ) the first ever process which gets created in the system
    also called as init process / systemd (System - OS system , d - daemon (which runs in the background) ).

    So in linux everything is a process everything starts with a process.

    a very good example:

    when you do :: terminal> cp folder1 folder2

    then the " cp " command calls the binary ( cp is a binary file ), located in the /bin folder and in that binary file there was C program
    written which gets executed and this whole thing is nothing but ... a process.

    when you do ls, cp, mkdir, mv , ping, grep... a new process gets created with it with a PID,

    how to find whether the process got executed ? what is the PID(process ID) ?
    
    * terminal> ps aux
    this will list all the process running,
    * terminal> ps aux | grep mv
    And this will list only those processes which is running after mv command got executed.
    * terminal> top
    And this command will list all the running processes BUT IN REAL TIME ( LIVE CHANGES ).

    <h3>Now lets discuss about the file structure in linux OS:</h3>
    - Remember everything in linux is a " file or a process ",
      have you ever seen an octopus ? must be, the octopus has a head at the top or its root and then there are octopus hands below that head
      similar to that

      there is a root directory ( / ) this is a root , a root directory before this there is nothing this is the starting.
      now there are so many sub-folders and each sub-folder ( created by linux not by you) serves a very imoportant role
      lets discuss them one by one:

      1) /
        = The base of the Linux directory is the root. This is the starting point of Filesystem Hierarchy.
          Every directory arises from the root directory. It is represented by a forward slash (/).
          If someone says to look into the slash directory, they refer to the root
          directory.

      2) /root
         = It is the home directory for the root user (superuser).

      3) /bin → User Binaries
         = Contains binary executable.
           Common Linux commands you need to use in single-user modes are located under this directory.
           Commands used by all the users of the system are located here.

      4) /sbin → SystemBinaries
         = Just like /bin, /sbin also contains binary executables.
           But, the linux commands located under this directory are used typically by system aministrator, for system maintenance purpose.
           For example: iptables, reboot, fdisk, ifconfig, swapon

      5) /dev → Device Files
         = it contains hardware device files,
           Contains device files.
           These include terminal devices, usb, or any device attached to the system.
           For example: /dev/tty1, /dev/usbmon0

      6) /var→ Variable Files
          = The variable data files such as log files are located in the /var directory.
            File contents that tend to grow are located in this directory.
            * This includes :
          ◦ /var/log: System log files generated by OS and other applications.
          ◦ /var/lib: Contains database and package files.
          ◦ /var/mail: Contains Emails.
          ◦ /var/tmp: Contains Temporary files needed for reboot.

      7) /mnt→ Mount Directory
          = This directory is used to mount a file system temporarily.

      8) /media→ Removable Media Devices
          = The /media directory contains subdirectories where removable media devices
            inserted into the computer are mounted.

      9) /usr→ User Binaries
          = The /usr directory contains applications and files used by users, as opposed to
            applications and files used by the system.

      10) /etc → Configuration files
          = It contains all configuration files of server
            The core configuration files are stored in the /etc directory. It controls the behavior of an operating system or application.
            This directory also contains startup and shutdown program scripts that are used
            to start or stop individual programs.

      11) /boot → Boot Loader Files
          = The /boot directory contains the files needed to boot the system
            * for example,
            the GRUB boot loader's files and your Linux kernels are stored here.
            
      12) /opt → Optional Applications
          = The opt directory is used for installing the application software from thirdparty vendors that are not available in the Linux
            distribution. Usually, the software code is stored in the opt directory and the binary code is linked to the bin directory so
            that all users can run that software.
          
      13) /home → Home Directory
          = It contains secondary users home directory.
          
      14) /tmp → Temporary Files
          = Directory that contains temporary files created by system and users.
            Files under this directory are deleted when system is rebooted.

    <h3>why this Systemd is so important?</h3>
    = systemd is the first user-space process in the linux system, it helps you to create any system or check the status or to restart any
      service or system,

      example:
      I want to start a nginx application so I will do systemctl start nginx (here systemctl is like a command line or control which can be
      used for executing system commands provided by systemd).

    <h3>Now, we will discuss what is process and what are the different kind of states in process:</h3>
    = Process state, is like a current status of a process running on the OS, It tells what process is doing at the moment.

    50 shades of process (just kidding it's:)
    <h4>states of process:</h4>
    1) Running:
       = when the state is (R) that means the process is currently running, or is ready or waiting for CPU time, (Only one process per CPU
         core can run at a time).
    2) Sleeping (S)
       = The process is waiting for some event to happen, such as user input, network response, or disk read. Once the event occurs,
         the process wakes up and continues execution
    3) Uninterruptible Sleep (D)
       = The process is waiting for a hardware or kernel operation, usually disk I/O. While in this state, the process cannot be interrupted
         until the operation finishes.
    4) Stopped (T)
       = The process has been paused, usually by a signal. For example, when you press Ctrl + Z in the terminal, the running process is
         stopped.
    5) Zombie (Z)
       = The process has finished execution, but its parent process has not yet read its exit status. The process no longer runs but still
         exists in the process table until the parent collects the status. the process is dead but still has entry in OS.

    <h3>And these are my top 5 command which I use in my day to day life:</h3>

    1) grep -r "some_string" /folder_path : this command finds all the macthing line which contains that particular sting.
    2) ls -a : shows the last modified files and folders in an acending order
    3) cd : to get into any folder
    4) crontan -e : file is not a command but a file which can be used for scheduling execution automatically.
    5) chmod +x file_name : this command is very useful for modifying the permissions, I do use it to give or to take permission
   
    

       
