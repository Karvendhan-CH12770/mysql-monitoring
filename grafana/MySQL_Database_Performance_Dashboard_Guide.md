# MySQL/MariaDB Database Performance Dashboard
## Enterprise Monitoring Guide

### 🎯 Dashboard Overview

**Dashboard Name:** Database Performance - MySQL/MariaDB  
**UID:** enterprise-mysql-performance  
**Panels:** 27 comprehensive monitoring panels  
**Sections:** 6 major monitoring areas  
**Filters:** Team, Application, Database, Environment (all multi-select, cascading)

---

## 📊 Dashboard Structure

### **Section 1: Database Health Overview (6 KPI Stats)**

1. **Databases Online** - Count of healthy MySQL/MariaDB instances
2. **Current QPS** - Queries Per Second across all databases
3. **Active Connections** - Currently connected clients
4. **Slow Queries (Last Hour)** - Total slow queries in past hour
5. **InnoDB Buffer Pool Usage** - Memory utilization percentage
6. **Database Uptime** - Average uptime across fleet

---

### **Section 2: Query Performance & Throughput (4 Panels)**

7. **Queries Per Second (QPS)** - Query rate by database
8. **Query Operations Breakdown** - SELECT/INSERT/UPDATE/DELETE breakdown
9. **Slow Queries Over Time** - Slow query rate by database
10. **Query Cache Hit Rate** - Cache effectiveness percentage

---

### **Section 3: Connection Management (5 Panels)**

11. **Active Connections** - Connected clients over time
12. **Connection Usage %** - Connection pool utilization
13. **Running Threads** - Actively executing queries
14. **Aborted Connections** - Failed connection attempts
15. **Connection Status by Database Table** - Heat-map showing connection usage

---

### **Section 4: InnoDB Buffer Pool & Cache Performance (4 Panels)**

16. **InnoDB Buffer Pool Usage** - Buffer pool memory utilization
17. **InnoDB Buffer Pool Read Requests** - Read request rate
18. **InnoDB Buffer Pool Hit Rate** - Cache hit effectiveness
19. **InnoDB Pages Read/Written** - Disk I/O activity

---

### **Section 5: Table Locks & Replication (4 Panels)**

20. **Table Locks Waited** - Lock contention by database
21. **Table Locks Immediate vs Waited** - Lock acquisition patterns
22. **Replication Lag (Seconds Behind Master)** - Replica delay
23. **Replication IO/SQL Running Status** - Replication thread health

---

### **Section 6: Network I/O & Temporary Tables (4 Panels)**

24. **Network Traffic (Sent/Received)** - Database network I/O
25. **Temporary Tables Created** - Memory vs Disk temp tables
26. **Temporary Tables on Disk %** - Percentage requiring disk
27. **Full Table Scans** - Queries without index usage

---

## ⚙️ Template Variables (Multi-Select with Cascading)

### **1. Team Filter**
```
Type: Multi-select dropdown
Source: label_values(mysql_up, team)
Default: All
Example Values: platform, backend, data-engineering
```

### **2. Application Filter**
```
Type: Multi-select dropdown  
Source: label_values(mysql_up{team=~"$team"}, application)
Default: All
Cascades from: Team
Example Values: web-api, mobile-api, analytics, admin-portal
```

### **3. Database Filter**
```
Type: Multi-select dropdown
Source: label_values(mysql_up{team=~"$team", application=~"$application"}, database)
Default: All
Cascades from: Team, Application
Example Values: production-db, staging-db, user-db, analytics-db
```

### **4. Environment Filter**
```
Type: Multi-select dropdown
Source: label_values(mysql_up{team=~"$team", application=~"$application", database=~"$database"}, environment)
Default: All
Cascades from: Team, Application, Database
Example Values: production, staging, development
```

---

## 📋 Required Prometheus Metrics

### **MySQL Exporter Metrics:**

```promql
# Database Status
mysql_up{team, application, database, environment}
mysql_global_status_uptime{team, application, database, environment}

# Query Metrics
mysql_global_status_queries{team, application, database, environment}
mysql_global_status_slow_queries{team, application, database, environment}
mysql_global_status_com_select{team, application, database, environment}
mysql_global_status_com_insert{team, application, database, environment}
mysql_global_status_com_update{team, application, database, environment}
mysql_global_status_com_delete{team, application, database, environment}

# Query Cache
mysql_global_status_qcache_hits{team, application, database, environment}
mysql_global_status_qcache_inserts{team, application, database, environment}

# Connection Metrics
mysql_global_status_threads_connected{team, application, database, environment}
mysql_global_status_threads_running{team, application, database, environment}
mysql_global_variables_max_connections{team, application, database, environment}
mysql_global_status_aborted_connects{team, application, database, environment}

# InnoDB Buffer Pool
mysql_global_status_innodb_buffer_pool_pages_data{team, application, database, environment}
mysql_global_status_innodb_buffer_pool_pages_total{team, application, database, environment}
mysql_global_status_innodb_buffer_pool_read_requests{team, application, database, environment}
mysql_global_status_innodb_buffer_pool_reads{team, application, database, environment}
mysql_global_status_innodb_pages_read{team, application, database, environment}
mysql_global_status_innodb_pages_written{team, application, database, environment}

# Table Locks
mysql_global_status_table_locks_immediate{team, application, database, environment}
mysql_global_status_table_locks_waited{team, application, database, environment}

# Replication
mysql_slave_status_seconds_behind_master{team, application, database, environment}
mysql_slave_status_slave_io_running{team, application, database, environment}
mysql_slave_status_slave_sql_running{team, application, database, environment}

# Network
mysql_global_status_bytes_sent{team, application, database, environment}
mysql_global_status_bytes_received{team, application, database, environment}

# Temporary Tables
mysql_global_status_created_tmp_tables{team, application, database, environment}
mysql_global_status_created_tmp_disk_tables{team, application, database, environment}

# Full Scans
mysql_global_status_select_scan{team, application, database, environment}
```

---

## 🔧 Prometheus Configuration Example

### **prometheus.yml - MySQL Exporter Setup:**

```yaml
scrape_configs:
  # MySQL Database - Production Web API
  - job_name: 'mysql'
    static_configs:
      - targets: ['mysql-exporter-web:9104']
        labels:
          team: 'backend'
          application: 'web-api'
          database: 'production-db'
          environment: 'production'
          instance: 'mysql-prod-01'
          replica_role: 'master'

      # MySQL Database - Production Web API Replica
      - targets: ['mysql-exporter-web-replica:9104']
        labels:
          team: 'backend'
          application: 'web-api'
          database: 'production-db-replica'
          environment: 'production'
          instance: 'mysql-prod-02'
          replica_role: 'slave'

      # MySQL Database - Mobile API
      - targets: ['mysql-exporter-mobile:9104']
        labels:
          team: 'backend'
          application: 'mobile-api'
          database: 'mobile-db'
          environment: 'production'
          instance: 'mysql-prod-03'
          replica_role: 'master'

      # MySQL Database - Analytics
      - targets: ['mysql-exporter-analytics:9104']
        labels:
          team: 'data-engineering'
          application: 'analytics'
          database: 'analytics-db'
          environment: 'production'
          instance: 'mysql-analytics-01'
          replica_role: 'master'

      # MySQL Database - User Service
      - targets: ['mysql-exporter-users:9104']
        labels:
          team: 'backend'
          application: 'user-service'
          database: 'user-db'
          environment: 'production'
          instance: 'mysql-prod-04'
          replica_role: 'master'

      # MySQL Database - Staging
      - targets: ['mysql-exporter-staging:9104']
        labels:
          team: 'backend'
          application: 'web-api'
          database: 'staging-db'
          environment: 'staging'
          instance: 'mysql-staging-01'
          replica_role: 'master'
```

---

## 🎨 Key Design Features

### **1. Cascading Filters (4 Levels)**
```
Team → Application → Database → Environment

Example Flow:
1. Select Team: "backend"
2. Application filter shows: web-api, mobile-api, user-service
3. Select Application: "web-api"
4. Database filter shows: production-db, staging-db
5. Select Database: "production-db"
6. Environment filter shows: production
```

### **2. Color-Coded Thresholds**

**Databases Online:**
- Red: < 1 (critical)
- Yellow: 1 (warning)
- Green: ≥ 2 (healthy)

**Current QPS:**
- Green: < 1000 (normal)
- Yellow: 1000-5000 (moderate)
- Red: > 5000 (high load)

**Active Connections:**
- Green: < 100 (normal)
- Yellow: 100-200 (watch)
- Orange: 200-300 (high)
- Red: > 300 (critical)

**Slow Queries:**
- Green: < 10/hour (excellent)
- Yellow: 10-50/hour (watch)
- Orange: 50-100/hour (optimize)
- Red: > 100/hour (critical)

**InnoDB Buffer Pool:**
- Green: < 70% (good)
- Yellow: 70-85% (watch)
- Orange: 85-95% (high)
- Red: > 95% (critical)

**Connection Usage %:**
- Green: < 60% (healthy)
- Yellow: 60-80% (monitor)
- Orange: 80-90% (scale)
- Red: > 90% (critical)

### **3. Heat-Map Table**
- Connection Status Table shows gradient colors
- Red = High connection usage
- Green = Low connection usage
- Sortable by usage percentage

---

## 📈 Alert Rules (Recommended)

### **Critical Database Alerts:**

```yaml
groups:
  - name: mysql_database_critical
    rules:
      # Database Down
      - alert: MySQLDatabaseDown
        expr: mysql_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "MySQL Database Down"
          description: "MySQL database {{$labels.database}} is down"

      # High Connection Usage
      - alert: MySQLHighConnectionUsage
        expr: |
          (mysql_global_status_threads_connected / mysql_global_variables_max_connections) * 100 > 90
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "MySQL High Connection Usage (> 90%)"
          description: "Database {{$labels.database}} has {{$value}}% connection usage"

      # Replication Lag
      - alert: MySQLReplicationLag
        expr: mysql_slave_status_seconds_behind_master > 300
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "MySQL Replication Lag > 5 minutes"
          description: "Replica {{$labels.database}} is {{$value}} seconds behind master"

      # Replication Stopped
      - alert: MySQLReplicationStopped
        expr: |
          mysql_slave_status_slave_io_running == 0 or 
          mysql_slave_status_slave_sql_running == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "MySQL Replication Stopped"
          description: "Replication stopped on {{$labels.database}}"

      # Buffer Pool Low
      - alert: MySQLBufferPoolLow
        expr: |
          (mysql_global_status_innodb_buffer_pool_pages_data / 
           mysql_global_status_innodb_buffer_pool_pages_total) * 100 < 10
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "MySQL InnoDB Buffer Pool Low Usage"
          description: "Buffer pool usage is {{$value}}% on {{$labels.database}}"
```

### **Warning Database Alerts:**

```yaml
  - name: mysql_database_warning
    rules:
      # High Slow Queries
      - alert: MySQLHighSlowQueries
        expr: rate(mysql_global_status_slow_queries[5m]) > 10
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "MySQL High Slow Query Rate"
          description: "{{$value}} slow queries/sec on {{$labels.database}}"

      # Too Many Temporary Tables on Disk
      - alert: MySQLHighDiskTempTables
        expr: |
          (rate(mysql_global_status_created_tmp_disk_tables[5m]) / 
           rate(mysql_global_status_created_tmp_tables[5m])) * 100 > 25
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "MySQL High Temp Tables on Disk (> 25%)"
          description: "{{$value}}% temp tables on disk for {{$labels.database}}"

      # High Aborted Connections
      - alert: MySQLHighAbortedConnections
        expr: rate(mysql_global_status_aborted_connects[5m]) > 5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "MySQL High Aborted Connection Rate"
          description: "{{$value}} aborted connections/sec on {{$labels.database}}"

      # Low Query Cache Hit Rate
      - alert: MySQLLowQueryCacheHitRate
        expr: |
          (rate(mysql_global_status_qcache_hits[5m]) / 
          (rate(mysql_global_status_qcache_hits[5m]) + 
           rate(mysql_global_status_qcache_inserts[5m]))) * 100 < 50
        for: 30m
        labels:
          severity: warning
        annotations:
          summary: "MySQL Low Query Cache Hit Rate (< 50%)"
          description: "Query cache hit rate is {{$value}}% on {{$labels.database}}"
```

---

## 🚀 Usage Examples

### **1. Monitor Specific Application:**
```
Filters:
  Team: backend
  Application: web-api
  Database: All
  Environment: production

Use Case: Monitor all web-api databases in production
```

### **2. Compare Production vs Staging:**
```
Filters:
  Team: backend
  Application: web-api
  Database: production-db, staging-db (multi-select)
  Environment: All

Use Case: Compare performance between prod and staging
```

### **3. Team-Wide Database Health:**
```
Filters:
  Team: backend
  Application: All
  Database: All
  Environment: production

Use Case: Overall backend team database health
```

### **4. Troubleshoot Specific Database:**
```
Filters:
  Team: backend
  Application: web-api
  Database: production-db
  Environment: production

Check:
  - Slow Queries Over Time
  - Connection Usage %
  - InnoDB Buffer Pool Hit Rate
  - Replication Lag (if replica)
```

### **5. Replication Monitoring:**
```
Filters:
  Team: backend
  Application: All
  Database: All (with replica_role=slave label)
  Environment: production

Check:
  - Replication Lag panel
  - Replication IO/SQL Running Status
  - Connection differences vs master
```

---

## 🎯 Key Metrics Explained

### **1. QPS (Queries Per Second):**
```promql
sum(rate(mysql_global_status_queries[5m]))
```
**Interpretation:**
- < 1000 QPS: Low/normal load
- 1000-5000 QPS: Moderate load
- > 5000 QPS: High load (may need scaling)

### **2. Connection Usage %:**
```promql
(mysql_global_status_threads_connected / mysql_global_variables_max_connections) * 100
```
**Interpretation:**
- < 60%: Healthy headroom
- 60-80%: Monitor closely
- 80-90%: Consider increasing max_connections
- > 90%: Critical - connections will be rejected

### **3. InnoDB Buffer Pool Hit Rate:**
```promql
(1 - (rate(mysql_global_status_innodb_buffer_pool_reads[5m]) / 
      rate(mysql_global_status_innodb_buffer_pool_read_requests[5m]))) * 100
```
**Interpretation:**
- > 99%: Excellent (most reads from memory)
- 95-99%: Good
- 90-95%: Consider increasing buffer pool
- < 90%: Poor - increase innodb_buffer_pool_size

### **4. Replication Lag:**
```promql
mysql_slave_status_seconds_behind_master
```
**Interpretation:**
- 0-10 seconds: Normal
- 10-60 seconds: Watch
- 60-300 seconds: Warning
- > 300 seconds: Critical issue

### **5. Temp Tables on Disk %:**
```promql
(rate(mysql_global_status_created_tmp_disk_tables[5m]) / 
 rate(mysql_global_status_created_tmp_tables[5m])) * 100
```
**Interpretation:**
- < 10%: Excellent
- 10-25%: Acceptable
- > 25%: Optimize queries, increase tmp_table_size

---

## 📊 Panel Count Summary

| Section | Panels | Types |
|---------|--------|-------|
| Health Overview | 6 | Stats |
| Query Performance | 4 | Time series |
| Connection Mgmt | 5 | Time series + Table |
| InnoDB Buffer Pool | 4 | Time series |
| Locks & Replication | 4 | Time series |
| Network & Temp Tables | 4 | Time series |
| **Total** | **27** | **Mixed** |

---

## 🔥 What Makes This Dashboard Enterprise-Level?

1. ✅ **Comprehensive Coverage** - All critical MySQL/MariaDB metrics
2. ✅ **4-Level Cascading Filters** - Team → App → DB → Env
3. ✅ **Replication Monitoring** - Master/slave lag and status
4. ✅ **Performance Optimization** - Buffer pool, temp tables, full scans
5. ✅ **Connection Management** - Usage %, aborted connections, threads
6. ✅ **Query Analytics** - QPS, slow queries, operation breakdown
7. ✅ **Lock Monitoring** - Table lock contention analysis
8. ✅ **Network I/O** - Database traffic monitoring
9. ✅ **Color-Coded Alerts** - Visual problem identification
10. ✅ **Production-Ready** - 27 comprehensive panels

---

## ✅ Import Checklist

```
□ Import dashboard JSON to Grafana
□ Select Prometheus data source
□ Verify all 4 filters work (team, application, database, environment)
□ Test multi-select on each filter
□ Confirm 27 panels load data correctly
□ Test filter cascading (team → app → db → env)
□ Verify replication panels (if using replicas)
□ Check heat-map table displays gradient colors
□ Set default time range (1 hour)
□ Configure auto-refresh (30 seconds)
□ Set up alert rules for critical metrics
□ Create team folders and save dashboard
□ Configure MySQL Exporter on all databases
```

---

## 🔧 MySQL Exporter Installation

### **Docker Compose Example:**

```yaml
version: '3.8'

services:
  mysql-exporter:
    image: prom/mysqld-exporter:latest
    container_name: mysql-exporter
    restart: unless-stopped
    ports:
      - "9104:9104"
    environment:
      - DATA_SOURCE_NAME=exporter:password@(mysql-host:3306)/
    command:
      - "--collect.global_status"
      - "--collect.info_schema.innodb_metrics"
      - "--collect.auto_increment.columns"
      - "--collect.info_schema.processlist"
      - "--collect.binlog_size"
      - "--collect.info_schema.tablestats"
      - "--collect.global_variables"
      - "--collect.info_schema.query_response_time"
      - "--collect.info_schema.userstats"
      - "--collect.info_schema.tables"
      - "--collect.perf_schema.tablelocks"
      - "--collect.perf_schema.file_events"
      - "--collect.perf_schema.eventswaits"
      - "--collect.slave_status"
```

### **Create MySQL Exporter User:**

```sql
CREATE USER 'exporter'@'%' IDENTIFIED BY 'STRONG_PASSWORD' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'%';
FLUSH PRIVILEGES;
```

---

## 📝 Notes

- **Replication Panels:** Only show data for replica databases
- **Query Cache:** Deprecated in MySQL 8.0+, panels will show no data
- **Performance:** Optimized queries with 5m rate intervals
- **Scalability:** Works with 1 to 100+ databases
- **Multi-Tenancy:** Team/Application/Database/Environment isolation

---

**Dashboard Created:** December 2025  
**Version:** 1.0 Enterprise  
**Status:** Production-Ready ✅
