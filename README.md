# NOC SLA & Uptime Management Framework

![SLA Management](https://img.shields.io/badge/Domain-SLA%20Management-blue)
![Uptime](https://img.shields.io/badge/Target-99.99%25%20Uptime-brightgreen)
![Grafana](https://img.shields.io/badge/Tool-Grafana%20Dashboards-orange)
![Prometheus](https://img.shields.io/badge/Tool-Prometheus-orange)
![ITIL](https://img.shields.io/badge/Framework-ITIL%20v4-green)
![NOC](https://img.shields.io/badge/Operations-24%2F7%20NOC-red)

---

## Project Overview

This project documents a comprehensive NOC SLA and uptime management framework — covering SLA definitions, availability calculation methodology, uptime tracking, capacity management, performance baselines, automated alerting, and executive reporting dashboards for 24/7 infrastructure operations.

Built to demonstrate operational readiness for:
- NOC Engineer roles
- Infrastructure Monitoring Engineer roles
- Data Center Operations Engineer roles
- Service Delivery Manager roles

---

## Repository Structure
---

noc-sla-uptime-management/

│

├── docs/

│   └── sla-framework-overview.md          # Framework overview

│

├── sla-definitions/

│   ├── tier-definitions.md               # SLA tier breakdown

│   ├── measurement-methodology.md        # How uptime is calculated

│   └── exclusion-windows.md             # Planned maintenance exclusions

│

├── uptime-tracking/

│   ├── availability-calculation.md        # Availability formulas

│   ├── downtime-logging.md               # Downtime log procedures

│   └── monthly-reporting.md             # Monthly report process

│

├── capacity-management/

│   ├── baseline-establishment.md          # Performance baselines

│   ├── trend-analysis.md                 # Capacity trend analysis

│   └── growth-planning.md               # Future capacity planning

│

├── dashboards/

│   ├── grafana-sla-dashboard.md          # Grafana SLA dashboard

│   ├── executive-dashboard.md            # Executive visibility

│   └── operations-dashboard.md           # NOC operations view

│

├── alerting/

│   ├── sla-breach-alerts.md              # SLA breach alerting

│   └── capacity-threshold-alerts.md      # Capacity alerts

│

├── reporting/

│   ├── weekly-report-template.md         # Weekly NOC report

│   ├── monthly-report-template.md        # Monthly SLA report

│   └── executive-summary-template.md     # Executive summary

│

├── runbooks/

│   ├── sla-breach-response.md            # SLA breach procedure

│   └── capacity-breach-response.md       # Capacity breach response

│

└── images/

└── README.md                          # Dashboard screenshots


## SLA Tier Definitions

| Tier | Service Type | Availability | Monthly Downtime | RTO | RPO |
|---|---|---|---|---|---|
| Platinum | Mission Critical | 99.99% | 4.3 minutes | 5 min | 1 min |
| Gold | Business Critical | 99.9% | 43.8 minutes | 15 min | 5 min |
| Silver | Business Important | 99.5% | 3.6 hours | 1 hour | 30 min |
| Bronze | Standard | 99.0% | 7.3 hours | 4 hours | 1 hour |
| Development | Non-Production | 95.0% | 36.5 hours | Best effort | Best effort |

---

##  Availability Calculation Methodology

### Formula

Availability % = ((Total Time - Downtime) / Total Time) × 100
Monthly Total Time = 30 days × 24 hours × 60 minutes = 43,200 minutes
Example:

Total Time: 43,200 minutes

Unplanned Downtime: 30 minutes

Availability = ((43,200 - 30) / 43,200) × 100 = 99.93%

### What Counts as Downtime

| Event Type | Counts as Downtime |
|---|---|
| Unplanned outage | ✅ Yes |
| Partial degradation above 50% | ✅ Yes |
| Failed deployments causing outage | ✅ Yes |
| Planned maintenance (approved) | ❌ No — excluded |
| External dependency failure | ❌ No — if documented |
| Force majeure events | ❌ No — if documented |

### Python Availability Calculator

```python
# availability_calculator.py
from datetime import datetime, timedelta

def calculate_availability(
    period_start: datetime,
    period_end: datetime,
    downtime_events: list
) -> dict:
    """
    Calculate service availability for a given period
    
    Args:
        period_start: Start of measurement period
        period_end: End of measurement period
        downtime_events: List of dicts with 'start', 'end', 'type'
    
    Returns:
        Dict with availability metrics
    """
    
    total_minutes = (period_end - period_start).total_seconds() / 60
    
    # Calculate unplanned downtime only
    unplanned_downtime = 0
    for event in downtime_events:
        if event['type'] == 'unplanned':
            duration = (event['end'] - event['start']).total_seconds() / 60
            unplanned_downtime += duration
    
    availability = ((total_minutes - unplanned_downtime) / total_minutes) * 100
    
    return {
        'period_start': period_start.isoformat(),
        'period_end': period_end.isoformat(),
        'total_minutes': total_minutes,
        'unplanned_downtime_minutes': unplanned_downtime,
        'availability_percent': round(availability, 4),
        'sla_met': availability >= 99.9,
        'downtime_budget_remaining': 43.8 - unplanned_downtime
    }

# Example usage
events = [
    {
        'start': datetime(2026, 6, 1, 14, 0),
        'end': datetime(2026, 6, 1, 14, 25),
        'type': 'unplanned',
        'cause': 'Database failover'
    }
]

result = calculate_availability(
    period_start=datetime(2026, 6, 1),
    period_end=datetime(2026, 6, 30),
    downtime_events=events
)

print(f"Availability: {result['availability_percent']}%")
print(f"SLA Met: {result['sla_met']}")
print(f"Downtime Budget Remaining: {result['downtime_budget_remaining']} minutes")
```

---

## Grafana SLA Dashboard Configuration

### Dashboard Panels

```json
{
  "dashboard": {
    "title": "NOC SLA & Uptime Dashboard",
    "panels": [
      {
        "title": "Current Availability %",
        "type": "stat",
        "targets": [{"expr": "availability_percent"}],
        "thresholds": [
          {"color": "red", "value": 0},
          {"color": "yellow", "value": 99.5},
          {"color": "green", "value": 99.9}
        ]
      },
      {
        "title": "Monthly Uptime Trend",
        "type": "timeseries",
        "targets": [{"expr": "uptime_percent[30d]"}]
      },
      {
        "title": "Downtime Budget Remaining",
        "type": "gauge",
        "targets": [{"expr": "downtime_budget_minutes_remaining"}],
        "max": 43.8
      },
      {
        "title": "Incident Count by Severity",
        "type": "barchart",
        "targets": [
          {"expr": "incidents_total{severity='p1'}", "legendFormat": "P1"},
          {"expr": "incidents_total{severity='p2'}", "legendFormat": "P2"},
          {"expr": "incidents_total{severity='p3'}", "legendFormat": "P3"}
        ]
      },
      {
        "title": "MTTR Trend",
        "type": "timeseries",
        "targets": [{"expr": "avg(incident_resolution_time_minutes)"}]
      },
      {
        "title": "SLA Compliance by Service",
        "type": "table",
        "targets": [{"expr": "sla_compliance_percent by (service)"}]
      }
    ]
  }
}
```

---

##  Weekly NOC Report Template

```markdown
## Weekly NOC Operations Report

**Week:** _______________
**Prepared By:** _______________
**Date:** _______________

---

### Executive Summary
Overall infrastructure availability this week: ___%
SLA breaches: ___
P1 Incidents: ___
P2 Incidents: ___

---

### Availability Summary

| Service | SLA Target | Actual | Status |
|---|---|---|---|
| | 99.99% | | 🟢/🟡/🔴 |
| | 99.9% | | 🟢/🟡/🔴 |

---

### Incident Summary

| ID | Severity | Description | Duration | RCA Complete |
|---|---|---|---|---|
| | | | | |

---

### Capacity Alerts

| System | Metric | Current | Threshold | Trend |
|---|---|---|---|---|
| | | | | ↑/↓/→ |

---

### Maintenance Windows

| Date | System | Type | Duration | Status |
|---|---|---|---|---|
| | | | | |

---

### Action Items

| Item | Owner | Due | Status |
|---|---|---|---|
| | | | |

---

### Next Week Planned Maintenance

| Date | System | Type | Risk |
|---|---|---|---|
| | | | |
```

---

## SLA Breach Response Procedure

```markdown
### Phase 1 — Detection (0–2 minutes)
1. SLA breach alert fires in monitoring platform
2. Create P2 incident ticket immediately
3. Calculate current availability against SLA target
4. Notify service owner and NOC supervisor
5. Begin downtime clock documentation

### Phase 2 — Assessment (2–10 minutes)
1. Identify root cause of availability impact
2. Assess likelihood of SLA breach for the month
3. Calculate remaining downtime budget
4. Determine if emergency escalation needed
5. Prepare initial stakeholder notification

### Phase 3 — Communication
Send SLA breach notification:
"SLA BREACH ALERT — [Service]
Current availability: [X]%
SLA target: [X]%
Breach by: [X] minutes
Action: [what is being done]
ETA resolution: [time]"

### Phase 4 — Resolution
1. Resolve underlying incident causing breach
2. Verify availability recovering to target
3. Document complete downtime period
4. Update SLA tracking spreadsheet
5. Notify stakeholders of resolution

### Phase 5 — Post-Breach
1. Schedule SLA review meeting with stakeholders
2. Complete PIR for incidents causing breach
3. Update capacity planning if trend identified
4. Review alert thresholds for earlier detection
5. Document lessons learned
```

---

## Standards & Frameworks Referenced

- **ITIL v4** — Service level management
- **ISO/IEC 20000** — IT service management
- **Google SRE Book** — Error budgets and SLOs
- **NIST SP 800-61** — Incident handling
- **PagerDuty Incident Response** — On-call best practices

---

##  Tools & Technologies

![Grafana](https://img.shields.io/badge/Grafana-SLA%20Dashboards-orange)
![Prometheus](https://img.shields.io/badge/Prometheus-Metrics-orange)
![Python](https://img.shields.io/badge/Python-Availability%20Calculator-blue)
![Datadog](https://img.shields.io/badge/Datadog-SLA%20Monitoring-purple)
![ITIL](https://img.shields.io/badge/ITIL%20v4-Service%20Management-green)
![CloudWatch](https://img.shields.io/badge/CloudWatch-AWS%20Monitoring-yellow)

---

##  Author

**George Amankwaa Sarpong**
NOC Engineer | SLA & Uptime Management
📍 Accra, Ghana 
🔗 [LinkedIn](https://linkedin.com/in/georgesarpong)
🌐 [GitHub Portfolio](https://github.com/GeorgeSarpong)

---

*This project is part of a broader portfolio demonstrating readiness for NOC Engineer and Infrastructure Monitoring roles in the US and global market.*
