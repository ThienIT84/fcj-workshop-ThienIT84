---
title: "Self-Assessment"
date: 2026-07-12
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

## Internship Context

My internship was carried out at **Amazon Web Services Vietnam Co., Ltd.** from **20/04/2026 to 12/07/2026**. During this period, I worked on the project **AI-Powered Multi-Model Hybrid Cloud Security Monitoring Platform**, a security monitoring platform that combines cloud infrastructure, network telemetry, AI model outputs, backend processing, and an analyst-facing dashboard.

The project required me to connect several areas that are usually learned separately: AWS networking, IAM, storage, database services, Zeek/Suricata-style security logs, AI model evaluation, FastAPI backend integration, asynchronous processing with SQS, RDS persistence, deployment evidence, and cleanup discipline. Because of that, the internship helped me understand the difference between a feature that works locally and a system that can be deployed, tested, documented, and cleaned up responsibly.

## Main Contributions

Across the 12-week internship, my main contributions were:

| Area | Contribution |
| --- | --- |
| AWS foundation | Practiced IAM, VPC, subnet, route table, EC2, Security Group, S3, CloudFront, RDS, CloudWatch, and cost-control concepts |
| Data and telemetry | Built and reviewed lab traffic sources using Zeek logs, HTTP events, connection logs, and attack/normal traffic profiles |
| AI model work | Supported AI2A and AI2B workflows, including feature review, dataset quality checks, model freeze notes, and SQL Injection/XSS prediction validation |
| Backend integration | Connected FastAPI ingestion, model adapters, evidence routing, fusion decision output, and alert API contracts |
| AWS deployment | Helped validate the deployed thin slice with CloudFront, ALB, EC2 backend, API routes, SQS, DLQ, worker service, and RDS PostgreSQL |
| Persistence and reliability | Verified RDS `final_alerts`, `/api/alerts/latest`, and D5-5 idempotency behavior using `event_id` |
| Final reporting | Organized worklog evidence, cleanup documentation, Vercel deployment notes, and final workshop publication material |

## Self-Assessment Table

| No. | Criteria | Self rating | Evidence from internship |
| --- | --- | --- | --- |
| 1 | Professional knowledge and technical skills | Good | I applied AWS, backend, security monitoring, AI model, and database knowledge in one integrated SOC AI project. |
| 2 | Ability to learn | Good | I learned unfamiliar AWS services and debugging flows while working through deployment, IAM, SQS/RDS, and dashboard issues. |
| 3 | Proactiveness | Good | I actively tested API paths, checked logs, traced dashboard behavior, and prepared cleanup/deployment runbooks instead of only waiting for fixed instructions. |
| 4 | Sense of responsibility | Good | I preserved evidence, avoided unsupported claims, used RDS final snapshot thinking, and documented CloudFront/WAF/OAC deferred cleanup clearly. |
| 5 | Discipline and consistency | Fair to Good | I maintained a 12-week worklog and final report, but I still need to organize screenshots and documentation earlier instead of concentrating too much work near the deadline. |
| 6 | Communication and reporting | Fair to Good | I produced technical documentation and runbooks, but I still need to make status reports shorter, clearer, and easier for others to follow quickly. |
| 7 | Teamwork and collaboration | Good | I followed the FCAJ workshop structure, aligned work with mentor guidance, and converted technical progress into reviewable workshop pages. |
| 8 | Problem-solving ability | Good | I debugged issues across frontend API usage, WebSocket assumptions, CloudFront/WAF behavior, S3 permissions, SQS/RDS wiring, and cleanup dependencies. |
| 9 | Security and cost awareness | Good | I considered least privilege, evidence retention, S3 versioning, RDS snapshots, Secrets Manager recovery window, DLQ handling, and AWS cleanup order. |
| 10 | Contribution to the project | Good | I helped bring the project from local AI/backend experiments to deployed API evidence, persisted alerts, idempotency validation, cleanup documentation, and a public report site. |
| 11 | Overall performance | Good | I completed the internship with a working technical story, documented evidence, known limitations, and a clear final submission path. |

## Strengths

My strongest improvement during this internship was the ability to connect multiple technical layers into one working flow. At the beginning, I mostly treated cloud services, AI models, backend APIs, and security logs as separate topics. By the end, I could reason about an end-to-end path such as:

```text
Zeek / replay event
-> CloudFront / ALB
-> FastAPI backend
-> SQS main queue
-> worker processing
-> RDS final_alerts
-> API / dashboard evidence
```

I also improved my debugging mindset. Instead of only checking whether one command returned success, I learned to compare the behavior across logs, API responses, frontend rendering, database records, queue state, and AWS console evidence. This was especially important when the backend API worked but the dashboard did not render the latest alert, and when CloudFront/WAF behavior blocked or deferred cleanup steps.

Another strength was evidence discipline. I learned that a final report is stronger when it records what was proven, what was not proven, and what was deferred. For example, I did not claim worker-to-dashboard realtime WebSocket broadcast without evidence, and I recorded CloudFront/WAF/OAC cleanup as deferred instead of marking it as fully deleted.

## Areas for Improvement

The first area I need to improve is early organization. Some screenshots, evidence files, and documentation paths were handled late in the project. In future projects, I should define evidence folder names, screenshot naming rules, and report structure earlier so that final publication is smoother.

The second area is communication. I can explain technical details, but I sometimes include too many intermediate details at once. I need to practice summarizing current status, root cause, next action, and risk in a shorter format, especially when reporting to a mentor or team.

The third area is frontend integration planning. Backend/API/RDS evidence was strong, but dashboard rendering and realtime behavior needed separate validation. In future work, I should define UI integration tests earlier, including API polling, WebSocket behavior, stale build/cache checks, and production environment variables.

## Final Reflection

This internship showed me that building a security monitoring platform is not only about training a model or deploying a server. A useful system also needs clean data, reliable APIs, correct IAM permissions, observable logs, persistent storage, careful frontend integration, and a cleanup process that protects both cost and evidence.

The most valuable lesson for me was learning to be honest about system boundaries. A project can still be strong when it clearly says which parts are complete, which parts are proven by evidence, and which parts need follow-up. That mindset helped me finish the SOC AI workshop with a realistic final report instead of a superficial demo.
