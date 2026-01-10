# Complete Alerting Setup Guide
## Email Notifications with Team Routing and Custom Templates

---

## 📋 **Table of Contents**

1. [Overview](#overview)
2. [Directory Structure](#directory-structure)
3. [Gmail SMTP Setup](#gmail-smtp-setup)
4. [Configuration Files](#configuration-files)
5. [Team Email Configuration](#team-email-configuration)
6. [Testing Alerts](#testing-alerts)
7. [Troubleshooting](#troubleshooting)
8. [Advanced Features](#advanced-features)

---

## 🎯 **Overview**

### **What You Get:**

✅ **Team-Based Routing** - Alerts routed to correct team emails
✅ **Severity-Based Templates** - Different email formats for critical/warning/info
✅ **Root Cause Analysis** - Custom templates with troubleshooting guidance
✅ **Email Grouping** - Multiple alerts grouped into single email
✅ **Alert Deduplication** - Prevents spam from repeated alerts
✅ **Inhibition Rules** - Lower severity alerts suppressed when critical fires

---

## 📁 **Directory Structure**

```bash
/etc/
├── prometheus/
│   ├── prometheus.yml                    # Main Prometheus config
│   └── rules/
│       └── alerts.yml                    # Alert rules (creates ALERTS metric)
│
├── alertmanager/
│   ├── alertmanager.yml                  # Alertmanager config (routing & receivers)
│   ├── data/                             # Alertmanager data directory
│   └── templates/
│       ├── email_template_critical.tmpl  # Critical alert template
│       ├── email_template_warning.tmpl   # Warning alert template
│       ├── email_template_info.tmpl      # Info alert template
│       └── email_template_default.tmpl   # Default template
```

---

## 📧 **Gmail SMTP Setup**

### **Step 1: Create Gmail App Password**

```
1. Go to: https://myaccount.google.com/apppasswords
2. Sign in to your Gmail account
3. Click "Generate" under "App passwords"
4. Select "Mail" and "Other (Custom name)"
5. Name it: "Prometheus Alerts"
6. Click "Generate"
7. Copy the 16-character password (e.g., "abcd efgh ijkl mnop")
8. Save this password - you'll use it in alertmanager.yml
```

### **Step 2: Update alertmanager.yml**

```yaml
global:
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'your-email@gmail.com'              # ← Your Gmail address
  smtp_auth_username: 'your-email@gmail.com'     # ← Your Gmail address
  smtp_auth_password: 'abcd efgh ijkl mnop'      # ← App password (remove spaces)
  smtp_require_tls: true
```

### **Alternative SMTP Providers:**

#### **Microsoft 365 / Outlook:**
```yaml
smtp_smarthost: 'smtp.office365.com:587'
smtp_from: 'alerts@yourcompany.com'
smtp_auth_username: 'alerts@yourcompany.com'
smtp_auth_password: 'your-password'
smtp_require_tls: true
```

#### **SendGrid:**
```yaml
smtp_smarthost: 'smtp.sendgrid.net:587'
smtp_from: 'alerts@yourcompany.com'
smtp_auth_username: 'apikey'
smtp_auth_password: 'YOUR_SENDGRID_API_KEY'
smtp_require_tls: true
```

#### **AWS SES:**
```yaml
smtp_smarthost: 'email-smtp.us-east-1.amazonaws.com:587'
smtp_from: 'alerts@yourcompany.com'
smtp_auth_username: 'YOUR_SMTP_USERNAME'
smtp_auth_password: 'YOUR_SMTP_PASSWORD'
smtp_require_tls: true
```

---

## ⚙️ **Configuration Files**

### **File 1: prometheus.yml** (Alert Rules Integration)

```yaml
# /etc/prometheus/prometheus.yml

global:
  scrape_interval: 15s
  evaluation_interval: 30s  # Evaluate alert rules every 30 seconds

# Load alert rules
rule_files:
  - '/etc/prometheus/rules/alerts.yml'

# Configure Alertmanager endpoint
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - 'localhost:9093'  # Alertmanager address
```

### **File 2: alerts.yml** (Alert Rules)

```yaml
# /etc/prometheus/rules/alerts.yml

groups:
  - name: application_alerts
    rules:
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
          description: "Error rate is {{ $value | humanizePercentage }} on {{ $labels.application }}"
```

### **File 3: alertmanager.yml** (Routing & Email Config)

See the complete `alertmanager.yml` file I created.

### **File 4: Email Templates** (Custom HTML)

See the 4 email template files I created.

---

## 👥 **Team Email Configuration**

### **Update Email Addresses in alertmanager.yml:**

```yaml
receivers:
  # Backend Team
  - name: 'backend-critical'
    email_configs:
      - to: 'backend-team@yourcompany.com'        # ← Update this
      - to: 'backend-lead@yourcompany.com'        # ← Update this
        
  # Platform Team  
  - name: 'platform-critical'
    email_configs:
      - to: 'platform-team@yourcompany.com'       # ← Update this
      - to: 'cto@yourcompany.com'                 # ← Update this
      
  # Data Team
  - name: 'data-critical'
    email_configs:
      - to: 'data-engineering@yourcompany.com'    # ← Update this
```

### **Email Routing Logic:**

```yaml
# Alert fired with labels:
{
  alertname: "HighErrorRate",
  team: "backend",
  environment: "production",
  severity: "critical"
}

# Routing flow:
1. Check team: "backend" → Routes to backend-team receiver
2. Check severity: "critical" → Routes to backend-critical receiver
3. Send email to:
   - backend-team@yourcompany.com
   - backend-lead@yourcompany.com
```

---

## 🧪 **Testing Alerts**

### **Step 1: Deploy Configuration**

```bash
# 1. Copy files to correct locations
sudo mkdir -p /etc/alertmanager/templates
sudo cp alertmanager.yml /etc/alertmanager/
sudo cp email_template_*.tmpl /etc/alertmanager/templates/

# 2. Copy alert rules
sudo mkdir -p /etc/prometheus/rules
sudo cp prometheus_alert_rules_sample.yml /etc/prometheus/rules/alerts.yml

# 3. Update prometheus.yml
sudo nano /etc/prometheus/prometheus.yml
# Add rule_files and alerting sections (see above)

# 4. Set correct permissions
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R alertmanager:alertmanager /etc/alertmanager
```

### **Step 2: Restart Services**

```bash
# Reload Prometheus (graceful)
curl -X POST http://localhost:9090/-/reload

# Restart Alertmanager
docker restart alertmanager
# OR
sudo systemctl restart alertmanager

# Verify services are running
curl http://localhost:9090/-/healthy     # Prometheus
curl http://localhost:9093/-/healthy     # Alertmanager
```

### **Step 3: Create Test Alert**

Add this to your alerts.yml:

```yaml
- alert: TestEmailAlert
  expr: vector(1)  # Always fires
  labels:
    severity: warning
    team: backend
    environment: production
  annotations:
    summary: "Test Email Alert"
    description: "This is a test alert to verify email configuration"
```

### **Step 4: Verify Alert Firing**

```bash
# Check Prometheus alerts page
open http://localhost:9090/alerts

# Check Alertmanager
open http://localhost:9093/#/alerts

# You should see TestEmailAlert in FIRING state
```

### **Step 5: Check Email**

```
1. Wait 30 seconds for Alertmanager to group and send
2. Check inbox: backend-team@yourcompany.com
3. You should receive email with subject: "⚠️ [WARNING] Backend Alert - TestEmailAlert"
```

---

## 🔍 **Troubleshooting**

### **Problem 1: No Emails Received**

**Check Alertmanager Logs:**
```bash
# Docker
docker logs alertmanager | grep -i email

# Systemd
sudo journalctl -u alertmanager -f | grep -i email
```

**Common Issues:**
```
Error: "authentication failed"
Solution: 
  - Verify Gmail App Password is correct
  - Remove spaces from password
  - Try re-generating App Password

Error: "connection refused"
Solution:
  - Check smtp_smarthost is correct
  - Verify port 587 is not blocked by firewall
  - Try smtp.gmail.com:465 with smtp_require_tls: false

Error: "template not found"
Solution:
  - Verify template files exist in /etc/alertmanager/templates/
  - Check file permissions (should be readable by alertmanager user)
  - Verify template names match in alertmanager.yml
```

### **Problem 2: Emails Sent But Wrong Template**

**Check Routing:**
```bash
# Test routing with amtool
amtool config routes test \
  --config.file=/etc/alertmanager/alertmanager.yml \
  team=backend severity=critical

# Expected output should show: backend-critical receiver
```

### **Problem 3: Too Many Emails (Spam)**

**Adjust Grouping:**
```yaml
route:
  group_wait: 30s        # Wait 30s before sending first notification
  group_interval: 5m     # Wait 5m before sending updates
  repeat_interval: 4h    # Wait 4h before repeating for unresolved alerts
```

### **Problem 4: Alert Rules Not Loading**

**Validate Alert Rules:**
```bash
# Check syntax
promtool check rules /etc/prometheus/rules/alerts.yml

# Check Prometheus logs
docker logs prometheus | grep -i rule
```

---

## 🎨 **Email Template Customization**

### **Customize Critical Alert Template:**

Edit `/etc/alertmanager/templates/email_template_critical.tmpl`:

```html
<!-- Change colors -->
<style>
    .header {
        background: linear-gradient(135deg, #YOUR_COLOR 0%, #YOUR_COLOR2 100%);
    }
</style>

<!-- Add company logo -->
<div class="header">
    <img src="https://yourcompany.com/logo.png" alt="Logo" style="height: 50px;">
    <h1>🚨 CRITICAL ALERT</h1>
</div>

<!-- Customize links -->
<a href="https://your-grafana-url.com/d/enterprise-alerts-sla-slo" class="button">
    📊 View Dashboard
</a>
```

### **Add More Root Cause Analysis:**

```html
{{ else if eq .CommonLabels.alertname "YourCustomAlert" }}
<ul>
    <li><strong>Issue:</strong> Description of the issue</li>
    <li><strong>Impact:</strong> What users will experience</li>
    <li><strong>Possible Causes:</strong>
        <ul>
            <li>Cause 1 with explanation</li>
            <li>Cause 2 with explanation</li>
            <li>Cause 3 with explanation</li>
        </ul>
    </li>
</ul>
{{ end }}
```

---

## 📊 **Alert Dashboard Integration**

### **Link Email to Grafana Dashboard:**

In email templates, the links point to:
```
http://grafana.yourcompany.com/d/enterprise-alerts-sla-slo
```

**Update to your Grafana URL:**
1. Open each email template (.tmpl files)
2. Find: `http://grafana.yourcompany.com`
3. Replace with: `http://your-actual-grafana-url.com`

### **Dashboard Filters Pre-Selected:**

To link directly to filtered view:

```html
<!-- Link to specific team -->
<a href="http://grafana.com/d/enterprise-alerts-sla-slo?var-team=backend&var-severity=critical">
    View Backend Critical Alerts
</a>

<!-- Link to specific environment -->
<a href="http://grafana.com/d/enterprise-alerts-sla-slo?var-environment=production">
    View Production Alerts
</a>
```

---

## 🚀 **Advanced Features**

### **1. Slack + Email (Multi-Channel)**

```yaml
receivers:
  - name: 'backend-critical'
    email_configs:
      - to: 'backend-team@yourcompany.com'
        html: '{{ template "email.critical.html" . }}'
    
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
        channel: '#backend-alerts'
        title: '🚨 CRITICAL: {{ .GroupLabels.alertname }}'
        text: '{{ .CommonAnnotations.description }}'
```

### **2. PagerDuty Integration (On-Call)**

```yaml
receivers:
  - name: 'backend-critical'
    email_configs:
      - to: 'backend-team@yourcompany.com'
    
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_SERVICE_KEY'
        severity: 'critical'
        description: '{{ .CommonAnnotations.summary }}'
```

### **3. Time-Based Routing (Business Hours)**

```yaml
routes:
  - match:
      severity: critical
    receiver: 'pagerduty-oncall'
    active_time_intervals:
      - business_hours
      
time_intervals:
  - name: business_hours
    time_intervals:
      - times:
          - start_time: '09:00'
            end_time: '17:00'
        weekdays: ['monday:friday']
```

### **4. Auto-Acknowledge After Resolution**

```yaml
receivers:
  - name: 'backend-critical'
    email_configs:
      - to: 'backend-team@yourcompany.com'
        send_resolved: true  # Send email when alert resolves
        headers:
          Subject: '✅ [RESOLVED] {{ .GroupLabels.alertname }}'
```

---

## 📋 **Setup Checklist**

```bash
□ 1. Create Gmail App Password
□ 2. Update alertmanager.yml with SMTP credentials
□ 3. Update team email addresses in receivers
□ 4. Copy alertmanager.yml to /etc/alertmanager/
□ 5. Copy email templates to /etc/alertmanager/templates/
□ 6. Copy alerts.yml to /etc/prometheus/rules/
□ 7. Update prometheus.yml with rule_files and alerting sections
□ 8. Set correct file permissions
□ 9. Reload Prometheus configuration
□ 10. Restart Alertmanager
□ 11. Create test alert (TestEmailAlert)
□ 12. Verify alert fires in Prometheus UI
□ 13. Check email received in inbox
□ 14. Test critical/warning/info severity routing
□ 15. Verify email templates render correctly
□ 16. Update Grafana dashboard URLs in templates
□ 17. Configure inhibition rules (optional)
□ 18. Set up Slack integration (optional)
□ 19. Document team escalation procedures
□ 20. Train team on alert acknowledgment
```

---

## 📞 **Support Contacts**

**For Configuration Issues:**
- Prometheus: https://prometheus.io/docs/
- Alertmanager: https://prometheus.io/docs/alerting/latest/alertmanager/
- Go Templates: https://pkg.go.dev/text/template

**For Email Issues:**
- Gmail SMTP: https://support.google.com/mail/answer/7126229
- SendGrid: https://docs.sendgrid.com/
- AWS SES: https://docs.aws.amazon.com/ses/

---

## 🎯 **Summary**

**You now have:**
✅ Complete alerting pipeline with email notifications
✅ Team-based alert routing (backend, platform, data, frontend)
✅ Severity-based templates (critical, warning, info)
✅ Custom email templates with root cause analysis
✅ Alert grouping and deduplication
✅ Integration with Grafana dashboard

**Total Files Created:**
- alertmanager.yml (1 file)
- Email templates (4 files)
- Alert rules sample (1 file)
- Prometheus config sample (1 file)
- Setup guide (this file)

**Next Steps:**
1. Deploy configuration files
2. Update email addresses for your teams
3. Test with TestEmailAlert
4. Monitor email delivery
5. Customize templates as needed

---

**Created:** December 2025  
**Version:** 1.0 Production-Ready  
**Status:** ✅ Complete Enterprise Setup
