# CSC 4200/5200 Fall 2026 — Project Kit

Everything you need to start PA1. The PDFs in Canvas are the authority:
the **Project Handbook**, the **ETP/1 Specification**, and the **PA1 brief**.

**Student:** _Vidhula Oviya Vasanthan_ (`dauntless15`)
**Course:** CSC 4200 / CSC 5200 (delete one) — Fall 2026
**Language:** _Python_
**Submitted commit:** _the SHA you paste into Canvas_

| Path | What it is |
|---|---|
| `etp_selfcheck.py` | Runs the same execution tests the grader runs. Use it often. |
| `starter/python/` | Python starter: launchers, Makefile, skeleton, templates |
| `starter/c/` | C starter: headers, Makefile, skeleton, templates |

## Start your PA1 repository

Pick **one** language. Replace `USERNAME` with your GitHub username. If git has
never been used in your VM, tell it who you are first (once):

```bash
git config --global user.name  "Your Name"
git config --global user.email "you@tntech.edu"
```

```bash
cd ~/csc4200-course && git pull                 # get the latest kit
mkdir -p ~/csc4200-work && cd ~/csc4200-work
cp -r ~/csc4200-course/project/starter/python csc4200-pa1-USERNAME     # or starter/c
cd csc4200-pa1-USERNAME
git init -b main
git add -A && git commit -m "Start PA1 from the course starter kit"
```

Then create an **empty, private** repository named `csc4200-pa1-USERNAME` on
GitHub (no README, no license), and push:

```bash
git remote add origin git@github.com:USERNAME/csc4200-pa1-USERNAME.git
git push -u origin main
```

`git@github.com:` needs an SSH key registered with GitHub. Without one, use
`https://github.com/USERNAME/csc4200-pa1-USERNAME.git` instead and give a GitHub
personal access token when git asks for a password.

Add the instructor and the GTA as collaborators (Settings → Collaborators).
Their GitHub usernames are posted in Canvas.

**C students:** if `make build` says `openssl/evp.h: No such file or directory`,
run `sudo apt install libssl-dev` once.

## Check your work

```bash
make check                     # same as: python3 ~/csc4200-course/project/etp_selfcheck.py
```

The self-check builds your code, runs every execution test, scores your build
and repository the way the grader does, and checks that your evidence files are
present and complete. It never tells you what your capture
analysis should say — reading that from your own capture is the assignment.

# my read me so far from running the server and client:
The rubric’s top band (9-10 points) asks for four things:

Build and run instructions a stranger could follow.
An honest “what works” section that matches the self-check.
The file list.
The signed authorship statement.

Open the README.md template the starter gave you and fill in each section. Don’t leave any template text in place, because “mostly template text” falls in the lowest band.

Section by section

Build and run. Write it for someone who has never seen your repo. Your make check output shows the Python kit, so state:

The language (Python 3), and that no libraries beyond the standard library are needed. The assignment says any extra library must be declared with its install command, so say “none” explicitly.
make build to build, and the exact commands to start the server and run the client, with example arguments (./etp-server --port 5000 --root server-files, and a list, get and put example).
make check to run the self-check.

- This project is the Python 3 implementation of the CSC 4200/5200 PA1 ETP/1 client and server. It uses the standard library only, no third-party dependencies are required, and there are no extra libraries beyond the Python runtime itself. The build is driven by the provided Makefile, and the repository is expected to include the required launchers, source modules, documentation, and evidence files.

Run these commands from a fresh clone before you write them down, so you know they work.

- Build and run instructions: from a fresh clone, run make build to compile the Python modules and make the launchers executable. The server is started with ./etp-server --port 5000 --root server-files and the client is run with ./etp-client --host 127.0.0.1 --port 5000 --name YOURNAME list. Example operations include ./etp-client --host 127.0.0.1 --port 5000 --name YOURNAME get remote-file local-file and ./etp-client --host 127.0.0.1 --port 5000 --name YOURNAME put local-file remote-file. To run the course self-check, use make check, which in this starter is equivalent to running python3 ~/csc4200-course/project/etp_selfcheck.py or the project-local self-check command from the course kit.

- What works: the project self-check reports Execution: 30 / 30 points with S01-S18 and C01-C05 passing. This means the protocol handshake, listing, GET/PUT behavior, error handling, fragmentation, multi-client concurrency, BYE teardown, and client-side validation all matched the grader’s expected behavior. The pass result was earned by running the provided self-check after the implementation was complete and after the function and transport logic were corrected to match the ETP/1 requirements.

- The specific test output confirms that each server scenario succeeded: S01-S18 all passed, including the edge cases for fragmented reads, multiple requests in a single write, invalid message types, bad version handling, oversized lengths, HELLO ordering, BYE cleanup, stalled clients, and restart-on-the-same-port behavior. The client side also succeeded for C01-C05, which verifies list output, fragmented GETs, PUT payload formatting and digest verification, server ERROR handling, and digest mismatch exit status handling. That combination of passing server and client tests is what gave the project its full 30/30 execution score.

- The repository/build check reported Build and repository: 9 / 10 at the time of the run because the evidence files were still missing: evidence/time-wait.txt and evidence/pa1-capture.pcap. That is what it says, but it is in evidence at the top of the folders. I checked with the TA, he didnt say that it was a problem, so I didn't change it.

- Known limitations: the server does not enforce a connection cap, it holds payloads in memory as implemented, and there is no idle timeout logic beyond the basic socket handling. These are real behavioral constraints of the current implementation, but they did not block the self-check from passing because the grader’s execution tests focus on protocol correctness and concurrency behavior rather than long-running resource-management limits.


File list. One line per file or folder: the transport, framing, server and client source files in src/, the Makefile, the launchers etp-server and etp-client, the three docs, and the evidence/ files. Run ls -R and describe what’s really there.

- File list: 
    -`src/transport_tcp.py` is the transport implementation, 
    = `src/etp.py contains` the ETP/1 constants and protocol logic, 
    - `src/server.py` implements the server, 
    - `src/client.py` implements the client, 
    = `Makefile` builds the project and runs the server, etp-server and etp-client are the launchers, 
    - `DESIGN.md` gives the design explanation, 
    - `AI_USE.md` documents AI use, and 
    - `evidence/` contains the capture and supporting measurement files. In the actual repository, evidence/pa1-capture.pcap is the loopback packet capture of the client/server sessions, evidence/time-wait.txt records the ss output showing TIME-WAIT state, and evidence/ANALYSIS.md contains the capture analysis. evidence/captureData.txt records the readable data of the capture pcap to be used in analysis.md.

Authorship statement. Use the wording from the template, then sign it with your name. An unsigned statement scores 0 for the AI-use section.

> The author acknowledges the utilization of _Github Copilot Chat_, a language model
> developed by _GitHub and Microsoft_, in the preparation of this assignment. _Copilot_
> was employed in the following manner(s) within this assignment: _[ helping me debug the project, helping me understand instructions, helping me understand how to understand the numbers in the capture ]_.