---
title: "Sharing and Feedback"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

### Overall Evaluation

**1. Working and Learning Environment**  
During my internship in the First Cloud AI Journey program, I studied and worked in a hybrid format. This gave me the flexibility to complete AWS labs remotely while still participating in direct guidance sessions, seminars, and group activities. The learning environment was open and encouraged students to ask questions, share difficulties, and learn from one another. Through this experience, I gradually became more confident when working with AWS services and collaborating with other members.

**2. Support from Mentors and the Administration Team**  
The mentors and program administrators provided learning materials, weekly assignments, technical guidance, and necessary announcements throughout the internship. When I encountered problems while configuring AWS services, I was encouraged to review the system flow, read error messages, and find the cause before asking for support. This approach helped me improve my independent problem-solving skills instead of relying only on ready-made instructions.

**3. Relevance to My Academic Major**  
The internship was closely related to my Information Technology and Software Engineering studies. I had opportunities to apply knowledge of backend development, APIs, authentication, authorization, cloud infrastructure, databases, and system security. At the same time, I learned many new concepts that had not been covered deeply at university, such as AWS IAM roles and policies, Amazon Cognito, JWT authentication, API Gateway authorizers, serverless architecture, event-driven processing, data analytics, and AI/ML services on AWS.

**4. Learning and Skill Development**  
Throughout the program, I completed weekly labs involving services such as Amazon EC2, Amazon S3, Amazon RDS, Amazon VPC, AWS IAM, Amazon CloudWatch, AWS Tags, Resource Groups, data analytics services, and AI/ML services. These activities helped me understand how AWS services can be combined to build a complete system rather than being used independently.

In the group project, I took the role of **Authentication & API Developer** for the Secure AI-Driven Document Platform. I configured an Amazon Cognito User Pool, created a Single Page Application client, used Authorization Code Flow with PKCE, created an HTTP API on Amazon API Gateway, configured a JWT Authorizer, defined custom OAuth scopes, and created the `users` and `admins` groups. I also tested authentication and authorization cases, including missing tokens, invalid tokens, valid users, unauthorized users, and administrators. This was the part of the internship that helped me improve my technical skills the most.

**5. Teamwork and Professional Skills**  
The group project helped me understand the importance of dividing responsibilities and agreeing on technical contracts between modules. My authentication and API components had to work correctly with the frontend and the Lambda functions developed by the backend member. Therefore, I learned to communicate API routes, JWT claim locations, scopes, group permissions, expected status codes, and request/response formats clearly. I also improved my presentation, documentation, time-management, and teamwork skills through seminars, weekly reports, and project discussions.

**6. Most Satisfying Experience**  
The most satisfying moment was when I successfully completed the authentication and authorization flow from end to end. A user could sign in through Amazon Cognito, receive an access token, call a protected API, and have Amazon API Gateway validate the JWT before invoking AWS Lambda. The test results were clear: requests without a token or with an invalid token returned `401`, normal users calling an administrative API returned `403`, and users in the `admins` group received a successful `200` response. This result helped me understand the difference between authentication and authorization through an actual implementation.

---

### Suggestions for Improvement

I believe the program could be improved by providing a clearer project roadmap earlier, especially the expected architecture, team roles, naming conventions, and integration milestones. This would help students coordinate their modules more effectively and reduce rework when different members make different technical assumptions.

It would also be useful to organize more short technical review sessions during the group project. In these sessions, each team could present its current architecture, completed modules, integration issues, and next steps. Early reviews would help students detect problems related to API contracts, permissions, service selection, and system flow before the final integration stage.

In addition, more team-building or cross-team sharing activities would help interns become more comfortable communicating with one another. Because the internship was conducted in a hybrid format, students sometimes focused mainly on individual tasks and had fewer opportunities to exchange experiences with other groups.

---

### Recommendation and Future Expectations

I would recommend the First Cloud AI Journey program to other students who want to learn cloud computing through practical activities. The program covers many AWS services and gives students opportunities to build a project that combines cloud infrastructure, security, APIs, data, and AI. However, participants should be proactive in reviewing documentation, recording their progress, and practicing independently because the amount of new knowledge is relatively large.

I would like to continue learning AWS after the internship, especially in serverless backend development, cloud security, Infrastructure as Code, monitoring, and CI/CD. I also hope to improve the group project further by integrating all production APIs, strengthening document ownership checks, improving logging and monitoring, and automating deployment.

Overall, the internship gave me a more practical understanding of cloud systems and helped me identify the areas I need to continue developing for a future career in backend and cloud development.