<h1>Day 14 – Networking Fundamentals & Hands-on Checks</h1>

Task
  Get comfortable with core networking concepts and the commands you’ll actually run during troubleshooting.
  
  You will:
  
  Map the OSI vs TCP/IP models in your own words
  Run essential connectivity commands
  Capture a mini network check for a target host/service

My answer:

## OSI vs TCP/IP

    OSI → 7 layers (Physical → Application)  
    
    A -> Application Layer
    P -> presentation Layer
    S -> Session Layer
    T -> Transport Layer
    N -> Network Layer
    D -> Data Link Layer
    P -> Physical Layer
    
    how to remember this :( 
    --> ez :: remember : Aree Pagali Saath Toh Nibha Deti Pyaari ( A P S T N D P ) . XD 
    
    TCP/IP → 4 layers (Link, Internet, Transport, Application) 
    
    A -> Application layer
    T -> Transport Layer
    I -> Internet Layer
    N -> Network access Layer
    
    How to remember --> just remember the name "NITA ambani" ( N,I,T,A )
    
    IP → Internet layer  
    TCP/UDP → Transport layer  
    HTTP/HTTPS → Application layer  
    DNS → Application layer  

    curl https://google.com → HTTP over TCP over IP

    <img width="800" height="400" alt="image" src="https://github.com/user-attachments/assets/0e6c2e58-0889-4e3a-a800-b2453f4d7b6d" />

## Hands-on Checklist (run these; add 1–2 line observations)
  Identity: hostname -I (or ip addr show) — note your IP.
  
  Reachability: ping <target> — mention latency and packet loss.
  
  Path: traceroute <target> (or tracepath) — note any long hops/timeouts.
  
  Ports: ss -tulpn (or netstat -tulpn) — list one listening service and its port.
  
  Name resolution: dig <domain> or nslookup <domain> — record the resolved IP.
  
  HTTP check: curl -I <http/https-url> — note the HTTP status code.
  
  Connections snapshot: netstat -an | head — count ESTABLISHED vs LISTEN (rough).
  
  Pick one target service/host (e.g., google.com, your lab server, or a local service) and stick to it for ping/traceroute/curl where possible.

My Answer:
<img width="1435" height="663" alt="Screenshot 2026-03-26 at 12 55 24 PM" src="https://github.com/user-attachments/assets/f91ac485-22a3-4245-ad40-42e183a6d970" />

<img width="1022" height="267" alt="Screenshot 2026-03-26 at 12 54 42 PM" src="https://github.com/user-attachments/assets/1de5c099-d3cd-44a9-879f-13c80930bead" />

<img width="1429" height="504" alt="Screenshot 2026-03-26 at 12 56 05 PM" src="https://github.com/user-attachments/assets/031c0c23-8793-4964-984a-b87c69d2735c" />

<img width="2524" height="930" alt="image" src="https://github.com/user-attachments/assets/fce1a8e8-af39-4dd7-8d02-cba5943a353f" />

<img width="1752" height="488" alt="image" src="https://github.com/user-attachments/assets/4bef4cf6-a40a-4ae2-a31e-3201cecffba6" />

## Mini Task: Port Probe & Interpret

  Identify one listening port from ss -tulpn (e.g., SSH on 22 or a local web app).
  
  From the same machine, test it: nc -zv localhost <port> (or curl -I http://localhost:<port>).
  
  Write one line: is it reachable? If not, what’s the next check? (e.g., service status, firewall).

<img width="1732" height="472" alt="image" src="https://github.com/user-attachments/assets/e41fb885-e542-4906-a350-84381f26230f" />

<img width="1778" height="580" alt="image" src="https://github.com/user-attachments/assets/31b86057-3881-4a56-ad0a-ffb557870c55" />

## Reflection

  Fastest command:
  ping → quickly tells if system reachable  
  
  If DNS fails:
  Check dig / DNS layer  
  
  If HTTP 500:
  Check application logs ( Application layer )
  
  Next checks:
  1. systemctl status service  
  2. journalctl logs
