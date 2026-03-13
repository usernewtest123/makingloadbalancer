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

PART 2 — First TCP Server in C++ (Linux)

We will learn the 5 core system calls:

socket()
bind()
listen()
accept()
send()/recv()

These are Linux kernel networking APIs.

1. Basic Server Workflow

The TCP server always follows this order:

1. socket()   -> create socket
2. bind()     -> attach socket to IP + PORT
3. listen()   -> wait for connections
4. accept()   -> accept client
5. send/recv  -> communicate
6. close()

Visual:

Client  --------connect()------->  Server
                                   |
                              accept()
                                   |
                              send/recv
2. Required Header Files
#include <iostream>
#include <cstring>

#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>

What they do:

sys/socket.h   -> socket APIs
netinet/in.h   -> sockaddr_in structure
unistd.h       -> close()
cstring        -> memset
3. Step 1 — Create a Socket
int server_fd = socket(AF_INET, SOCK_STREAM, 0);

Explanation:

AF_INET      -> IPv4
SOCK_STREAM  -> TCP
0            -> default protocol

Think of it as:

create TCP communication endpoint

If socket fails:

if (server_fd < 0) {
    perror("socket failed");
    return 1;
}
4. Step 2 — Define Address and Port

Servers must specify where they listen.

sockaddr_in address;

address.sin_family = AF_INET;
address.sin_addr.s_addr = INADDR_ANY;
address.sin_port = htons(8080);

Explanation:

AF_INET     -> IPv4
INADDR_ANY  -> listen on all network interfaces
8080        -> port

Why htons()?

Network uses big-endian format.

htons = host to network short
5. Step 3 — Bind Socket to Port
bind(server_fd, (struct sockaddr*)&address, sizeof(address));

Meaning:

Attach socket → IP + PORT

Example:

0.0.0.0:8080
6. Step 4 — Listen for Connections
listen(server_fd, 5);

Meaning:

Kernel starts waiting for clients

5 = backlog queue

Max pending connections
7. Step 5 — Accept Client
int client_socket = accept(server_fd, NULL, NULL);

What happens:

client connect()
        ↓
server accept()
        ↓
new socket created for client

Important concept:

server_fd      -> listening socket
client_socket  -> communication socket
8. Step 6 — Send Data
const char* message = "Hello from server\n";
send(client_socket, message, strlen(message), 0);
9. Step 7 — Close Socket
close(client_socket);
close(server_fd);
10. Full Working TCP Server Code
#include <iostream>
#include <cstring>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>

int main() {

    int server_fd = socket(AF_INET, SOCK_STREAM, 0);

    if (server_fd < 0) {
        perror("Socket failed");
        return 1;
    }

    sockaddr_in address;

    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(8080);

    if (bind(server_fd, (struct sockaddr*)&address, sizeof(address)) < 0) {
        perror("Bind failed");
        return 1;
    }

    listen(server_fd, 5);

    std::cout << "Server listening on port 8080...\n";

    int client_socket = accept(server_fd, NULL, NULL);

    const char* message = "Hello from server\n";

    send(client_socket, message, strlen(message), 0);

    close(client_socket);
    close(server_fd);

    return 0;
}
11. Compile
g++ server.cpp -o server

Run:

./server
Now we start I/O multiplexing, which is the first step toward event-driven servers.
This is the idea that allowed systems like NGINX and HAProxy to handle thousands of connections efficiently.

PART 6 — select() (Monitoring Multiple Sockets)

Problem with previous server:

accept client
recv from client
handle client

If you have many clients, the server must check many sockets.

Instead of checking each socket manually, we ask the kernel:

Tell me which sockets are ready for reading.

This is what select() does.

1. What select() Does

select() monitors multiple file descriptors (including sockets).

It tells you when a socket is:

ready to read
ready to write
has error

Diagram:

        Kernel
           |
   -------------------
   |   |   |   |   |
 sock sock sock sock
   |   |   |   |
   |---select()---|
           |
     returns ready sockets
2. Function Signature
int select(
    int nfds,
    fd_set *readfds,
    fd_set *writefds,
    fd_set *exceptfds,
    struct timeval *timeout
);

Simplified idea:

select()
waits until one socket has data
3. Important Data Structure

fd_set

This represents a set of file descriptors.

Example:

fd_set readfds;
4. Important Macros
FD_ZERO()
FD_SET()
FD_ISSET()
FD_CLR()
Clear the set
FD_ZERO(&readfds);
Add socket
FD_SET(sockfd, &readfds);
Check if ready
FD_ISSET(sockfd, &readfds)
5. Example Concept

Suppose you have:

server_fd
client1
client2
client3

Server creates set:

readfds = {server_fd, client1, client2, client3}

Then call:

select()

Kernel waits until any socket has data.

Example result:

client2 ready

Then server reads only client2.

6. Architecture of a select() Server
while(true)
{
    FD_ZERO()
    FD_SET(server_fd)
    FD_SET(all client sockets)

    select()

    if(new connection)
        accept()

    if(client has data)
        recv()
}

This is called an event loop.

7. Basic Example Code

Simplified server using select():

#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>

int main() {

    int server_fd = socket(AF_INET, SOCK_STREAM, 0);

    sockaddr_in addr;
    addr.sin_family = AF_INET;
    addr.sin_port = htons(8080);
    addr.sin_addr.s_addr = INADDR_ANY;

    bind(server_fd, (sockaddr*)&addr, sizeof(addr));
    listen(server_fd, 5);

    fd_set readfds;

    while(true)
    {
        FD_ZERO(&readfds);
        FD_SET(server_fd, &readfds);

        select(server_fd + 1, &readfds, NULL, NULL, NULL);

        if(FD_ISSET(server_fd, &readfds))
        {
            int client = accept(server_fd, NULL, NULL);
            std::cout << "Client connected\n";
            close(client);
        }
    }
}

This server can detect new connections using select().

8. Handling Multiple Clients with select()

Real server:

server_fd
client_sockets[1000]

Algorithm:

1. Add all sockets to fd_set
2. call select()
3. kernel marks ready sockets
4. process only those sockets
9. Why select() Was Revolutionary

Before this:

1 thread per client

With select():

1 thread
1000 sockets

Huge performance improvement.

10. Limitations of select()

Unfortunately it has problems:

1. Max sockets
FD_SETSIZE = 1024

So you can monitor only ~1024 sockets.

2. Slow for large systems

Kernel must scan the entire set every call.

Complexity:

O(n)
11. Why Modern Servers Don't Use select()

High-performance servers need:

100k connections
500k connections
1M connections

So they use epoll instead.

Used by:

NGINX

Envoy

HAProxy

12. Evolution of Linux Network APIs
select()  → old
poll()    → improved
epoll()   → modern Linux

For your load balancer project, we will eventually use:

epoll()

Because it can handle millions of connections.

Assignment (Important)

Modify your server:

1️⃣ Maintain an array:

int clients[100];

2️⃣ Add new clients to fd_set.

3️⃣ When select() shows a socket ready:

recv()
print message

You will now have a multi-client event-driven server.

Next Part (Extremely Important)

PART 7 — poll()

You'll learn:

better alternative to select()
no 1024 limit
cleaner API

Then we move to:

epoll()

Which is the core technology behind modern load balancers.
