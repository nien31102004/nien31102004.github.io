---
title: "Event 3 - Seminar on Professional Knowledge and Personal Development Skills"
date: 2026-07-25
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Recap: Seminar on Professional Knowledge and Personal Development Skills

### Event Information

* **Date:** July 25, 2026
* **Location:** Amazon Office / AWS Training Center
* **Program week:** Week 6 of the internship program

### General Introduction

During the final week of the internship program, I attended a seminar that created a vibrant learning environment by combining professional knowledge, practical project experience, and personal development skills.

Unlike the regular AWS lab sessions, this event went beyond technical instructions. It brought together real stories about learning Cloud, participating in Hackathons, developing products under time pressure, overcoming personal limitations, and collaborating effectively within a team.

The seminar helped me understand that technical knowledge alone is not enough to create a successful product. A developer must also know how to identify user problems, communicate ideas, manage time, divide responsibilities, and remain confident when facing unfamiliar challenges.

### Event Objectives

* Explore effective approaches to learning AWS Cloud.
* Learn from real Hackathon projects and team experiences.
* Understand how an idea can be transformed into a working product.
* Improve confidence, communication, and teamwork skills.
* Learn practical methods for managing projects and overcoming procrastination.
* Apply the lessons from the seminar to the **Secure AI-Driven Document Platform** group project.

---

## 1. AWS Cloud Quest and the Cloud Learning Roadmap

The first session introduced **AWS Cloud Quest** and discussed how learners can build a practical Cloud learning roadmap.

Instead of relying only on technical documentation, AWS Cloud Quest allows learners to interact with simulated scenarios and solve tasks based on real-world Cloud situations. This learning method makes technical concepts easier to understand because learners can immediately observe how AWS services behave after each configuration.

One of the most valuable recommendations was to begin with the core AWS services before moving to more advanced topics. A suitable learning path should start with:

* AWS Identity and Access Management (IAM)
* Amazon EC2
* Amazon VPC
* Amazon S3
* Amazon RDS

After understanding these foundational services, learners can continue with Serverless architecture, Data Analytics, Artificial Intelligence, and Machine Learning.

The main message of this session was **“learning by doing.”** Reading documentation is important, but practical skills are built through hands-on experience. Each lab should include personal notes about the implementation process, errors encountered, their causes, and the solutions used.

### Lessons Learned

* Cloud knowledge should be developed through both theory and practice.
* Learning objectives should be divided into small and measurable milestones.
* Errors during labs are valuable learning opportunities.
* Recording solutions helps save time when similar problems occur again.
* Unused AWS resources should be removed after testing to avoid unnecessary costs.

### Personal Application

This learning approach was useful throughout my internship. When working with Cognito, Lambda, API Gateway, JWT, and Amazon S3, I did not only follow instructions. I also recorded configuration steps, test results, errors, and troubleshooting methods so that I could reuse the knowledge in later stages of the project.

---

## 2. Hackathon Experience: From Idea to Product

The Hackathon session was the most energetic part of the event. Four standout teams shared their journeys from identifying a problem to developing and presenting a working solution.

Their stories showed that a Hackathon is not simply a coding competition. Teams must work under strict time constraints while handling idea selection, business analysis, task allocation, technical integration, presentation, and unexpected problems.

### Team Signal Scout

Team Signal Scout shared their experience building a system for collecting and analyzing strategic business signals.

The team had to overcome not only technical difficulties but also challenges related to business logic and domain understanding. Their story about continuing development despite exhaustion at 3 AM reflected the pressure, determination, and team commitment required during a Hackathon.

The key lesson was that a strong solution must be based on a clear understanding of the business problem, not only on advanced technology.

### Team Plan V

Team Plan V introduced **SA Professional**, a solution that uses natural language to generate AWS architecture diagrams and estimate Cloud costs.

The project addressed a real pain point for Solution Architects: the amount of time required to design architecture, select services, and calculate estimated costs.

This solution demonstrated the value of combining Artificial Intelligence with AWS knowledge to automate repetitive tasks and improve productivity.

### Team 3KA

Team 3KA presented their journey of building **S.H.E.P.H.E.R.D**, a crowd-monitoring system powered by Computer Vision.

The team described the development process as an “emotional rollercoaster,” including moments of excitement, pressure, technical failure, and recovery.

Their most memorable message was that the experience, relationships, and lessons gained during the competition could be more valuable than the final prize.

### Team Six Six Fuller

Team Six Six Fuller introduced **Adaptive AML**, a solution designed to support banks in detecting and preventing money laundering.

The project demonstrated how Artificial Intelligence can be integrated into complicated banking workflows. It also showed that technology products in regulated industries must consider business rules, risk evaluation, explainability, and operational processes.

### Key Lessons from the Hackathon Teams

* Start with a real and clearly defined user problem.
* Avoid building too many features within a limited timeframe.
* Assign responsibilities according to each member’s strengths.
* Agree on architecture and technologies early.
* Integrate modules continuously instead of waiting until the end.
* Prepare demo data and a backup plan.
* Present the problem, solution, and value as a clear story.
* Trust teammates and focus on the shared objective.

### Connection to My Group Project

These lessons were highly relevant to the **Secure AI-Driven Document Platform**.

Our project includes multiple modules such as Frontend, Authentication & API, Document Processing, Security, AI Analysis, Administration, and Infrastructure. Therefore, the team must agree on architecture, API contracts, resource names, request formats, and response formats before integration.

The Hackathon stories helped me understand that a successful project requires more than individual modules that work separately. The entire workflow must be connected, tested, and presented as one complete product.

---

## 3. Confidence and Soft Skills Development

Another important session focused on confidence, communication, and personal development.

The speakers explained that confidence is not something people automatically possess. It develops through preparation, practice, knowledge, and the courage to participate.

Many students avoid asking questions because they are afraid of being wrong or judged. However, remaining silent may cause misunderstandings to continue and make teamwork less effective.

Confidence becomes stronger when a person:

* Prepares carefully before speaking or presenting.
* Understands the main message they want to communicate.
* Practices repeatedly.
* Accepts that small mistakes are part of learning.
* Actively asks questions when something is unclear.
* Participates in discussions and group activities.
* Receives feedback with a progressive mindset.

### Application to Technical Teamwork

This lesson was especially useful when discussing Cognito, API Gateway, JWT Authorizer, scopes, and User/Admin permissions with my group.

Instead of investigating every uncertainty alone, openly discussing technical issues helped the team verify assumptions and align the system architecture more quickly.

I also realized that technical communication is an important engineering skill. A developer should not only know how to implement a solution but also explain why that solution was selected and how it works within the overall architecture.

---

## 4. Project Management and Overcoming Procrastination

The final topic introduced **Task Decomposition**, a practical method for managing large tasks and reducing procrastination.

Procrastination does not always come from laziness. It may be caused by unclear objectives, fear of making mistakes, excessive pressure, or tasks that appear too large to begin.

For example, a broad task such as:

> Complete the Authentication module

does not clearly show what should be done first. A more effective approach is to divide it into smaller tasks:

1. Create a Cognito User Pool.
2. Configure the App Client.
3. Enable the login interface.
4. Register and verify a test user.
5. Create the `/me` Lambda function.
6. Create the HTTP API.
7. Configure the JWT Authorizer.
8. Define Resource Server scopes.
9. Create the `users` and `admins` groups.
10. Test User and Admin permissions.

Each small task has a clear result and can be verified independently. This approach reduces pressure, supports progress tracking, and makes troubleshooting easier.

### Lessons Learned

* Large goals should be divided into small, actionable tasks.
* Each task should have a clear expected result.
* Difficult work becomes easier after completing the first small step.
* Progress should be reviewed regularly.
* Teams should identify dependencies between tasks.
* Technical documentation should be updated throughout development rather than only at the end.

---

## Outcomes and Lessons Learned

After attending the seminar, I gained several important lessons:

* **Product mindset:** A good product must solve a real user pain point instead of only demonstrating that the code works.
* **Cost awareness:** Cloud architecture should be designed with both technical performance and cost optimization in mind.
* **Teamwork:** Team members should define responsibilities clearly, trust each other, and avoid allowing personal ego to affect the project.
* **Communication:** Asking questions and discussing uncertainty can prevent major technical mistakes.
* **Confidence:** Confidence is developed through preparation, practice, and active participation.
* **Task management:** Large objectives should be divided into smaller and measurable tasks.
* **Willingness to engage:** Students should not wait until they feel fully qualified before joining Hackathons or technology events.
* **Continuous learning:** Technical knowledge must be continuously improved through practice, feedback, and community interaction.

---

## Personal Reflection

The seminar helped me connect technical knowledge with the realities of product development.

The Hackathon stories showed that successful teams must balance technology, business value, time, and communication. The soft-skills sessions reminded me that confidence and teamwork directly influence technical progress. The task-management session also provided a practical method that I could immediately apply during the internship.

These lessons became valuable preparation for the **Secure AI-Driven Document Platform**, especially while implementing the Authentication & API module and coordinating integration with other project components.

### Conclusion

Although the seminar has concluded, its lessons about confidence, teamwork, product thinking, and continuous improvement will remain valuable throughout my internship and future career.

Becoming a capable Cloud engineer is not only about configuring AWS services or writing working code. It also requires the ability to understand user needs, communicate solutions, cooperate with teammates, manage pressure, and continuously learn from practical experience and the technology community.

![Event overview](/images/event3.jpg)

*Figure 1. Students attending the seminar at the Amazon Office / AWS Training Center.*