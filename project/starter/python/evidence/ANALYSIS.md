# PA1 Capture Analysis

_Analysis reviewed on 2026-11-12 using the current packet capture and its
companion packet listing. The packet facts below are from the capture, not
reconstructed from the earlier analysis._

## How I captured it

From the repository root, the analyzed file is
`evidence/pa1-capture.pcap`. Its pcap header reports a 200-byte snapshot
length, and it contains 105 complete packet records. The companion
`evidence/captureData.txt` is a TShark packet listing refreshed from that
capture on 2026-11-12. The original packet-capture, server, and client shell
commands were not saved, so I cannot reliably report them as commands I ran.

The capture contains two loopback ETP/1 sessions: a PUT session followed by a
GET session for `photo.bin`. The file in `server-files/photo.bin` is 262144
bytes. The first session uses client port 60886; the GET session analyzed below
uses client port 60905. Both connect to server port 5000.

The 200-byte snapshot setting truncates the saved bytes of the large packets;
it does not indicate that the transfer itself was only 200 bytes. TShark's
packet list reports the original frame and TCP payload lengths.

## Facts

```etp-facts
server_port: 5000
client_port: 60905
client_isn: 4293186993
server_isn: 3419342164
client_mss: 16344
server_to_client_payload_bytes: 262180
server_to_client_data_segments: 19
first_fin_from: server
```


## Q1. The handshake

The GET session's three-way handshake is:

- Frame 53: client `127.0.0.1:60905` to server `127.0.0.1:5000`, `SYN`,
  raw sequence number 4293186993, acknowledgment number 0.
- Frame 54: server to client, `SYN, ACK`, raw sequence number 3419342164,
  acknowledgment number 4293186994.
- Frame 55: client to server, `ACK`, raw sequence number 4293186994,
  acknowledgment number 3419342165.

The SYN consumes one sequence number, so the server acknowledges the client's initial sequence number plus one: 4293186993 + 1 = 4293186994. 
The client's final ACK similarly acknowledges the server's SYN at 3419342164 + 1 = 3419342165.

The client advertised an MSS of 16344 bytes in its SYN. MSS is derived from the
network path/interface MTU and the IP and base TCP header sizes; it is not set
by the ETP application. TCP options take additional room in actual packets, so
the data segments below carry 16332 bytes rather than the advertised MSS.

## Q2. Your protocol on the wire

The client's GET request is in frame 61. Its complete TCP payload is 17 bytes:
the 8-byte ETP header followed by the 9-byte filename `photo.bin`. The header
bytes are:

```
01 03 00 00 00 00 00 09
```

Decoded:

- VERSION: `1`
- TYPE: `0x03` (GET)
- STATUS: `0`
- RESERVED: `0`
- LENGTH: `9`

The server's GET_OK header begins in frame 63:

```
01 83 00 00 00 04 00 00
```

Decoded:

- VERSION: `1`
- TYPE: `0x83` (GET_OK)
- STATUS: `0`
- RESERVED: `0`
- LENGTH: `0x00040000` = 262144 bytes

The GET_OK length equals the size of `server-files/photo.bin` (262144 bytes).
The server-to-client TCP payload total comprises all three responses:

```
HELLO_OK: 8-byte ETP header + 12-byte payload = 20 bytes (frame 59)
GET_OK: 8-byte ETP header + 262144-byte file = 262152 bytes (frames 63-89)
BYE_OK: 8-byte ETP header + 0-byte payload = 8 bytes (frame 100)

Total: 20 + 262152 + 8 = 262180 bytes
```

There are 19 server-to-client data segments in total: one for HELLO_OK, 17 for
GET_OK, and one for BYE_OK.

## Q3. A byte stream, not messages

The GET_OK response spans 17 TCP data segments: frames 63-72 and 83-89. Sixteen
segments carry 16332 bytes each, and the final segment carries 840 bytes:

```
16 * 16332 + 840 = 262152 bytes
```

The segments are not all the same size, and none reaches the client's
advertised MSS of 16344 bytes. The IP packet length for a full-sized data frame
is 16384 bytes. It contains a 20-byte IPv4 header and a 32-byte TCP header
(including options), leaving 16332 bytes for TCP data.

No segment in this GET response also carries the end or beginning of another
ETP message. GET_OK occupies its own multi-segment TCP byte range BYE_OK is
sent later, after the client sends BYE. TCP doesnt preserve application
message boundaries, so a receiver must read the fixed-size ETP header, parse
LENGTH, and then read exactly that many payload bytes. In this implementation,
`src/etp.py`'s `read_message()` reads the header and body using
`transport.recv_exact()` from `src/transport_tcp.py`.

## Q4. Teardown and TIME-WAIT

In the GET session the client sends BYE in frame 98, the server replies with
BYE_OK in frame 100, and the server sends the first FIN in frame 102. The
client ACKs that FIN in frame 103, then sends its FIN in frame 104, and the server
ACKs it in frame 105. This matches the session-close behavior in which the
server acknowledges a valid BYE and then closes its side of the connection.
Because the server sends the first FIN it is the active closer and its
endpoint `127.0.0.1:5000` is the endpoint expected to enter TIME-WAIT.
The repository-root `evidence/time-wait.txt` does noy show a TIME-WAIT
socket. It contains only these LISTEN rows:

```
tcp4       0      0  127.0.0.1.5000         *.*                    LISTEN
tcp6       0      0  *.5000                 *.*                    LISTEN
tcp4       0      0  *.5000                 *.*                    LISTEN
```

That listing was updated before the current pcap, and it does not prove the
state of the connection in this capture. The packet trace supports the
expected TIME-WAIT endpoint, but a matching socket-inspection snapshot after
the session is still needed as direct evidence. 

The server sets `SO_REUSEADDR` before `bind()` in `src/transport_tcp.py`.
This supports restarting the server on the same local address and port while
recent connections may still be in TIME-WAIT, subject to the operating
system's socket-reuse rules.

## Q5. What an observer learns

This capture contains both sessions, not only the GET session. An observer
learns:

- Both endpoints are on loopback (`127.0.0.1`), with server port 5000.
- The PUT client's ephemeral port is 60886 & the GET client's is 60905.
- The HELLO request contains the literal client name `YOURNAME`, and the
  server identifies itself as `ETP/1 Server`.
- Both operations use the filename `photo.bin`; the GET_OK length reveals
  that its size is 262144 bytes.
- The sessions use ETP/1 and exchange HELLO, PUT or GET, and BYE messages.
The PUT_OK response also exposes its 32-byte digest.
- Packet metadata exposes sequence and acknowledgment numbers, advertised
  MSS and window values, timing, transfer direction, and approximate data
  volume. The GET session begins about 14.908 seconds after the PUT session
  in the capture's relative timeline.
- The first bytes of the large file data are visible in captured packet
  payloads; the trace is not encrypted.

With a 200-byte snapshot, a full-sized frame whose original length is 16388
bytes retains only its first 200 bytes. In this capture that includes the
4-byte loopback header, 20-byte IPv4 header, 32-byte TCP header, and the first
144 bytes of TCP payload. The remaining bytes of each large frame are absent
from the pcap, so it does not contain a complete copy of the transferred file.
Short control packets that fit within the snapshot are captured in full. The
original frame and TCP lengths remain visible in packet metadata despite the
missing payload bytes.

End-to-end TLS (another authenticated-encryption protocol between the
applications) would protect the application contents from an on-path
observer. It would not hide the network endpoints, timing, or approximate
traffic volume. A VPN can conceal inner addresses and contents from an
observer outside the tunnel, but that observer can still see the VPN
endpoints and traffic metadata.
