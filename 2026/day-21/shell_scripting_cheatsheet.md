# Day 21 – Shell Scripting Cheat Sheet: Build Your Own Reference Guide

## Task

You've spent the last several days learning Shell scripting — from basics to real-world projects. Now it's time to consolidate everything into a **personal cheat sheet** that you can use as a quick-reference guide for the rest of your DevOps journey.

The best way to revise is to **teach it back**. Writing a cheat sheet forces you to organize your understanding and identify gaps.

---


## Challenge Tasks

### Task 1: Basics
Document the following with short descriptions and examples:
1. Shebang (`#!/bin/bash`) — what it does and why it matters
2. Running a script — `chmod +x`, `./script.sh`, `bash script.sh`
3. Comments — single line (`#`) and inline
4. Variables — declaring, using, and quoting (`$VAR`, `"$VAR"`, `'$VAR'`)
5. Reading user input — `read`
6. Command-line arguments — `$0`, `$1`, `$#`, `$@`, `$?`

---

## My answer : 

    - #!/bin/bash - also known as shebang, it is used to tell the system which interpreter to use, if not mentioned then it will take                          default
    - chmod +x <file_name> - used to give the Execute permission to all ( Owner + Group + Others ) for the mentioned file.
    - ./script.sh - this will run the script
    - bash script.sh - runs the script using bash directly
    - "#" - this is used for comments single line, eg: # this is the commented line.
    - inline '#' - this is used to put a comment line in existing line eg: echo "Hello " # this is inline comment
    - Variables ('$VAR', "$VAR", ''$VAR''):
  Example:

    1)NAME = "Vishal D"
    2)echo $NAME
    3)echo "$NAME"A
    4)echo '$NAME'

  Here, the 2nd point will print "Vishal D"      : variable expands but treats the string as 2 words/value in this example,
        the 3rd point will print "Vishal D"      : variable expands but treats the string as 1 single value,
        the 4th point will simply prints "$NAME" : variable does not expands in this case.

    - read - is used to read the input from the user, EX: read -p "Enter your name" NAME
    
      Here -p means prompt without it the message "Enter your name" won't get print, EX means example not literally your ex bro XD
    
    - Command-line arguments — $0 : this will return the current script file name
                               $1 : this will be the first argument you can pass while executing your script( ./script.sh log.log)
                               $# : this will return number of arguments passed to sript
                               $@ : this will return all arguments.
                               $? : This will return exit status of last command.
  
---

### Task 2: Operators and Conditionals
Document with examples:
1. String comparisons — `=`, `!=`, `-z`, `-n`
2. Integer comparisons — `-eq`, `-ne`, `-lt`, `-gt`, `-le`, `-ge`
3. File test operators — `-f`, `-d`, `-e`, `-r`, `-w`, `-x`, `-s`
4. `if`, `elif`, `else` syntax
5. Logical operators — `&&`, `||`, `!`
6. Case statements — `case ... esac`


---

My answer

1) String comparisons -

       =  : is equal to?
       != : is not equal to?
       -z : is empty?
       -n : is not empty?
    
       Example:
    
        [ "$a" = "$b" ]     # equal  
        [ "$a" != "$b" ]    # not equal  
        [ -z "$a" ]         # empty  
        [ -n "$a" ]         # not empty

3) Integer Comparisons:

       -eq : equals to
       -ne : not equal to
       -lt : less than
       -gt : greater than
       -le : less than or equal to
       -ge : greater than or equal to
    
       Eg:
    
        [ $a -eq $b ]   # equal  
        [ $a -ne $b ]   # not equal  
        [ $a -lt $b ]   # less than  
        [ $a -gt $b ]   # greater than
        [ $a -le $b ]   # greater than
        [ $a -ge $b ]   # greater than

4) File test Operations:

       -f : file ( file exists ? eg: if [ -f "$FILENAME"]; then)
       -d : dir ( directory exists?)
       -r : file ( is it readable ?)
       -w : file (is it writable ?)
       -x : file (is it executable ?)
       -e : file ( file exists? (( any type of file eg: symlink , directory)) )
       -s : file (does the file exist and NOT empty ??)

5) if, elif & else:

       Eg:
    
       if [ -f text.txt ]; then
           echo "file exists"
       elif [ -d text.txt ]; then
           echo "its a directory"
       else
           echo "Not found"

6) &&, ||, !:

       cmd1 && cmd2 --> return true if both of them are true else false
       cmd1 || cmd2 --> return true if either of them is true
       ! cmd1       ---> return true if the cmd1 is not cmd1 ( NOT operator)

7) case and esac :
       Eg:
    
       case $VAR in
         start) echo "started the process"
         end) echo "ended the process"
         *) echo "invalid"
       esac

---

### Task 3: Loops
Document with examples:
1. `for` loop — list-based and C-style
2. `while` loop
3. `until` loop
4. Loop control — `break`, `continue`
5. Looping over files — `for file in *.log`
6. Looping over command output — `while read line`

---

My answer:

1) for loop list-based:

       for i in 1 2 3 4 5; do
           echo $i
       done
    
       C-style:
    
       for ((i=1; i<=5; i++))
       do
           echo $i
       done

2) while loop:

       i=1
       while [ $i -le 5 ];
       do
           echo $i
           ((i++))
       done

3) until loop:

       i=1
       until [ $i -gt 5 ]:
       do
           echo $1
           (( i++ ))
       done

4) loop control: break ( break will exit the entire loop ), continue ( continue will just skip the current iteration and move to the next iteration)

5) Loop over files:

       for file in *.log
       do
           echo "processing files: $file"
           wc -l "$file"
       done

6) while read line loop:

       while read line_anything
       do
           echo "line : $line_anything"
       done
    
       --> this will ask for input and prints the same.


---

### Task 4: Functions
  Document with examples:
  1. Defining a function — `function_name() { ... }`
  2. Calling a function
  3. Passing arguments to functions — `$1`, `$2` inside functions
  4. Return values — `return` vs `echo`
  5. Local variables — `local`

---

    1) functions:
       greet(){
    3)    echo "Hello $1"
        }
    
    2)  greet vishal
    
    4) return 1 --> this will not print anything but returns someting to function, echo "Hello" --> this will print the text passed inside                       it.
    
    5) myfunc() {
      local name="Vishal"
    } --> only available in funciton inside function.

---

### Task 5: Text Processing Commands
  Document the most useful flags/patterns for each:
  1. `grep` — search patterns, `-i`, `-r`, `-c`, `-n`, `-v`, `-E`
  2. `awk` — print columns, field separator, patterns, `BEGIN/END`
  3. `sed` — substitution, delete lines, in-place edit
  4. `cut` — extract columns by delimiter
  5. `sort` — alphabetical, numerical, reverse, unique
  6. `uniq` — deduplicate, count
  7. `tr` — translate/delete characters
  8. `wc` — line/word/char count
  9. `head` / `tail` — first/last N lines, follow mode

---

My answer:

1)

      grep "error" file.txt        # search pattern
      grep -i "error" file.txt     # ignore case ( NAME can be considered as name as well)
      grep -r "error" /logs        # recursive search
      grep -c "error" file.txt     # count matches
      grep -n "error" file.txt     # show line numbers
      grep -v "error" file.txt     # invert (exclude)
      grep -E "error|fail" file.txt # multiple patterns (regex)

4)

       awk '{ print $1 }'file   --> prints 1st column only fro the file "file"
       awk '{print $1,$3}' file.txt  --> prints 1st and 3rd column
       awk -F "," '{print $1}' file.csv --> this will print the 1st column but how it will differentiate the content of 1st and                                           rest of the column this can be possible using delimiter but providing delimiter it                                            will seperate.
       awk '/error/ {print}' file.txt  --> print only pattern match
       awk 'BEGIN {print "Start"} {print $1} END {print "End"}' file.txt --> this will print start at start and END at the end.


5)
 
      sed 's/error/warning/' file.txt     --> replace first match per line (ex: error with error on to warniing with warning                                               on), this will not edit the file it will just print the modified output.
      sed 's/error/warning/g' file.txt    --> replace all
      sed '2d' file.txt                   --> delete line 2
      sed -i 's/error/warning/g' file.txt --> edit file in-place, different than sed 's/er/war/' file.txt, this will edit the                                               original file

6)

      root:x:0:0:root:/root:/bin/bash
      ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
  
      Command:
      
      cut -d ":" -f1 file.txt
  
      Output:
  
      root
      ubuntu

7)

      sort file.txt        --> alphabetical wise sort
      sort -n numbers.txt  --> numeric wise sort 1,10,12,23...
      sort -r file.txt     --> reverse wise sort d,c,b,a
      sort -u file.txt     --> unique, apple , apple, apple will be shown as one apple.

8)

      uniq file.txt        --> remove duplicates
      uniq -c file.txt     --> count duplicates Ex: 3.apple,2.banana
      uniq -d file.txt     --> show duplicates only.

9) tr = translate / transform character now::

        echo "abc" | tr 'a' 'x'     --> replace a → x
        echo "abc" | tr 'a-z' 'A-Z' --> lowercase → uppercase
        echo "a b c" | tr -d ' '    --> delete spaces

10)  wc file.txt     --> lines, words, chars

        == 3 7 44 file.txt
    
        Meaning:
          3 → lines
          7 → words
          44 → characters (bytes)
        
        wc -l file.txt  --> number of lines
        wc -w file.txt  --> count of words
        wc -c file.txt  --> count of characters


11)

      head -5 file.txt   --> first 5 lines
      tail -5 file.txt   --> last 5 lines
      tail -f file.txt   --> follow mode(live logs)

---

### Task 6: Useful Patterns and One-Liners
Include at least 5 real-world one-liners you find useful. Examples:
- Find and delete files older than N days
- Count lines in all `.log` files
- Replace a string across multiple files
- Check if a service is running
- Monitor disk usage with alerts
- Parse CSV or JSON from command line
- Tail a log and filter for errors in real time

---

My answer:

    - find . -name "*.log" -mtime +7 -delete
    - wc -l "*.log"
    - sed -i "s/error/warnings/g" *.log
    - systemctl is-active nginx, systemctl status, nginx
    - df -h | awk '$5+0 > 80 {print "High usage on", $6}'
    - cut -d "," -f2 file.csv
    - tail -f app.log | grep -i error

---

### Task 7: Error Handling and Debugging
Document with examples:
1. Exit codes — `$?`, `exit 0`, `exit 1`
2. `set -e` — exit on error
3. `set -u` — treat unset variables as error
4. `set -o pipefail` — catch errors in pipes
5. `set -x` — debug mode (trace execution)
6. Trap — `trap 'cleanup' EXIT`

---

My answer:

1)

      ls file.txt
      echo $?
    
    Output:
    
    0 -> success
    non-zero -> error

1)  exit :
   
        exit 0   -> success
        exit 1   -> failure
    
    Used to stop script with status


2) set -e (exit on error) : if any error occurs anywhere in the script then this command will help to exit immediately.

3)  set -u (unset variable error):

        set -u
        
        echo "$name"

    If name not defined → script fails

4)  set -o pipefail (important 🔥):
    
        set -o pipefail
        grep "error" file.txt | wc -l

    Normally:
  
    only last command checked (wc)
  
    With pipefail:
  
    whole pipeline fails if any command fails
  
5) set -x (debug mode):
        
        set -x
        echo "Hello"

  Output:

  + echo Hello  ( shows this in the console step by step)
  Hello

  Shows command execution step-by-step

6)  trap (cleanup handler):
       
        cleanup() {
          echo "Cleaning up..."
          rm -f temp.txt
        }
        
        trap cleanup EXIT
        
        touch temp.txt
    
    When script exits:
    Cleaning up...

7) Combination of all:

        #!/bin/bash
        
        set -euo pipefail
        
        cleanup() {
          echo "Cleanup done"
        }
        trap cleanup EXIT
        
        echo "Running script..."
        ls file.txt                     --> if fails → script stops




---

### Task 8: Bonus — Quick Reference Table
Create a summary table like this at the top of your cheat sheet:

| Topic | Key Syntax | Example |
|-------|-----------|---------|
| Variable | `VAR="value"` | `NAME="DevOps"` |
| Argument | `$1`, `$2` | `./script.sh arg1` |
| If | `if [ condition ]; then` | `if [ -f file ]; then` |
| For loop | `for i in list; do` | `for i in 1 2 3; do` |
| Function | `name() { ... }` | `greet() { echo "Hi"; }` |
| Grep | `grep pattern file` | `grep -i "error" log.txt` |
| Awk | `awk '{print $1}' file` | `awk -F: '{print $1}' /etc/passwd` |
| Sed | `sed 's/old/new/g' file` | `sed -i 's/foo/bar/g' config.txt` |

---
My answer:

## 🧠 Quick Reference

| Topic        | Key Syntax                                 | Example                            |
|--------------|--------------------------------------------|------------------------------------|
| Variable     | `VAR="value"`                              | `NAME="DevOps"`                    |
| Argument     | `$1`, `$2`, `$#`, `$@`                     | `./script.sh file.txt`             |
| Exit Code    | `$?`, `exit`                               | `echo $?`                          |
| If           | `if [ condition ]; then`                   | `if [ -f file ]; then`             |
| For loop     | `for i in list; do`                        | `for i in 1 2 3; do`               |
| While loop   | `while [ cond ]; do`                       | `while read line; do`              |
| Function     | `name() { ... }`                           | `greet() { echo "Hi"; }`           |
| File Check   | `-f`, `-d`, `-e`, `-s`                     | `[ -f file.txt ]`                  |
| Grep         | `grep pattern file`                        | `grep -i "error" log.txt`          |
| Awk          | `awk '{print $1}' file`                    | `awk -F: '{print $1}' /etc/passwd` |
| Sed          | `sed 's/old/new/g' file`                   | `sed -i 's/foo/bar/g' config.txt`  |
| Cut          | `cut -d delim -fN`                         | `cut -d: -f1 file.txt`             |
| Sort         | `sort`, `-n`, `-r`, `-u`                   | `sort -nr file.txt`                |
| Uniq         | `uniq`, `-c`                               | `sort file | uniq -c`              |
| WC           | `wc -l/-w/-c`                              | `wc -l file.txt`                   |
| Head/Tail    | `head -n`, `tail -f`                       | `tail -f app.log`                  |
| Read         | `read -p`                                  | `read -p "Enter name: " name`      |
| Date         | `date +FORMAT`                             | `date +%Y-%m-%d`                   |
| Pipe         | `|`                                        | `cat file | grep error`            |
| Redirect     | `>`, `>>`                                  | `echo hi > file.txt`               |
| Debug        | `set -x`                                   | `set -x script.sh`                 |
| Safe Mode    | `set -euo pipefail`                        | `set -euo pipefail`                |




***********************************THIS IS THE REFERENCE SHELL SCRIPTING CHEATSHEET*************************************

