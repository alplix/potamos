# Potamos

Potamos is the **sequential brute-force scan** of the [BitBOINC](https://bitboinc.athena.org.tr/) project. It looks for the private key of a Bitcoin puzzle address in a limited key interval, one disjoint piece at a time.

> **This repository contains documentation only. It does not contain source code, work-unit parameters, searched ranges or server code.** See [What is not published, and why](#what-is-not-published-and-why).

## The problem

A Bitcoin puzzle gives an address and says that the private key lies in an interval. When only the **address** (a hash of the public key) is known, there is no shortcut: every candidate private key has to be turned into an address and compared. Keraunos, the project's kangaroo search, needs the public key to be known; Potamos is for the puzzles where it is not.

## The method

For each candidate private key `k` in the piece assigned to a task:

1. compute the public point `k·G` on secp256k1 (consecutive keys differ by one addition, so this is done incrementally);
2. compute its hash (SHA-256, then RIPEMD-160);
3. compare with the target address hash.

A match is a candidate solution. Nothing else is stored.

## How it runs on BOINC

- The server splits the key interval into **disjoint chunks** and hands them out; each chunk is claimed atomically so that two tasks never receive the same one.
- A task scans its chunk and writes a checkpoint, so it can resume after an interruption.
- The result file only records a match; it does not list the keys that were tried.
- One puzzle is worked on at a time. Credit is a flat amount per task. Earlier GPU variants have been retired; the current work runs on CPUs.

## Correctness and safety

- **Server-side verification.** A reported match is recomputed on the server (`k·G`, SHA-256, RIPEMD-160) and compared with the target address before the puzzle counts as solved. Wrong or fake reports cannot mark a puzzle as solved.
- A separate sweeper checks result files for a match that the normal path missed, so no match is lost.

## What happens if a key is found

- The candidate key is **encrypted immediately with an RSA-4096 (OAEP) public key**. Only the **ciphertext** is stored (database, log and an extra backup file); the plaintext key never touches the server's disk or logs.
- The matching **private key is kept offline** by the project owner and has never been on the server.
- Notifications about a solve never contain the key.
- The key is only used for puzzles that were created to be solved. The project does not search for the keys of arbitrary addresses.

## What volunteers' computers do and do not do

They scan their chunk and report whether it contained a match. They do not mine, do not touch any wallet or file of the user, and the project stores no personal data about the search itself. See the project's privacy page.

## What is not published, and why

The source code, the chunking and bookkeeping, and the progress of the scan are deliberately **not published**. The prize goes to the first finder, and knowing which chunks were already scanned would let others skip or copy the work. The correctness and security design above is published so that it can be reviewed without the scanner.

## License

The text of this repository is released under the GPL-3.0 (see `LICENSE`).
