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
**Objective:** Establish a purely native TCP web server, dynamically map the host's Local Area Network (LAN) architecture, and ensure stable port management.

### ⚙️ What We Built
In this initial phase, I engineered the foundation of the Direct-Drop server. Instead of hardcoding IP addresses or relying on `localhost` (which mobile devices cannot reach), I built a **dynamic routing** function that automatically hunts down the host machine's true Wi-Fi IP address. I also implemented a Command Line Interface (CLI) to accept file paths, dynamically generate an ASCII QR code, and secured the server's lifecycle to prevent port hanging.

### 🔍 How It Works (Under the Hood)
* **The CLI Trigger:** We used the `argparse` library **so that** the user can pass specific file paths directly via the terminal, allowing the script to instantly validate that the files actually exist before booting the server.
  
* **The UDP Ping Trick:** We created a dummy UDP socket and attempted a connection to a public DNS server (`8.8.8.8`) **so that** the Windows OS is forced to consult its internal routing table. This **safely** reveals the **active Private IP** address being used for Wi-Fi without actually sending a payload over the internet.
* **The Server Bootup:** We bound Python's native `http.server` to that specific Private IP on Port 8080 **so that** external devices (like a phone) on the exact same Wi-Fi network can route traffic directly to the laptop.
* **The QR Handshake:** We mathematically generated an ASCII QR code using the `qrcode` library and printed it to the terminal **so that** the user can instantly pair their phone to the server URL (`http://<IP>:8080/`) without manually typing IP addresses.
* **The Port Reuse Fix:** We explicitly set `socketserver.TCPServer.allow_reuse_address = True` **so that** if the user stops the script and instantly restarts it, Windows immediately **frees up Port 8080 instead of crashing the application** with an "Address already in use" network error.
* **The Graceful Shutdown:** We wrapped the server execution in a `try/except KeyboardInterrupt` block **so that** when the user presses `CTRL+C`, the server **securely** and cleanly closes the port, preventing silent "ghost" ports from continuously listening in the background.

### 🧠 Core Networking & Security Concepts Applied
* **Socket Programming (UDP vs. TCP):** Exploited the connectionless nature of UDP to securely query the OS routing table without opening unnecessary outbound TCP streams.
  
* **LAN Architecture & NAT:** Deepened understanding of Network Address Translation **(NAT)** by ensuring the server binds strictly to the internal Private IP (e.g., `192.168.x.x`) rather than the loopback address (`127.0.0.1`), allowing local peer-to-peer routing.
* **Zero-Dependency System Design:** Bypassed massive frameworks like Flask. By using native `socketserver` and `http.server`, the application attack surface is drastically minimized, and the footprint stays incredibly lightweight.
* **Resource Management:** Ensured strict operating system compliance by writing explicit port-release commands and handling manual keyboard interrupts safely.


## 🛠️ Phase 2: The HTML Interface & "Security Jail" (Secure GET Requests)
**Objective:** Override the default HTTP handler to serve a custom UI and rigorously protect the host machine against Directory Traversal attacks.

### ⚙️ What We Built
In Phase 2, I engineered the custom web interface and the download engine. Python’s default `http.server` is highly dangerous for file sharing because it automatically exposes your entire working directory to anyone on the network. To fix this, I subclassed the request handler, built a strict **"Security Jail"** to sandbox the application, and injected a native HTML/CSS UI directly into the server's response.

### 🔍 How It Works (Under the Hood)
* **The Custom GET Handler:** We subclassed `SimpleHTTPRequestHandler` and overrode the `do_GET` method **so that** we could intercept every single incoming web request from the phone and apply our **own strict security routing** instead of letting Python auto-serve the entire `C:\` drive.
  
* **The Single-File UI Architecture:** We injected raw HTML and CSS directly as a UTF-8 encoded string **so that** the server serves a responsive, dark-mode user interface to the phone without relying on external static files, keeping the application completely zero-dependency.
* **The URL Parser:** We utilized `urllib.parse` **so that** special characters, symbols, and spaces in filenames sent by the phone's browser are correctly decoded before the server attempts to locate them on the laptop's hard drive.
* **The Security Jail (Whitelist Sandbox):** We implemented a strict `TARGET_FILES` dictionary mapping **so that** when a device requests a file, it is validated against an approved whitelist. If a hacker attempts a Path Traversal attack (e.g., requesting `../../Windows/System32/config/SAM`), the server instantly blocks it and throws a `403 Forbidden` or `404 Not Found` error.
* **The Download Headers:** We manually injected explicit `Content-Disposition: attachment` and `Content-Length` HTTP headers into the response **so that** the mobile browser natively understands it is receiving a file download (triggering the OS progress bar) rather than attempting to play or display the raw file in a new browser tab.
* **The Chunked Byte Stream:** We utilized a while loop with `f.read(1024 * 1024)` to stream the file in continuous 1MB blocks **so that** transferring a massive 4GB 4K video doesn't instantly overflow and crash the laptop's RAM.

### 🧠 Core Networking & Cybersecurity Concepts Applied

* **Directory Traversal Mitigation:** **Replaced** dangerous default file-serving behavior with strict server-side validation and dictionary whitelisting to sandbox the application.
  
* **HTTP Protocol Fundamentals:** Deep dive into how browsers interpret TCP headers (`Content-Type`, `Content-Disposition`) to trigger native operating system download behaviors.
* **Memory Management (I/O Streams):** Engineered efficient chunked byte streaming to handle massive binary payloads without memory exhaustion.
* **Secure UI Delivery:** **Bypassed** the need for external web directories by dynamically generating front-end code at runtime.

---

> * **Phase 3:** *Coming soon...*
>* **Phase 4:** *Coming soon...*
>* **Phase 5:** *Coming soon...*
>* **Phase 6:** *Coming soon...*

---
*Engineered by **Anisur Rahman***
