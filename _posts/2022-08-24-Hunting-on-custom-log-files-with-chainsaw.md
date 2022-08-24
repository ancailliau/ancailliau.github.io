---
title: Hunting on custom log files with Chainsaw
tags: Sigma threat-hunting chainsaw
---

The previous posts looked on how we could hunt on [forged EVTX 
files](/2022/08/17/Creating-EVTX-for-malicious-activity.html). However, in the course of an incident response or advanced threat
hunting, not all logs lies in properly formated EVTX files. For example, some firewalls
export their logs in JSON format, some application will output XML. On Windows
servers, these might eventually reach an event log but on Linux, it will most
likely remains as it is. As I don't like wasting my time, I want to leverage
Sigma rules on JSON and XML files. Today, I focus my post on JSON files.

In an ideal scenario, these logs are ingested in your SIEM and you can
compile your [Sigma](https://github.com/SigmaHQ/Sigma) rule to the proper backend. But in real life, that is not always the
case and you sometimes need to fallback on good ol' text files. During an 
incident response, you don't have the luxury to wait for the logs to be ingested;
during a threat hunt you don't want the SOC to spend time ingesting a useless
log source.

To illustrate the process, let's take a log entry from [AWS Network Firewall](https://docs.aws.amazon.com/network-firewall/latest/developerguide/firewall-logging.html)

```
{
  "firewall_name": "test-firewall",
  "availability_zone": "us-east-1b",
  "event_timestamp": "1602627001",
  "event": {
    "timestamp": "2020-10-13T22:10:01.006481+0000",
    "flow_id": 1582438383425873,
    "event_type": "alert",
    "src_ip": "203.0.113.4",
    "src_port": 55555,
    "dest_ip": "192.0.2.16",
    "dest_port": 111,
    "proto": "TCP",
    "alert": {
      "action": "allowed",
      "signature_id": 5,
      "rev": 0,
      "signature": "test_tcp",
      "category": "",
      "severity": 1
    }
  }
}
```

While `grep`, `cut`, `sort`, `awk` or `jq` are of great help (in particular when the
log file is `jsonl`), they form
recipe that are difficult to share and document. Good luck if your log format is XML however. You could probably create the
`grep` expression that would matches the log entries you assessed malicious but
you would like to hoard that knowledge.

When the SOC has completed its job of ingesting the log source, you don't want to come back and re-create a detection rule. 

A solution is to write a Sigma rule and apply it directly to the text file.
[Chainsaw 2.0](https://github.com/WithSecureLabs/chainsaw/releases/tag/v2.0.0) released yesterday supports JSON logs, with some caveat. 

For our example, we could assess for example that this log entry is malicious
because it is an alert (`"event_type":"alert"`) to the destination ip `192.0.2.16` (so `"dest_ip":"192.0.2.16"`).
A possible rule is then

```yaml
title: Suspicious activity to compromised machine
id: 94f9921a-23b2-11ed-861d-0242ac120002
status: experimental
description: An alert for the test rule test_tcp to a compromised machine
author: Antoine Cailliau
date: 2022/08/24
detection:
    selection:
        EventType: 'alert'
        DestinationIp: '192.0.2.16'
    condition: selection
level: low
fields:
    - Signature
    - SourceIp
    - DestinationIp
falsepositives:
    - Unknown    
```

Before being able to execute the rule against the JSON file, we need to create a mapping
that will translate the fields of the Sigma rule to the correct JSON fields. 
For the example here, I decided the following mapping:

| -- | -- | -- |
| Sigma Field | JSON Path | Example Value |
| -- | -- | -- |
| FirewallName | firewall_name | test-firewall | 
| AvailabilityZone | availability_zone | us-east-1b | 
| TimeCreated | event_timestamp | 1602627001 | 
| Timestamp | event.timestamp | 2020-10-13T22:10:01.006481+0000 | 
| FlowID | event.flow_id | 1582438383425873 |
| EventType | event.event_type | alert | 
| SourceIp | event.src_ip | 203.0.113.4 | 
| SourcePort | event.src_port | 55555 |
| DestinationIp | event.dest_ip | 192.0.2.16 | 
| DestinationPort | event.dest_port | 111 |
| Protocol | event.proto | TCP | 
| Action | event.alert.action | allowed | 
| SignatureID | event.alert.signature_id | 5 |
| Revision | event.alert.rev | 0 |
| Signature | event.alert.signature | test_tcp | 
| Category | event.alert.category |  | 
| Severity | event.alert.severity | 1 |

That can be translate in Chainsaw format as follow. Note that the format requires
a `filter` section, commonly on the provider. Here I decided to call the provider
`AWS-Network-Firewall`.

```
---
name: Chainsaw's Sigma mappings to AWS Firewall logs
kind: json
rules: sigma

groups:
  - name: Sigma
    timestamp: Timestamp
    filter:
      Provider: "*"
    fields:
      - name: Firewall Name
        from: FirewallName
        to: firewall_name
      - name: Availability Zone
        from: AvailabilityZone
        to: availability_zone
      - name: Time Created
        from: TimeCreated
        to: event_timestamp
      - name: Timestamp
        from: Timestamp
        to: event.timestamp 
      - name: Flow ID
        from: FlowID
        to: event.flow_id
      - name: Event Type
        from: EventType
        to: event.event_type
      - name: Source Ip
        from: SourceIp
        to: event.src_ip
      - name: Source Port
        from: SourcePort
        to: event.src_port
      - name: Destination Ip
        from: DestinationIp
        to: event.dest_ip 
      - name: Destination Port
        from: DestinationPort
        to: event.dest_port 
      - name: Protocol
        from: Protocol
        to: event.proto 
      - name: Action
        from: Action
        to: event.alert.action 
      - name: SignatureID
        from: Signature ID
        to: event.alert.signature_id 
      - name: Revision
        from: Revision
        to: event.alert.rev 
      - name: Signature
        from: Signature
        to: event.alert.signature
      - name: Category
        from: Category
        to: event.alert.category
      - name: Severity
        from: Severity
        to: event.alert.severity 
      - name: Provider
        from: Provider
        to: provider
```

For our JSON lines to have a chance to match the rule, we need to massage it for Chainsaw:
1. Our log file is composed on JSON lines and Chainsaw requires an array of JSON objects.
2. Our log file does not contain the *provider* field.
3. The format of the date `2020-10-13T22:10:01.006481+0000` does not please Chainsaw.

Hopefully, `jq` can apply such transform very easily. 

```shell
$ cat 20220824-aws-network-firewall.json \
    | jq '.provider = "AWS-Network-Firewall"' \
    | jq '.event.timestamp = (.event.timestamp | split(".")[0] | strptime("%Y-%m-%dT%H:%M:%S") | strftime("%Y-%m-%dT%H:%M:%S.0000Z"))' \
    | jq -s . \
    > 20220824-aws-network-firewall_chainsaw.json
```

and we can now run the Sigma rule against the log file,

```shell
$ ./chainsaw hunt ./20220824-aws-network-firewall_chainsaw.json \
    --sigma ./example-aws-rule.yaml \
    --mapping ./mappings/sigma-aws-firewall.yml
```

For example,

```shell
$ ./chainsaw hunt ./20220824-aws-network-firewall_chainsaw.json --sigma ./example-aws-rule.yaml --mapping ./mappings/sigma-aws-firewall.yml  --json -q | jq .
[
  {
    "group": "Sigma",
    "kind": "individual",
    "document": {
      "kind": "json",
      "data": {
        "availability_zone": "us-east-1b",
        "event": {
          "alert": {
            "action": "allowed",
            "category": "",
            "rev": 0,
            "severity": 1,
            "signature": "test_tcp",
            "signature_id": 5
          },
          "dest_ip": "192.0.2.16",
          "dest_port": 111,
          "event_type": "alert",
          "flow_id": 1582438383425873,
          "proto": "TCP",
          "src_ip": "203.0.113.4",
          "src_port": 55555,
          "timestamp": "2020-10-13T22:10:01.0000Z"
        },
        "event_timestamp": "1602627001",
        "firewall_name": "test-firewall",
        "provider": "AWS-Network-Firewall"
      }
    },
    "name": "Suspicious activity to compromised machine",
    "timestamp": "2020-10-13T22:10:01+00:00",
    "authors": [
      "Antoine Cailliau"
    ],
    "level": "low",
    "source": "sigma",
    "status": "experimental",
    "falsepositives": [
      "Unknown"
    ],
    "id": "94f9921a-23b2-11ed-861d-0242ac120002"
  }
]
```

Happy hunting :bow_and_arrow:
