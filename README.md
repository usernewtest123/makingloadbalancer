PART 1 — Networking Fundamentals (Very Important)

Before writing sockets, you must understand how network communication works.

1. What Happens When Two Programs Communicate?

Example:

Browser → Server

When you visit a website:

Your browser connects to a server like:

Client (your laptop)
        |
        | TCP connection
        |
Server (google.com)

Programs communicate using network protocols.

The most important protocol stack:

Application Layer → HTTP
Transport Layer   → TCP / UDP
Network Layer     → IP
Link Layer        → Ethernet/WiFi

This is called the TCP/IP Model.

2. What is TCP?

TCP = Transmission Control Protocol

Features:

Reliable

Ordered delivery

Error checking

Connection based

Example:

Client ----TCP connection----> Server

Used by:

HTTP

HTTPS

SSH

FTP

3. What is UDP?

UDP = User Datagram Protocol

Features:

Fast

No guarantee of delivery

No connection

Used by:

video streaming

games

DNS

For load balancers and web servers, we mostly use TCP.

4. IP Address

Every machine has an IP address.

Example:

192.168.1.10
8.8.8.8
127.0.0.1

Important ones:

127.0.0.1 = localhost

It means:

your own computer

5. Ports

Ports identify which application should receive the data.

Example:

IP + PORT = SOCKET ADDRESS

Example servers:

HTTP   → port 80
HTTPS  → port 443
SSH    → port 22

Example:

127.0.0.1:8080

means:

local computer + port 8080
6. What is a Socket?

A socket is the interface that allows programs to communicate over the network.

Think of it like:

File descriptor for network communication

In Linux:

Everything is a file descriptor.

Socket example:

int sockfd = socket(...);
7. Client–Server Architecture

Networking usually follows:

SERVER
  |
  | waits for connection
  |
CLIENT

Flow:

Server:
socket()
bind()
listen()
accept()

Client:
socket()
connect()

Then both:
send()
recv()
8. TCP Connection Lifecycle

The server workflow:

1. socket()
2. bind()
3. listen()
4. accept()
5. recv()
6. send()
7. close()

Client workflow:

1. socket()
2. connect()
3. send()
4. recv()
5. close()
9. Important Linux Networking Commands

Run these in terminal.

Check open ports
ss -tuln

or

netstat -tuln
Check network interfaces
ip addr
Test connection
ping google.com
Check listening servers
lsof -i
10. Important Header Files in C++

When writing socket programs on Linux:

#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>

These provide:

socket()
bind()
listen()
accept()
connect()
send()
recv()
Small Visualization
          CLIENT
        (Browser)
            |
            | connect()
            |
        TCP CONNECTION
            |
            |
        SERVER
       (Program)

Your load balancer will sit like this:

Clients
   |
   |
Load Balancer
   |
   |---- Server 1
   |---- Server 2
   |---- Server 3
Mini Homework (Important)

Before Part 2 run these:

1
ip addr

Find your IP address

2

Run

ss -tuln

See which ports are open.

3

Try

nc -l 8080

Open another terminal:

nc localhost 8080

Type something.

You just created a manual TCP connection.

In PART 2 we will build your FIRST TCP SERVER in C++

You will learn:

socket()

bind()

listen()

accept()

And write a real working TCP server in Linux.

After Part 5–6, you will already be able to build a basic load balancer.

If you want, I can also give you a FULL INDUSTRY ROADMAP for a C++ Load Balancer (similar to Nginx/Envoy) so the project becomes resume + interview level.


