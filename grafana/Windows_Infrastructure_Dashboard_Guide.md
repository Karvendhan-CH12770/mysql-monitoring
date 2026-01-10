# Windows/OS Infrastructure Dashboard - Enterprise Monitoring
## Complete Implementation Guide

### 🎯 Dashboard Overview

**Dashboard Name:** Infrastructure - Windows/OS Monitoring  
**UID:** enterprise-windows-infrastructure  
**Panels:** 25 comprehensive monitoring panels  
**Sections:** 5 major monitoring areas  
**Filters:** Team, Environment, IP Address (all multi-select)

---

## 📊 Dashboard Structure

### **Section 1: Infrastructure Health Overview (6 KPI Stats)**

1. **Servers Online** - Count of healthy Windows servers
2. **Average CPU Usage** - CPU usage averaged across all servers
3. **Average Memory Usage** - Memory usage averaged across all servers
4. **Average Disk Usage** - Disk usage averaged across all volumes
5. **Total Network Traffic** - Combined network throughput
6. **Average System Uptime** - Average uptime across fleet

### **Section 2: CPU Monitoring (Enhanced - 5 Panels)**

**NEW ENHANCED CPU PANELS:**

7. **Current CPU Usage (Per Server)** - Individual server CPU with multi-line chart
   - Shows each server's CPU usage separately
   - Color-coded by IP address
   - Best for: Identifying which specific server has high CPU

8. **Average CPU Usage (Across All Servers)** - Trend analysis
   - Single line showing fleet-wide average
   - Smooth out individual server spikes
   - Best for: Understanding overall fleet health
   - **Color:** Blue line

9. **Maximum CPU Usage (Highest Server) - ALERT ON THIS**
   - Shows the HIGHEST CPU among all servers
   - **THIS IS THE METRIC TO ALERT ON**
   - Prevents masking issues on individual servers
   - **Color:** Red line with threshold markers
   - **Alert Rule:** Trigger if > 80% for 5 minutes

10. **CPU Usage by Mode (User, System, Interrupt)** - Stacked area chart
    - User mode CPU (applications)
    - System mode CPU (kernel)
    - Interrupt CPU (hardware interrupts)

11. **CPU Usage by Server Table** - Current snapshot
    - Heat-map visualization
    - Sortable by CPU percentage
    - Real-time server comparison

---

### **Section 3: Memory Management (4 Panels)**

12. **Memory Usage by Server** - Per-server memory trends
13. **Total Memory (Used vs Free)** - Fleet-wide memory consumption
14. **Memory Page Faults by Server** - Memory pressure indicators
15. **Memory Status by Server Table** - Total, Free, Used% with heat-map

---

### **Section 4: Disk Storage & I/O (4 Panels)**

16. **Disk Usage by Server & Volume** - C: and D: drive monitoring
17. **Disk I/O Operations (Read/Write IOPS)** - Disk performance
18. **Disk Throughput (Read/Write Bandwidth)** - Data transfer rates
19. **Disk Space by Server & Volume Table** - Current disk status

---

### **Section 5: Network Activity (4 Panels)**

20. **Network Traffic (Total Sent/Received)** - Combined network I/O
21. **Network Traffic by Server** - Per-server network usage
22. **Network Errors & Dropped Packets** - Network health
23. **Network Traffic by Server Table** - Current bandwidth usage

---

### **Section 6: System Information & Uptime (2 Panels)**

24. **System Uptime by Server** - Uptime trends
25. **Server Information Summary Table** - OS version, RAM, uptime

---

## ⚙️ Template Variables (Multi-Select Enabled)

### **1. Team Filter**
```
Type: Multi-select dropdown
Source: label_values(windows_os_info, team)
Default: All
Example Values: platform, frontend, backend, ops
```

### **2. Environment Filter**
```
Type: Multi-select dropdown  
Source: label_values(windows_os_info{team=~"$team"}, environment)
Default: All
Cascades from: Team
Example Values: production, staging, development
```

### **3. IP Address Filter**
```
Type: Multi-select dropdown
Source: label_values(windows_os_info{team=~"$team", environment=~"$environment"}, ip)
Default: All
Cascades from: Team, Environment
Example Values: 192.168.1.10, 192.168.1.11, 10.0.0.5
```

---

## 📋 Required Prometheus Metrics

### **Windows Exporter Metrics:**

```promql
# CPU Metrics
windows_cpu_time_total{mode, ip, team, environment}

# Memory Metrics
windows_os_physical_memory_free_bytes{ip, team, environment}
windows_cs_physical_memory_bytes{ip, team, environment}
windows_memory_page_faults_total{ip, team, environment}

# Disk Metrics
windows_logical_disk_size_bytes{volume, ip, team, environment}
windows_logical_disk_free_bytes{volume, ip, team, environment}
windows_logical_disk_reads_total{volume, ip, team, environment}
windows_logical_disk_writes_total{volume, ip, team, environment}
windows_logical_disk_read_bytes_total{volume, ip, team, environment}
windows_logical_disk_write_bytes_total{volume, ip, team, environment}

# Network Metrics
windows_net_bytes_total{ip, team, environment}
windows_net_bytes_sent_total{ip, team, environment}
windows_net_bytes_received_total{ip, team, environment}
windows_net_packets_received_errors_total{ip, team, environment}
windows_net_packets_sent_errors_total{ip, team, environment}
windows_net_packets_received_discarded_total{ip, team, environment}
windows_net_packets_sent_discarded_total{ip, team, environment}

# System Info
windows_os_info{ip, team, environment, product, version}
windows_os_time{ip, team, environment}
up{job="windows", ip, team, environment}
```

---

## 🔧 Prometheus Configuration Example

### **prometheus.yml - Windows Exporter Setup:**

```yaml
scrape_configs:
  # Windows Server 01
  - job_name: 'windows'
    static_configs:
      - targets: ['192.168.1.10:9182']
        labels:
          environment: 'production'
          team: 'platform'
          ip: '192.168.1.10'
          hostname: 'app-server-01'
          datacenter: 'dc1'

      # Windows Server 02
      - targets: ['192.168.1.11:9182']
        labels:
          environment: 'production'
          team: 'platform'
          ip: '192.168.1.11'
          hostname: 'app-server-02'
          datacenter: 'dc1'

      # Windows Server 03
      - targets: ['192.168.1.12:9182']
        labels:
          environment: 'production'
          team: 'platform'
          ip: '192.168.1.12'
          hostname: 'app-server-03'
          datacenter: 'dc2'

      # Windows Server 04
      - targets: ['192.168.1.13:9182']
        labels:
          environment: 'production'
          team: 'platform'
          ip: '192.168.1.13'
          hostname: 'app-server-04'
          datacenter: 'dc2'

      # Windows Server 05
      - targets: ['192.168.1.14:9182']
        labels:
          environment: 'production'
          team: 'platform'
          ip: '192.168.1.14'
          hostname: 'app-server-05'
          datacenter: 'dc1'

      # Staging Server
      - targets: ['192.168.2.10:9182']
        labels:
          environment: 'staging'
          team: 'platform'
          ip: '192.168.2.10'
          hostname: 'staging-server'
          datacenter: 'dc1'
```

---

## 🎨 Key Design Features

### **Enhanced CPU Monitoring (NEW):**

1. **Three Different Views:**
   - **Current (Per Server):** Individual server lines - for troubleshooting
   - **Average:** Single line showing fleet average - for trend analysis
   - **Maximum:** Highest CPU among all servers - **FOR ALERTING**

2. **Why Maximum CPU for Alerts?**
   ```
   Scenario: 5 servers, 4 at 20% CPU, 1 at 95% CPU
   
   Average CPU: (20+20+20+20+95)/5 = 35% ✅ Looks fine
   Maximum CPU: 95% ❌ ALERT! Problem detected!
   
   Maximum prevents masking issues on individual servers!
   ```

3. **Color Coding:**
   - Green: < 60% (healthy)
   - Yellow: 60-80% (watch)
   - Orange: 80-90% (high)
   - Red: > 90% (critical)

### **IP-Based Filtering (Instead of Instance):**
- More intuitive for network teams
- Easier to correlate with network monitoring tools
- Direct mapping to infrastructure diagrams
- Better for firewall and security correlation

### **Heat-Map Tables:**
- CPU Usage Table: Red gradient for high utilization
- Memory Usage Table: Red gradient for high usage
- Disk Space Table: Red gradient for low free space
- Network Traffic Table: Blue gradient for high bandwidth

---

## 📈 Alert Rules (Recommended)

### **Critical CPU Alert:**
```yaml
groups:
  - name: windows_infrastructure_critical
    rules:
      - alert: WindowsHighCPU
        expr: max(100 - (avg by (ip) (rate(windows_cpu_time_total{mode="idle"}[5m])) * 100)) > 90
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Windows Server Critical CPU (> 90%)"
          description: "At least one Windows server has CPU > 90% for 5 minutes"
```

### **Memory Pressure Alert:**
```yaml
      - alert: WindowsHighMemory
        expr: max((1 - (windows_os_physical_memory_free_bytes / windows_cs_physical_memory_bytes)) * 100) > 95
        for: 3m
        labels:
          severity: critical
        annotations:
          summary: "Windows Server Critical Memory (> 95%)"
          description: "At least one Windows server has memory > 95% for 3 minutes"
```

### **Disk Space Alert:**
```yaml
      - alert: WindowsLowDiskSpace
        expr: max(100 - ((windows_logical_disk_free_bytes{volume=~"C:|D:"} / windows_logical_disk_size_bytes{volume=~"C:|D:"}) * 100)) > 90
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "Windows Server Low Disk Space (> 90%)"
          description: "At least one Windows server volume has < 10% free space"
```

---

## 🚀 Usage Examples

### **1. Fleet-Wide Health Check:**
```
Filters:
  Team: All
  Environment: production
  IP: All

Check:
  - Average CPU Usage stat
  - Average Memory Usage stat
  - Servers Online count
  - Maximum CPU Usage graph (for hotspots)
```

### **2. Troubleshoot Specific Server:**
```
Filters:
  Team: platform
  Environment: production
  IP: 192.168.1.10

Check:
  - Current CPU Usage (single line)
  - Memory Usage by Server
  - Disk Usage by Server & Volume
  - Network Traffic by Server
```

### **3. Environment Comparison:**
```
Filters:
  Team: platform
  Environment: production, staging (multi-select)
  IP: All

Check:
  - Average CPU Usage (compare prod vs staging)
  - Memory Usage trends
  - Disk I/O differences
```

### **4. Team Resource Monitoring:**
```
Filters:
  Team: frontend, backend (multi-select)
  Environment: production
  IP: All

Check:
  - CPU Usage by Server Table (sorted by usage)
  - Memory Status Table
  - Disk Space Table
```

---

## ✅ Import Checklist

```
□ Import dashboard JSON to Grafana
□ Select Prometheus data source
□ Verify all 3 filters work (team, environment, ip)
□ Test multi-select on each filter
□ Confirm 25 panels load data correctly
□ Verify CPU panels show: Current, Average, Maximum
□ Check heat-map tables display gradient colors
□ Test filter cascading (team → environment → ip)
□ Set default time range (1 hour)
□ Configure auto-refresh (30 seconds)
□ Set up alert rules for Maximum CPU
□ Create platform team folder
□ Save dashboard with appropriate permissions
```

---

## 🎯 Key Metrics Formulas

### **Average CPU:**
```promql
avg(100 - (avg by (ip) (rate(windows_cpu_time_total{mode="idle", team=~"$team", environment=~"$environment", ip=~"$ip"}[5m])) * 100))
```

### **Maximum CPU (ALERT ON THIS):**
```promql
max(100 - (avg by (ip) (rate(windows_cpu_time_total{mode="idle", team=~"$team", environment=~"$environment", ip=~"$ip"}[5m])) * 100))
```

### **Memory Usage %:**
```promql
(1 - (windows_os_physical_memory_free_bytes / windows_cs_physical_memory_bytes)) * 100
```

### **Disk Usage %:**
```promql
100 - ((windows_logical_disk_free_bytes{volume=~"C:|D:"} / windows_logical_disk_size_bytes{volume=~"C:|D:"}) * 100)
```

---

## 📊 Panel Count Summary

| Section | Panels | Types |
|---------|--------|-------|
| Health Overview | 6 | Stats |
| CPU Monitoring | 5 | Time series + Table |
| Memory | 4 | Time series + Table |
| Disk & I/O | 4 | Time series + Table |
| Network | 4 | Time series + Table |
| System Info | 2 | Time series + Table |
| **Total** | **25** | **Mixed** |

---

## 🔥 What Makes This Dashboard Enterprise-Level?

1. ✅ **Centralized Monitoring** - Single view for all Windows servers
2. ✅ **Multi-Select Filters** - Flexible team/environment/IP filtering
3. ✅ **Enhanced CPU Monitoring** - Current, Average, **Maximum** (for alerting)
4. ✅ **IP-Based Filtering** - Network-friendly server identification
5. ✅ **Heat-Map Visualizations** - Quick visual problem identification
6. ✅ **Comprehensive Metrics** - CPU, Memory, Disk, Network, System Info
7. ✅ **Cascading Filters** - Team → Environment → IP logical flow
8. ✅ **Production-Ready Alerts** - Pre-defined alert rules included
9. ✅ **Professional Design** - Color-coded thresholds, smooth animations
10. ✅ **Scalable Architecture** - Supports 5 to 500+ servers

---

## 📝 Notes

- **Why IP instead of Instance?** 
  - Easier correlation with network tools
  - More intuitive for infrastructure teams
  - Direct mapping to network diagrams

- **Why Maximum CPU for Alerts?**
  - Prevents masking high CPU on individual servers
  - Industry best practice for infrastructure monitoring
  - Average can hide critical issues

- **Dashboard Performance:**
  - Optimized queries with 5m rate intervals
  - Efficient aggregations
  - Works well with 50+ servers

---

**Dashboard Created:** December 2025  
**Version:** 1.0 Enterprise  
**Status:** Production-Ready ✅
