---
title: "Week 7 - Local IDS Network Segmentation & Zeek Telemetry Validation"
date: 2026-06-03
weight: 7
chapter: false
draft: false
pre: " <b> 1.7. </b> "
---

## Week at a Glance

| Item | Result |
| --- | --- |
| Primary focus | Local IDS lab baseline |
| Main objective | Validate network segmentation and Zeek telemetry before dataset generation |
| Firewall/router | pfSense |
| Attacker host | Kali Linux in DMZ / OPT1 |
| Victim host | Ubuntu Victim in LAN |
| Telemetry host | Zeek sensor on LAN |
| Main evidence | pfSense rules/logs, Kali-to-Victim tests, Zeek packet visibility, `conn.log`, `http.log`, dataset export |
| Kibana visualization | Deferred |
| Week status | Implemented from local lab notes; evidence re-validation completed; time estimates require owner confirmation |

## Evidence Re-validation Note

The local IDS lab had already been built before this Week 7 worklog page was prepared. Week 7 does not claim that pfSense, Kali, Victim Ubuntu, and Zeek were installed from scratch during the evidence re-validation session.

Instead, the Week 7 worklog focuses on validating the existing lab:

```text
review topology
-> confirm pfSense interfaces
-> tighten OPT1 firewall rules
-> validate Kali-to-Victim access
-> review default-deny behavior
-> confirm Victim services
-> confirm Zeek packet visibility
-> inspect conn.log and http.log
-> export structured dataset
```

Evidence was re-collected on `01/07/2026` from the active local lab. Screenshots are sanitized public copies stored under:

```text
static/images/worklog/week7/
```

Kibana was not restored during this evidence re-validation, so Kibana visualization remains deferred.

## Objective

The objective of Week 7 was to prove that the local IDS lab can generate controlled network traffic and convert it into Zeek telemetry.

This matters because the Hybrid IDS/AI SOC project should not depend only on public datasets. A local lab provides a controlled source of traffic, known roles, known IPs, and explainable labels for later dataset generation and AI model training.

## Project Context

Weeks 1-6 established the AWS foundation, storage, database, static delivery, and operational monitoring concepts. Week 7 shifts focus to the telemetry source.

The local lab provides the raw security signals that can later be:

- captured by Zeek,
- exported as `conn.log` and `http.log`,
- parsed into CSV datasets,
- uploaded to S3 as artifacts,
- analyzed by AI detection components,
- summarized by the SOC backend.

Week 7 is therefore a bridge between infrastructure learning and the security-data pipeline.

## Local IDS Architecture Overview

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e01-local-ids-topology.svg" alt="Local IDS topology">
  <figcaption>Figure 1 - Local IDS lab topology used for Week 7 segmentation and Zeek telemetry validation.</figcaption>
</figure>

The validated topology was:

| Segment | Component | IP address |
| --- | --- | --- |
| VMware NAT / VMnet8 | pfSense WAN | `192.168.44.133/24` |
| DMZ / OPT1 | pfSense OPT1 | `10.10.10.1/24` |
| DMZ / OPT1 | Kali attacker | `10.10.10.10/24` |
| LAN | pfSense LAN | `192.168.1.1/24` |
| LAN | Ubuntu Victim | `192.168.1.10/24` |
| LAN | Zeek sensor | `192.168.1.20/24` |

The intended traffic path is:

```text
Kali 10.10.10.10
-> pfSense OPT1
-> pfSense LAN
-> Victim 192.168.1.10
-> Zeek telemetry on LAN
```

## Daily Worklog

| Activity | Date | Time spent | Work completed | Result | Issue / decision | Next step |
| --- | --- | --- | --- | --- | --- | --- |
| Local IDS topology review | 03/06/2026 | Estimated 3 hours | Reviewed the pfSense, Kali, Victim, and Zeek VM topology and documented the DMZ/LAN addressing model | The lab segmentation model was defined around Kali in OPT1 and Victim/Zeek in LAN | Keep private lab IPs visible because they explain the evidence | Validate pfSense interfaces |
| pfSense access and rule review | 04/06/2026 | Estimated 4 hours | Recovered pfSense WebGUI access, confirmed interfaces, identified that the troubleshooting rule `OPT1 net -> Any` was too broad, and replaced it with a host-to-host rule | Kali was limited to the Victim target instead of unrestricted LAN access | WebGUI used HTTP port 80 during the lab; public screenshots were cropped to remove browser warning bars | Validate allowed and blocked traffic paths |
| Victim and Kali connectivity validation | 05/06/2026 | Estimated 4 hours | Confirmed Victim IP/gateway, Apache and SSH status, Kali route to Victim, ICMP, HTTP response, and Nmap service results | Kali could reach Victim HTTP and SSH through pfSense | Victim services are lab targets, not production exposure | Validate Zeek visibility |
| Zeek visibility and log validation | 06/06/2026 | Estimated 4 hours | Used `tcpdump` on the Zeek VM while Kali generated traffic, checked Zeek runtime, inspected `conn.log`, inspected `http.log`, and correlated records using Zeek UID | Zeek saw Kali-to-Victim traffic and produced connection/application metadata | Running Zeek is not enough; packet visibility and log content must be checked | Export telemetry to a structured dataset |
| Evidence re-validation and dataset export | 01/07/2026 | Estimated 4.5 hours | Re-captured Week 7 screenshots, reviewed pfSense pass/default-deny evidence, exported a structured CSV dataset, checked dataset shape, columns, and label distribution | Week 7 evidence was consolidated into public-safe screenshots | Kibana was not restored for screenshot collection and remains deferred | Use Week 8 for traffic campaigns and event diary |

## Technical Implementation Summary

Week 7 validated a local IDS network where Kali was isolated in a DMZ-style segment and the Victim/Zeek hosts were placed in the LAN segment.

pfSense routed traffic between the two segments and enforced the access policy. The final OPT1 rule allowed only:

```text
Source:      10.10.10.10
Destination: 192.168.1.10
Protocol:    IPv4 any
```

This still allowed controlled lab traffic such as ICMP, HTTP, SSH, and Nmap testing against the Victim. The earlier broad troubleshooting rule was disabled.

Zeek was then validated from the telemetry side. Packet visibility was confirmed with `tcpdump`, and Zeek logs were inspected through `conn.log` and `http.log`. A structured dataset export was also checked with Pandas.

## Network Segmentation and pfSense Rules

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e02-pfsense-interface-assignments.png" alt="pfSense interface assignments">
  <figcaption>Figure 2 - pfSense interface assignments confirmed WAN, LAN, OPT1, and OPT2 addressing.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** WAN, LAN, and OPT1 were available, with LAN on `192.168.1.1/24` and OPT1 on `10.10.10.1/24`.  
**Scope:** OPT2 was visible but not part of the Week 7 telemetry path.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e03-pfsense-opt1-firewall-rule.png" alt="pfSense OPT1 firewall rule">
  <figcaption>Figure 3 - The OPT1 rule allows Kali `10.10.10.10` to reach only the Victim `192.168.1.10`.</figcaption>
</figure>

**Result:** Implemented  
**Key observation:** The host-to-host rule is active, while the earlier broad `OPT1 net -> Any` troubleshooting rule is disabled.  
**Security value:** Kali remains useful for controlled traffic generation without receiving unrestricted LAN access.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e03a-pfsense-kali-victim-pass-log.png" alt="pfSense pass logs for Kali to Victim">
  <figcaption>Figure 4 - pfSense firewall logs show Kali-to-Victim traffic matching the allow rule.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** ICMP, TCP/80, and TCP/22 traffic from `10.10.10.10` to `192.168.1.10` matched the `Allow Kali attacker to Victim lab target` rule.  
**Why it matters:** A firewall rule should be verified by traffic logs, not only by its configuration screen.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e03b-pfsense-kali-zeek-block-log.png" alt="pfSense default deny firewall log">
  <figcaption>Figure 5 - A pfSense default-deny log example was captured during firewall re-validation.</figcaption>
</figure>

**Result:** Supporting evidence  
**Key observation:** The screenshot demonstrates default-deny firewall logging, but it does not clearly preserve the full Kali-to-Zeek tuple in the public-safe crop.  
**Publication note:** Before switching this page to public, a stronger screenshot should be added if the report needs visual proof of `10.10.10.10 -> 192.168.1.20` being blocked.

## Victim and Kali Validation

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e04-victim-network-services.png" alt="Victim Ubuntu network and service status">
  <figcaption>Figure 6 - Victim Ubuntu had the expected LAN address and exposed HTTP/SSH services for the lab.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** Victim had `192.168.1.10/24`, default gateway `192.168.1.1`, Apache active, SSH active, TCP/80 listening, and TCP/22 listening.  
**Scope:** These services are intentional lab targets for telemetry generation.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e05-kali-to-victim-validation.png" alt="Kali to Victim validation">
  <figcaption>Figure 7 - Kali successfully reached the Victim through ICMP, HTTP, and service detection.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** Kali used `10.10.10.10`, routed to `192.168.1.10` through pfSense, received HTTP `200 OK`, and detected SSH/HTTP with Nmap.  
**Project value:** This proves the controlled traffic path needed before collecting IDS logs.

## Zeek Traffic Visibility

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e06-zeek-live-traffic-visibility.png" alt="Zeek tcpdump live traffic visibility">
  <figcaption>Figure 8 - The Zeek VM observed live Kali-to-Victim traffic on `ens33`.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** `tcpdump` showed packets in both directions between `10.10.10.10` and `192.168.1.10`, including HTTP and SSH-related traffic.  
**Why it matters:** Installing Zeek does not prove visibility; the sensor interface must actually see the traffic.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e07-zeek-runtime-status.png" alt="Zeek runtime status">
  <figcaption>Figure 9 - Zeek installation and runtime context were checked on the telemetry VM.</figcaption>
</figure>

**Result:** Supporting evidence  
**Key observation:** Zeek version and host network interfaces were visible, and process/runtime context was reviewed.  
**Scope:** Runtime status supports the packet/log evidence, but it is not sufficient by itself.

## Zeek Log Validation and UID Correlation

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e08-zeek-conn-log.png" alt="Zeek conn log validation">
  <figcaption>Figure 10 - `conn.log` contained connection-level metadata for Kali-to-Victim HTTP traffic.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** `conn.log` records included `10.10.10.10` as origin, `192.168.1.10` as responder, TCP/80, service `http`, bytes, packets, connection state, and Zeek UIDs.  
**Project value:** `conn.log` provides the core network-level features for later dataset generation.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e09-zeek-http-uid-correlation.png" alt="Zeek http log UID correlation">
  <figcaption>Figure 11 - `http.log` exposed HTTP metadata and shared Zeek UIDs with connection records.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** HTTP methods, host, URI, status fields, and UIDs were present for requests from `10.10.10.10` to `192.168.1.10`.  
**Correlation value:** The UID links connection-level records in `conn.log` with application-layer records in `http.log`.

## Dataset Export Validation

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e10-normal-http-session.png" alt="Normal HTTP session generation">
  <figcaption>Figure 12 - Repeated HTTP requests generated normal baseline traffic for Zeek capture.</figcaption>
</figure>

**Result:** Implemented  
**Key observation:** Repeated HTTP requests returned successful responses and created baseline traffic for comparison against suspicious activity.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e11-controlled-port-scan.png" alt="Controlled port scan traffic">
  <figcaption>Figure 13 - A controlled port scan generated suspicious but bounded lab traffic toward the Victim.</figcaption>
</figure>

**Result:** Implemented  
**Key observation:** The scan was limited to a known source, known destination, and defined port range.  
**Scope:** This proves suspicious traffic generation and logging, not production-grade detection accuracy.

<figure class="worklog-evidence">
  <img src="/images/worklog/week7/w7-e12-zeek-dataset-export.png" alt="Zeek dataset export validation">
  <figcaption>Figure 14 - Zeek-derived telemetry was exported into a structured CSV dataset and checked with Pandas.</figcaption>
</figure>

**Result:** Validated  
**Key observation:** The dataset shape, column list, and label distribution were inspected.  
**Scope:** This validates dataset export only. Missing values, duplicates, class balance, label quality, leakage, and AI readiness remain Week 9 work.

## Validation Matrix

| Validation | Actual result | Status |
| --- | --- | --- |
| pfSense interfaces | WAN, LAN, and OPT1 addresses verified | Pass |
| Kali segmentation | Kali placed in `10.10.10.0/24` | Pass |
| Victim segmentation | Victim placed in `192.168.1.0/24` | Pass |
| Restricted OPT1 rule | Kali allowed only to Victim in final rule design | Pass |
| Positive firewall test | Kali-to-Victim traffic logged as pass | Pass |
| Negative firewall test | Block behavior documented in notes; stronger tuple screenshot recommended | Review before publish |
| Victim HTTP/SSH | TCP/80 and TCP/22 active | Pass |
| Kali-to-Victim access | Ping, HTTP, and Nmap validation succeeded | Pass |
| Zeek packet visibility | Sensor saw Kali-to-Victim traffic | Pass |
| `conn.log` | Correct source, destination, service, and UID fields present | Pass |
| `http.log` | HTTP method, URI, and UID fields present | Pass |
| UID correlation | Connection and HTTP layers can be linked by UID | Pass |
| Normal traffic | Baseline HTTP traffic recorded | Pass |
| Suspicious traffic | Controlled scan traffic generated and recorded | Pass |
| Dataset export | Structured CSV inspected with Pandas | Pass |
| Kibana visualization | ELK/Kibana not restored during re-validation | Deferred |
| Model-training readiness | Data-quality checks not completed in Week 7 | Deferred |

## Security Decisions

| Decision | Reason |
| --- | --- |
| Place Kali in a DMZ-style segment | Isolate the attacker role from monitoring and LAN infrastructure |
| Route Kali traffic through pfSense | Force traffic through an observable policy point |
| Restrict Kali to one Victim IP | Prevent unrestricted lateral access inside the lab |
| Disable the broad OPT1 troubleshooting rule | Avoid leaving `OPT1 net -> Any` as the final state |
| Enable logging on the allow rule | Prove the rule matched real traffic |
| Keep Zeek/ELK separate from Kali | Protect the monitoring system from direct attacker access |
| Keep private lab IPs visible | They are required to explain topology and evidence |
| Keep raw logs and PCAPs private | They may contain detailed traffic data beyond the public report |
| Defer Kibana claim | Avoid claiming visualization evidence that was not freshly captured |

## Issues and Troubleshooting

| Issue | Root cause | Resolution |
| --- | --- | --- |
| pfSense HTTPS timed out | WebGUI was listening on HTTP port `80`, not HTTPS `443` | Checked service listening state and used the correct lab URL |
| Browser displayed insecure management warning | The lab WebGUI used HTTP during re-validation | Cropped public screenshots and documented this as a lab-only management limitation |
| Default admin-password warning appeared in pfSense screenshots | pfSense displayed a lab warning banner | Cropped public evidence so the banner is not published |
| OPT1 rule was too broad | Temporary `OPT1 net -> Any` rule was used for troubleshooting | Added a host-to-host allow rule and disabled the broad rule |
| Zeek VM had multiple LAN IPs at one point | Interface had both intended and extra address | Used routing/source-IP checks and documented the configuration risk |
| Need to prove sensor visibility | Zeek installation alone did not prove packets were visible | Used `tcpdump` while generating Kali-to-Victim traffic |
| Kibana evidence unavailable | ELK stack was not restored for screenshot collection | Marked Kibana visualization as deferred |
| Negative block screenshot was weak | Public crop did not clearly show Kali-to-Zeek tuple | Kept it as supporting default-deny evidence and recommended stronger evidence before publishing |

## Lessons Learned

Network segmentation evidence should show both configuration and behavior. A firewall rule screen is useful, but firewall logs prove whether traffic actually matched the intended rule.

Ping success does not mean a web interface is reachable over HTTPS. Troubleshooting should check ICMP, TCP port state, and the actual service listener.

Zeek must be validated at three levels: interface visibility, process/runtime status, and log content. The strongest evidence is not `zeek --version`; it is traffic seen by the sensor and represented in `conn.log` and `http.log`.

Zeek UID correlation is important for dataset work because it links connection-level metadata to application-layer records.

Dataset export is not the same as a model-training-ready dataset. Week 7 proves that structured export is possible, while data quality and label validation should be handled later.

## Weekly Outcome

By the end of Week 7, the local IDS lab had a validated network segmentation and telemetry baseline.

Kali was isolated in the OPT1/DMZ segment and allowed to reach the Victim host through a controlled pfSense rule. Victim HTTP and SSH services were active for lab traffic generation. Zeek observed live Kali-to-Victim traffic, produced connection and HTTP metadata, and exported telemetry into a structured dataset.

Kibana visualization and full model-training-readiness validation were intentionally deferred to avoid unsupported claims.

## Next Week Plan

Week 8 will move from lab validation to traffic campaigns and event diary management.

The focus will be:

- normal traffic campaigns,
- controlled attack traffic campaigns,
- event diary and timestamp tracking,
- Zeek `conn.log` and `http.log` collection,
- Suricata or additional IDS evidence if available,
- clean separation between raw traffic, parsed logs, and labeled events.
