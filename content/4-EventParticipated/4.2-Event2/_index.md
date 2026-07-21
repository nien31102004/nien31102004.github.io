---
title: "Event 2 - From Student to Cloud, Data, and DevOps Professional"
date: 2026-06-15
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# Reflection Report: From Student to Cloud, Data, and DevOps Professional

### General Introduction

The second event I attended connected career development, AWS system architecture, data analysis, corporate culture, and the realities of DevOps work.

The program consisted of five presentations. Each speaker contributed a different perspective: a former Cloud Journey member's career story, the architecture of a scalable URL Shortener, the mindset of a Data Analyst, the importance of corporate culture, and the responsibilities of a DevOps Engineer.

A major strength of the event was its practical focus. The speakers used real projects, workplace situations, and personal lessons to help students understand professional expectations beyond classroom theory.

### Event Objectives

* Connect students, mentors, and technology professionals.
* Share career paths from university to professional roles.
* Introduce practical AWS system-design principles.
* Expand understanding of Data Analytics, corporate culture, and DevOps.
* Help students build projects, CVs, and portfolios supported by evidence.
* Encourage balanced development of technical and customer-facing skills.

---

## 1. Hiếu Kỵ – Renova Cloud and Former Cloud Journey Member

The speaker shared the development of the Cloud Journey community and explained how former members had become mentors, engineers, and managers at AWS partners.

He described an AI Robot project for a city that failed during an important live demonstration. Instead of giving up, the team investigated the issue and recovered the project. The experience later became evidence of his problem-solving ability and contributed to his opportunity at Renova Cloud.

This story taught me that real projects are rarely perfect. How a person responds to failure can demonstrate more value than a flawless classroom exercise.

The speaker emphasized personal projects as concrete proof of technical ability. A strong project should explain the problem, architecture, technology choices, individual contribution, challenges, outcomes, and future improvements.

He also introduced a 50/50 mindset: approximately half of professional success comes from technical ability, while the other half comes from customer-facing skills such as listening, explaining, questioning, and building trust.

The “AWS Everything” mindset encouraged students to integrate Cloud and AI into university subjects. A web assignment could use S3 and CloudFront, a database assignment could use RDS or DynamoDB, and a backend assignment could use Lambda and API Gateway.

Another important lesson was that growth often occurs in uncomfortable situations, such as accepting difficult projects, speaking in public, and resolving production-like incidents.

Finally, the speaker highlighted documentation as a long-term asset. Even basic technical guides may later support larger customer projects.

---

## 2. Trung Kim and Minh Thọ – Building a Scalable URL Shortener

The second presentation analyzed a URL Shortener. Although the user-facing function appeared simple, large-scale operation required security, low latency, unique key generation, caching, database storage, and independent services.

CloudFront was used to improve global access speed, while AWS WAF implemented rate limiting and traffic filtering. This demonstrated the importance of placing protection near the user.

A Key Generation Service was deployed on Amazon ECS. Redis through Amazon ElastiCache provided microsecond-level access for frequently requested data, while DynamoDB offered durable storage.

The speakers summarized four major best practices:

### Workflow Separation

Separate responsibilities so that one service failure does not stop the entire system.

### Defense in Depth

Use multiple security layers instead of relying on one control.

### Pre-generation

Create resources or keys before user demand to reduce latency and absorb traffic spikes.

### Cache First

Check frequently accessed data in Redis before querying the database.

The live demonstration allowed participants to scan a QR code and use the URL Shortener directly. This reinforced the value of preparing an interactive and reliable demo.

The architecture was relevant to our Secure AI-Driven Document Platform. Our project also separated upload from inspection, used SQS for asynchronous processing, applied CloudFront and WAF near the user, protected APIs with JWT, separated Lambda responsibilities, and stored metadata in DynamoDB.

---

## 3. Phạm Thành Đạt – Data Analyst

The third speaker redefined a Data Analyst as a professional who uses data to provide strategic advice rather than merely creating reports.

Examples involving VAT changes, administrative-area changes, and predictive maintenance showed that a Data Analyst must monitor external factors and understand how they affect business systems.

The predictive-maintenance example used vibration sensors to estimate equipment failure before breakdown. This demonstrated how data can reduce downtime and operational cost.

The speaker discussed capability levels ranging from following instructions to problem solving, system thinking, and strategic contribution. The key message was not to measure a career only by job title. Practical ability should be evaluated through problems solved and evidence produced.

Data Storytelling was another major topic. Dashboards should move from management-level overview to specialist-level detail. Every visualization should answer a question and reflect the reader's needs.

The speaker also emphasized cross-department communication. A Data Analyst must work with Marketing, Sales, Supply Chain, and other departments to understand the reasons behind the numbers.

This session helped me understand that cloud logs, metrics, and dashboards should be designed for their audiences. Developers need technical errors, while managers may need document volume, processing time, and rejection rates.

---

## 4. Nguyễn Mạnh Cường – Manufacturing Excellence and Corporate Culture

The fourth presentation focused on Culture Fit. Strong technical candidates may still fail final interviews if their behavior and values do not align with the organization.

Culture was defined as the way people think, live, and work. For a new intern, this can be demonstrated through punctuality, honesty about progress, early communication of difficulties, respect for teammates, and responsibility for mistakes.

The speaker discussed global standards such as ISO, GMP, and GDP. The underlying lesson also applies to software and cloud systems: reliable work requires documented processes, security controls, change management, logging, and access governance.

The AWS principle “It is always Day 1” emphasized curiosity, speed, and a startup mindset. For me, Day 1 means continuing to ask questions and never assuming that current knowledge is sufficient.

Customer Obsession was presented as a core principle. Technology creates value only when it solves a real user problem.

The speaker also emphasized kindness and purpose. Global companies value people who respect others and create responsible products and services.

---

## 5. Trương Hoàng Trọng – DevOps Engineer at Renova Cloud

The final speaker explained that DevOps is not only a trending or well-paid role. It carries significant responsibility for stability, delivery, cost, and security.

Typical responsibilities include on-call rotation, production incident response, log and metric monitoring, deployment support, cloud cost optimization, access control, automation, and collaboration with developers and security teams.

The incident-handling discussion showed that recovery is only the first step. Teams should also identify root causes, document the event, and create preventive actions.

The speaker recommended focusing on fundamentals rather than chasing every tool:

* Linux.
* Networking.
* One programming language such as Python or Go.
* Git.
* Software delivery processes.
* Operating systems, processes, and containers.

He also emphasized understanding the entire SDLC from idea and development to testing, build, deployment, monitoring, and logging.

The Container Mindset means understanding why containers are used and how they operate on Linux, not merely memorizing Docker commands.

A practical learning path was proposed:

1. Build a small application on localhost.
2. Publish it through a VPS or cloud environment.
3. Configure domain and HTTPS.
4. Containerize the application.
5. Automate build and deployment.
6. Add monitoring, logging, and security.

The speaker repeatedly encouraged students to ask “Why?” before adopting tools. Tool knowledge becomes valuable only when the problem, alternatives, and trade-offs are understood.

AI was described as leverage rather than a replacement for thinking. It can accelerate debugging and learning, but copying code without understanding weakens long-term ability.

The final recommendation was to document everything. Shared incident notes, configuration guides, and troubleshooting records reduce team dependency on individual knowledge.

---

## Overall Event Experience

The event successfully combined motivation, system architecture, data thinking, corporate culture, and operational responsibility.

The five presentations complemented each other:

* The first explained the value of projects and community.
* The URL Shortener demonstrated scalable architecture.
* The Data Analyst session connected data with business decisions.
* The culture session emphasized professional values.
* The DevOps session described real operational responsibility.

Interactive discussion and live demonstration made the event more engaging and practical.

<!-- Suggested image:
![Event overview](/images/4-EventParticipated/4.2-Event2/event-overview.jpg)
-->

---

## Lessons Applied to My Project and Career Direction

For the Secure AI-Driven Document Platform, I applied the following ideas:

* Defense in Depth across Frontend, API, and Lambda.
* CloudFront and WAF near the user.
* SQS to separate upload from inspection.
* Independent Lambda responsibilities.
* Cognito, JWT scopes, and groups for authorization.
* Logging, test cases, and demo preparation.
* Documentation for integration and troubleshooting.
* Explaining user value rather than only listing AWS services.

For my personal career direction, I plan to:

* Strengthen Linux, Networking, Python, and cloud fundamentals.
* Complete projects with working demos and documentation.
* Improve presentation and customer-facing skills.
* Connect with mentors and technology communities.
* Use AI to support learning without depending on blind copy-paste.
* Measure progress through ability and project evidence rather than job titles alone.

---

## Outcomes

After the event, I:

* Better understood the value of the Cloud Journey community.
* Recognized projects as important evidence in recruitment.
* Learned four practical best practices for scalable system design.
* Understood how Cache, Database, ECS, CloudFront, and WAF work together.
* Expanded my understanding of Data Analysis and Data Storytelling.
* Recognized the importance of Culture Fit and Customer Obsession.
* Understood the real responsibilities of a DevOps Engineer.
* Built a learning path focused on fundamentals, projects, and automation.
* Strengthened the habit of documenting errors, configuration, and solutions.

### Conclusion

This event was highly valuable because it connected multiple aspects of the technology industry. I learned not only about AWS Architecture, Data, and DevOps, but also about customer communication, corporate culture, responsibility, and evidence-based professional growth.

The event reinforced that career success is not based on the number of tools a person knows. Strong fundamentals, real problem solving, collaboration, and the ability to explain technical decisions clearly are more important.
