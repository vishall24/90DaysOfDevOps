# Day 12 – Breather & Revision (Days 01–11)
## Goal
  Take a one-day pause to consolidate everything from Days 01–11 so you don’t forget the fundamentals you just built.

# Day 12 – Revision ::

## Key Commands I Revisited
  ps aux → check running processes  
  systemctl status nginx → check service status  
  journalctl -u nginx → check logs  

## File Operations Practice
  echo "test" >> file.txt → append content  
  chmod 755 script.sh → change permissions  
  chown user:group file → change ownership  

## Top 5 Commands I Will Use
  ps aux  
  top  
  systemctl status  
  journalctl  
  chmod

## User/Group Practice
  Created a user and checked using id  
  Changed file ownership and verified using ls -l  

---

## Self Check

### 1.) 3 most useful commands
  ps aux → quick process check  
  top → live CPU usage  
  journalctl → logs debugging  

Remember :

  du -h → disk usage of files/folders (du -h /var/log)
  df -h → disk usage of filesystem (whole disk)

### 2.) How to check service health
  systemctl status nginx  
  journalctl -u nginx  
  ps aux | grep nginx  

### 3.) Safe permission/ownership change
  chmod 755 script.sh  
  chown user:group file.txt  

### 4.) Focus for next 3 days
  Improve troubleshooting speed  
  Practice more real scenarios  
  Get comfortable with logs and debugging  
