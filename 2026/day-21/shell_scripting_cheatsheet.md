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

- #!/bin/bash - also known as shebang, it is used to tell the system which interpreter to use, if not mentioned then it will take default
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

2) Integer Comparisons:
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

3) File test Operations:
   -f : file ( file exists ? eg: if [ -f "$FILENAME"]; then)
   -d : dir ( directory exists?)
   -r : file ( is it readable ?)
   -w : file (is it writable ?)
   -x : file (is it executable ?)
   -e : file ( file exists? (( any type of file eg: symlink , directory)) )
   -s : file (does the file exist and NOT empty ??)

4) if, elif & else:
   Eg:

   if [ -f text.txt ]; then
       echo "file exists"
   elif [ -d text.txt ]; then
       echo "its a directory"
   else
       echo "Not found"

5) &&, ||, !:
   cmd1 && cmd2 --> return true if both of them are true else false
   cmd1 || cmd2 --> return true if either of them is true
   ! cmd1       ---> return true if the cmd1 is not cmd1 ( NOT operator)

6) case and esac :
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
