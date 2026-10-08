# INC-001 — Web Server Unreachable

**Incident ID:** INC-001  
**Title:** Web Server Unreachable  
**Date:** 2026-10-08  
**Severity:** Medium  
**Status:** Resolved  
**Environment:** AWS Cloud Support & Troubleshooting Lab  
**AWS Region:** `eu-west-1`  
**Affected Service:** Nginx HTTP web server on EC2  

---

## 1. Summary

The lab web server became unreachable over HTTP.

External requests to the EC2 public IP on TCP port 80 failed with:

```text
Failed to connect to 108.130.99.59:80
Could not connect to server
```

Investigation showed that the EC2 instance itself remained reachable over SSH, but Nginx was inactive and TCP port 80 was no longer listening.

The Nginx service had been cleanly stopped. The Nginx configuration remained valid.

The service was restored using:

```bash
sudo systemctl start nginx
```

Recovery was verified both locally on the EC2 instance and externally from the Windows client, with HTTP `200 OK` returned.

---

## 2. Impact

### User Impact

The HTTP web service was unavailable.

Users attempting to access:

```text
http://108.130.99.59
```

could not establish a connection.

### Scope

The failure affected the HTTP service on TCP port 80.

SSH access on TCP port 22 remained available, allowing the EC2 instance to be investigated and recovered.

---

## 3. Detection

The incident was detected through an external HTTP connectivity test:

```bash
curl -I --connect-timeout 5 http://108.130.99.59
```

Result:

```text
Failed to connect to 108.130.99.59:80 after 3000 ms: Could not connect to server
```

This established the initial symptom:

> The web server could not be reached externally over HTTP.

---

## 4. Known-Good Baseline

Before introducing the controlled failure, the environment was verified as healthy.

### Nginx

```text
Version: nginx/1.30.5
State: active (running)
```

### Listening Ports

```text
TCP 22 → LISTEN
TCP 80 → LISTEN
```

### Local HTTP

```bash
curl -I http://localhost
```

Returned:

```text
HTTP/1.1 200 OK
Server: nginx/1.30.5
```

### External HTTP

```bash
curl -I http://108.130.99.59
```

Returned:

```text
HTTP/1.1 200 OK
Server: nginx/1.30.5
```

The known-good request path was therefore:

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

---

## 5. Incident Introduction

This incident was intentionally introduced as a controlled failure for the laboratory.

The Nginx service was stopped using:

```bash
sudo systemctl stop nginx
```

The purpose was to simulate a web-server outage while keeping the underlying AWS networking and EC2 instance operational.

---

## 6. Investigation

### Evidence #1 — External HTTP Failure

Test:

```bash
curl -I --connect-timeout 5 http://108.130.99.59
```

Result:

```text
Failed to connect to 108.130.99.59:80 after 3000 ms: Could not connect to server
```

This showed that the external client could not establish an HTTP connection to TCP port 80.

At this point, the cause was not assumed.

---

### Evidence #2 — EC2 and Service-Level Checks

SSH access to the EC2 instance remained available.

Checks showed:

```text
Nginx → inactive/dead
Port 80 → not listening
Port 22 → listening
```

This established that the EC2 host was still accessible and that the failure was specifically affecting the HTTP service.

---

### Evidence #3 — Local HTTP Failure

From inside the EC2 instance:

```bash
curl -I --connect-timeout 5 http://localhost
```

Result:

```text
curl: (7) Failed to connect to localhost:80 after 0 ms: Could not connect to server
```

Service state:

```bash
sudo systemctl is-active nginx
```

Result:

```text
inactive
```

This proved that the HTTP failure existed locally on the EC2 instance.

The problem was therefore not limited to the external network path.

---

### Evidence #4 — Nginx Service History

The service status showed:

```text
Active: inactive (dead)
```

The systemd journal showed:

```text
Stopping nginx.service...
nginx.service: Deactivated successfully.
Stopped nginx.service...
```

The service state also showed:

```text
Result=success
ExecMainStatus=0
ActiveState=inactive
SubState=dead
```

This indicated a clean service shutdown rather than an observed crash or failed execution.

---

### Evidence #5 — Configuration Validation

Nginx was confirmed to be enabled:

```bash
sudo systemctl is-enabled nginx
```

Result:

```text
enabled
```

The service configuration was inspected using:

```bash
sudo systemctl cat nginx
```

The Nginx configuration was then validated:

```bash
sudo nginx -t
```

Result:

```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

This ruled out a current Nginx configuration syntax failure.

---

## 7. Hypotheses

The investigation considered several possible fault domains.

### Hypothesis 1 — External Network Failure

**Status:** Rejected.

SSH access remained available, and the HTTP failure also occurred when testing `localhost` from the EC2 instance.

---

### Hypothesis 2 — Security Group Blocking Port 80

**Status:** Rejected as the immediate cause.

The HTTP request failed locally on the EC2 instance itself, before any external security-group path was relevant.

---

### Hypothesis 3 — Nginx Configuration Failure

**Status:** Rejected.

```bash
sudo nginx -t
```

returned a successful configuration test.

---

### Hypothesis 4 — Nginx Crash

**Status:** No evidence found.

Systemd reported a successful deactivation:

```text
Result=success
ExecMainStatus=0
```

The journal did not show a crash or startup failure.

---

### Hypothesis 5 — Nginx Was Stopped

**Status:** Confirmed.

The service was inactive, port 80 was not listening, systemd recorded a clean shutdown, and the controlled failure procedure had explicitly stopped Nginx.

---

## 8. Root Cause

### Root Cause

**Nginx was manually stopped as the controlled failure used to simulate the incident.**

This caused:

```text
Nginx stopped
    ↓
TCP port 80 stopped listening
    ↓
Local HTTP connection failed
    ↓
External HTTP connection failed
    ↓
Web service became unreachable
```

The underlying EC2 instance, SSH service, VPC routing, and network path remained operational.

---

## 9. Resolution

The Nginx service was restarted:

```bash
sudo systemctl start nginx
```

Systemd then reported:

```text
Active: active (running)
```

Nginx successfully passed its startup configuration test.

---

## 10. Recovery Verification

Recovery was verified at multiple layers.

### Service State

```text
Nginx → active (running)
```

### TCP Port 80

```text
0.0.0.0:80 → LISTEN
[::]:80 → LISTEN
```

### SSH Port

```text
0.0.0.0:22 → LISTEN
[::]:22 → LISTEN
```

### Local HTTP

```bash
curl -I http://localhost
```

Returned:

```text
HTTP/1.1 200 OK
Server: nginx/1.30.5
```

### External HTTP

From the Windows client:

```bash
curl -I http://108.130.99.59
```

Returned:

```text
HTTP/1.1 200 OK
Server: nginx/1.30.5
```

The full request path was therefore restored:

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

---

## 11. Prevention / Follow-Up

For this controlled incident, the primary prevention lesson is service-state awareness.

Recommended operational checks include:

```bash
sudo systemctl status nginx
sudo systemctl is-active nginx
sudo ss -tuln | grep ':80'
curl -I http://localhost
```

Future phases of the lab will introduce monitoring and CloudWatch-based health checks so that service failures can be detected automatically rather than only through manual testing.

Additional future improvements may include:

- HTTP health checks
- CloudWatch metrics
- CloudWatch alarms
- Service monitoring
- Automated recovery procedures
- Runbooks for common Nginx failures

---

## 12. Lessons Learned

### 1. Start from the symptom

The investigation began with the externally reported failure rather than immediately assuming Nginx was the cause.

### 2. Test from multiple locations

Testing both externally and from `localhost` helped separate network-path problems from host/service problems.

### 3. Check the listening port

A service being installed does not mean it is currently accepting connections.

```bash
ss -tuln
```

was important evidence.

### 4. Check service history

`systemctl status` and `journalctl` helped distinguish a clean service stop from a crash or configuration failure.

### 5. Validate configuration before restarting

```bash
sudo nginx -t
```

confirmed that the configuration was healthy before recovery.

### 6. Verify recovery from the user's perspective

A successful `systemctl start nginx` was not considered sufficient.

Recovery was confirmed through:

1. Service state
2. Listening port
3. Local HTTP request
4. External HTTP request

This provided end-to-end verification.

---

## 13. Commands Used

### External connectivity

```bash
curl -I --connect-timeout 5 http://108.130.99.59
```

### SSH

```bash
ssh -i ~/.ssh/CloudSupportLab-Key.pem ec2-user@108.130.99.59
```

### Service status

```bash
sudo systemctl status nginx --no-pager
sudo systemctl is-active nginx
sudo systemctl is-enabled nginx
```

### Service logs

```bash
sudo journalctl -u nginx --since "2 hours ago" --no-pager
```

### Service definition

```bash
sudo systemctl cat nginx
```

### Port inspection

```bash
sudo ss -tuln
sudo ss -tuln | grep ':80'
sudo ss -tuln | grep ':22'
```

### Local HTTP testing

```bash
curl -I --connect-timeout 5 http://localhost
curl -I http://localhost
```

### Configuration validation

```bash
sudo nginx -t
```

### Recovery

```bash
sudo systemctl start nginx
```

---

## 14. Final Status

```text
Incident: INC-001
Status: RESOLVED

Root Cause:
Nginx was manually stopped as a controlled failure.

Resolution:
Nginx was restarted using systemctl.

Local Verification:
HTTP 200 OK

External Verification:
HTTP 200 OK

Service:
Nginx active/running

Port 80:
LISTENING
```

---

## 15. Incident Lifecycle

```text
Symptom
   ↓
External HTTP Failure
   ↓
Impact Identified
   ↓
Server Investigation
   ↓
Local HTTP Failure Confirmed
   ↓
Nginx Service Investigated
   ↓
Service History Checked
   ↓
Configuration Validated
   ↓
Root Cause Identified
   ↓
Nginx Restarted
   ↓
Local Recovery Verified
   ↓
External Recovery Verified
   ↓
Incident Resolved
   ↓
Lessons Documented
```
