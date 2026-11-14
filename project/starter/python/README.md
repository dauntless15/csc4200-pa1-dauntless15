# PA1 — ETP/1 Client and Server

**Student:** _Vidhula Oviya Vasanthan_ (`dauntless15`)
**Course:** CSC 4200 / CSC 5200 (delete one) — Fall 2026
**Language:** _Python_
**Submitted commit:** _the SHA you paste into Canvas_

## Build and run

```
make build
make run-server PORT=5000 ROOT=server-files      # in one terminal
./etp-client --host 127.0.0.1 --port 5000 --name YOURNAME list    # in another
```

_Replace this with anything else a grader needs: dependencies you installed, the
Python or gcc version you used, how to run your own tests._

## What works and what does not

_Be specific and honest. Paste the `Execution: N / 30 points` and `Build and
repository: N / 10` lines from `make check`. List any test
that fails and what you believe the cause is. Known limitations go here, not in a
footnote. An honest "S16 fails because ..." earns more than silence._

- This project is the Python 3 implementation of the CSC 4200/5200 PA1 ETP/1 client and server. It uses the standard library only, no third-party dependencies are required, and there are no extra libraries beyond the Python runtime itself. The build is driven by the provided Makefile, and the repository is expected to include the required launchers, source modules, documentation, and evidence files.

Run these commands from a fresh clone before you write them down, so you know they work.

- Build and run instructions: from a fresh clone, run make build to compile the Python modules and make the launchers executable. The server is started with ./etp-server --port 5000 --root server-files and the client is run with ./etp-client --host 127.0.0.1 --port 5000 --name YOURNAME list. Example operations include ./etp-client --host 127.0.0.1 --port 5000 --name YOURNAME get remote-file local-file and ./etp-client --host 127.0.0.1 --port 5000 --name YOURNAME put local-file remote-file. To run the course self-check, use make check, which in this starter is equivalent to running python3 ~/csc4200-course/project/etp_selfcheck.py or the project-local self-check command from the course kit.

- What works: the project self-check reports Execution: 30 / 30 points with S01-S18 and C01-C05 passing. This means the protocol handshake, listing, GET/PUT behavior, error handling, fragmentation, multi-client concurrency, BYE teardown, and client-side validation all matched the grader’s expected behavior. The pass result was earned by running the provided self-check after the implementation was complete and after the function and transport logic were corrected to match the ETP/1 requirements.

- The specific test output confirms that each server scenario succeeded: S01-S18 all passed, including the edge cases for fragmented reads, multiple requests in a single write, invalid message types, bad version handling, oversized lengths, HELLO ordering, BYE cleanup, stalled clients, and restart-on-the-same-port behavior. The client side also succeeded for C01-C05, which verifies list output, fragmented GETs, PUT payload formatting and digest verification, server ERROR handling, and digest mismatch exit status handling. That combination of passing server and client tests is what gave the project its full 30/30 execution score.

- The repository/build check reported Build and repository: 9 / 10 at the time of the run because the evidence files were still missing: evidence/time-wait.txt and evidence/pa1-capture.pcap. This was the honest state at that moment, and it reflects that the implementation itself was passing while the documentation/evidence artifacts were not yet included. The execution score is separate from the repository score, so the protocol behavior could still be fully passing even though the evidence requirements were not yet complete.

- Known limitations: the server does not enforce a connection cap, it holds payloads in memory as implemented, and there is no idle timeout logic beyond the basic socket handling. These are real behavioral constraints of the current implementation, but they did not block the self-check from passing because the grader’s execution tests focus on protocol correctness and concurrency behavior rather than long-running resource-management limits.

- Known limitations: the server does not enforce a connection cap, it holds payloads in memory as implemented, and if no idle timeout logic was added, there is no idle timeout behavior beyond the basic socket handling.

## Files

| File | What it is |
|---|---|
| `src/...` | 
| `src/transport_tcp.py` | transport impelmentation |
| `src/etp.py contains` | ETP/1 constants and protocol logic |
| `src/server.py` | impelment the server |
| `src/client.py` | impelmentat the client |
| `DESIGN.md` | design document |
| `AI_USE.md` | AI-use disclosure and log |
| `evidence/pa1-capture.pcap` | loopback capture of my PUT and GET sessions |
| `evidence/time-wait.txt` | `ss` output showing TIME-WAIT after my session |
| `evidence/ANALYSIS.md` | my capture analysis |
| `evidence/captureData.txt` | my pcap capture's data |
| `Makefile` | uilds the project and runs the server, etp-server and etp-client are the launchers |


---

## Authorship statement

I am the author of this submission. Except where this README or `AI_USE.md`
says otherwise, I wrote, or rewrote and understand, every part of it — code,
documents and evidence — and I can explain any line of it to the instructor or
GTA. I am responsible for its correctness, including any part a
generative AI tool helped me produce. My use of AI tools is disclosed in full in
`AI_USE.md`. The capture and measurements in `evidence/` come from my own programs,
run by me.

**Vidhula Oviya Vasanthan** _Vidhula Oviya Vasanthan_  **Date:** _11/12/26_
