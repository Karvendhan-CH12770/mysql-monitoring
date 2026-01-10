# Alerts & SLA/SLO Tracking Dashboard
## Enterprise Monitoring & Reliability Guide

### 🎯 Dashboard Overview

**Dashboard Name:** Alerts & SLA/SLO Tracking  
**UID:** enterprise-alerts-sla-slo  
**Panels:** 28 comprehensive monitoring panels  
**Sections:** 7 major monitoring areas  
**Filters:** Team, Environment, Severity (all multi-select)

**Purpose:** Real-time alert monitoring, SLO compliance tracking, and error budget management

---

## 📊 Dashboard Structure

### **Section 1: Alert Overview (6 KPI Stats)**

1. **Active Alerts (All)** - Total active alerts across all severities
2. **Critical Alerts** - Number of critical severity alerts
3. **Warning Alerts** - Number of warning severity alerts
4. **Alerts Last Hour** - Alert volume in past hour
5. **Alerts Last 24h** - Alert volume in past 24 hours
6. **Alert Rate (Last Hour)** - Alerts per minute firing rate

---

### **Section 2: Active Alerts & History (2 Panels)**

7. **Active Alerts Table** - Real-time alert details with severity color-coding
8. **Alert Timeline (Last 24h)** - Visual timeline of alert firing patterns

---

### **Section 3: Alert Breakdown & Analysis (5 Panels)**

9. **Alerts by Team** - Alert distribution across teams
10. **Alerts by Severity** - Critical/Warning/Info breakdown
11. **Alerts by Environment** - Production/Staging/Dev alert patterns
12. **Top Firing Alerts (Last 24h)** - Most frequently triggered alerts
13. **Alert Firing Rate** - Rate of alerts being triggered

---

### **Section 4: SLO/SLI Tracking (6 Stats)**

14. **Overall Availability SLO** - System-wide availability percentage
15. **Error Budget Remaining** - Percentage of error budget left
16. **P99 Latency SLO** - 99th percentile response time
17. **Success Rate (Last Hour)** - Request success percentage
18. **Database SLO (Uptime)** - Database availability percentage
19. **Infrastructure SLO** - Server uptime percentage

---

### **Section 5: SLO Detailed Metrics (4 Panels)**

20. **Availability SLO Trend (30 days)** - Long-term availability tracking
21. **Error Budget Burn Rate** - Rate of error budget consumption
22. **P50/P90/P99 Latency SLI** - Latency percentile tracking
23. **Request Success Rate by Team** - Team-specific success rates

---

### **Section 6: Service Level Indicators (1 Table)**

24. **SLI Summary by Service** - Comprehensive table showing:
    - Availability %
    - P99 Latency (ms)
    - Request Rate (RPS)

---

### **Section 7: Error Budget Management (4 Panels)**

25. **Error Budget Remaining (30 days)** - Error budget trend
26. **Error Budget Consumption Rate** - Rate of budget usage
27. **SLO Compliance Over Time** - Historical SLO compliance
28. **Error Budget by Team** - Team-specific error budget tracking

---

## ⚙️ Template Variables (Multi-Select)

### **1. Team Filter**
```
Type: Multi-select dropdown
Source: label_values(up, team)
Default: All
Example Values: platform, backend, frontend, data-engineering
```

### **2. Environment Filter**
```
Type: Multi-select dropdown  
Source: label_values(up{team=~"$team"}, environment)
Default: All
Cascades from: Team
Example Values: production, staging, development
```

### **3. Severity Filter**
```
Type: Multi-select dropdown (custom)
Options: All, critical, warning, info
Default: All
Use Case: Filter alerts by severity level
```

---

## 📋 Required Prometheus Metrics

### **Alert Metrics:**

```promql
# Active Alerts
ALERTS{team, environment, severity, alertname, alertstate}

# Alertmanager Metrics
alertmanager_alerts_received_total{team, environment}
alertmanager_alerts_invalid_total{team, environment}
```

### **SLO/SLI Metrics:**

```promql
# HTTP Request Metrics
http_requests_total{team, environment, application, status}
http_request_duration_seconds_bucket{team, environment, application, le}

# Service Uptime
up{job, team, environment, application}

# Database Uptime
up{job="mysql", team, environment, database}

# Windows Infrastructure Uptime
up{job="windows", team, environment, ip}
```

---

## 🔧 Prometheus Alert Rules Configuration

### **Create Alert Rules File:**

```yaml
# /etc/prometheus/rules/alerts.yml

groups:
  - name: application_alerts
    interval: 30s
    rules:
      # High Error Rate
      - alert: HighErrorRate
        expr: |
          (sum(rate(http_requests_total{status=~"5.."}[5m])) by (application, team, environment) / 
           sum(rate(http_requests_total[5m])) by (application, team, environment)) * 100 > 5
        for: 5m
        labels:
          severity: critical
          team: "{{ $labels.team }}"
          environment: "{{ $labels.environment }}"
        annotations:
          summary: "High Error Rate on {{ $labels.application }}"
          description: "Error rate is {{ $value | humanize }}% on {{ $labels.application }}"

      # High Latency
      - alert: HighLatency
        expr: |
          histogram_quantile(0.99, 
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le, application, team, environment)
          ) > 1
        for: 5m
        labels:
          severity: warning
          team: "{{ $labels.team }}"
          environment: "{{ $labels.environment }}"
        annotations:
          summary: "High P99 Latency on {{ $labels.application }}"
          description: "P99 latency is {{ $value }}s on {{ $labels.application }}"

      # Service Down
      - alert: ServiceDown
        expr: up{job!~"prometheus|alertmanager"} == 0
        for: 1m
        labels:
          severity: critical
          team: "{{ $labels.team }}"
          environment: "{{ $labels.environment }}"
        annotations:
          summary: "Service Down: {{ $labels.job }}"
          description: "{{ $labels.job }} on {{ $labels.instance }} is down"

  - name: database_alerts
    interval: 30s
    rules:
      # MySQL Down
      - alert: MySQLDatabaseDown
        expr: mysql_up == 0
        for: 1m
        labels:
          severity: critical
          team: "{{ $labels.team }}"
          environment: "{{ $labels.environment }}"
        annotations:
          summary: "MySQL Database Down"
          description: "MySQL {{ $labels.database }} on {{ $labels.ip }} is down"

      # High Connection Usage
      - alert: MySQLHighConnectionUsage
        expr: |
          (mysql_global_status_threads_connected / 
           mysql_global_variables_max_connections) * 100 > 90
        for: 5m
        labels:
          severity: critical
          team: "{{ $labels.team }}"
          environment: "{{ $labels.environment }}"
        annotations:
          summary: "MySQL High Connection Usage"
          description: "Connection usage is {{ $value | humanize }}% on {{ $labels.database }}"

  - name: infrastructure_alerts
    interval: 30s
    rules:
      # High CPU
      - alert: WindowsHighCPU
        expr: |
          100 - (avg by (ip, team, environment) 
            (rate(windows_cpu_time_total{mode="idle"}[5m])) * 100) > 90
        for: 5m
        labels:
          severity: critical
          team: "{{ $labels.team }}"
          environment: "{{ $labels.environment }}"
        annotations:
          summary: "Windows High CPU Usage"
          description: "CPU usage is {{ $value | humanize }}% on {{ $labels.ip }}"

      # High Memory
      - alert: WindowsHighMemory
        expr: |
          (1 - (windows_os_physical_memory_free_bytes / 
                windows_cs_physical_memory_bytes)) * 100 > 95
        for: 3m
        labels:
          severity: critical
          team: "{{ $labels.team }}"
          environment: "{{ $labels.environment }}"
        annotations:
          summary: "Windows High Memory Usage"
          description: "Memory usage is {{ $value | humanize }}% on {{ $labels.ip }}"

  - name: slo_alerts
    interval: 1m
    rules:
      # SLO Violation - Availability
      - alert: SLOViolationAvailability
        expr: |
          (1 - (sum(rate(http_requests_total{status=~"5.."}[1h])) / 
                sum(rate(http_requests_total[1h])))) * 100 < 99.9
        for: 5m
        labels:
          severity: critical
          team: "platform"
        annotations:
          summary: "SLO Violation: Availability below 99.9%"
          description: "Current availability is {{ $value | humanize }}%"

      # Error Budget Exhausted
      - alert: ErrorBudgetExhausted
        expr: |
          ((0.001 * sum(rate(http_requests_total[30d])) * 2592000) - 
           sum(increase(http_requests_total{status=~"5.."}[30d]))) / 
          (0.001 * sum(rate(http_requests_total[30d])) * 2592000) * 100 < 0
        for: 1m
        labels:
          severity: critical
          team: "platform"
        annotations:
          summary: "Error Budget Exhausted for 30-day window"
          description: "Error budget is {{ $value | humanize }}%"

      # High Error Budget Burn Rate
      - alert: HighErrorBudgetBurnRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[1h])) / 
          (0.001 * sum(rate(http_requests_total[1h]))) > 5
        for: 15m
        labels:
          severity: warning
          team: "platform"
        annotations:
          summary: "High Error Budget Burn Rate"
          description: "Burn rate is {{ $value }}x the allowed rate"
```

### **Update prometheus.yml:**

```yaml
# prometheus.yml

global:
  scrape_interval: 15s
  evaluation_interval: 30s

rule_files:
  - '/etc/prometheus/rules/alerts.yml'

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']
```

---

## 🎨 Key Design Features

### **1. Color-Coded Alert Severities**

**Active Alerts (All):**
- Green: 0 alerts
- Yellow: 1-4 alerts
- Orange: 5-9 alerts
- Red: 10+ alerts

**Critical Alerts:**
- Green: 0 alerts
- Red: 1+ alerts

**Warning Alerts:**
- Green: 0 alerts
- Yellow: 1-4 alerts
- Orange: 5+ alerts

### **2. SLO Thresholds**

**Availability SLO:**
- Red: < 99%
- Orange: 99-99.5%
- Yellow: 99.5-99.9%
- Green: > 99.9%

**Error Budget:**
- Red: < 0% (exhausted)
- Orange: 0-10%
- Yellow: 10-30%
- Green: > 30%

**P99 Latency:**
- Green: < 200ms
- Yellow: 200-500ms
- Orange: 500-1000ms
- Red: > 1000ms

### **3. Active Alerts Table**

- **Color-coded Severity:** Red (critical), Yellow (warning), Blue (info)
- **Color-coded State:** Red (firing), Orange (pending)
- **Sortable:** Click column headers
- **Real-time:** Auto-refresh every 30 seconds

---

## 📈 SLO Calculations Explained

### **1. Availability SLO (99.9% Target):**

```promql
(1 - (sum(rate(http_requests_total{status=~"5.."}[30d])) / 
      sum(rate(http_requests_total[30d])))) * 100
```

**Interpretation:**
- 99.9% = 43.2 minutes downtime per month allowed
- 99.95% = 21.6 minutes per month
- 99.99% = 4.32 minutes per month

### **2. Error Budget Remaining:**

```promql
((0.001 * sum(rate(http_requests_total[30d])) * 2592000) - 
 sum(increase(http_requests_total{status=~"5.."}[30d]))) / 
(0.001 * sum(rate(http_requests_total[30d])) * 2592000) * 100
```

**Calculation:**
- Total Budget = 0.1% of requests (for 99.9% SLO)
- Used Budget = Actual 5xx errors
- Remaining = (Total - Used) / Total * 100

### **3. Error Budget Burn Rate:**

```promql
sum(rate(http_requests_total{status=~"5.."}[1h])) / 
(0.001 * sum(rate(http_requests_total[1h])))
```

**Interpretation:**
- 1.0 = Consuming budget at expected rate
- > 1.0 = Burning budget faster than allowed
- > 5.0 = Critical - will exhaust budget in < 6 days

### **4. P99 Latency SLI:**

```promql
histogram_quantile(0.99, 
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
) * 1000
```

**SLO Targets:**
- API endpoints: < 200ms
- Web pages: < 500ms
- Batch operations: < 2000ms

---

## 🚀 Usage Examples

### **1. Monitor Active Alerts:**
```
Filters:
  Team: All
  Environment: production
  Severity: critical

View:
  - Active Alerts count
  - Active Alerts Table (see details)
  - Alert Timeline (visualize patterns)
```

### **2. Check SLO Compliance:**
```
Filters:
  Team: backend
  Environment: production
  Severity: All

View:
  - Overall Availability SLO
  - Error Budget Remaining
  - SLO Compliance Over Time
```

### **3. Investigate Error Budget:**
```
Filters:
  Team: platform
  Environment: production
  Severity: All

View:
  - Error Budget Remaining (30 days)
  - Error Budget Burn Rate
  - Error Budget Consumption Rate
```

### **4. Team-Specific Alerts:**
```
Filters:
  Team: backend
  Environment: All
  Severity: critical, warning

View:
  - Alerts by Team
  - Top Firing Alerts
  - Alert Firing Rate
```

---

## 🎯 SLO Best Practices

### **1. Define Clear SLOs:**

```yaml
# Example SLO Definitions

Service: Web API
  - Availability: 99.9% (monthly)
  - Latency P99: < 200ms
  - Latency P50: < 100ms
  
Service: Database
  - Availability: 99.95% (monthly)
  - Query Latency P99: < 50ms
  
Service: Infrastructure
  - Server Uptime: 99.9% (monthly)
  - CPU Availability: < 90% utilization
```

### **2. Error Budget Policy:**

| Error Budget Remaining | Action |
|------------------------|--------|
| > 50% | Deploy freely, innovate |
| 30-50% | Normal operations |
| 10-30% | Reduce deployment frequency |
| < 10% | Feature freeze, focus on reliability |
| < 0% | Emergency freeze, all hands on deck |

### **3. Alert Escalation:**

```yaml
Severity: critical
  - Immediate page on-call
  - Auto-escalate after 15 minutes
  - Notify team lead after 30 minutes

Severity: warning
  - Slack notification
  - Escalate if 3+ warnings in 1 hour
  - Review in next day's standup

Severity: info
  - Log only
  - Weekly review
```

---

## 📊 Monitoring Metrics Summary

| Metric Type | Count | Purpose |
|-------------|-------|---------|
| Alert Stats | 6 | Active alert counts and rates |
| Alert Tables | 2 | Detailed alert information |
| Alert Charts | 5 | Alert trends and breakdowns |
| SLO Stats | 6 | Key SLO indicators |
| SLO Charts | 4 | SLO trend analysis |
| SLI Table | 1 | Service-level indicators |
| Error Budget | 4 | Budget tracking and burn rate |
| **Total** | **28** | **Complete monitoring coverage** |

---

## ✅ Import Checklist

```
□ Import alerts_sla_slo_tracking_dashboard.json
□ Select Prometheus data source
□ Configure Prometheus alert rules (/etc/prometheus/rules/alerts.yml)
□ Configure Alertmanager
□ Verify ALERTS metric exists in Prometheus
□ Test alert firing (trigger test alert)
□ Verify Active Alerts Table populates
□ Check all filters work (Team, Environment, Severity)
□ Verify SLO calculations display correctly
□ Set up SLO definitions for your services
□ Configure error budget policies
□ Document SLO targets in team runbooks
□ Train team on error budget management
```

---

## 🔍 Troubleshooting

### **Problem: Active Alerts Table is Empty**

**Solution:**
```bash
# Check if ALERTS metric exists
curl 'http://prometheus:9090/api/v1/query?query=ALERTS' | jq

# Verify alert rules are loaded
curl 'http://prometheus:9090/api/v1/rules' | jq

# Check Prometheus logs
docker logs prometheus 2>&1 | grep -i alert
```

### **Problem: SLO Metrics Show No Data**

**Solution:**
```bash
# Verify http_requests_total metric exists
curl 'http://prometheus:9090/api/v1/query?query=http_requests_total' | jq

# Check if histogram buckets exist
curl 'http://prometheus:9090/api/v1/query?query=http_request_duration_seconds_bucket' | jq

# Ensure applications are exporting metrics
curl 'http://your-app:9090/metrics' | grep http_request
```

---

## 🎯 Key Formulas Reference

### **Availability Calculation:**
```
Availability = (1 - (Errors / Total Requests)) * 100
Target: 99.9% = 43.2 min downtime/month
```

### **Error Budget:**
```
Budget = Total Allowed Errors - Actual Errors
Remaining % = (Budget / Total Allowed) * 100
```

### **Burn Rate:**
```
Burn Rate = Current Error Rate / Allowed Error Rate
Burn Rate > 1 = Consuming budget faster than allowed
```

---

**Dashboard Created:** December 2025  
**Version:** 1.0 Enterprise  
**Status:** Production-Ready ✅  
**SLO Framework:** Google SRE Best Practices
