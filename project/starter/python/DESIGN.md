# PA1 Design — ETP/1 Client and Server

_One to two pages. Short, specific answers beat long general ones. Every section
should be true of the code you submit. Delete these italic prompts when you write._

## 1. Architecture

_Your modules and what calls what. A small diagram (ASCII is fine) or a list:
which file holds the protocol (encode/decode), which holds the transport, which
holds the server's session logic, which holds the client._

- The ETP logic is split across the files in the Python starter: 
    src/etp.py defines the protocol constants, the 8-byte header format, and the shared encode/read_message logic
    
    src/transport.py is the transport seam and simply re-exports whichever concrete transport is active
    
    src/transport_tcp.py is the only file that touches the socket API and implements listen/accept/connect/send_all/recv_exact/close
    
    src/server.py owns the session loop, message dispatch, file-system operations, and per-client threading
    
    src/client.py performs the HELLO/request/response flow, local file I/O, digest checks, and exit-status handling
    
    - a smal visualization
    client.py -> transport.py -> transport_tcp.py -> socket
    
    server.py -> transport.py -> transport_tcp.py -> socket



## 2. The transport seam

_Which file is your transport? List the six operations as you implemented them.
Confirm that no other file calls the socket API. In two or three sentences: what
would you have to change, and what would stay the same, when you swap TCP for your
PA2 transport in the final project?_

- The seam is in src/transport.py: server.py and client.py import transport, never transport_tcp directly. In PA1 that concrete module is src/transport_tcp.py, which provides the six required operations: listen, accept, connect, send_all, recv_exact, and close (plus close_listener). 
- No other file calls socket.recv, socket.sendall, or socket.create_connection. 
- If we switched to the PA2 transport, the only required code change would be the implementation behind the same function names in a new module and the import line in transport.py. the protocol logic, server session logic, and client behavior would stay the same.

## 3. Framing

_How does your code read one ETP message from a byte stream? Where and how do you
handle a header that arrives in pieces, or two messages that arrive in one read?
Where do you check VERSION and LENGTH, and why before reading the payload?_

- The ETP framing lives in src/etp.py with HEADER = struct.Struct("!BBBBI"), which is 8 bytes: VERSION, TYPE, STATUS, RESERVED, LENGTH. The code reads exactly 8 bytes first, unpacks the header, and only then reads the declared payload using transport.recv_exact(conn, length). TCP may split one message across reads or coalesce several messages into one read, so the fixed header must be parsed before any payload is consumed. The code checks the protocol version and maximum length before it reads the payload body because a malformed or impossible length would otherwise make the server/client consume bytes that may belong to the next message or to a different session.



## 4. Concurrency

_Threads, processes or an event loop? What happens to everyone else when one
client stalls half-way through a header? Is any state shared between sessions
(for example, two clients PUT and GET the same file at once), and how is it
protected?_

- The server uses a thread per accepted client connection, created in src/server.py with threading.Thread(target=handle_session, ...). Each session maintains its own state machine and reads requests in a loop, so a stalled client in the middle of a header only blocks that worker threa. The listener continues accepting other clients. The only explicit shared mutable object is the stdout log lock, which serializes log writes, while the actual data store is the shared root directory on disk. There is no global in-memory lock around files, so two clients can PUT or GET the same file concurrently, so the code relies on OS filesystem behavior and the fact that PUT writes a temp file and then os.replace() to avoid partial writes, rather than a stronger application-level mutex.

## 5. Errors and disconnects

_How do fatal and non-fatal errors differ in your code? What happens when a client
disconnects in the middle of a GET of a large file? (C: how did you deal with
SIGPIPE?) How does your client choose its exit code?_

- The server distinguishes non-fatal and fatal protocol errors by status and by the etp.FATAL_ERRORS set: BAD_VERSION, TOO_LARGE, and NOT_READY close the current session, while errors like BAD_REQUEST, BAD_NAME, and NOT_FOUND are reported and the session continues. send_error() emits an ETP ERROR response, logs it, and returns whether the error is fatal. If a client disconnects mid-transfer, transport.recv_exact() raises ConnectionClosed(clean=False), which the session loop catches and closes that connection without crashing the server. The client chooses exit codes from the protocol and local-file results: 0 for success, 1 for server error, 2 for client/runtime failure, and 3 for digest mismatch after PUT. SIGPIPE is avoided by catching BrokenPipeError/ConnectionResetError in the send path and by the graceful TCP shutdown in transport.close(), which calls shutdown(SHUT_WR) and drains remaining bytes before closing.

## 6. Threat model

_One paragraph. Who could attack an ETP/1 server, and how? Name one attack your
server resists (and how), and one it does not. Chapter 5 §5.9 and the
specification's Security Considerations are good starting points._

- The server can be attacked by any remote client that reaches the port, including malformed packets, oversized headers, impossible lengths, invalid message types, path traversal names, or repeated requests to exhaust CPU or disk space. The code resists several attacks. it validates the ETP version and message type, rejects lengths above MAX_LEN, checks file names with etp.valid_name(), prevents symlink traversal via os.O_NOFOLLOW and stat.S_ISREG, and uses temporary files plus atomic replace during PUT. The server doesnt fully resist a malicious client that fills the root directory with large uploads or floods the server with many concurrent connections, because there is no quota enforcement or cross-session locking beyond the OS and the per-session thread model.

## 7. Testing

_What did you test beyond `etp_selfcheck.py`? How? Name one bug your testing
found, how you found it, and the fix._

- I checked the repo with the self-check (make build, then python3 etp_selfcheck.py --no-build), and the execution score was 30/30: all server tests and client tests passed. I also exercised the server and client manually with the loopback commands in the repo and reviewed the packet capture/evidence files. One concrete bug I found during debugging was a connection/setup issue caused by misspelled function names and bad wiring in the server/client call paths; once those were corrected, the self-check and inter-process runs succeeded. The repo’s build-and-repository score is 9/10 because the required evidence files are still missing: evidence/time-wait.txt and evidence/pa1-capture.pcap. The code itself passes the execution checks, and the missing artifacts are documentation/evidence items, not runtime protocol failures. I was also having some issues with caputring the evidence and reading the capture with tshark, but I worked it out with my ta.