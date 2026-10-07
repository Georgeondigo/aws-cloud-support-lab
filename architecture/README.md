# AWS Cloud Support Lab — Architecture

## Purpose

This lab provides a practical environment for developing AWS Cloud Support
Engineering skills through infrastructure deployment, monitoring,
troubleshooting, incident response, and documentation.

The environment is intentionally designed to allow controlled failures and
repeatable troubleshooting exercises.

---

## Initial Architecture

The initial environment consists of:

- Amazon VPC
- Public subnet
- Internet Gateway
- Route table
- Security Group
- Amazon EC2 Linux instance
- Amazon EBS volume
- Amazon S3 bucket
- AWS IAM
- Amazon CloudWatch

---

## Architecture

```text
                         INTERNET
                            |
                            v
                    +----------------+
                    |      VPC       |
                    |  10.0.0.0/16   |
                    +-------+--------+
                            |
                            v
                    +----------------+
                    | Public Subnet  |
                    |  10.0.1.0/24   |
                    +-------+--------+
                            |
                            v
                    +----------------+
                    |      EC2       |
                    |  Linux Server  |
                    +-------+--------+
                            |
                            v
                    +----------------+
                    |      EBS       |
                    |    Storage     |
                    +----------------+

        +----------------+       +----------------+
        |   CloudWatch   |       |      S3        |
        |   Monitoring   |       | Object Storage |
        +----------------+       +----------------+

                    +----------------+
                    |      IAM       |
                    |  Permissions   |
                    +----------------+

```
## Network Design

### VPC

CIDR:

`10.0.0.0/16`

### Public Subnet

CIDR:

`10.0.1.0/24`

### Internet Gateway

Provides internet connectivity for resources in the public subnet.

### Route Table

The public subnet will use a route to the Internet Gateway for internet
traffic.

## Compute

### EC2

The lab will use a Linux-based EC2 instance as the primary support target.

The instance will be used to practice:

- Linux administration
- SSH access
- service management
- process management
- disk management
- log investigation
- networking diagnostics
- system monitoring
- troubleshooting

## Storage

### EBS

EBS will provide persistent block storage for the EC2 instance.

We will investigate:

- volume attachment
- filesystem usage
- persistence
- disk exhaustion
- storage troubleshooting

### S3

S3 will be used for object storage and AWS service integration exercises.

## Security

Security will be implemented using:

- IAM
- Security Groups
- least-privilege access
- controlled network access

Security Groups will initially be configured to allow only the traffic
required for the lab.

## Monitoring

Amazon CloudWatch will be used to investigate:
- CPU utilization
- instance health
- system/application logs where applicable
- resource behavior
- operational incidents

## Support Engineering Goals

The environment will eventually be used to simulate real support cases,

including:

1. EC2 connectivity failure
2. SSH access failure
3. HTTP service failure
4. Security Group misconfiguration
5. IAM permission failure
6. High CPU utilization
7. Disk space exhaustion
8. S3 access failure
9. DNS/network connectivity problems
10. Application/service failure

Each incident will be documented using:

Symptom → Investigation → Evidence → Root Cause → Resolution → Verification → Prevention

## Design Principles

- Keep infrastructure simple and understandable.
- Prefer least-privilege access.
- Document infrastructure before deployment.
- Make changes reproducible.
- Monitor resources.
- Introduce controlled failures for troubleshooting practice.
- Document every significant incident.
- Avoid unnecessary AWS resources and costs.
