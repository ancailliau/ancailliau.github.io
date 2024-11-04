---
title: Highlights from Hack.lu 2024 (Day 4)
tags: conference
---

The fourth day of Hack.lu 2024 brought valuable insights into the evolving
tactics of cybercriminals, proactive network defense, and innovative threat
detection methods. Covering a range of topics from router compromises to
firmware analysis, the sessions offered practical knowledge for security
professionals focused on anticipating adversary actions, securing network
infrastructure, and improving threat detection accuracy.

**The Ouroboros of Cybercrime - Witnessing Threat Actors go from Pwn to Pwn'd**:
This talk highlighted the cyclical nature of cybercrime, where threat actors who
compromise systems are themselves targeted and compromised by other groups. It
illustrated how attacks evolve and often come full circle, underscoring the
volatility and risks within the cybercrime ecosystem.

**The XE Files - Trust No Router**: James Atack provided an in-depth look into
Cisco Talos’s recent findings on widespread router compromises, revealing that
tens of thousands of routers remain vulnerable to backdoors. He emphasized the
persistence of the adversary and the concerning lack of response from some
organizations, calling for enhanced vigilance around network devices.

**New Features in the Zeek Network Monitor**: Christian Kreibich discussed the
latest advancements in Zeek, including scriptability with JavaScript,
performance improvements, and enhanced telemetry integration with Prometheus.
These updates open new possibilities for network analysis, making Zeek an even
more robust tool for proactive network monitoring.

**Sigma Unleashed - A Realistic Implementation**: Mathieu Le Cleach from CERT-EU
shared a real-world implementation of Sigma rules for better detection across
multiple SIEMs and EDRs. His tool, Droid, supports versioning, modular rule
validation, and cross-platform deployment, helping organizations maintain
adaptable and effective detection capabilities.

**Predictive Analytics for Adversary Techniques in the MITRE ATT&CK Framework
using Rule Mining**: Tristan Madani presented a predictive model using
association rule mining to anticipate adversary techniques based on historical
data within the MITRE ATT&CK framework. By applying Apriori and FP-Growth
algorithms, he showcased how this approach can enhance threat hunting by
identifying probable attack patterns and improving proactive defense.

**Exploring Firmwares - Tools and Techniques for New Cartographers**: Eloïse
Brocas introduced attendees to firmware analysis, guiding them through file
system extraction and cartography using tools like Pyrrha and Numbat. She
demonstrated how custom parsers with Kaitai Struct and visual mapping can
uncover vulnerabilities and provide deep insights into firmware structure,
aiding in securing IoT and embedded devices.
