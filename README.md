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

## 📅 Development Phases
* **Phase 1:** *[Dropping Tomorrow!]*
* **Phase 2:** *Coming soon...*
* **Phase 3:** *Coming soon...*
* **Phase 4:** *Coming soon...*
* **Phase 5:** *Coming soon...*
* **Phase 6:** *Coming soon...*

---
*Engineered by **Anisur Rahman***
