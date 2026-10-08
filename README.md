# AWS Cloud Support & Troubleshooting Lab

A hands-on cloud engineering and technical support lab focused on building, operating, monitoring, troubleshooting, and documenting AWS infrastructure.

The goal of this project is not simply to deploy AWS resources, but to develop practical cloud support engineering skills through controlled infrastructure changes, realistic incidents, evidence-based troubleshooting, root-cause analysis, recovery, and technical documentation.

---

## 1. Purpose

This lab is designed to provide practical experience with:

- AWS infrastructure
- Cloud support engineering
- Linux system administration
- Networking and connectivity troubleshooting
- IAM and access control
- Security groups and network security
- HTTP/HTTPS troubleshooting
- DNS and network diagnosis
- Storage and disk management
- CloudWatch monitoring and logs
- S3 access and permissions
- Bash and Python troubleshooting utilities
- Incident investigation and root-cause analysis
- Technical runbooks and operational documentation

The project is also intended to serve as a practical portfolio project demonstrating the ability to investigate and resolve cloud infrastructure problems systematically.

---

# 2. Lab Philosophy

The lab follows a simple principle:

> **Build it → Verify it → Break it → Investigate it → Fix it → Verify recovery → Document it**

The objective is to develop the ability to reason from symptoms and evidence rather than simply following predefined troubleshooting commands.

Every significant infrastructure change should therefore have:

1. A documented purpose
2. A known expected state
3. A verification method
4. A rollback or recovery procedure where applicable
5. Appropriate documentation

---

# 3. Incident Investigation Method

All incidents in this lab follow the same investigation lifecycle:

```text
Symptom
   ↓
Impact
   ↓
Investigation
   ↓
Evidence
   ↓
Hypothesis
   ↓
Root Cause
   ↓
Resolution
   ↓
Verification
   ↓
Prevention
   ↓
Documentation
```

The goal is to avoid guessing.

Each incident should answer:

- What happened?
- What was affected?
- How was the problem detected?
- What evidence was collected?
- What hypotheses were considered?
- What was the actual root cause?
- What fixed the problem?
- How was recovery verified?
- How could the incident be prevented or detected earlier?

---

# 4. Lab Lifecycle

The project follows this lifecycle:

```text
Design
  ↓
Build
  ↓
Verify
  ↓
Document
  ↓
Monitor
  ↓
Introduce Controlled Failure
  ↓
Investigate
  ↓
Resolve
  ↓
Verify Recovery
  ↓
Document Prevention
```

Controlled failures are introduced deliberately and only when the healthy baseline has already been documented.

This allows the troubleshooting process to be compared against a known-good state.

---

# 5. Current AWS Environment

## AWS Account

The lab is running in AWS in the:

```text
Region: eu-west-1
Location: Europe (Ireland)
```

The lab uses a dedicated IAM identity:

```text
IAM User: george-lab
IAM Group: CloudSupportLab-Admins
```

The root account is not used for normal laboratory operations.

Root access is reserved for account-level tasks that require root privileges, such as initial account security and billing configuration.

---

# 6. Phase 1 — AWS Environment Foundation

## Status: Completed

The AWS environment foundation has been established and verified.

### Completed

- AWS CLI v2 installed
- AWS CLI configured
- AWS region configured
- Root MFA enabled
- Dedicated IAM user created
- Dedicated IAM group created
- IAM permissions configured
- AWS CLI authentication verified
- STS identity verified
- IAM API access verified
- S3 API access verified
- AWS Free Plan reviewed
- Monthly AWS budget configured
- Billing email verification completed

### Identity Verification

The AWS CLI successfully authenticates as:

```text
arn:aws:iam::510735888710:user/george-lab
```

The environment was also verified using:

```bash
aws sts get-caller-identity
```

---

# 7. Phase 2 — Core Infrastructure

## Status: Completed

The initial AWS network and compute environment has been deployed.

## Architecture

The current environment consists of:

```text
                         Internet
                            │
                            ▼
                   Internet Gateway
                            │
                            ▼
                    Public Route Table
                            │
                            ▼
                    Public Subnet
                    10.0.1.0/24
                            │
                            ▼
                    Security Group
                            │
                            ▼
                       EC2 Instance
                       t3.micro
                            │
             ┌──────────────┴──────────────┐
             ▼                             ▼
          EBS Volume                    Nginx
           8 GiB                        HTTP :80
             │
             ▼
        Linux Filesystem
```

Additional AWS services are managed separately from the EC2 instance:

```text
                    ┌──────────────┐
                    │     IAM      │
                    └──────────────┘

                    ┌──────────────┐
                    │      S3      │
                    └──────────────┘

                    ┌──────────────┐
                    │  CloudWatch  │
                    └──────────────┘
```

S3 and CloudWatch are planned operational components and are not currently required for the baseline web server.

---

# 8. Network Infrastructure

## VPC

```text
Name: CloudSupportLab-VPC
VPC ID: vpc-0699284c3650117f8
CIDR: 10.0.0.0/16
Region: eu-west-1
```

DNS support and DNS hostnames are enabled.

## Public Subnet

```text
Name: CloudSupportLab-PublicSubnet
Subnet ID: subnet-0bdb058d667106382
CIDR: 10.0.1.0/24
Availability Zone: eu-west-1b
```

## Internet Gateway

```text
Name: CloudSupportLab-IGW
ID: igw-0a71429466864a9a9
```

The Internet Gateway is attached to the lab VPC.

## Route Table

```text
Name: CloudSupportLab-PublicRT
Route Table ID: rtb-00aa4b2c76999be06
```

Routes:

```text
10.0.0.0/16 → local
0.0.0.0/0   → Internet Gateway
```

The public subnet is associated with this route table.

---

# 9. Security Group

```text
Name: CloudSupportLab-EC2-SG
Security Group ID: sg-01ca6aea1a6b7140c
```

Current inbound rules:

| Protocol | Port | Source | Purpose |
|---|---:|---|---|
| TCP | 22 | Lab client public IP `/32` | SSH administration |
| TCP | 80 | `0.0.0.0/0` | HTTP access |

Outbound traffic currently uses the default IPv4 allow rule.

### Security Consideration

The SSH rule is restricted to the administrator's public IP rather than allowing SSH from the entire internet.

Because public IP addresses can change, an SSH connectivity failure caused by an outdated security-group rule is a potential future incident scenario.

---

# 10. EC2 Instance

## Instance

```text
Instance ID: i-06d61b2da5b1915a6
Instance Type: t3.micro
AMI: ami-087fc0fcee2c7570a
Private IP: 10.0.1.145
```

The instance is running Amazon Linux 2023.

Current host:

```text
ip-10-0-1-145.eu-west-1.compute.internal
```

The EC2 instance is located in the public subnet and has internet connectivity through the Internet Gateway.

---

# 11. SSH Access

The instance is accessed using the AWS key pair:

```text
CloudSupportLab-Key
```

The corresponding private key is stored locally and is not committed to this repository.

SSH access is performed using:

```bash
ssh -i ~/.ssh/CloudSupportLab-Key.pem ec2-user@<PUBLIC_IP>
```

The public IP may change after stopping and starting the instance because the instance does not currently use an Elastic IP.

---

# 12. EBS Storage

The EC2 instance uses an EBS root volume:

```text
Volume ID: vol-0a40a3c767cafdb53
Device: /dev/xvda
Size: 8 GiB
Type: gp3
Encrypted: false
Delete on Termination: true
```

Inside Linux, the volume is exposed through NVMe:

```text
/dev/nvme0n1p1
```

Current baseline disk usage was approximately:

```text
Filesystem       Size  Used  Avail  Use%
/dev/nvme0n1p1   8.0G  1.6G  6.4G   20%
```

Disk exhaustion will later be used as a controlled troubleshooting scenario.

---

# 13. Linux Baseline

The EC2 instance runs:

```text
Amazon Linux 2023
Version: 2023.12.20260930
Kernel: 6.18.51-120.163.amzn2023.x86_64
```

Baseline system checks include:

```bash
hostname
uname -a
cat /etc/os-release
uptime
df -h
free -h
ip addr
ip route
ss -tuln
```

These commands establish the initial state of:

- Operating system
- Kernel
- Host identity
- Uptime
- Memory
- Disk
- Network interfaces
- Routing
- Listening services

---

# 14. Nginx Web Server Baseline

Nginx has been installed on the EC2 instance.

Version:

```text
nginx/1.30.5
```

The service is enabled and running:

```bash
sudo systemctl enable --now nginx
```

Service status:

```bash
sudo systemctl status nginx
```

Configuration validation:

```bash
sudo nginx -t
```

The configuration test succeeds.

Nginx is listening on HTTP port 80:

```text
0.0.0.0:80
[::]:80
```

---

# 15. Known-Good HTTP Baseline

Before introducing incidents, the web server was verified from both the EC2 instance and the external client.

Local validation:

```bash
curl -I http://localhost
```

Expected response:

```text
HTTP/1.1 200 OK
Server: nginx/1.30.5
```

External validation:

```bash
curl -I http://<PUBLIC_IP>
```

The external request successfully returned:

```text
HTTP/1.1 200 OK
Server: nginx/1.30.5
```

This establishes the known-good path:

```text
Client
  ↓
Internet
  ↓
Internet Gateway
  ↓
VPC
  ↓
Public Subnet
  ↓
Security Group
  ↓
EC2
  ↓
Nginx
  ↓
HTTP 200
```

This baseline is important because future incidents can be compared against a verified healthy state.

---

# 16. Monitoring Baseline

## Current Status

Basic operating-system and service-level verification is currently being performed manually.

Current baseline checks include:

```bash
systemctl status nginx
ss -tuln
df -h
free -h
ip addr
ip route
curl -I http://localhost
```

CloudWatch monitoring is planned for a later phase.

Planned monitoring areas include:

- EC2 CPU utilization
- Instance status checks
- Disk usage
- Application/service health
- System logs
- Network-related metrics
- Incident detection

---

# 17. Planned Support Incidents

The lab will introduce controlled failures and investigate them using evidence.

Planned scenarios include:

### INC-001 — Web Server Unreachable

Potential causes:

- Nginx stopped
- Nginx configuration failure
- Port 80 unavailable
- Security-group misconfiguration
- Network routing problem

---

### INC-002 — SSH Connection Failure

Potential causes:

- Incorrect security-group rule
- Client public IP changed
- SSH service unavailable
- Incorrect key or permissions
- Network connectivity issue

---

### INC-003 — HTTP Service Failure

Potential causes:

- Nginx stopped
- Nginx configuration error
- Application failure
- Port conflict
- Incorrect service configuration

---

### INC-004 — Security Group Misconfiguration

Introduce a controlled security-group change and investigate why connectivity fails.

---

### INC-005 — IAM Permission Failure

Create a controlled IAM permission problem and investigate:

- Identity
- Policy
- Resource
- Action
- Access denied response

---

### INC-006 — High CPU Utilization

Generate controlled CPU load and investigate:

- CPU utilization
- Running processes
- Process resource consumption
- System behavior
- Recovery

---

### INC-007 — Disk Exhaustion

Create controlled disk pressure and investigate:

- Filesystem usage
- Large files
- Processes
- Logs
- Available space
- Recovery

---

### INC-008 — S3 Access Failure

Introduce a controlled S3 permission or configuration issue and investigate the resulting access failure.

---

### INC-009 — DNS / Network Failure

Investigate failures involving:

- DNS resolution
- Routing
- Connectivity
- Network configuration

---

### INC-010 — Application / Service Failure

Deploy a simple application or service and introduce a controlled failure requiring service-level troubleshooting.

---

# 18. Incident Documentation Standard

Each incident should eventually be documented using a structure similar to:

```text
Incident ID
Title
Date
Severity
Status

1. Summary
2. Impact
3. Detection
4. Known-Good State
5. Symptoms
6. Investigation
7. Evidence
8. Hypotheses
9. Root Cause
10. Resolution
11. Verification
12. Prevention
13. Lessons Learned
14. Commands / Evidence
```

The purpose is to demonstrate the complete troubleshooting process rather than only documenting the final fix.

---

# 19. Cost Management

AWS resources must be created deliberately.

Every resource should have:

- A documented purpose
- A reason for existing
- A verification procedure
- A teardown or cleanup procedure where applicable

Potential cost-generating resources must be treated carefully.

Examples include:

- EC2
- EBS
- Elastic IPs
- NAT Gateways
- Load Balancers
- RDS
- Other continuously running services

The lab currently uses a simple architecture specifically to minimize unnecessary AWS costs.

A monthly AWS budget has been configured:

```text
Budget: CloudSupportLab-Monthly
Limit: $5/month
```

The EC2 instance should be stopped when active experimentation is complete and it is no longer required.

---

# 20. Repository Structure

```text
aws-cloud-support-lab/
│
├── README.md
│
├── architecture/
│   └── README.md
│
├── incidents/
│   └── README.md
│
├── infrastructure/
│   └── README.md
│
├── monitoring/
│   └── README.md
│
├── runbooks/
│   └── README.md
│
└── scripts/
    └── README.md
```

## Directory Responsibilities

### `architecture/`

Contains:

- Architecture decisions
- Network topology
- System relationships
- Architecture diagrams
- Design assumptions

### `infrastructure/`

Contains:

- AWS resource inventory
- Resource IDs
- Configuration details
- Deployment notes
- Infrastructure verification

### `incidents/`

Contains:

- Incident reports
- Investigation evidence
- Root-cause analysis
- Resolution steps
- Prevention actions

### `monitoring/`

Contains:

- Monitoring configuration
- Metrics
- Logs
- Health checks
- Alerting documentation

### `runbooks/`

Contains repeatable operational procedures for common support tasks.

### `scripts/`

Contains troubleshooting and operational automation scripts.

---

# 21. Engineering Principles

The lab follows these principles:

### 1. Evidence before assumptions

Do not immediately assume the cause of an incident.

Collect evidence first.

### 2. Understand before fixing

The objective is not merely to restore service.

The objective is to understand why the failure occurred.

### 3. Least privilege

Use the minimum permissions required for a task whenever practical.

### 4. Reproducibility

Commands and procedures should be documented well enough to reproduce the environment or troubleshooting process.

### 5. Controlled failure

Failures should be introduced deliberately and safely.

### 6. Known-good baselines

A healthy baseline must exist before introducing an incident.

### 7. Verification after recovery

A service is not considered recovered simply because a command succeeds.

Recovery must be verified from the relevant user or system perspective.

### 8. Document the lesson

Every significant incident should produce knowledge that improves future operations.

### 9. Cost awareness

Infrastructure should exist because it serves a learning or engineering objective.

### 10. Continuous improvement

The lab should evolve as new troubleshooting and cloud-support concepts are learned.

---

# 22. Current Phase

## Phase 1 — AWS Environment Foundation

**Status: COMPLETE**

---

## Phase 2 — Core Infrastructure

**Status: COMPLETE**

Completed:

- VPC
- Public subnet
- Internet Gateway
- Route table
- Security group
- EC2
- EBS
- SSH access
- Linux baseline
- Nginx
- HTTP verification
- External connectivity verification
- Cost budget

---

## Phase 3 — Incident Simulation & Troubleshooting

**Status: NEXT**

The first controlled incident will be:

```text
INC-001 — Web Server Unreachable
```

The incident will be introduced only after the current healthy baseline has been documented.

The investigation will follow:

```text
Symptom
→ Impact
→ Investigation
→ Evidence
→ Hypothesis
→ Root Cause
→ Resolution
→ Verification
→ Prevention
→ Documentation
```

---

# 23. Immediate Next Task

Create and document:

```text
INC-001 — Web Server Unreachable
```

Before introducing the failure:

1. Capture the known-good Nginx state.
2. Capture the current listening ports.
3. Verify local HTTP access.
4. Verify external HTTP access.
5. Record the baseline.
6. Introduce one controlled failure.
7. Investigate without assuming the cause.
8. Collect evidence.
9. Identify the root cause.
10. Restore the service.
11. Verify recovery.
12. Document the incident and prevention.

---

# 24. Current Lab Identity

```text
Project:
AWS Cloud Support & Troubleshooting Lab

Purpose:
Cloud Support Engineering / AWS / Linux / Networking / Troubleshooting

AWS Region:
eu-west-1

Primary Compute:
Amazon EC2 t3.micro

Operating System:
Amazon Linux 2023

Web Server:
Nginx

Primary Network:
10.0.0.0/16

Public Subnet:
10.0.1.0/24

Primary Focus:
Evidence-driven troubleshooting and incident response
```

---

# 25. Long-Term Objective

The long-term objective is to transform this repository from a simple AWS deployment into a practical cloud support engineering portfolio.

The final lab should demonstrate the ability to:

- Build AWS infrastructure
- Understand how the infrastructure works
- Operate Linux systems
- Troubleshoot network connectivity
- Diagnose application and service failures
- Investigate IAM problems
- Work with storage
- Monitor infrastructure
- Analyze logs and metrics
- Write technical runbooks
- Perform root-cause analysis
- Document incidents professionally
- Automate repetitive troubleshooting tasks
- Communicate technical findings clearly

The end result should demonstrate not only that infrastructure can be deployed, but that it can be **operated, supported, investigated, recovered, and continuously improved.**
