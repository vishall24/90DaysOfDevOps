<h1>Day 06 – Linux Fundamentals: Read and Write Text Files</h1>

<h3>Todays task is to read write and append on a file. This file will be having very basic commads practice.</h3>

Terminal> touch notes.txt → created empty file  

Terminal> echo "Learning Linux" > notes.txt -> added first line  

Terminal> echo "Practicing file commands" >> notes.txt -> appended second line  

Terminal> echo "DevOps journey Day 06" | tee -a notes.txt -> appended and displayed  ( -a : appends, tee is used to write as well as print ).


OUTPUT:

Learning Linux is fun
Practicing file commands
DevOps journey Day 06


Terminal> head -n 2 notes.txt  -> prints stating 2 lines


OUTPUT:

Learning Linux

Practicing file commands


Terminal> tail -n 2 notes.txt prints last 2 lines


OUTPUT:

Practicing file commands

DevOps journey Day 06
