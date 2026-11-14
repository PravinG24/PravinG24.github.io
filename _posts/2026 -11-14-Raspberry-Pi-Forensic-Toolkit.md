---
title: "Building a Raspberry-Pi Forensic Toolkit (And Trying Not to Melt It)"
date: 2026-11-14 12:00:00 +0800
categories: [Architecture, Forensics, Red Teaming, Network]
tags: [Raspberry-Pi, Tailscale, Kotlin, PowerShell]
---

# Building a Raspberry-Pi Forensic Toolkit (And Trying Not to Melt It)

Clicking through pre-configured virtual simulations in a digital forensics lab isn't exactly the thrilling, high-stakes hacker montage we all hoped for when choosing a cybersecurity major. Professional forensic hardware is notoriously expensive, and most students graduate with a gap in practical, hands-on evidence acquisition. For my Final Year Project, I decided to fix that by building a Raspberry Pi-Based Forensic Toolkit—an affordable, portable, and entirely ethical digital extraction platform.

The dream was ambitious but straightforward: create an Integrated Mobile Forensic Ecosystem (IMFE) where I could walk up to a target Windows machine, automatically extract password hashes, and crack them, all while coordinating the entire operation from an Android smartphone. I essentially wanted to build a forensic lab that fits in a backpack.

If you're picturing a sleek, glowing Hollywood-style briefcase, let me stop you there. Take a look at this:

![Development Setup](assets/setup.jpeg)

My actual development setup looked a bit more... chaotic. It was just me, a laptop, a portable monitor, a Raspberry Pi, and a terrifying tangle of wires and power adapters scattered across a desk and the floor. Fun fact about that portable monitor: I originally had zero intention of using one, but I was having massive troubles trying to SSH into the Pi for the very first time to configure it headlessly. In a fit of frustration, I went out and got the monitor just to see what was failing on the backend. The absolute most annoying part? The exact second the Pi's desktop showed up on the monitor, the SSH magically fixed itself and started working perfectly. Hardware is a troll.

## Hardware Hates Me

No engineering project is complete without your hardware violently rejecting your ideas. My first brilliant plan for the toolkit was to tether my Android phone directly to the Raspberry Pi via a phone cable for seamless communication. The result? Modern Android fast-charging aggressively tried to push power to the Pi, completely overwhelming its power management system and causing unexpected, random shutdowns with a hint of smoky smells being produced in my room.

To save my Pi from a crispy demise, I had to ditch the phone cable entirely and switch to a dedicated USB-C power supply just for the Pi. By ensuring there were fewer physical connections drawing from or pushing power to the Pi's board, I drastically reduced the electrical strain, guaranteeing I didn't accidentally burn out the circuitry. But this created a new problem: since I could no longer tether the phone directly for data, I had to pivot the entire architecture, shifting instead to secure wireless coordination using a Tailscale VPN mesh network.

You can actually see the glorious result of this pivot here:

![Tailscale VPN Connected](assets/homescreen.jpeg)

Right on the app's main dashboard, you'll notice that satisfying "Tailscale VPN: Connected" and "GPU Server: Available" status. That is the digital equivalent of my hardware breathing a sigh of relief while offering one-tap access to start investigations or run standalone hash cracking.

Then came the password cracking itself. I initially thought I could just let the Pi’s ARM processor handle the decryption natively. Fast forward through testing, and the Pi was practically weeping; complex hash cracking operations were taking several hours, sometimes days, to complete. Since students need timely results, I outsourced the heavy lifting, reconfiguring the Pi to act as a stealthy network coordinator that delegates the intense computational work over WebSocket connections to a remote, RTX 3070-equipped GPU server.

---

## What Exactly Did I Build?

To pull off this portable forensic wizardry, I had to architect a system that stitched together edge computing, mobile app development, and remote GPU processing into one seamless, rapid-application-development (RAD) lifecycle. Here is the breakdown of the digital heavy-lifting happening whenever I tap the screen:

*   **The Raspberry Pi Backend:** At the core of the toolkit is a Raspberry Pi running a custom Python Flask API server. When plugged into a target Windows machine via ethernet, the Pi automatically detects the system and mounts it. I wrote a credential-less extraction engine that utilizes tools like `samdump2` to quietly pull the SAM and SECURITY registry hives directly over SMB/WMI protocols—meaning it grabs the hashes without ever needing the target's actual password.
*   **The GPU Server:** Because the Pi's processor would take centuries to crack complex hashes on its own, it acts purely as a middleman. Once the hashes are extracted, the Pi shoots them through the encrypted Tailscale VPN tunnel directly to the remote RTX 3070 GPU server. I programmed this server to execute an ultra-efficient 4-phase attack: a quick dictionary check, rule-based mutations, context-aware AI password generation using a PassGPT model, and finally, a progressive brute force.
*   **The Android App:** I didn't want to be typing clunky Linux commands in the field, so I built a native Android app from scratch using Kotlin. This app acts as the ultimate remote control, allowing me to view real-time cracking progress, execute forensic imaging, and browse the target's file system safely in read-only mode. 

You can see this slick UI in action here:

![Extracted Hashes](assets/hashes.jpeg)

![File Browser](assets/files.jpeg)

Here, the extracted NTLM hashes and target file structures (like AppData and Documents) are laid out, ready for analysis.

*   **The Testing Lab:** To ensure everything worked without accidentally committing cybercrimes on production networks, I built a heavily isolated Windows 11 Virtual Machine. I made an automated PowerShell script that instantly strips down the VM's security, configuring the exact SMB, WMI, and firewall rules needed to turn it into the perfect, repeatable forensic punching bag.

Building this was a massive journey in adapting on the fly, learning to respect hardware limitations, and realizing that sometimes, delegating the math to an RTX 3070 is the only way to stay sane. The toolkit successfully bridged the gap between forensic theory and reality, proving that you don't need an unlimited budget to do advanced data recovery—just a Raspberry Pi, an Android phone, and a high tolerance for troubleshooting.

*(You can check out the sanitized, API-stripped version of the code on my GitHub here: [https://github.com/PravinG24/Raspberry-Pi-Based-Forensic-Toolkit])*