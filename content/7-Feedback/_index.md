---
title: "Sharing and Feedback"
date: 2026-07-10
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

## Overall Reflection

After completing the First Cloud Journey internship and workshop activities, I gained a much clearer understanding of how cloud knowledge is applied in a real project environment. Before joining the program, I mostly understood AWS services as separate concepts. Through the internship, I learned how those services work together to form a complete system.

The most valuable part of the experience was building and deploying the **AI-Powered Multi-Model Hybrid Cloud Security Monitoring Platform**. This project helped me connect cloud infrastructure, backend services, machine learning models, security monitoring, and operational cleanup into one practical workflow.

Instead of only learning theory, I had the opportunity to work with real AWS services such as VPC, EC2, S3, CloudFront, ALB, SQS, RDS, CloudWatch, IAM, Secrets Manager, KMS, and WAF. I also learned that a successful cloud project is not only about making the application run, but also about controlling cost, managing access, collecting evidence, monitoring operations, and cleaning up resources correctly after the workshop.

---

## Program and Working Environment

The First Cloud Journey program provided a practical and open learning environment. The workshop structure encouraged hands-on practice, which helped me understand AWS concepts more deeply than only reading documentation or watching tutorials.

The environment also encouraged self-learning and problem solving. During the project, I often had to investigate issues by checking AWS Console, reading error messages, reviewing service dependencies, and comparing the expected architecture with the actual deployed resources.

Some of the most valuable learning moments came from unexpected problems, such as:

- CloudFront and WAF behavior during XSS testing.
- CloudFront flat-rate pricing plan restrictions during cleanup.
- IAM Identity Center differences compared with traditional IAM users.
- RDS final snapshot and KMS retention decisions.
- SQS idempotency and DLQ handling.
- Evidence-first cleanup before deleting resources.

These situations made the internship feel closer to a real cloud engineering experience rather than a simple lab exercise.

---

## Support from Mentor and Team

The mentor and support team provided useful guidance throughout the internship. The guidance was not only about giving direct answers, but also about helping me understand how to analyze a problem step by step.

When I encountered issues, I learned to ask better technical questions, collect evidence, check AWS service behavior, and explain my decisions more clearly. This helped me improve both technical thinking and communication skills.

Teamwork was also an important part of the project. The group had to divide responsibilities across evidence collection, backend deployment, frontend delivery, security configuration, network cleanup, and documentation. Through this process, I learned how important it is to define ownership, record decisions, and avoid deleting shared resources before confirming dependencies.

---

## Relevance to My Academic Major

My major is **Machine Learning and Applications**, and the internship was highly relevant to this field. The project combined machine learning with cloud deployment and cybersecurity monitoring.

From the machine learning perspective, I worked with dataset preparation, feature engineering, model validation, threshold selection, holdout evaluation, and AI model handoff. The AI2A model focused on network behavior, while the AI2B model focused on HTTP-based SQL Injection and Cross-Site Scripting detection.

From the cloud engineering perspective, I learned that a machine learning model alone is not enough to create a usable system. The model must be integrated with backend APIs, data queues, databases, storage, monitoring, access control, and a user-facing dashboard. This helped me understand the full lifecycle from model artifact to production-style deployment.

This experience made the connection between academic machine learning knowledge and practical cloud system implementation much clearer.

---

## Learning and Skill Development

### Cloud Architecture

During the internship, I improved my understanding of core AWS architecture. I practiced designing and deploying components such as:

- VPC, public subnets, private subnets, route tables, and Internet Gateway.
- EC2 backend and worker instances.
- Application Load Balancer and Target Groups.
- S3 static frontend bucket and data/artifact bucket.
- CloudFront distribution for frontend and API delivery.
- SQS main queue and Dead-Letter Queue.
- RDS PostgreSQL for final alert storage.
- CloudWatch logs, alarms, and operational evidence.

I learned that each service has a specific role, but the real challenge is understanding how they depend on each other. For example, EC2 depends on IAM roles and security groups, ALB depends on target groups, CloudFront depends on origins and behaviors, and RDS deletion must consider final snapshots and encrypted data.

### Security and Access Management

Security was one of the most important parts of the internship. I learned how to work with:

- Security Groups and network access control.
- IAM runtime roles for backend and worker services.
- IAM Identity Center for human access management.
- Secrets Manager for database credentials.
- KMS key retention decisions.
- AWS WAF managed rules and CloudFront Web ACL association.
- MFA evidence for team users.

One important lesson was that access management should be separated into two layers: human access and runtime access. Human access should be managed through IAM Identity Center users, groups, permission sets, and account assignments. Runtime access should be managed through IAM roles and policies used by EC2, worker, S3, SQS, RDS, and CloudWatch.

### AI and Security Engineering

The SOC AI project helped me better understand how AI can support cybersecurity monitoring. I worked with Zeek logs, attack traffic, normal traffic profiles, model adapters, and alert fusion logic.

The main detection flows included:

- SQL Injection detection from HTTP request data.
- Cross-Site Scripting detection from HTTP URI/query data.
- Network behavior analysis from Zeek connection logs.
- Alert generation and storage in RDS.
- Dashboard/API output for analyst review.

Through this work, I learned that AI security systems must be evaluated carefully. It is important to avoid data leakage, check model shortcuts, validate holdout results, and design clear API contracts between the model and backend system.

### Operations, Cost Control, and Cleanup

A major lesson from the internship was that cloud engineering does not end after deployment. Cleanup and cost control are also part of the system lifecycle.

During cleanup, I learned to follow an evidence-first teardown process:

1. Capture final evidence.
2. Stop producers and replay scripts.
3. Drain or inspect queues and DLQs.
4. Stop and terminate compute resources.
5. Delete ALB and target groups.
6. Delete SQS queues and RDS with final snapshot.
7. Archive and remove S3 buckets when safe.
8. Review CloudFront, WAF, OAC, IAM, CloudWatch, NAT Gateway, EIP, and KMS.
9. Record deferred resources when AWS prevents immediate deletion.

The CloudFront/WAF cleanup issue was especially useful. I learned that some AWS resources cannot always be deleted immediately because of pricing-plan or service constraints. In that case, the correct engineering decision is not to force deletion, but to document the reason, reduce the resource to a safe state, and schedule a follow-up cleanup.

---

## Most Satisfying Experience

The most satisfying part of the internship was seeing the SOC AI system work as an end-to-end cloud-based demo.

It was meaningful to see local lab traffic and test events move through the system, reach the backend, pass through the AI detection layer, generate final alerts, store them in RDS, and display them on the dashboard.

Another satisfying part was solving real infrastructure problems. Issues such as CloudFront behavior, WAF blocking, RDS deletion protection, SQS idempotency, IAM role permissions, and Vercel/Hugo deployment helped me understand that cloud engineers need both technical knowledge and systematic troubleshooting skills.

---

## Suggestions for Program Improvement

The program is already very helpful for students who want to learn AWS through practical projects. However, I think the internship experience could be improved further with a few additions:

- Add a standard cleanup checklist for common AWS workshop resources.
- Add more guidance about cost control after labs and workshops.
- Add a short session comparing IAM Identity Center, IAM users, IAM roles, and permission sets.
- Add more examples of CloudFront, WAF, and pricing-plan behavior because these services can be confusing during cleanup.
- Provide an evidence register template so students know which screenshots should be captured and which sensitive fields should be masked before publishing.

These improvements would help future interns avoid common mistakes and make their final reports more professional.

---

## Future Expectations

After this internship, I want to continue improving my skills in AWS architecture, cloud security, DevOps, MLOps, and AI-based security monitoring.

I also want to continue developing the SOC AI project into a more complete system. Some possible future improvements include better model evaluation, stronger data pipeline design, improved dashboard usability, automated deployment, infrastructure as code, and better observability.

In the future, I hope to continue participating in cloud learning communities and contribute back by sharing my own workshop notes, lessons learned, and troubleshooting experience.

---

## Final Sharing

Overall, the First Cloud Journey internship was a valuable experience for me. It helped me move from learning separate cloud concepts to understanding how to build, operate, secure, and clean up a real cloud-based system.

The internship also helped me realize that a good technical project requires more than implementation. It requires documentation, teamwork, cost awareness, security awareness, evidence collection, and clear decision making.

This experience gave me more confidence to continue learning AWS and to pursue future work related to cloud engineering, machine learning applications, and cybersecurity systems.