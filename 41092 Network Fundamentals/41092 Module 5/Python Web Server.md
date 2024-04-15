```python
from socket import *

# set port up
serverPort = 1200
serverSocket = socket(AF_INET, SOCK_STREAM)
serverSocket.bind(("", serverPort))
serverSocket.listen(1)

while True:
    connectionSocket, addr = serverSocket.accept()
    try:
        # create new socket
        message = connectionSocket.recv(1024)
        
        # read requested file
        filename = message.split()[1]
        f = open(filename[1:])
        
        # send data
        outputdata = f.read()
        connectionSocket.send("HTTP/1.1 200 OK\r\n\r\n".encode('utf-8'))
        
        for i in range(0, len(outputdata)):
            connectionSocket.send(outputdata[i].encode('utf-8'))
        connectionSocket.send("\r\n".encode('utf-8'))
    except IOError:
        connectionSocket.send("HTTP/1.1 404 Not Found\r\n\r\n".encode('utf-8'))
        connectionSocket.send("<html><head></head><body><h1>404 Not Found</h1></head></html>".encode('utf-8'))
    except KeyboardInterrupt:
        connectionSocket.close()
serverSocket.close()
```

`from socket import *` - Importing modules
`serverPort = 1200` - Defining the port for the server
`serverSocket = socket(AF_INET, SOCK_STREAM)` - Creating a socket object for the server
`serverSocket.bind(("", serverPort))` - Binding the server socket to the specified port
`serverSocket.listen(1)` - Setting the server to listen for incoming connections
`while True:` - Beginning of an infinite loop to keep the server running
`connectionSocket, addr = serverSocket.accept()` - Accepting a connection request from a client and assigning connectionSocket and addr with values
`try:` - Beginning of a try block to handle exceptions
`message = connectionSocket.recv(1024)` - Receiving the request message from the client
	1024 is the buffer size, message is a variable
`filename = message.split()[1]` - Parsing the requested filename from the request message
	delimeter of white space,  the second string in the created list
`f = open(filename[1:])` - Opening the requested file, enabling the python program to read
	1: is chosing the second character in the filename variable
`outputdata = f.read()` - Reading the content of the requested file and putting it to a variable
`connectionSocket.send("HTTP/1.1 200 OK\r\n\r\n".encode('utf-8'))` - Sending a success response header
`for i in range(0, len(outputdata)):` - Iterating through the file content to send it
`connectionSocket.send(outputdata[i].encode('utf-8'))` - Sending each character of the file content and encoding one by one
`connectionSocket.send("\r\n".encode('utf-8'))` - Sending an end-of-response marker
`except IOError:` - Handling IOError if the requested file is not found
`connectionSocket.send("HTTP/1.1 404 Not Found\r\n\r\n".encode('utf-8'))` - Sending a 404 response header
`connectionSocket.send("<html><head></head><body><h1>404 Not Found</h1></head></html>".encode('utf-8'))` - Sending a custom HTML error page
`except KeyboardInterrupt:` - Handling KeyboardInterrupt exception to gracefully exit the server.
`connectionSocket.close()` - Closing the connection socket
`serverSocket.close()` - Closing the server socket.


# Good version
```python
from socket import *

# set port up
serverPort = 1200
serverSocket = socket(AF_INET, SOCK_STREAM)
serverSocket.bind(("", serverPort))
serverSocket.listen(1)

while True:
    connectionSocket, addr = serverSocket.accept()
    try:
        # create new socket
        message = connectionSocket.recv(1024)
        
        # read requested file
        filename = message.split()[1]
        f = open(filename[1:])
        
        # send data
        outputdata = f.read()
        connectionSocket.send("HTTP/1.1 200 OK\r\n\r\n".encode('utf-8'))
        
        connectionSocket.sendall(outputdata.encode('utf-8))
        connectionSocket.send("\r\n".encode('utf-8'))
        connectionSocket.close()
    except IOError:
        connectionSocket.send("HTTP/1.1 404 Not Found\r\n\r\n".encode('utf-8'))
        connectionSocket.send("<html><head></head><body><h1>404 Not Found</h1></head></html>".encode('utf-8'))
        connectionSocket.close()
    except KeyboardInterrupt:
        serverSocket.close()
```

The packet sent from the browser looks like this:
```
b'GET /simpleWeb.html HTTP/1.1\r\nHost: 127.0.0.1:1200\r\nConnection: keep-alive\r\nCache-Control: max-age=0\r\nsec-ch-ua: "Not_A Brand";v="8", "Chromium";v="120", "Brave";v="120"\r\nsec-ch-ua-mobile: ?0\r\nsec-ch-ua-platform: "Windows"\r\nUpgrade-Insecure-Requests: 1\r\nUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36\r\nAccept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8\r\nSec-GPC: 1\r\nSec-Fetch-Site: cross-site\r\nSec-Fetch-Mode: navigate\r\nSec-Fetch-User: ?1\r\nSec-Fetch-Dest: document\r\nAccept-Encoding: gzip, deflate, br\r\nAccept-Language: en-US,en;q=0.9\r\n\r\n'
```

You can notice some changes when it's loaded in another browser
```
b'GET /simpleWeb.html HTTP/1.1\r\nHost: 127.0.0.1:1200\r\nConnection: keep-alive\r\nCache-Control: max-age=0\r\nsec-ch-ua: "Not A(Brand";v="99", "Opera GX";v="107", "Chromium";v="121"\r\nsec-ch-ua-mobile: ?0\r\nsec-ch-ua-platform: "Windows"\r\nUpgrade-Insecure-Requests: 1\r\nUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36 OPR/107.0.0.0\r\nAccept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7\r\nSec-Fetch-Site: none\r\nSec-Fetch-Mode: navigate\r\nSec-Fetch-User: ?1\r\nSec-Fetch-Dest: document\r\nAccept-Encoding: gzip, deflate, br\r\nAccept-Language: en-US,en;q=0.9\r\nCookie: SLG_G_WPT_TO=en; SLG_GWPT_Show_Hide_tmp=1; SLG_wptGlobTipTmp=1; X-CSRF-Token=2931da15db1301e6ca6896d2b7a23ce6fdd0b2b7e4902059f8de4aba9228b8b6\r\n\r\n'
```

If we use `.split()` with a default delimiter, we get a list that looks like this:
`[b'GET', b'/simpleWeb.html', b'HTTP/1.1\r\nHost:',...]`
An index of 1 would select `b'/simpleWeb.html'` which is the file we want
This b is apart of the encoding to utf-8, we don't want it so when opening the file we use `[1:]` to cut it out

```
# serverSocket before bind
<socket.socket fd=340, family=2, type=1, proto=0>
# serverSocket after bind
<socket.socket fd=340, family=2, type=1, proto=0, laddr=('0.0.0.0', 1200)>
# serverSocket after listen
<socket.socket fd=340, family=2, type=1, proto=0, laddr=('0.0.0.0', 1200)>
# connectionSocket
<socket.socket fd=328, family=2, type=1, proto=0, laddr=('127.0.0.1', 1200), raddr=('127.0.0.1', 53511)>
# addr
('127.0.0.1', 53511)
```

