# AWS Cloud Support & Troubleshooting Lab

A hands-on learning and portfolio project focused on cloud support engineering,
AWS infrastructure, Linux, networking, troubleshooting, monitoring, and technical
runbooks.

## Purpose

This repository will document the deliberate build-out of a small AWS environment
and a series of controlled troubleshooting exercises. The goal is to develop
practical evidence of cloud support skills rather than simply listing AWS services.

## Planned Scope

- AWS compute and networking
- Linux administration
- IAM and least-privilege access
- DNS and HTTP/HTTPS troubleshooting
- Security groups and network connectivity
- PostgreSQL/database connectivity
- S3 access and permissions
- CloudWatch monitoring and logs
- Bash/Python troubleshooting utilities
- Incident investigation and root-cause analysis
- Technical runbooks

## Incident Method

Each completed incident will follow:

1. Symptom
2. Impact
3. Investigation
4. Root cause
5. Resolution
6. Verification
7. Prevention / follow-up

## Status

**Phase 1 — AWS environment foundation**

The repository and initial AWS access environment have been established.

### Verified

- AWS CLI v2 installed and operational
- AWS CLI integrated with Git Bash
- AWS region configured as `eu-west-1`
- AWS root account protected with MFA
- Dedicated IAM user `george-lab` created for lab administration
- IAM group `CloudSupportLab-Admins` established for lab permissions
- CLI authentication verified using the dedicated lab identity
- AWS STS identity verification completed
- IAM user API access verified
- Amazon S3 API access verified
- No S3 buckets have been created yet

### Not yet deployed

The following infrastructure remains planned and will be deployed progressively:

- VPC
- Public subnet
- Internet Gateway
- Route table
- Security Group
- EC2 Linux instance
- EBS storage
- S3 bucket
- CloudWatch monitoring

Infrastructure will not be created until cost controls and the deployment design have been documented.

### Current identity

The AWS CLI is authenticated using the dedicated lab IAM identity rather than the AWS root user.

The root account is reserved for account-level administration and protected with MFA.

## Planned Structure

```text
architecture/     Architecture diagrams and design notes
incidents/        Completed troubleshooting incidents
monitoring/       Monitoring and observability notes
runbooks/         Repeatable support procedures
scripts/          Bash/Python support utilities
infrastructure/  Infrastructure configuration and notes