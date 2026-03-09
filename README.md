# ⚡ Direct-Drop
**A native, zero-dependency LAN file-sharing architecture.**

> **Status:** 🟢 Completed & Deployed. 

## 🎯 What is Direct-Drop?
Direct-Drop is a lightweight, cross-platform file transfer utility engineered from absolute scratch. The objective was to securely and instantly beam files between a Windows host and a mobile device over a local Wi-Fi network without relying on the internet, Bluetooth, or heavy third-party web frameworks (like Flask or Django). 

With a strict focus on backend infrastructure and memory management, the final build features a concurrent multi-threaded server, on-the-fly ZIP compression, and is compiled into a standalone executable (`.exe`) natively hooked into the Windows "Send To" context menu for zero-friction execution.

## 🧠 Networking & Security Concepts Explored
Instead of importing pre-built abstraction libraries, the core engine was built natively to deeply understand raw network communication and system architecture. Throughout this build, the following concepts were implemented:

* **Socket Programming:** Establishing and maintaining raw TCP connections between local devices.
* **Concurrency & Multi-Threading:** Utilizing `socketserver.ThreadingMixIn` to handle simultaneous multi-client connections and prevent network bottlenecks.
* **Dynamic IP Routing:** Bypassing hardcoded IPs by scanning the local network configuration via UDP sockets.
* **Custom HTTP Packet Parsing:** Manually slicing and extracting binary file payloads from raw `multipart/form-data` streams while stripping protocol artifacts.
* **AppSec & Sandboxing:** Implementing strict dictionary whitelisting to mitigate Directory/Path Traversal attacks and keeping the application securely sandboxed in user-space.
* **Memory Management:** Managing chunked byte-reading and ephemeral storage for ZIP compilation to prevent RAM overflow during massive file transfers.
* **OS-Level Integration:** Mapping native Windows GUI actions directly to Python CLI arguments via a compiled standalone binary.

## 🗓️ Development Phases

## 🛠️ Phase 1: The Core Engine & Dynamic IP Routing
**Objective:** Establish a purely native TCP web server, dynamically map the host's Local Area Network (LAN) architecture, and ensure stable port management.

### ⚙️ What I Built
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

### ⚙️ What I Built
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

## 🛠️ Phase 3: The Upload Engine & Raw Multipart Parsing (Secure POST Requests)
**Objective:** Implement secure two-way file transfer by engineering a custom HTTP POST handler to parse raw binary payloads directly from the TCP stream and prevent file corruption.

### ⚙️ What I Built
In Phase 3, I engineered the reverse pipeline: beaming files from the mobile device back to the host laptop. Native `http.server` does not support file uploads out-of-the-box. Instead of importing external libraries, I built a custom packet parser to intercept the HTTP POST request, strip the browser's metadata, identify the **cryptographic** boundary, and securely reconstruct the raw binary file onto the laptop's hard drive **without data corruption**.

### 🔍 How It Works (Under the Hood)
* **The Custom POST Handler:** I overrode the `do_POST` method **so that** the server can actively intercept and process incoming HTML form submissions from the mobile device.
  
* **Strict Endpoint Routing:** I restricted the POST execution strictly to the `/upload` path and enforced a `403 Forbidden` response for all other routes **so that** malicious actors cannot blindly throw POST payloads at the root server or download endpoints.
* **Dynamic Boundary Extraction:** I parsed the `Content-Type` header to extract the unique `boundary=` string **so that** the server knows exactly where the file's binary data begins and ends within the massive HTTP network packet.
* **Raw Byte Reading:** We utilized `self.rfile.read(int(content_length))` **so that** the server pulls the exact number of bytes specified by the browser, preventing the socket from hanging indefinitely while waiting for non-existent data.
* **Header & Payload Separation:** I manually split the raw byte stream at the `\r\n\r\n` (Carriage Return Line Feed) marker **so that** we could perfectly isolate the file's actual binary payload from the browser's metadata headers.
* **Safe Header Decoding:** I decoded the multipart headers using `utf-8` with the `errors='ignore'` flag **so that** any maliciously crafted or corrupted bytes in the header block do not trigger a fatal `UnicodeDecodeError` and crash the application.
* **Regex Filename Extraction:** I implemented the `re` (Regular Expression) module to search the raw header block for `filename="..."` **so that** the laptop securely saves the incoming payload with its original name and extension.
* **Input Sanitization:** I wrapped the extracted filename in `os.path.basename()` **so that** a compromised phone or malicious actor cannot send a file named `../../Windows/System32/malware.exe` to execute a reverse Path Traversal attack against the host PC.
* **Trailing Byte Trimming:** I explicitly stripped the trailing `\r\n` bytes from the end of the binary payload using array slicing (`file_bytes[:-2]`) **so that** uploaded files (especially sensitive formats like images or compiled binaries) **do not get corrupted** by leftover HTTP protocol artifacts.
* **Dynamic Directory Routing:** We utilized `os.path.expanduser('~')` to dynamically resolve the host operating system's user profile **so that** incoming files are safely and predictably routed directly to the native `Downloads` folder, rather than dumping them haphazardly into the script's execution directory.
* **The Acknowledgment Handshake:** I explicitly sent an HTTP 200 OK response and a plain-text confirmation back through the socket **so that** the mobile device's browser instantly knows the file was **safely received** and severs the connection cleanly without hanging.
* **Fault Tolerance:** I wrapped the entire parsing engine in a strict `try/except` block returning a `400 Bad Request` **so that** if a network drop or malformed packet corrupts the upload, the server cleanly rejects it and stays online for the next request.

### 🧠 Core Networking & Cybersecurity Concepts Applied
* **HTTP Protocol Deep-Dive:** Gained hands-on experience with how modern browsers encode and transmit files using `multipart/form-data` and MIME boundaries.
  
* **Binary Data Stream Parsing & Integrity:** Engineered manual byte-level manipulation to separate application-layer headers from raw payload data while preserving file integrity.
* **Reverse Path Traversal Mitigation:** Implemented strict server-side sanitization of user-controlled input (the uploaded filename) to protect the host operating system from arbitrary file writes.
* **Server Resilience & Fault Tolerance:** Hardened the server against malformed packets and invalid routing attempts to ensure continuous uptime during file transfers.

---

## 🛠️ Phase 4: Asynchronous Telemetry & On-the-Fly Payload Compression (AJAX & ZIP)
**Objective:** Implement real-time asynchronous upload tracking and engineer a dynamic, memory-safe ZIP compiler for multi-file network transfers.

### ⚙️ What I Built
In Phase 4, I solved two massive user-experience and backend infrastructure problems. First, I engineered an asynchronous JavaScript payload tracker to monitor byte-transfer rates in real-time, preventing the mobile browser from appearing "frozen" during massive uploads. Second, I built an on-the-fly ZIP compiler that intercepts multi-file download requests, compresses the assets into a temporary OS directory, streams the package over TCP, and instantly shreds the leftover cache to protect the host machine's hard drive from silent storage bloat.

### 🔍 How It Works (Under the Hood)
* **Asynchronous `XMLHttpRequest`:** We implemented raw AJAX via JavaScript's `XMLHttpRequest` and `FormData` **so that** the mobile browser can beam massive binary payloads to the laptop in the background without freezing the screen or forcing a clunky page reload.
  
* **Real-Time Byte Telemetry:** We attached an event listener to the `xhr.upload.progress` state **so that** the UI dynamically calculates `(evt.loaded / evt.total) * 100` and renders a live visual progress bar, providing critical network feedback during heavy data transfers.
* **State-Based UI Color Coding:** We programmed the JavaScript to dynamically shift the progress bar's hex colors (Blue for uploading, Green for 200 OK success, Red for failure) **so that** the user instantly understands the HTTP response state without needing to read terminal logs.
* **On-the-Fly ZIP Compilation:** We engineered a `/download_zip` endpoint using Python's native `zipfile` module **so that** when a user requests multiple files, the server dynamically bundles them into a single archive before streaming, drastically reducing the number of open TCP connections required.
* **Deflated Network Compression:** We specifically configured the compiler with `zipfile.ZIP_DEFLATED` **so that** the data footprint is actively reduced before transmission, optimizing LAN bandwidth utilization.
* **Host Privacy Preservation (Path Masking):** We mapped `arcname=fname` inside the `zipf.write()` command **so that** the archive only stores the raw filename, permanently preventing the host machine's absolute directory structure (e.g., `C:\Users\Name\Desktop\...`) from being exposed inside the ZIP file.
* **Secure Temp Directory Routing:** We utilized `tempfile.gettempdir()` **so that** the massive temporary ZIP archive is safely constructed in the operating system's designated ephemeral storage, rather than cluttering the user's active working directory or the script's folder.
* **Automated Cache Shredding:** We wrapped the ZIP stream in a strict `try/finally` block with `os.remove(temp_zip_path)` **so that** the archive is instantly and permanently deleted from the host machine the millisecond the network transfer completes or fails, preventing silent storage exhaustion (a common cause of local Denial of Service).
* **Dynamic Payload Detection:** We embedded conditional logic (`if len(TARGET_FILES) > 1`) directly into the HTML generator **so that** the server intelligently analyzes the payload queue and only renders the "Download All (ZIP)" endpoint if it is mathematically necessary, keeping the UI clean.

### 🧠 Core Networking & Cybersecurity Concepts Applied
* **Asynchronous Network Telemetry:** Mastered background data streaming and real-time state calculation without interrupting the main UI thread.
  
* **Ephemeral Storage Management:** Engineered strict backend infrastructure to handle massive temporary file generation and automated deletion, protecting the host operating system from storage exhaustion.
* **Bandwidth Optimization:** Implemented server-side compression algorithms to minimize payload weight across local network channels.
* **Directory Structure Obfuscation:** Protected the host machine's internal file paths from being leaked to network peers during multi-file bundling.
* **Dynamic System Integration:** Bridged frontend user actions directly to heavy backend file-system operations securely and efficiently.

---

## 🛠️ Phase 5: Concurrency, The "Receive-Only" Engine & Native OS Integration
**Objective:** Implement non-blocking multi-threaded network transfers, a robust standby mode for zero-file booting, and integrate the system directly into the Windows File Explorer via a standalone executable.

### ⚙️ What I Built
In this final phase, the server was upgraded to production-grade status. First, the single-thread network bottleneck was eliminated by engineering a concurrent, multi-threaded server class. Next, the application was engineered to function perfectly as a dedicated "Receive-Only" bridge when no files are explicitly queued. Finally, the UI branding was polished, the development environment was sanitized, the raw Python architecture was compiled into a standalone Windows executable, and the binary was hooked directly into the native OS right-click context menu for instantaneous, zero-navigation execution.

### 🔍 How It Works (Under the Hood)
* **The Multi-Threading Engine:** Injected Python's `socketserver.ThreadingMixIn` into a custom HTTP server class **so that** the application instantly spins up a new, independent background thread for every connected device, allowing multiple users to download massive payloads simultaneously without blocking or freezing the local network.
  
* **Zero-Argument CLI Patch:** Modified the `argparse` configuration from `nargs='+'` (one or more) to `nargs='*'` (zero or more) **so that** the application can be double-clicked and launched without any initial file inputs, entirely bypassing the fatal command-line parsing crash.
* **Kill-Switch Removal:** Bypassed the `sys.exit(1)` termination block that previously triggered when `TARGET_FILES` was empty **so that** the server continues its boot sequence and safely binds to the network port purely as a listening receiver.
* **Strict Environment Sanitization:** Explicitly uninstalled experimental cryptographic libraries via the terminal before the final compilation **so that** the production environment was perfectly restored, strictly adhering to the core "zero-dependency / no-pip" architectural philosophy.
* **Clean Build Architecture:** Explicitly deleted the old `build`, `dist`, and `.spec` directories before the final compilation **so that** PyInstaller was forced to package the fresh multi-threaded logic rather than aggressively caching the outdated single-threaded architecture.
* **Standalone Binary Compilation:** Utilized the `pyinstaller --onefile directdrop.py` command **so that** the entire Python runtime, local networking libraries, and the custom server script are permanently compressed down into a single, portable executable (`directdrop.exe`), completely removing the need for the end-user to have Python installed.
* **Visible Console Architecture:** Intentionally omitted the `--noconsole` flag during compilation **so that** the Windows command prompt remains actively visible upon launch, ensuring the user can interact with the dynamically generated ASCII QR code and monitor real-time network transfer logs.
* **OS Context Menu Integration (Send To):** Injected an application shortcut directly into the hidden Windows `shell:sendto` directory **so that** users can natively right-click any file within the Windows File Explorer to launch the server. 
* **Native Argument Passing:** By utilizing the Send To menu, Windows natively intercepts the right-clicked file and passes its absolute path directly into the script's `argparse` `sys.argv` array **so that** the file is instantly and automatically pre-loaded into the payload queue without opening a terminal. 

### 🧠 Core Networking & Cybersecurity Concepts Applied
* **Asynchronous Concurrency:** Eliminated TCP socket bottlenecks by implementing thread-based request handling for simultaneous multi-client connections.
  
* **State-Driven Application Logic:** Managed dynamic backend routing and front-end rendering based entirely on payload queue states.
* **Secure Environment Management:** Practiced strict dependency hygiene by removing unused cryptographic libraries to minimize the final binary's bloat and attack surface.
* **Cross-Platform Portability (Deployment):** Engineered a friction-free deployment model via standalone binary compilation, drastically reducing the application's attack surface by hiding the source code.
* **OS-Level System Hooks:** Integrated custom application execution directly into the native Windows Explorer shell environment without requiring administrative registry edits, successfully mapping GUI actions to CLI arguments.

---

### 🛡️ Architectural Note: The HTTP vs. HTTPS (TLS) Trade-off
***Why Direct-Drop runs on raw HTTP instead of encrypted HTTPS:***

As a security-conscious build, the final roadmap included wrapping the TCP socket in a dynamically generated self-signed RSA-2048 certificate to achieve End-to-End Encryption over the local network. However, a deliberate engineering decision was made to revert to raw HTTP due to the strict Public Key Infrastructure (PKI) rules enforced by modern mobile browsers.

* **The Local IP Limitation:** Public Certificate Authorities (like Let's Encrypt) do not issue trusted SSL/TLS certificates for local Wi-Fi IP addresses (e.g., `192.168.x.x`).
  
* **The UX vs. Security Dilemma:** Utilizing a self-signed certificate dynamically generated by Python triggers severe, unavoidable "Your connection is not private" security blocks on iOS Safari and Android Chrome. 
* **The Threat Model Assessment:** Direct-Drop is explicitly designed as a zero-friction, ephemeral tool for trusted private home networks. Forcing an end-user to manually install custom Root CA certificates into their mobile operating system to bypass the browser warnings completely violates the application's "zero-setup, plug-and-play" philosophy.
* **The Negative Impact (The Catch):** Because the internal traffic is raw HTTP, there is zero *application-layer* encryption. If you run Direct-Drop on a public, open Wi-Fi network (like a local cafe), any other user connected to that exact same network could use packet-sniffing tools (like Wireshark) to intercept and reconstruct your files in plain text.
* **The Final Verdict (Is it still secure?):** Yes. Direct-Drop is highly secure **if used on a trusted, password-protected network** (like your home Wi-Fi & own network). Modern routers enforce WPA2/WPA3 encryption, meaning the data is already cryptographically secured in the air against outside attackers.
 

**Conclusion:** Therefore, raw HTTP was chosen as a calculated risk. Direct-Drop is engineered to be a lightning-fast, seamless bridge for private environments, but it relies on the underlying security of the local network and should not be used to transmit payloads on untrusted public Wi-Fi.

---

*Engineered by **Anisur Rahman***

---
