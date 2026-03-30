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
 *) When we run dig google.com , the browser sends the request --> DNS resolves domain (google.com) to IP --> connection is made to server 
    --> the server will return the response

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

==

  IPv4 is a protocol which is used to identify devices on network & to route internet traffic, it is divied into four 8 -bit octets from 0     to 255

  EG:
   192.168.1.1 ( 4 octets ( each 8 bits )) so 8 x 4 = 32 bits in total.

  Public VS private IPs:

  Public -> 8.8.8.8 ( for internet accessable ( accessable by all ))
    - Accessible from anywhere in the world
    - Must be globally unique
    - Assigned by ISP / cloud (AWS, etc.)
  
  
  private -> 192.168.2.3 ( for internal use only (internal network)). 
    - NOT accessible from internet
    - Used inside home, office, VPC
    - Can be reused in many networks
  
  Private ranges:
    10.x.x.x  
    172.16.x.x  - 172.31.x.x  
    192.168.x.x

  Command:
    
    ip addr show  

  Observation:
    My IP is private.

  Additional:
     
  Why private IP exists?
    
      Because IPv4 addresses are limited
      Total IPv4 ≈ 4.3 billion
      But devices = billions × billions
    
  So solution:
    
      Use private IP inside networks
      Use 1 public IP to represent many devices


### Task 3: CIDR & Subnetting
    
    1. What does `/24` mean in `192.168.1.0/24`?
    2. How many usable hosts in a `/24`? A `/16`? A `/28`?
    3. Explain in your own words: why do we subnet?
    4. Quick exercise — fill in:
    
    | CIDR | Subnet Mask | Total IPs | Usable Hosts |
    |------|-------------|-----------|--------------|
    | /24  | ?           | ?         | ?            |
    | /16  | ?           | ?         | ?            |
    | /28  | ?           | ?         | ?            |


====

    1) the /24 means the subnet or the available addresses to use
    
    2) usable hosts in /24 --> 2 ^ 7 (32-24 = 7) == 256 so 254 usable since first and last are reserved. similarly /16 = 65536 total
       usable 65534 , /28 = 2^4 = 16, usable 14.
    
    3) why we do subneting? :: to divide the network into multiple networks to make the best use of one IP / Manage traffic efficiently. 
       Example: one home but no rooms, dividing home into rooms, and assign each room to each person ( device ).  
    
    4) 
          | CIDR | Subnet Mask      | Total IPs | Usable Hosts |
          |------|------------------|-----------|--------------|
          | /24  | 255.255.255.0    | 256       | 254          |
          | /16  | 255.255.0.0      | 65536     | 65534        |
          | /28  | 255.255.255.240  | 16        | 14           |


### Task 4: Ports – The Doors to Services

    1. What is a port? Why do we need them?
    2. Document these common ports:
    
    | Port | Service |
    |------|---------|
    | 22   | ?       |
    | 80   | ?       |
    | 443  | ?       |
    | 53   | ?       |
    | 3306 | ?       |
    | 6379 | ?       |
    | 27017| ?       |
  
  3. Run `ss -tulpn` — match at least 2 listening ports to their services

==

    1) Port are entry point for services, IP == your building , and port is your room number 
    
    2)    | Port | Service |
          |------|---------|
          | 22   | SSH     |
          | 80   | HTTP    |
          | 443  | HTTPS   |
          | 53   | DNS     |
          | 3306 | MySQL   |
          | 6379 | Redis   |
          | 27017| MongoDB |

    3) Observed services like SSH running on 22 and DNS running on 53

## Task 5: Putting It Together

    Answer in 2–3 lines each:
    - You run `curl http://myapp.com:8080` — what networking concepts from today are involved?
    - Your app can't reach a database at `10.0.1.50:3306` — what would you check first?

==

    - `curl http://myapp.com:8080` - DNS -> IP -> TCP connection -> HTTP request.
    - I would check the port number using ss -tulpn which port is running on which port? ping 10.0.1.50:3306,
      systemctl status service_name

What I learned:

    - what is IP, subnet, subnet mask.
    - port services, which service runs on which port
    - what is public and private IPs.


Cias Adios :)
