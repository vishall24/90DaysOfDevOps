<h1>Day 15 – Networking Concepts: DNS, IP, Subnets & Ports</h1>
Task
  Build on Day 14 by understanding the building blocks of networking every DevOps engineer must know.
  
  You will:
  
  Understand how DNS resolves names to IPs
  Learn IP addressing (IPv4, public vs private)
  Break down CIDR notation and subnetting basics
  Know common ports and why they matter
  This is concept-focused — research, understand, and document in your own words.


Task 1: DNS – How Names Become IPs
    Explain in 3–4 lines: what happens when you type google.com in a browser?
    What are these record types? Write one line each:
    A, AAAA, CNAME, MX, NS
    Run: dig google.com — identify the A record and TTL from the output

== 
 *) When we run dig google.com , the browser sends the request --> DNS resolves domain (google.com) to IP --> connection is made to server --> the server will return the response

 SO here the IP is nothing but the IP of some machine where google website is hosted.

 Record Types:
  A record    --> connects a domain to an IPv4 address (like google.com → 142.250.x.x)
  AAAA record --> connects a domain to an IPv6 address (newer, longer IP format)
  CNAME       --> makes one domain point to another domain (like alias/shortcut)
  MX record   --> tells where emails should go for that domain
  NS record   --> tells which DNS servers manage the domain

Task 2: IP Addressing
  What is an IPv4 address? How is it structured? (e.g., 192.168.1.10)
  Difference between public and private IPs — give one example of each
  What are the private IP ranges?
  10.x.x.x, 172.16.x.x – 172.31.x.x, 192.168.x.x
  Run: ip addr show — identify which of your IPs are private


