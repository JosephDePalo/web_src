---
title: "PyChat"
weight: 1
date: 2025-01-05T16:51:44-05:00
# bookFlatSection: false
# bookToc: true
# bookHidden: false
# bookCollapseSection: false
# bookComments: false
# bookSearchExclude: false
---

# PyChat

This project started as a way of brushing up on my socket programming skills in Python. I have built very basic chat rooms in the past, but this time I wanted to make something a little more sophisticated. As of now, this is still a work in progress, but I would like to share what I've built so far nonetheless.

## Initial Client/Server Connection

Setting up a simple socket connection between a client and server program can be done with only a few lines of code. First, we must set up our project. We will create a directory for this project, a virtual environment for any dependencies we may have, and python files for our client and server.

```bash
> mkdir pychat
> cd pychat
> python -m venv .venv
> touch client.py server.py
```

### Server Creation

We will start with `server.py`:

```python
import socket

DEFAULT_SERVER = "127.0.0.1"
DEFAULT_PORT = 1234
MAX_CONNS = 5
MAX_MSG_SIZE = 1024

server_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # Create a socket for the server
server_sock.bind((DEFAULT_SERVER, DEFAULT_PORT)) # Bind the server to the default server and port
server_sock.listen(MAX_CONNS) # Listen for incoming connections
print(f"Listening on {DEFAULT_SERVER}:{DEFAULT_PORT}...")
```

First, we import the `socket` library, a necessity for any program utilizing sockets. Next, we define useful constants to avoid hard-coded values. This includes the server and port we want to listen on, the maximum number of connections our socket will listen for, and the maximum size for a message sent to the server. We then write some basic code for setting up a server-side socket. This includes:

- Creating the socket object with `socket.socket`. `socket.AF_INET` indicates that we want to use IPv4 and `socket.SOCK_STREAM` indicates that we will use a TCP socket.
- Binding the socket to our previously specified server and port.
- Starting to listen on the binded server and port for any incoming connections.

For now, we will create a `main()` functiion that accepts a single connection and then loops to receive an arbitrary number of messages from that client.

```python
def main():
    server_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # Create a socket for the server
    server_sock.bind((DEFAULT_SERVER, DEFAULT_PORT)) # Bind the server to the default server and port
    server_sock.listen(MAX_CONNS) # Listen for incoming connections
    print(f"Listening on {DEFAULT_SERVER}:{DEFAULT_PORT}...")

    (client_sock, client_addr) = server_sock.accept() # Accept an incoming connection
    print(f"Connection from {client_addr}")

    while True:
        msg = client_sock.recv(MAX_MSG_SIZE).decode() # Block until a message is received from the client.
        print(f"{client_addr}> {msg}")

if __name__ == "__main__":
    main()
```

When we run the server with `python server.py`, our server will wait for a connection:

```bash
> python server.py
Listening on 127.0.0.1:1234...
```

### Client Creation

Now we need to build the client to connect to our server. Just like with our server, we will start by importing `socket` in `client.py`, defining relevant constants, and then writing our socket code:

```python
import socket

DEFAULT_SERVER = "127.0.0.1"
DEFAULT_PORT = 1234
MSG_SIZE = 1024

def main():
    client_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # Create a socket for the client
    client_sock.connect((DEFAULT_SERVER, DEFAULT_PORT)) # Connect to the server
    print(f"Successfully connected to {DEFAULT_SERVER}:{DEFAULT_PORT}")
    while True:
        msg = input("client> ")
        client_sock.send(msg.encode())

if __name__ == "__main__":
    main()
```

Unlike server sockets, client sockets can be created and used in just 2 lines of code: creating the socket and then pointing it to a server and port to connect to.

Now we are able to test what we've created so far! On one terminal we run `python server.py` and on another we run `python client.py`. We see that our client is successfully able to connect to our server and send messages to it.

```bash
# Server Terminal
> python server.py
Listening on 127.0.0.1:1234 # Server is waiting for a connection
Connection from ('127.0.0.1', 38708) # Server receives a connection from our client
('127.0.0.1', 38708)> Hello # Server receives the 'Hello' message from our client

# Client Terminal
> python client.py
Successfully connected to 127.0.0.1:1234 # Client connects to our server
client> Hello # Client sends a 'Hello' message to the server
client> # Prompt awaiting input
```

### Basic Error Handling

To make our code more reliable and to prevent problems down the road, we will now add error handling functionality to our code. Starting with `server.py`, we want to make sure to throw an error if the socket is not created properly or a message cannot be received from the client. We also want the server to know when a client disconnects so the client socket can be properly closed on the server side. For this, we will check if the message if equal to `!quit` and close the socket connection if it is.

```python
def main():
   try:
        server_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # Create a socket for the server
        server_sock.bind((DEFAULT_SERVER, DEFAULT_PORT)) # Bind the server to the default server and port
        server_sock.listen(MAX_CONNS) # Listen for incoming connections
        print(f"Listening on {DEFAULT_SERVER}:{DEFAULT_PORT}...")

        (client_sock, client_addr) = server_sock.accept() # Accept an incoming connection
        print(f"Connection from {client_addr}")

        while True:
            msg = client_sock.recv(MAX_MSG_SIZE).decode() # Block until a message is received from the client.
            if msg == "!quit": # If the client quits, break out of the loop.
                print(f"{client_addr} has disconnected.")
                break
            print(f"{client_addr}> {msg}")
    except KeyboardInterrupt:
        print("Interrupt received, terminating server...")
    except Exception as e:
        print(f"An error occurred: {e}")
    
    try: # Try to close any sockets that may be open
        client_sock.close()
        server_sock.close()
    except:
        pass
```

We will perform similar additions on the client side in `client.py`; we will catch any errors that can arise from the code and send the `!quit` message when an exception occurs.

```python
def main():
    try:
        client_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # Create a socket for the client
        client_sock.connect((DEFAULT_SERVER, DEFAULT_PORT)) # Connect to the server
        print(f"Successfully connected to {DEFAULT_SERVER}:{DEFAULT_PORT}")
        while True:
            msg = input("client> ")
            client_sock.send(msg.encode())
            if msg == "!quit":
                print("Quitting...")
                break
    except KeyboardInterrupt:
        print("Interrupt received, quitting...")
        client_sock.send("!quit".encode()) # Tell the server the client is terminating
    except Exception as e:
        print(f"An error occurred: {e}")

    try: # Try to close any sockets that may be open
        client_sock.close()
    except:
        pass
```

## Creating a Chat Room

Thus far, we have written code to handle a single client sending messages to the server. This unidirectional form of communication, however, will not be sufficient when adding additional clients. Take an example where we have two clients: when one client sends a message, we not only want it to go to the server, but also to the other clients. The server, then, should not only receive messages but also send them to all the clients it is connected to. Performing the functions of sending and receiving messages requires concurrency. To achieve this, we will multithread our client and server.

### Server Side

#### Accepting Multiple Connections

Let's start with the server. First, we need the ability to connect to multiple clients at once. To receive messages from `n` clients concurrently, we will need `n` threads. Therefore, it makes the most sense to create a new thread to handle each client. First, we will create a `Client` class to better encapsulate all the information pertaining to a particular client. This will also help later if we decide to add additional features that require such information.

```python
MAX_MSG_SIZE = 1024

class Client:
    def __init__(self, sock, addr):
        self.sock = sock
        self.addr = addr

def main():
```

We must also move the `while True` loop into its own function so we can create a thread from it:

```python
def client_handler(client):
    while True:
            msg = client.sock.recv(MAX_MSG_SIZE).decode() # Block until a message is received from the client.
            if msg == "!quit": # If the client quits, break out of the loop.
                print(f"{client.addr} has disconnected.")
                break
            print(f"{client.addr}> {msg}")
```

Finally, we can update our `main()` to support creating multiple client threads:

```python
def main():
    try:
        server_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # Create a socket for the server
        server_sock.bind((DEFAULT_SERVER, DEFAULT_PORT)) # Bind the server to the default server and port
        server_sock.listen(MAX_CONNS) # Listen for incoming connections
        print(f"Listening on {DEFAULT_SERVER}:{DEFAULT_PORT}...")

        while True:
            (client_sock, client_addr) = server_sock.accept() # Accept an incoming connection
            client = Client(client_sock, client_addr)
            print(f"Connection from {client.addr}")

            client_thread = Thread(target=client_handler, args=(client,))
            client_thread.daemon = True
            client_thread.start()
```

Our server now supports connections to multiple clients:

```bash
> python server.py
Listening on 127.0.0.1:1234...
Connection from ('127.0.0.1', 35858)
('127.0.0.1', 35858)> Hello from client 1!
Connection from ('127.0.0.1', 52578)
('127.0.0.1', 52578)> Hello from client 2!
('127.0.0.1', 35858)> Another message from client 1!
```

#### Collecting & Sending Messages

Clients, however, are still unable to receive messages from other clients via the server. For this, we need a mechanism for the server to safely collect messages from the clients and then broadcast them to all connections. A queue is a good fit for several reasons:

- Queues can ingest data from multiple producers.
- Ingested messages will be properly ordered.
- Python's built-in queue is thread safe.
- Messages can be consumed from the queue with a "broadcasting" thread.

To implement the producer side of the queue, all we have to do is define the queue and put messages to it whenever necessary:


```python
message_queue = Queue()

class Client:
    def __init__(self, sock, addr):
        self.sock = sock
        self.addr = addr

def client_handler(client):
    while True:
            msg = client.sock.recv(MAX_MSG_SIZE).decode() # Block until a message is received from the client.
            if msg == "!quit": # If the client quits, break out of the loop.
                print(f"{client.addr} has disconnected.")
                message_queue.put((client, msg))
                break
            print(f"{client.addr}> {msg}")
            message_queue.put((client, msg))
```

Messages are being put into the queue; now we need something to pop and send them. We wouldn't want to do this with a client thread, as they have the sole responsibility of receiving messages from their specific client and cannot multitask. Our main thread is busy accepting connections, so that leaves us with the option of making a new thread for this task. We first define a function to handle broadcasting messages.

```python
def broadcast_handler():
    while True:
        if not message_queue.empty():
            (sender, msg) = message_queue.get()
            for c in clients:
                if c != sender:
                    c.sock.send(msg.encode())
```

The `clients` list is not yet defined, but it should be a list of all connected clients. Lists, unlike queues, are not inherently thread safe, so we will use a lock to add and remove client connections. First, for definining `clients`:

```python
MAX_MSG_SIZE = 1024

message_queue = Queue()
clients = []
clients_lock = Lock()

class Client:
```
For adding clients we modify `main()`:

```python
while True:
    (client_sock, client_addr) = server_sock.accept() # Accept an incoming connection
    client = Client(client_sock, client_addr)
    print(f"Connection from {client.addr}")

    with clients_lock: # Add the client to the list of clients
        clients.append(client)

    client_thread = Thread(target=client_handler, args=(client,)) # Create a thread for the client
    client_thread.daemon = True
    client_thread.start()
```

To remove clients from the list, we must improve the error handling for our `client_handler()` function:

```python
def client_handler(client):
    try:
        while True:
                msg = client.sock.recv(MAX_MSG_SIZE).decode() # Block until a message is received from the client.
                if msg == "!quit": # If the client quits, break out of the loop.
                    print(f"{client.addr} has disconnected.")
                    message_queue.put((client, msg))
                    break
                print(f"{client.addr}> {msg}")
                message_queue.put((client, msg))
    except:
        print(f"Connection from {client.addr} lost.")

    try:
        with clients_lock: # Remove the client from the list of clients.
            clients.remove(client)
        client.sock.close() # Close the client's socket.
    except:
        pass
```

Now that we have successfully implemented the `clients` list, we can get back to the problem of sending the messages to clients. We already implemented the `broadcast_handler()` function, so now all we have to do now is create a thread that will run it:

```python
def main():
    try:
        server_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # Create a socket for the server
        server_sock.bind((DEFAULT_SERVER, DEFAULT_PORT)) # Bind the server to the default server and port
        server_sock.listen(MAX_CONNS) # Listen for incoming connections
        print(f"Listening on {DEFAULT_SERVER}:{DEFAULT_PORT}...")

        broadcast_thread = Thread(target=broadcast_handler) # Create a thread for broadcasting messages
        broadcast_thread.daemon = True
        broadcast_thread.start()
```

### Client Side

Our client needs to now simultaneously be able to send messages and receive them from the server. Since we are only sending and receiving from one connection, this will only require 1 thread in addition to our main program. Currently, our `main()` function already implements sending messages correctly, so nothing has to change there; what we do need is an additional function for receiving messages, which we define here as `receive_handler()`:

```python
def receive_handler(client_sock):
    try:
        while True:
            msg = client_sock.recv(MSG_SIZE).decode()
            if msg == "!quit":
                raise Exception("Server quit")
            print(msg)
    except:
        print("Connection lost. Press enter to terminate.")
    client_sock.close()

def main():
    try:
        client_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # Create a socket for the client
        client_sock.connect((DEFAULT_SERVER, DEFAULT_PORT)) # Connect to the server
        print(f"Successfully connected to {DEFAULT_SERVER}:{DEFAULT_PORT}")

        recv_thread = Thread(target=receive_handler, args=(client_sock,)) # Create a thread for receiving messages
        recv_thread.daemon = True
        recv_thread.start()
```

Testing this, I realize I forgot a minor detail that helps clarify where output is coming from. Back in the server code, we do not want to send only the message, but also information about the sender. Therefore, we can modify `broadcast_handler()` as follows:

```python
def broadcast_handler():
    while True:
        if not message_queue.empty():
            (sender, msg) = message_queue.get()
            for c in clients:
                if c != sender: # Don't send the message back to the sender.
                    c.sock.send(f"{sender.addr}> {msg}".encode())
```

That aside, everything works! All messages sent by any of the clients are now being sent to all other clients.

### Error Handling

#### Handling Server Disconnects

As of now, when the server disconnects while the client is still connected, an infinite number of lines is printed. To fix this, we will have a server `!quit` message to let clients know that the server has terminated. To fix this, we will broadcast `!quit` when the server is shutting down in `main()`:

```python
except KeyboardInterrupt:
    print("Interrupt received, terminating server...")
except Exception as e:
    print(f"An error occurred: {e}")

message_queue.put((None, "!quit")) # Tell the broadcast handler to quit

server_sock.close()
```

#### Properly Closing Threads on the Server

So far, we have not been properly managing our threads. At the end of the program, we should ensure that all our threads are closed and then joined to the main program. To do this we first add a list `client_threads` to keep track of the threads that we create in `main()`:

```python
client_threads = []

while True:
    (client_sock, client_addr) = server_sock.accept() # Accept an incoming connection
    client = Client(client_sock, client_addr)
    print(f"Connection from {client.addr}")

    with clients_lock: # Add the client to the list of clients
        clients.append(client)

    client_thread = Thread(target=client_handler, args=(client,)) # Create a thread for the client
    client_thread.daemon = True
    client_thread.start()
    client_threads.append(client_thread)
```

We also need to give `broadcast_handler()` a way to terminate and handle our server `!quit` messages without error:

```python
def broadcast_handler():
    while True:
        if not message_queue.empty():
            (sender, msg) = message_queue.get()
            for c in clients:
                if c != sender: # Don't send the message back to the sender.
                    c.sock.send(f"{sender.addr if sender != None else "server"}> {msg}".encode())
            if sender is None and msg == "!quit":
                break
```

We also make a small modification to `client.py` to handle our new server `!quit` messages:

```python
def receive_handler(client_sock):
    try:
        while True:
            msg = client_sock.recv(MSG_SIZE).decode()
            if msg == "server> !quit":
                raise Exception("Server quit")
            print(msg)
    except:
        print("Connection lost. Press enter to terminate.")
    try:
        client_sock.close()
    except:
        pass
```

Finally, we modify the end of main to properly clean up our threads and sockets:

```python
message_queue.put((None, "!quit")) # Tell the broadcast handler to quit

broadcast_thread.join() # Wait for the broadcast handler to finish

try: # Close any open sockets
    for c in clients:
        c.sock.close()
except:
    pass

for ct in client_threads: # Wait for all client threads to finish
    ct.join()

server_sock.close() # Close the server socket
```

#### Properly Closing Threads on the Client

Closing threads on the client is much easier since all we have to worry about is the receiver thread. At the end of our `main()` in `client.py`, we get rid of the print statement in the final exception and join the receiver thread:

```python
except KeyboardInterrupt:
    print("Interrupt received, quitting...")
    client_sock.send("!quit".encode()) # Tell the server the client is terminating
except Exception as e:
    pass

recv_thread.join() # Wait for the receive thread to finish

try: # Try to close any sockets that may be open
    client_sock.close()
except:
    pass
```

That handles our previously open thread on the client. There is one last problem: when the client terminates now, the receiver thread has no way of terminating. To solve this, we add a global variable `quitting` that is normally set to False. When `main()` is in the process of terminating, it will set `quitting` to `True`. `receive_handler()` will check the value of `quitting` every loop and take action accordingly.

### Prettification

Although **PyChat** is now working as intended, the output is quite ugly, especially when dealing with multiple clients. To fix this, we can use ANSI escape codes in our receiver thread. Modifying `receive_handler()` as follows will enhance the printing of our client's output dramatically:

```python
def receive_handler(client_sock):
    try:
        while True:
            msg = client_sock.recv(MSG_SIZE).decode()
            if msg == "server> !quit":
                raise Exception("Server quit")
            print("\u001B[s", end="", flush=True)     # Save current cursor position
            print("\u001B[A", end="", flush=True)     # Move cursor up one line
            print("\u001B[999D", end="", flush=True)  # Move cursor to beginning of line
            print("\u001B[S", end="", flush=True)     # Scroll up/pan window down 1 line
            print("\u001B[L", end="", flush=True)     # Insert new line
            print(msg, end="", flush=True)            # Print output
            print("\u001B[u", end="", flush=True)     # Jump back to saved cursor position
```

ANSI escape codes can also be used to modify the color of text; we can color our messages!

**TO BE CONTINUED**

## Additional Features

### TLS Encryption

All of the traffic for **PyChat** thus far has been unencrypted. Fixing this is actually relatively easy using the `ssl` library to wrap our sockets. For `server.py`, we need to create an SSL context, load a certificate and private key, and then wrap our socket in the SSL context. The below code demonstrates this:

```python
def main():
    try:
        context = ssl.SSLContext(ssl.PROTOCOL_TLS_SERVER) # Create an SSL context for the server
        context.load_cert_chain(certfile="server.crt", keyfile="private.key") # Load the server's certificate and private key

        server_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # Create a socket for the server
        server_sock.bind((DEFAULT_SERVER, DEFAULT_PORT)) # Bind the server to the default server and port
        server_sock.listen(MAX_CONNS) # Listen for incoming connections
        secure_server_sock = context.wrap_socket(server_sock, server_side=True) # Wrap the server socket in an SSL context
        print(f"Listening on {DEFAULT_SERVER}:{DEFAULT_PORT}...")

        broadcast_thread = Thread(target=broadcast_handler) # Create a thread for broadcasting messages
        broadcast_thread.daemon = True
        broadcast_thread.start()

        client_threads = []

        while True:
            (client_sock, client_addr) = secure_server_sock.accept() # Accept an incoming connection
            client = Client(client_sock, client_addr)
```

For our client, we can just use the default SSL context. For the sake of demonstration, we also disable host name checks and allow self-signed certificates.

```python
def main():
    global quitting
    try:
        context = ssl.create_default_context() # Create an SSL context for the client
        context.check_hostname = False # Disable hostname verification
        context.verify_mode = ssl.CERT_NONE # Disable certificate verification
        client_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM) # Create a socket for the client
        client_sock = context.wrap_socket(client_sock, server_hostname=DEFAULT_SERVER) # Wrap the client socket in an SSL context
        client_sock.connect((DEFAULT_SERVER, DEFAULT_PORT)) # Connect to the server
        print(f"Successfully connected to {DEFAULT_SERVER}:{DEFAULT_PORT}")
```

Before testing this, we must generate our certificate and private key. This can be done with the following command:

```bash
openssl req -newkey rsa:2048 \
-x509 \
-sha256 \
-days 3650 \
-nodes \
-out server.crt \
-keyout private.key \
```

