---
title: Highlights from Hack.lu 2024 (Day 2)
tags: conference
---

The second day of the conference offered a lot of insights, covering topics
from incident response to emerging threats in IoT, Kubernetes, and edge-device
security. Both morning and afternoon sessions provided practical takeaways, as
well as deeper perspectives on evolving attack surfaces and cybersecurity
techniques. The talks spanned diverse subjects, from corporate cybersecurity
incidents to advanced APT tracking, making for a dynamic and engaging day.

## Morning Sessions

**CSIRT and the Chocolate Factory**: This case study detailed how a small
company in the chocolate machinery business faced a ransomware attack. Despite
paying the ransom, they were unable to decrypt their files. The CSIRT team
stepped in, uncovering a flaw in the ransomware that allowed them to recover
the files without needing the decryption key, providing key insights into
ransomware analysis and remediation.

**The Gist of Hundreds of Incident Response Cases**: Stephan Berger shared
lessons learned from his experience in incident response, emphasizing the
importance of focusing on key artifacts, such as Anti-Virus logs and suspicious
network activity. He also highlighted the importance of automating and
streamlining incident investigations to handle large environments more
efficiently.

**IoT Hacks Humans: Unexpected Angles of Human Process Compromise**: This
talk explored how IoT devices, which interact closely with human processes, can
be manipulated to create significant security risks. By collecting data from
connected devices, attackers can influence behaviors, spread disinformation,
and exploit digital identities. He highlighted real-world examples, such as
opinion manipulation and personal surveillance. The talk emphasized the
expanding human attack surface due to connected environments and stressed the
need for better defenses and awareness to counter these growing threats.

**KubeHound: Identifying Attack Paths in Kubernetes Clusters**: Julien
Terriac introduced KubeHound, a tool designed to automate the identification of
attack paths in Kubernetes clusters. By applying graph theory, KubeHound allows
security teams to visualize and prioritize vulnerabilities, making it easier to
protect large-scale Kubernetes environments.

**It Has Been 0 Days Since the Last Edge-Device Security Incident**: Paul
Rascagneres from Volexity discussed recent zero-day vulnerabilities in edge
devices, focusing on Ivanti Connect Secure and Palo Alto Global Protect. The
talk highlighted the challenges of detecting and responding to these
vulnerabilities and emphasized the need for better monitoring and forensics on
edge devices.

## Afternoon Sessions

**Chrome V8 Exploitation Training for Beginners**: Hoseok Lee led a hands-on
workshop that introduced participants to the inner workings of the Chrome V8
engine. The training covered debugging techniques, analyzing vulnerabilities,
and exploiting bugs, providing beginners with a comprehensive guide to V8
exploitation.

**Securing the Stars: Modern Satellite Vulnerabilities**: Vic Huang
presented an analysis of satellite vulnerabilities, including real-world cases
of compromised communication systems. He highlighted weaknesses in both
traditional satellite systems and open-source CubeSat projects, emphasizing the
urgent need for better security in space technology.

**DFIQ: Codifying Digital Forensic Intelligence**: Thomas Chopitea
introduced DFIQ, a framework designed to organize and automate digital forensic
investigations. By codifying investigative questions and creating structured
workflows, DFIQ aims to standardize forensic practices and improve consistency
in investigations.

**Cyber Threats to Advanced Intelligent Connected Vehicle Systems**: Shihao
Xue and Yuqiao Ning detailed the vulnerabilities present in Intelligent
Connected Vehicles (ICVs), including issues with the T-Box and CAN bus
interfaces. They demonstrated how these vulnerabilities could be exploited for
remote vehicle control and data theft, stressing the need for stronger security
in automotive systems.

**APT28: Following Bear Tracks Back to the Cave**: Golo Baulig from IBM
X-Force explored the operations of the Russian state-sponsored group ITG05
(APT28), detailing recent campaigns, evolving malware, and infrastructure
changes. Baulig’s analysis provided insights into ITG05’s activities, including
their targeting of NATO member states and their continuous adaptation to
geopolitical shifts.
