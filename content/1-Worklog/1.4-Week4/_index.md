---
title: "Week 4 Worklog"
date: 2026-05-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

* Deploy an EBS Provisioned IOPS SSD `io2` volume and attach it to EC2.
* Format and mount the volume, then share data internally through NFS.
* Validate read/write operations between EC2 instances in the same VPC.

### Tasks carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 2 | Create two EC2 instances as NFS server and client; configure Security Groups for SSH and NFS port 2049. | 11/05/2026 | 11/05/2026 | <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html> |
| 3 | Create a 10-GiB `io2` EBS volume with 1000 IOPS in the same Availability Zone as the server instance. | 12/05/2026 | 12/05/2026 | <https://docs.aws.amazon.com/ebs/latest/userguide/provisioned-iops.html> |
| 4 | Attach the volume to the EC2 server; inspect it with `lsblk`, format as `ext4`, and mount at `/mnt/shared`. | 13/05/2026 | 13/05/2026 | <https://docs.aws.amazon.com/ebs/latest/userguide/ebs-using-volumes.html> |
| 5 | Install NFS, configure `/etc/exports`, start the service, and grant internal access. | 14/05/2026 | 14/05/2026 | <https://docs.aws.amazon.com/efs/latest/ug/mounting-fs-old.html> |
| 6 | Mount the shared directory on the EC2 client; create, edit, and read files to verify data sharing. | 15/05/2026 | 15/05/2026 | <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volumes-multi.html> |
| 7 và CN | Unmount the file system, remove NFS configuration, detach and delete the EBS volume and EC2 instances. | 16/05/2026 | 17/05/2026 | <https://docs.aws.amazon.com/ebs/latest/userguide/ebs-deleting-volume.html> |

### Week 4 Achievements:

* Successfully deployed an `io2` volume with provisioned IOPS.
* Attached, formatted, and mounted the volume on the EC2 server.
* Configured NFS so the client could access and write data through the private VPC network.
* Understood EBS Availability Zone constraints and proper cleanup procedures.

### End-of-week Evaluation:

* The main objectives of the week were completed as planned.
* The activities strengthened my theoretical knowledge and practical AWS implementation skills.
* Issues encountered during the work were documented and resolved through AWS documentation and discussions with the mentor and team members.
* I reviewed the created resources and consolidated the necessary material for the internship report.
