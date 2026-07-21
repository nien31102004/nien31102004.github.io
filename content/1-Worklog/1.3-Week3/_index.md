---
title: "Week 3 Worklog"
date: 2026-05-04
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:

* Build a VPC with public and private subnets across multiple Availability Zones.
* Deploy Amazon RDS for MySQL in private subnets and restrict access with Security Groups.
* Connect EC2 to RDS, create and restore snapshots, and clean up resources.

### Tasks carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | Design VPC CIDR `10.0.0.0/16`; create two public and two private subnets across two Availability Zones. | 04/05/2026 | 04/05/2026 | <https://docs.aws.amazon.com/vpc/latest/userguide/vpc-getting-started.html> |
| 3 | Create an Internet Gateway and public route table; enable automatic public IPv4 assignment for public subnets. | 05/05/2026 | 05/05/2026 | <https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html> |
| 4 | Create Security Groups for EC2 and RDS; allow MySQL port 3306 only from the application Security Group. | 06/05/2026 | 06/05/2026 | <https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.RDSSecurityGroups.html> |
| 5 | Create a DB Subnet Group from private subnets and deploy a non-publicly accessible RDS MySQL instance. | 07/05/2026 | 07/05/2026 | <https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html> |
| 6 | Connect from EC2 to RDS; create a database, tables, and sample data to validate queries. | 08/05/2026 | 08/05/2026 | <https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ConnectToInstance.html> |
| 7 and Sun | Create and restore an RDS snapshot to verify recovery, then clean up resources. | 09/05/2026 | 10/05/2026 | <https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html> |

### Week 3 Achievements:

* Completed a multi-tier VPC with appropriate public/private subnets and routing.
* Deployed RDS MySQL in private networking without direct Internet access.
* Successfully connected EC2 to RDS through Security Group references and executed queries.
* Created and restored snapshots, gaining practical understanding of database backup and recovery.

### End-of-week Evaluation:

* The main objectives of the week were completed as planned.
* The activities strengthened my theoretical knowledge and practical AWS implementation skills.
* Issues encountered during the work were documented and resolved through AWS documentation and discussions with the mentor and team members.
* I reviewed the created resources and consolidated the necessary material for the internship report.
