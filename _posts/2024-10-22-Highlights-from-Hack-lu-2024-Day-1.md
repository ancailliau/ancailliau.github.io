---
title: Highlights from Hack.lu 2024 (Day 1)
tags: conference
---

The ongoing edition of Hack.lu, the well-known cybersecurity conference,
provided today a wealth of knowledge, with both morning and afternoon sessions
delivering insights into emerging threats and advanced defensive strategies.
Across the day's talks, we explored a range of topics—from the evolving nature
of botnets and Linux rootkits to unconventional malware persistence techniques
and the integration of new tools in threat intelligence workflows. These
sessions highlighted not only the sophistication of modern cyber threats but
also the innovative methods and tools being developed to counter them.
Additionally, a hands-on workshop on Return Oriented Programming (ROP) on ARM64
allowed me to gain practical skills in exploit development, making the day
highly valuable. Detailed feedback from each session will be published, in
accordance with the sharing rules.

## Morning Session Summary

The morning sessions at the conference provided an in-depth look at the
evolving cybersecurity landscape, covering a range of topics from the
resilience of modern botnets to advancements in automation and threat
intelligence tools. Each talk presented unique insights into how the threat
environment is changing and the innovations being developed to counter these
risks, offering valuable lessons for CTI professionals.

### Insights from Modern Botnets

Miguel Hernández’s presentation delved into the inner workings of modern
botnets, illustrating how these threats remain highly adaptable. Botnets like
Mirai and Shellbot continue to evolve, with numerous variants easily accessible
and used for DDoS attacks and cryptomining. Hernández highlighted how botnets
operate as a service, rebranding when discovered, and often target critical
sectors such as government and finance. The research emphasized the growing
sophistication and accessibility of these threats.

### NeuroCTI: Benchmarking, Successes, Failures, and Lessons Learned

Aaron Kaplan shared his work on NeuroCTI, a custom large language model
(LLM) fine-tuned for CTI purposes. Kaplan discussed the challenges of training
local models for CTI data extraction, the difficulty of building a reliable
dataset, and the privacy concerns around cloud-based solutions. The model shows
promise in tasks such as summarization, named entity recognition (NER), and
mapping TTPs to MITRE ATT&CK, but off-the-shelf LLMs still present tough
competition. The session underscored the need for fine-tuning and local
deployment to protect sensitive data.

### Tales of the Future Past

Saâd Kadhi from CERT-EU provided a reflective talk on the lessons learned from
past cyber incidents and how they shape the future of cybersecurity. He
discussed the ongoing importance of traditional attack vectors, such as
cyberespionage and information operations, while highlighting the potential
threats posed by emerging technologies like AI. Kadhi cautioned that while AI
offers automation potential, it lacks the human intuition necessary for
comprehensive threat detection, urging caution in over-relying on machine
intelligence for security tasks.

### Integrating New Tools in Your Workflows Within Minutes in MISP

Sami Mokaddem demonstrated the ease of integrating new tools into existing
cybersecurity workflows using MISP. The session showcased how the MISP
automation framework, including *pymisp*, *misp-modules*, and *workflows*,
allows for the quick deployment of external tools to enrich threat intelligence
data. Mokaddem highlighted the flexibility of MISP’s workflow editor, which can
automate enrichment of `.onion` addresses, while restricting the sharing of
undesirable content.

### Lessons Learned from (almost) 8 Years of Sigma Development

Thomas Patzke offered valuable insights from his experience maintaining the
open-source Sigma project, a language for creating detection rules across
different security platforms. Patzke discussed the challenges of balancing
simplicity and scalability, highlighting the benefits of a complete toolchain
rewrite in 2020. The separation of code and rules, along with the adoption of
better development practices like test-driven development, improved the
project’s maintainability and adoption, but also introduced new complexities
around coordination and dependency management.

## Afternoon Session Summary

This afternoon’s sessions provided in-depth insights into the evolving threat
landscape and defensive strategies across different platforms, from Windows
malware persistence to Linux rootkits, industrial protocol vulnerabilities, and
an innovative approach to honeypots using large language models (LLMs). Each
talk presented actionable strategies for detection and defense against these
sophisticated threats, alongside a hands-on workshop that introduced
participants to Return Oriented Programming (ROP) on ARM64, further equipping
attendees with practical skills in exploit development.

### Workshop: ROP on ARM64 - A Hands-On Tutorial

In the workshop, Saumil Shah provided participants with a practical
introduction to Return Oriented Programming (ROP) on ARM64. The tutorial
covered essential ARM64 assembly concepts, common ROP gadgets, and practical
exploit development techniques, including building ROP chains. Shah’s workshop
offered a hands-on experience with an ARM64 emulator and equipped participants
with the skills to navigate the unique challenges posed by the ARM64
architecture, making it a valuable session for anyone interested in exploit
development on this platform.

### Malware and Hunting for Persistence

Cocomelonc's presentation focused on unconventional malware persistence
techniques in Windows environments. The session highlighted how adversaries use
registry modifications and DLL hijacking to maintain persistence on compromised
systems. Through case studies, including vulnerabilities in Internet Explorer,
Microsoft Teams, and cryptographic APIs, Cocomelonc demonstrated how attackers
exploit legitimate Windows resources to evade detection and maintain long-term
access, emphasizing the importance of robust detection strategies to counter
these attacks.

### Breaking Industrial Protocol Translation Devices

Claire Vacherot shared insights from her research into vulnerabilities in
Anybus X-gateways, critical devices used for translating industrial protocols.
Vacherot revealed how these often-overlooked gateways are prime targets for
attackers, given their role in maintaining communication between OT systems.
She detailed three critical vulnerabilities in the device, including
unauthenticated access, manipulation of proprietary protocols, and
denial-of-service attacks, showing how attackers can exploit these weaknesses
to disrupt industrial processes.

### In-Depth Study of Linux Rootkits: Evolution, Detection, and Defense

Stephan Berger’s talk explored the evolution of Linux rootkits, from early
userland techniques to modern kernel-level rootkits. Berger outlined advanced
rootkit techniques, such as hooking kernel functions and exploiting eBPF, and
provided a thorough review of detection strategies using integrity checking,
behavioral analysis, and memory forensics. By analyzing rootkits like Bedevil
and Diamorphine, Berger underscored the importance of using modern detection
tools to uncover sophisticated threats in Linux environments.

### Decoding Galah: an LLM-Powered Web Honeypot

Adel Karimi introduced Galah, an innovative honeypot powered by large language
models (LLMs) that dynamically generates realistic HTTP responses to simulate
multiple web applications. Karimi discussed the challenges of building Galah,
including issues with response formatting and the difficulty of maintaining
deception. Despite these challenges, Galah successfully engaged attackers,
particularly in cases involving open proxy emulation. This approach showcased
the potential of using LLMs to improve honeypot realism and increase attacker
interaction.
