---
title: "FCAJ Hybrid SOC/AWS Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---


## Overview

This workshop documents the FCAJ Hybrid SOC/AWS project: a hybrid SOC pipeline that starts from local security evidence, moves events into AWS, processes them through cloud data services and AI/Fusion components, and prepares final alerts for downstream storage, monitoring, and dashboard views.

The current workshop is organized into 11 main sections, fully documented with verified configuration details and screenshots.

## Workshop content

1. [Overview and Architecture](5.1-Overview-and-Architecture/)
2. [Prerequisites and Naming](5.2-Prerequisites-and-Naming/)
3. [Networking, IAM and Security Baseline](5.3-Networking-IAM-Security-Baseline/)
4. [Frontend S3, CloudFront and WAF](5.4-Frontend-S3-CloudFront-WAF/)
5. [Backend EC2 Runtime](5.5-Backend-EC2-Runtime/)
6. [SQS, DLQ and S3 Data Bucket](5.6-SQS-DLQ-and-S3-Data-Bucket/)
7. [RDS, Secrets and Worker Pipeline](5.7-RDS-Secrets-Worker-Pipeline/)
8. [AWS Pipeline Validation and Dashboard Evidence](5.8-AWS-Pipeline-Validation-and-Dashboard-Evidence/)
9. [AI Fusion and Dashboard Validation](5.9-AI-Fusion-and-Dashboard-Validation/)
10. [Monitoring, Audit and Notification](5.10-Monitoring-Audit-Notification/)
11. [Cleanup and Cost Control](5.11-Cleanup/)

## Evidence policy

Only verified evidence should be included in workshop pages. All documented configurations are backed by active deployment and validation materials.
