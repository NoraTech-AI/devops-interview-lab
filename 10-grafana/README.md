#  Grafana Dashboarding & Alerting

## Objective

Build a monitoring stack using Prometheus and Grafana, visualize application metrics, and create basic alerting rules.

---

## Architecture

```text
+-------------+
|  Grafana    |
+------+------+
       |
       v
+------+------+
| Prometheus  |
+------+------+
       |
       +------------------+
       |                  |
       v                  v
+-------------+   +----------------+
| Nginx App   |   | Nginx Exporter |
+-------------+   +----------------+
```

Prometheus scrapes metrics from:

* Prometheus itself
* Nginx Exporter
* Nginx application endpoint

Grafana uses Prometheus as a datasource and visualizes collected metrics.

---

## Components

### Prometheus

Responsibilities:

* Metrics collection
* Metrics storage
* Rule evaluation
* Alert generation

### Grafana

Responsibilities:

* Dashboard creation
* Data visualization
* Alert visualization

### Nginx Exporter

Responsibilities:

* Expose Nginx metrics in Prometheus format
* Collect metrics from Nginx stub_status endpoint

---

## Kubernetes Resources

### Namespace

```bash
kubectl create namespace monitoring-lab
```

### Deployments

* Prometheus
* Grafana
* Nginx Application
* Nginx Exporter

### Services

* prometheus
* grafana
* nginx-service
* nginx-exporter

### ConfigMaps

* prometheus-config
* nginx-config
* alert-config

---

## Prometheus Configuration

Scrape jobs:

* prometheus
* nginx
* nginx-exporter

Alert rules are loaded using:

```yaml
rule_files:
  - /etc/prometheus/rules/alert.rules.yml
```

---

## Grafana Configuration

Datasource:

* Prometheus

Datasource URL:

```text
http://prometheus:9090
```

---

## Dashboard

Created panels:

### Prometheus Availability

Query:

```promql
up{job="prometheus"}
```

### Nginx Exporter Availability

Query:

```promql
up{job="nginx-exporter"}
```

### All Targets

Query:

```promql
up
```

---

## Alerting

### Alert Rule

```yaml
alert: NginxExporterDown
expr: up{job="nginx-exporter"} == 0
for: 1m
```

### Test Procedure

Scale exporter down:

```bash
kubectl scale deployment nginx-exporter \
--replicas=0 \
-n monitoring-lab
```

Verify alert:

```text
Prometheus → Alerts
State = Firing
```

Restore exporter:

```bash
kubectl scale deployment nginx-exporter \
--replicas=1 \
-n monitoring-lab
```

Verify alert recovery.

---

## Troubleshooting Performed

### Missing Prometheus Service

Problem:

Grafana could not resolve:

```text
http://prometheus:9090
```

Reason:

No Kubernetes Service existed for Prometheus.

Solution:

Created:

```yaml
kind: Service
metadata:
  name: prometheus
```

---

### ConfigMap Mount Issues

Problem:

Prometheus started without loading alert rules.

Reason:

Alert ConfigMap was not mounted correctly.

Solution:

Mounted alert-config under:

```text
/etc/prometheus/rules
```

and referenced it in:

```yaml
rule_files:
```

---

### Nginx Exporter Metrics

Problem:

stub_status endpoint unavailable.

Reason:

Nginx configuration not mounted correctly.

Solution:

Created nginx-config ConfigMap and mounted default.conf into the container.

---

## Interview Notes

### Why use Prometheus?

* Pull-based architecture
* Native Kubernetes integration
* Powerful PromQL language
* Efficient time-series storage

### Why use Grafana?

* Rich dashboards
* Multiple datasource support
* Alert visualization

### Difference between Pending and Firing alerts?

Pending:
Condition became true but duration defined in `for` has not elapsed.

Firing:
Condition remained true for the configured duration and alert became active.

### Why separate alert rules into a dedicated ConfigMap?

* Easier maintenance
* Independent lifecycle management
* Better GitOps workflow
* Cleaner configuration management

---

## Skills Demonstrated

* Kubernetes Deployments
* Kubernetes Services
* ConfigMaps
* Prometheus Configuration
* Prometheus Alert Rules
* Grafana Datasources
* Grafana Dashboards
* Metrics Collection
* Monitoring Architecture
* Troubleshooting and Root Cause Analysis
