# Week 7 Draft - Local IDS Network Segmentation and Zeek Telemetry Validation

This file is the internal source and implementation guide for the Week 7 Hugo worklog page.

Target Hugo page:

```text
content/1-Worklog/1.7-Week7/_index.md
```

Public evidence folder:

```text
static/images/worklog/week7/
```

Original screenshots are stored privately under:

```text
private-evidence/week7/originals/
```

## Positioning

Week 7 is:

```text
Week 7 - Local IDS Network Segmentation & Zeek Telemetry Validation
```

This week focuses on the local IDS lab baseline rather than a new AWS service.

The main story is:

```text
Kali DMZ -> pfSense -> Victim Ubuntu -> Zeek logs -> structured dataset export
```

## Topology and IP Plan

| Segment | Component | IP address |
| --- | --- | --- |
| VMware NAT / VMnet8 | pfSense WAN | `192.168.44.133/24` |
| DMZ / OPT1 | pfSense OPT1 | `10.10.10.1/24` |
| DMZ / OPT1 | Kali attacker | `10.10.10.10/24` |
| LAN | pfSense LAN | `192.168.1.1/24` |
| LAN | Ubuntu Victim | `192.168.1.10/24` |
| LAN | Zeek sensor | `192.168.1.20/24` |

## Source Notes Summary

Week 7 re-validates the existing local lab instead of reinstalling every VM.

Implemented and validated:

- pfSense interface assignments.
- OPT1 restricted rule for Kali-to-Victim.
- Kali-to-Victim positive traffic validation.
- Victim HTTP and SSH services.
- Zeek packet visibility with `tcpdump`.
- Zeek runtime context.
- `conn.log` content.
- `http.log` and UID correlation.
- Normal HTTP traffic generation.
- Controlled suspicious scan traffic.
- Dataset export to CSV and Pandas inspection.

Deferred:

- Kibana/ELK visualization.
- Model-training readiness validation.

## Evidence Map

| Public image | Use |
| --- | --- |
| `w7-e01-local-ids-topology.svg` | Local IDS topology and data flow |
| `w7-e02-pfsense-interface-assignments.png` | pfSense WAN/LAN/OPT1 interface addressing |
| `w7-e03-pfsense-opt1-firewall-rule.png` | Restricted OPT1 rule allowing Kali to Victim |
| `w7-e03a-pfsense-kali-victim-pass-log.png` | Positive firewall log for Kali-to-Victim traffic |
| `w7-e03b-pfsense-kali-zeek-block-log.png` | Supporting default-deny firewall evidence |
| `w7-e04-victim-network-services.png` | Victim IP, gateway, Apache, SSH, listening ports |
| `w7-e05-kali-to-victim-validation.png` | Kali route, ping, HTTP, and Nmap validation |
| `w7-e06-zeek-live-traffic-visibility.png` | Zeek VM sees live Kali-to-Victim packets |
| `w7-e07-zeek-runtime-status.png` | Zeek version, interface, and runtime context |
| `w7-e08-zeek-conn-log.png` | `conn.log` metadata validation |
| `w7-e09-zeek-http-uid-correlation.png` | `http.log` and UID correlation |
| `w7-e10-normal-http-session.png` | Normal HTTP traffic generation |
| `w7-e11-controlled-port-scan.png` | Controlled suspicious scan traffic |
| `w7-e12-zeek-dataset-export.png` | CSV dataset shape, columns, and label distribution |

## Redaction Checklist

Public images should not show:

- pfSense default admin password warning banner.
- Browser address bar with `Not Secure`.
- Browser profile/session indicators.
- Passwords, cookies, tokens, private keys, or credentials.
- Full raw PCAP/log payload beyond what is needed for the report.

Public images may show:

- Private lab IPs.
- Interface names.
- Firewall source/destination fields.
- Pass/block action icons.
- Zeek UIDs.
- `conn.log` and `http.log` fields.
- Dataset shape, column names, and label distribution.

## Evidence Caveat

The available block-log screenshot demonstrates pfSense default-deny behavior, but it does not clearly preserve the full `10.10.10.10 -> 192.168.1.20` tuple in the public-safe crop.

Before switching Week 7 to `draft: false`, capture a stronger block screenshot if the final report needs visual proof that Kali cannot reach the Zeek/ELK host.

## Publication Checklist

Before switching Week 7 to `draft: false`, confirm:

- Daily time estimates are approved or replaced with actual time spent.
- All public screenshots are reviewed in `static/images/worklog/week7/`.
- No raw/private evidence path is embedded in Hugo content.
- pfSense warning banners are not visible.
- Browser `Not Secure` bars are removed or explicitly handled.
- Kibana visualization is not presented as fresh evidence.
- Model-training readiness is deferred to Week 9.
- Week 8 transition points to traffic campaigns and event diary.
