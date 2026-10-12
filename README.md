## End-to-End SOC Investigation and Log Analysis using Splunk SIEM
An end-to-end SOC investigation in Splunk Enterprise: a network scan is generated in a home lab, ingested into Splunk, searched, visualised on a dashboard and analysed to produce findings and recommendations.

## Overview
This project follows the workflow of a SOC analyst from start to finish. An Nmap service scan was run against a lab host, the scan output was ingested into Splunk as its own sourcetype, and SPL searches were used to count, chart and summarise the activity. The results were then put on a custom "SOC Investigation Dashboard", and the scan findings were reviewed to identify exposed services and recommend hardening steps.

## Lab architecture
![Lab architecture](docs/screenshots/soc1.jpg)
Data sources (PC / VM / network device) -> Log shippers -> Splunk (indexer, search head, forwarder) -> Web interface -> Analyst

## Tech stack
Splunk Enterprise 9.3.0 (SIEM)
Kali Linux (lab VM)
Nmap 7.98 (scan generation)
SPL (Search Processing Language)
MITRE ATT&CK framework

## Investigation workflow
Generate activity:run an Nmap service-version scan against the lab host `10.0.2.15`.
Ingest:save the scan output to `nmap_scan.txt` and index it in Splunk with sourcetype `nmap_scan`.
Search: confirm the events are searchable.
Analyse:count the events and chart them over time.
Visualise:build the SOC Investigation Dashboard.
Report:review the findings and recommend hardening.

## Step 1: Scan activity (Nmap)
nmap -sV 10.0.2.15
![Nmap scan](docs/screenshots/soc2.jpg)
The scan reported the host as up, with 997 filtered ports and these open services:

| Port | State | Service | Version |
| 22/tcp | open | ssh | OpenSSH 10.3p1 Debian (protocol 2.0) |
| 8000/tcp | open | http | Splunkd httpd |
| 8089/tcp | open | ssl/http | Splunkd httpd |

## Step 2: Ingest and search in Splunk
spl
index=* sourcetype="nmap_scan"
![Events search](docs/screenshots/soc3.jpg)
The scan was indexed from `nmap_scan.txt` (host `kali`) and returned 2 events: the scan start with its results, and the completion line.

## Step 3: Count the events
spl
index=* sourcetype="nmap_scan" | stats count
![Stats count](docs/screenshots/soc4.jpg)

Result: 2 events.

## Step 4: Timeline of the scan
spl
index=* sourcetype="nmap_scan" | timechart count
![Timechart](docs/screenshots/soc5.jpg)
The chart shows activity at the start and at the end of the scan, which matches the scan duration.

## Step 5: SOC Investigation Dashboard
![Dashboard](docs/screenshots/soc6.jpg)



Panels:
Nmap scan timeline:events over time
Attack summary:breakdown by sourcetype
Event count by source:events per source file

## MITRE ATT&CK mapping
| Observed behaviour | Technique | ID |
| Scanning a host to find open ports and services | Network Service Discovery | T1046 |
| Probing a target for exposed services | Active Scanning | T1595 |

## Findings
The scan identified three exposed services: SSH on 22, and the Splunk web interface and management port on 8000 and 8089.
997 other ports were filtered, so the host exposes a small attack surface.
The scan was fully visible in Splunk, so a scan can be searched, timed and summarised in a few SPL searches.

## Recommendations
Restrict ports 8000 and 8089 to trusted IPs only.
Enable HTTPS for the Splunk web interface.
Use key-based SSH authentication, disable password login, and add rate limiting such as fail2ban.
Close any service that is not needed.
Monitor firewall or network logs to alert on port-scan behaviour in real time.

## What I learned
How to ingest custom text output into Splunk with its own sourcetype.
How to use `stats` and `timechart` to summarise events.
How to build a dashboard for investigation.
How to turn raw output into findings mapped to MITRE ATT&CK.

## Future improvements
Ingest firewall, Zeek or Suricata logs and detect port scans by counting distinct destination ports per source in a short time window.
Add alerts for scan behaviour.
Add more data sources such as Windows and web server logs.

## Disclaimer
Lab use only. All scanning was done against my own lab machine. Do not scan systems you do not own or have permission to test.
