---
title: Developping Sigma rules with Chainsaw
tags: sigma threat-hunting
---

From my experience, when you find evidence of malicious activity in a log file, 
there is probably more somewhere else, and the actor is likely to continue using
the same tools and techniques unless you detect it. Instead of just looking at
lines in a log file, when you discovered a malicious activity, write a detection
rule that can be applied to all your systems for retro-hunting, added to your
detection stack for future events, and shared with peers and partners to
benefit the greater good. 

My language of choice for detecting suspicious log entries is Sigma. Sigma is
to logs what Yara is to files and Snort is to network traffic. 
(Florian Roth)[https://twitter.com/cyb3rops] already shares a tremendous amount 
of Sigma rules that detect suspicious events but some events will be specific to
your environment and your incidents. For example, 

* If you detected an abused account, you can write a detection rule
  that triggers on any log mentioning the account and be notified of any 
  further event.
  
* If you detected a compromised host that is used to RDP to other machines, you
  can write a detection rules that, when executed against the event logs of all
  your machines, will help you scoping the intrusion.
  
Because Sigma rules are source-agnostic, you can write one rule and use it to
run against collected event log files, as a Kibana query or as a Sentinel KQL 
query. With the increasing diversity in systems, it is a bless to not have to 
maintain many detection rules that achieve the same logic.

To learn more about detection engineering with Sigma, I recommend the following 
course: [Detection Engineering with Sigma](https://www.networkdefense.co/courses/sigma/)
that has, in my opinion, a good return on investment.

My process to develop Sigma rules against Windows Event logs:

1. Find malicious or suspicious activity in event log entries.
2. Characterize **why** the activity is suspicious, focus on your rationale for 
   assessing that the activity is not right.
3. Write a Sigma rules that detects the activity
4. Run the rules against event logs that contains the suspicious activity (to 
   confirm the rule is detecting the activity), against event logs that are 
   known to be clean (to assess the false positive rate), and against a 
   collection of generic event logs containing malicious activity as in the 
   (evtx-attack-sample)[https://github.com/sbousseaden/EVTX-ATTACK-SAMPLES] 
   dataset.
5. Assess the true and false positives rates, refine and go to Step 4 until you
   are satisfied.
   
To run the rules, I use [Chainsaw](https://github.com/WithSecureLabs/chainsaw)
to run the sigma rules against EVTX files. 

You first need to install and compile the tool:

```shell
$ git clone https://github.com/countercept/chainsaw.git
$ cd chainsaw
$ cargo build --release
```
   
You might need to install dependencies first. On ubuntu:

```shell
$ sudo apt install cargo gcc-multilib
```

The tool is then available in the `target/release/` folder:

```shell
$ cd target/release/
$ ./chainsaw --help
```

When installed you can run the rules with `hunt` command:

```shell
$ ./chainsaw hunt /path/to/evtx/folder/ \
    --sigma /path/to/sigma/rules/ \
    --mapping ../../mappings/sigma-event-logs-all.yml
```

For example, to train, we would like to run the set of "standard" Sigma rules 
from the (official repository)[https://github.com/SigmaHQ/sigma] against the 
known-to-be-malicious evtx files from 
(evtx-attack-sample)[https://github.com/sbousseaden/EVTX-ATTACK-SAMPLES]. 

First, clone the rules and the samples

```shell
$ git clone https://github.com/SigmaHQ/sigma
$ git clone https://github.com/sbousseaden/EVTX-ATTACK-SAMPLES.git 
```

Then, we can detect malicious activity by running the tool (I assume that your
are in `chainsaw/target/release` folder)

```shell
$ ./chainsaw hunt /mnt/hgfs/Cases/EVTX-ATTACK-SAMPLES/ \
    --sigma /mnt/hgfs/Cases/sigma/rules \
    --mapping ../../mappings/sigma-event-logs-all.yml
```

Depending on your marchine it can take a while to complete. In my case, it 
detected quite a few log entries to check :-)

```shell
[+] 1353 Detections found on 924 documents
```

Other useful ressources:

* [Rapidly Search and Hunt through Windows Event Logs](https://rustrepo.com/repo/countercept-chainsaw)
* [Down the Chainsaw path to analyse Windows Event logs](https://www.vanimpe.eu/2022/08/16/down-the-chainsaw-path-to-analyse-windows-event-logs/)
