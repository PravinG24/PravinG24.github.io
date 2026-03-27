---
title: "From Homelab to 5G: Building a Pocket-Sized AI Command & Control Framework"
date: 2026-03-27 12:00:00 +0800
categories: [Red Teaming, Infrastructure]
tags: [python, tailscale, ai, c2]
---

If there is one thing that unites every penetration tester and red teamer, it’s a mutual disdain for the manual reconnaissance grind.

We’ve all been there: staring at a terminal, waiting for an Nmap scan to finish, then manually copying and pasting IP addresses into a bunch of other tools just to continue waiting even more in order to figure out what we are looking at. It is slow, it breaks your flow, and worse—it usually requires you to be hunched over a laptop.

I wanted to see if I could completely decouple the recon phase from my physical hardware. My goal? To build a stealthy, fully automated OSINT and Command-and-Control (C2) framework which implements AI analysis that I could trigger from my phone, while grabbing a coffee, over a 5G connection.

Here is how I turned my Linux homelab into a pocket-sized, AI-augmented threat hunter.

## The Hardware: Scrapyard Infrastructure
I didn't want to pay monthly cloud fees, so my "homelab" is a repurposed gaming laptop (featuring an old GTX 980m) with a completely shattered display. I stripped the OS, installed bare-metal Linux, and shoved it in a closet. Because it has no physical monitor, the entire C2 environment was engineered headless via SSH from my daily driver laptop. It is scrappy, silent, and proves you don't need expensive enterprise rack gear to build advanced offensive infrastructure.

## The Blueprint: Decoupling the Operator from the Hardware
To make this work, I needed four things: a stealthy network tunnel, a backend execution engine, an automated analyst (AI), and a UI that didn't look terrible on a mobile screen.

Here is the tech stack I engineered to make it happen:

* **The Network (Tailscale):** Opening inbound ports on my home router to host a C2 server is terrible OpSec. Instead, I deployed Tailscale (WireGuard) to create an end-to-end encrypted, zero-config mesh network. The C2 server strictly binds to the `100.x.x.x` IP address, making the weapon entirely invisible to the public internet and only accessible to my authenticated mobile device.

* **The Intelligence Arsenal (Active & Passive Recon):** The Bash script is programmed to pull from a highly specific stack of tools:
  * **Active Footprinting:** Nmap for localized service discovery and OS fingerprinting.
  * **Passive OSINT:** Shodan for external attack surface mapping and historical banner grabbing.
  * **The Threat Intel Pipeline:** I integrated a multi-layered API stack including VirusTotal and AbuseIPDB for reputation scoring, ThreatFox to cross-reference IPs against known active C2/botnet infrastructure, and most importantly, GreyNoise to filter out benign "internet weather" and mass-scanners.

* **The Orchestrator (Python/Flask & Bash):** The brain of the operation is a lightweight Python Flask server running in my homelab. When it receives an authenticated strike command, it triggers a hardened Bash script that orchestrates a massive concurrent data pull—firing active Nmap footprinting while scraping passive telemetry from GreyNoise, AbuseIPDB, and ThreatFox.

> **Under the Hood:** Before wrapping the tool in a mobile web interface, the core orchestration engine was engineered and tested entirely via the command line. Here is the raw Bash pipeline successfully resolving targets and extracting API telemetry.
![Project CORVUS CLI Execution](/assets/cli-execution.png)

* **The "Force Multiplier" (AI Synthesis via NVIDIA NIM):** Raw API JSON data from six different sources is completely unreadable on the fly. Instead of relying on slow, consumer-grade web AI, I engineered a direct integration with the NVIDIA NIM API, specifically leveraging the high-speed Step Flash 3.5 model. By utilizing NIM's low-latency microservices and applying strict prompt engineering, the AI bypasses standard conversational guardrails to instantly ingest, sanitize, and synthesize the asynchronous telemetry. Within milliseconds, it outputs a highly readable, actionable "Target Dossier" directly to my terminal.

* **The Field Console (PWA):** I built a mobile-first Progressive Web App using HTML5/CSS3 and Vanilla JS. It lives on my phone’s home screen as a standalone, edge-to-edge terminal interface.

> **The Operator's View:** The custom PWA interface running over a 5G connection. This decouples the reconnaissance phase from a traditional laptop, allowing for discreet, on-the-go intelligence gathering.
![Project CORVUS Mobile UI](/assets/mobile-app-ui.jpeg)

---

## Live Fire Exercises: Operational Output

To validate the pipeline's fidelity and the AI's ability to distinguish between actual threats and standard network traffic, I ran Project CORVUS against four distinct target profiles. 

### Test 1: The Internal Gateway (Benign Local IP)
> **Target:** `192.168.x.1` (Local Router)
> **Objective:** Verify that the AI correctly identifies private RFC 1918 IP space and does not hallucinate external threat intelligence data for internal network nodes.
> **Result:** Pipeline successfully profiled the local hardware without triggering false-positive alerts from external APIs.

![Local Router Output](/assets/router-test-1.jpeg)
![Local Router Output 2](/assets/router-test-2.jpeg)

### Test 2: Active Command & Control Node (Malicious IP)
> **Target:** `[Redacted Cobalt Strike C2 IP]`
> **Objective:** Test the asynchronous threat intel pipeline against a live, known-malicious adversary node.
> **Result:** ThreatFox and AbuseIPDB successfully flagged the host. The NVIDIA NIM model synthesized the raw JSON into a highly readable, "SUSPICIOUS" threat dossier instantly.

![Malicious IP Output](/assets/malicious-ip-test-1.jpeg)
![Malicious IP Output 2](/assets/malicious-ip-test-2.jpeg)

### Test 3: The Benign Domain (Google)
> **Target:** `google.com`
> **Objective:** Verify rapid DNS resolution and ensure that massive, globally distributed infrastructure is accurately categorized as safe internet weather.
> **Result:** DNS resolved perfectly. GreyNoise and VirusTotal returned clean bills of health, and the AI formatted a low-priority, benign intelligence report.

![Google DNS Output](/assets/google-test-1.jpeg)
![Google DNS Output 2](/assets/google-test-2.jpeg)

### Test 4: The Wormhole (Sinkholed Ransomware Domain)
> **Target:** `iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com` (WannaCry Killswitch)
> **Objective:** Feed the pipeline one of the most infamous, high-alert domains in history to test the AI's extreme-threat formatting.
> **Result:** The DNS resolution succeeded, triggering alerts across the OSINT stack. The AI parsed the historical ransomware association and generated a pristine dossier.

![WannaCry Output](/assets/wannacry-test-1.jpeg)
![WannaCry Output 2](/assets/wannacry-test-2.jpeg)

---

## The Battle Scars: Things I Broke (And Fixed)
You don’t build custom infrastructure without running into a few locked doors. Here were the most fun technical hurdles to bypass during the build:

1. **Bypassing the Strict JSON Bouncer:** When I upgraded the frontend to use asynchronous JavaScript (`fetch()`), the Python backend completely rejected the payloads. Flask was strictly expecting a `Content-Type: application/json` header, but my JS was sending raw form data. I had to explicitly forge a strict JSON envelope in the headers and meticulously align the dictionary keys to ensure the pipeline didn't drop the data.
2. **The Asynchronous SSH Wall:** My Python listener runs on a master host, but it needs to execute the Bash scripts laterally across different VMs in my homelab. Initially, the C2 pipeline just hung and died. Why? The target VMs were silently prompting for SSH passwords in the background. I bypassed this by generating an Ed25519 cryptographic key pair (with no passphrase) and injecting the public key into the target's `authorized_keys`, forging a persistent trust link for passwordless, automated execution.
3. **The Invisible Bash Artifacts:** Piping raw Bash terminal output directly into a mobile web browser resulted in chaotic text alignment. Bash scripts often leak invisible trailing spaces or carriage returns (`\r`). Standard JavaScript `.trim()` functions couldn't catch them, so I engineered a Regular Expression (Regex) scalpel (`/^\s+/`) to violently strip any invisible formatting characters the moment the payload hit the frontend, ensuring a perfect, retro-terminal text alignment.
4. **The API Handshake Desync:** During the transition from a local web app to an asynchronous remote C2, the pipeline completely decoupled. The mobile UI was throwing `404` errors while the backend was executing blindly. The issue was a routing desync: the JavaScript `fetch()` API was attempting to POST to legacy endpoints, and the JSON payload dictionary keys (`scan_type` vs `type`) were mismatched. I had to surgically realign the Flask `@app.route` decorators and the JSON extraction logic to perfectly match the frontend's data structure, establishing a flawless digital handshake.
5. **Headless Debugging & "Ghost" Errors:** Because the 980m host has a shattered screen, I was debugging the entire pipeline via remote SSH. At one point, the terminal was throwing HTTP `404` errors, making it look like the C2 was failing. By analyzing the raw traffic logs, I realized the 404s were just benign "Ghost" errors—the mobile browser was automatically searching for a `favicon.ico` image file that didn't exist, while the actual `POST /fire` payload was returning a successful HTTP `200` code. It was a masterclass in learning how to separate background web noise from actual pipeline failures.

## Why This Matters for Red Teaming
Building this framework wasn't just a fun project; it completely changed how I view the adversary lifecycle.

In offensive security, efficiency is everything. By automating the data correlation phase and piping it through an AI summarizer, I reduced my initial target profiling time by over 90%. Furthermore, forcing myself to build the infrastructure—managing API authentication, secure remote execution, and network tunneling—gave me a much deeper understanding of how modern threat actors engineer their own C2 environments.

I am constantly looking for the next technical rabbit hole to fall down. If you are building offensive tooling, hunting on Hack The Box, or engineering new ways to automate red team operations, I’d love to connect.

*(You can check out a sanitized, API-stripped version of the code on my GitHub here: [https://github.com/PravinG24/Corvus])*