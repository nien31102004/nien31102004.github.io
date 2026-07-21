---
title: "Week 5 Worklog"
date: 2026-05-18
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:

* Understand IAM Users, Groups, Roles, Policies, and authorization request evaluation.
* Practice Assume Role, Switch Role, and Trust Relationships.
* Apply IAM Conditions to restrict access by IP address and time.

### Tasks carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | Review IAM request structure including Principal, Action, Resource, Environment, and policy evaluation. | 18/05/2026 | 18/05/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html> |
| 3 | Create an IAM Group and four IAM Users with EC2, RDS, inherited, and no-permission profiles. | 19/05/2026 | 19/05/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html> |
| 4 | Sign in as each IAM User to validate assigned permissions and AccessDenied scenarios. | 20/05/2026 | 20/05/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_testing-policies.html> |
| 5 | Create an administrative IAM Role; update its Trust Relationship and allow a no-permission user to assume it. | 21/05/2026 | 21/05/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-console.html> |
| 6 | Add an IP-based Condition to Assume Role and test allowed and denied requests. | 22/05/2026 | 22/05/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html> |
| 7 and Sun | Add a time-based Condition, summarize tests, and delete created Users, Groups, Roles, and Policies. | 23/05/2026 | 24/05/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_aws-dates.html> |

### Week 5 Achievements:

* Clearly distinguished IAM Users, Groups, Roles, identity policies, and trust policies.
* Successfully switched roles and used temporary credentials issued by AWS STS.
* Validated IP- and time-based Conditions for restricting Assume Role.
* Strengthened understanding of least-privilege access control.

### End-of-week Evaluation:

* The main objectives of the week were completed as planned.
* The activities strengthened my theoretical knowledge and practical AWS implementation skills.
* Issues encountered during the work were documented and resolved through AWS documentation and discussions with the mentor and team members.
* I reviewed the created resources and consolidated the necessary material for the internship report.
