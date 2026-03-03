# ⚡ Direct-Drop (🚧 Active Development)
**A native, zero-dependency LAN file-sharing architecture.**

> **Status:** 🟢 Building in Public. I am developing this engine phase-by-phase and pushing daily updates. 

## 🎯 What is Direct-Drop?
Direct-Drop is a lightweight, cross-platform file transfer tool I am engineering from absolute scratch. The goal is to securely and instantly beam files between a Windows computer and a mobile device over a local Wi-Fi network without using the internet, Bluetooth, or any bloated third-party web frameworks (like Flask or Django). 

## 🧠 Networking & Security Concepts Explored
Instead of taking the easy route and importing pre-built libraries, I am building the core engine natively to deeply understand how computers talk to each other. Throughout this build, I am implementing:

* **Socket Programming:** Establishing raw TCP connections between devices.
* **Dynamic IP Routing:** Bypassing hardcoded IPs by scanning the local network configuration via UDP.
* **Custom HTTP Packet Parsing:** Manually slicing and extracting binary file payloads from raw `multipart/form-data` streams.
* **LAN Security & Least Privilege:** Keeping the application safely sandboxed in user-space without forcing administrative firewall overrides.
* **Asynchronous Data Transfer:** Managing chunked file reading to prevent RAM overflow when transferring massive files.

## 🗓️ Development Phases

## 🛠️ Phase 1: The Core Engine & Dynamic IP Routing
**Objective:** Establish a purely native TCP web server and dynamically map the host's Local Area Network (LAN) architecture.

### ⚙️ What We Built
In this initial phase, I engineered the foundation of the Direct-Drop server. Instead of hardcoding IP addresses or relying on `localhost` (which mobile devices cannot reach), I built a dynamic routing function that automatically hunts down the host machine's true Wi-Fi IP address. I also implemented the Command Line Interface (CLI) to accept file paths and dynamically generate an ASCII QR code directly inside the terminal.

### 🔍 How It Works (Under the Hood)
1. **The CLI Trigger:** The user runs `python directdrop.py <filepath>`. The `argparse` library validates the file path.
2. **The UDP Ping Trick:** The script creates a dummy UDP socket and attempts to connect to a public DNS server (`8.8.8.8`). Because it's UDP (connectionless), it doesn't actually send a packet over the internet; it simply forces the Windows OS to consult its internal routing table and reveal the active Private IP address being used for Wi-Fi. 
3. **The Server Bootup:** The script binds Python's native `http.server` to that specific Private IP on Port 8080. 
4. **The Handshake:** A QR code is mathematically generated using the `qrcode` library, encoding the `http://<IP>:8080` string and printing it to the terminal for the phone to scan.

### 🧠 Core Networking & Security Concepts Applied
* **Socket Programming (UDP vs. TCP):** Exploited the connectionless nature of UDP to securely query the OS routing table without opening unnecessary outbound TCP streams.
* **LAN Architecture & NAT:** Deepened understanding of Network Address Translation (NAT) by ensuring the server binds strictly to the internal Private IP (e.g., `192.168.x.x`) rather than the loopback address (`127.0.0.1`), allowing external LAN devices to route to it.
* **Zero-Dependency System Design:** Bypassed massive frameworks like Flask. By using native `socketserver` and `http.server`, the application attack surface is drastically minimized, and the footprint stays incredibly lightweight.
* **CLI UX Design:** Built a robust command-line argument parser that prevents the server from booting if invalid or malicious file paths are provided.




> * **Phase 2:** *Coming soon...*
> * **Phase 3:** *Coming soon...*
>* **Phase 4:** *Coming soon...*
>* **Phase 5:** *Coming soon...*
>* **Phase 6:** *Coming soon...*

---
*Engineered by **Anisur Rahman***
