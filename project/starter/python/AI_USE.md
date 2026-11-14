# AI Use Disclosure — PA1

_Generative AI is allowed on PA1 as a tutor, reviewer, debugging partner and
helper with tooling. It is not allowed to be the author of your work. See the PA1
brief, "AI, Authorship and You". Fill in every section. "None" is an acceptable
answer only if it is true._

## 1. Did you use generative AI on this assignment?

- [ ] No. I did not use any generative AI tool for any part of PA1.
- [x] Yes, as described below.

## 2. Statement (the syllabus form)

> The author acknowledges the utilization of GitHub Copilot (Copilot SDK in VS
> Code), a language model-based coding assistant developed by Microsoft, in the
> preparation of this assignment. GitHub Copilot was employed in the following
> manner(s) within this assignment: clarifying the PA1 AI-use disclosure
> requirements, checking the wording of the disclosure form, and reviewing the
> available repo/session history to complete the required disclosure sections.

## 3. Log

_One row per substantive interaction. Summaries are fine; you do not need to paste
whole conversations, but keep them in case you are asked._

| # | What I asked (summary) | What I used from the answer | What I changed or rejected | How I checked it was right |
|---|---|---|---|---|
| 1 | Asked for help interpreting the PA1 AI disclosure requirements and the exact wording needed for Sections 3-5. | I used the requirement to log substantive AI interactions, to identify AI-produced material that remains in the repo, and to explain either a wrong AI answer or a verification method. | I kept the answer limited to the disclosure form and did not add any AI-generated code or text into the repository. | I checked it against the wording in this document and the assignment guidance. |
| 2 | Asked whether there were other project sessions or prior AI-use records to include in the disclosure, and if so how to summarize them. | I used the result that no additional project sessions were present in the available local/cloud session history for this repo. | I did not include any undocumented or fabricated prior-project AI interactions; the available record is limited to this chat. | I verified this by checking the local and cloud session metadata for this repository, which returned no additional prior sessions. |
| 3 | Asked how to run `ss -tan state time-wait '( sport = :5000 or dport = :5000 )' > evidence/time-wait.txt` in the terminal on macOS. | I used the suggested `mkdir -p evidence` and macOS `netstat` command to collect port 5000 output into `evidence/time-wait.txt`. | The suggested `netstat -an -p tcp | grep '\.5000 '` command filters by port but not by TIME-WAIT state, so its output is not equivalent to the requested `ss` filter. | I checked `evidence/time-wait.txt`; it contains LISTEN rows and no TIME-WAIT entry, so it does not establish whether TIME-WAIT sockets exist. |
| 4 | Asked how it to give me step by step instruction on how I can execute the whole project- not the code, but just the instructions | I used it to understand what I am doing, what files I should be editing, etc. | I didn't change or reject anything, but I did get messed with the terminal instructions sometimes since I was in a mac, so I went to office hours to get it fixed. | I checked it by ensuring that I followed these instructions, still kept up with the packets provided, and met up with the ta to cross check or troubleshoot anything.
| 5| Asked it to debug why all my server tests were showing up as connection failed. | I was trying to understand why all my connections were failing- I didn't ask it to fix anything for me though- just use it for some troubleshooting. | I didn't change or reject anything | It helped me realize that I had misspelt functions in a lot of different places, and after fixing it the tests seemed to actually get the connection.
| 6 | Asked it to rewrite my Readme in a bit more consistent manner. | I was trying to use a bit more better wording to get my point across. | I didn't change or reject anything | I checked if it was still using the same points that I was making and used it to rewrite the readme.

## 4. Where AI-produced material remains

This file (`AI_USE.md`) contains AI-assisted disclosure wording and a summary of
this interaction. No application code was generated. `evidence/time-wait.txt`
is terminal output collected using an AI-suggested command, not AI-generated
content. (The terminal command wasn't executing properly, so with the suggestionsof my ta I gave copilot the terminal instruction and asked it to execute it for me in a suitable manner)

## 5. One thing the AI got wrong — or how you checked it

The macOS command suggested for the requested `ss` filter matched port 5000
without filtering for TIME-WAIT state. I checked `evidence/time-wait.txt` and
found only LISTEN rows, with no TIME-WAIT entry. The file therefore does not
provide evidence about TIME-WAIT sockets. But after talking with my TA I realized the problem actually wasn't with my timewait.txt, but with getting the data using the t shark command in `pa1-capture.pcap`.
