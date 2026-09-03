# SolarWinds SAM Physical Server Hardware Monitoring Dashboard for Grafana

[![Grafana Version](https://img.shields.io/badge/Grafana-12.0%2B-orange?logo=grafana)](https://grafana.com/)
[![Datasource](https://img.shields.io/badge/DataSource-MSSQL-blue?logo=microsoftsqlserver)](https://www.microsoft.com/sql-server/)
[![Platform](https://img.shields.io/badge/SolarWinds-Orion%20%2F%20SAM-yellow)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A production-ready, high-density **Grafana dashboard** built to monitor enterprise **bare-metal infrastructure and physical server hardware health** directly through your **SolarWinds SAM (Server & Application Monitor)** MSSQL database.

Eliminate hypervisor noise by automatically filtering out VMware/Hyper-V virtual machines while isolating true physical components across Cisco UCS, Dell PowerEdge, HPE ProLiant, and standalone hosts.

---

## 📸 Dashboard Overview

```
+----------------------------------------------------------------------------------------------------+
|  SOLARWINDS SAM • HARDWARE MONITORING & SYSTEM HEALTH                               [● LIVE]        |
+------------------------------------+--------------------------------+------------------------------+
| SAM INVENTORY (By Device Type)     | PROTOCOL DISTRIBUTION          | TOTAL PHYSICAL SERVERS: 142  |
| [ Donut Chart: Cisco UCS, Dell ]   | [ WMI | SNMP | Agent | ICMP ]  | ACTIVE DOWN (30d): 2         |
+------------------------------------+--------------------------------+------------------------------+
| CRITICAL HARDWARE SENSORS          | ACTION REQUIRED: PHYSICAL HARDWARE FAILURES                   |
| [ Red Alert Metric: 3 Sensors ]    | [ Table: Server | Category | Failed Component | Status ]     |
+------------------------------------+---------------------------------------------------------------+
| BLIND SPOTS & CONFIG FAILURES      | DECOMMISSION CANDIDATES (Down > 30 Days / Ghost Nodes)       |
| [ Bar: ICMP Only vs WMI Broken ]   | [ Table: Server | IP | Days Offline | Last Heartbeat ]       |
+----------------------------------------------------------------------------------------------------+
```

---

## ⚡ Key Features

* **Bare-Metal Physical Isolation:** Dedicated SQL predicates (`vim.NodeID IS NULL`) ensure virtual machines do not dilute physical failure metrics.
* **Real-Time Sensor Diagnostics:** Immediate alerting for component-level failures, including power supplies (PSUs), fan tachometers, RAID controller arrays, and physical disks.
* **Root-Cause Telemetry Breakdown:** Identifies configuration drift, bad credentials, and RPC/WMI port blocks (`TCP 135/445`) preventing reliable polling.
* **Ghost Node Remediation:** Surfaces inactive and decommission candidates untouched for over 30 days to free up SolarWinds node licenses.
* **Spare Parts & Drive Mapping:** Dynamic tables displaying connected drive capacity, model serial numbers, and storage array health for physical maintenance workflows.

---

## 📊 Panel Breakdown & Architecture

| Panel Title | Visualization | Polling / Logic Focus |
| :--- | :--- | :--- |
| **Physical Server Inventory** | Table | Categorizes systems (Hyper-V Hosts, ESXi/VMware, UCS, Standalone). |
| **Critical H/W Sensors** | Stat / Metric | Displays count of items reporting critical status (`Status = 14` or `2`). |
| **Hardware Failures** | Table | Lists broken components via `HWH_HardwareItem` & `APM_HardwareCategory`. |
| **System Blind Spots** | Bar / Table | Pinpoints authentication errors, network timeouts, and ICMP-only nodes. |
| **Decommission Candidates** | Table | Detects stale down nodes using `LastSystemUpTimePollUtc`. |
| **Spare Parts Lookup** | Table | Detailed inventory from `AssetInventory_HardDrive` by selected node. |

---

## 🛠️ Prerequisites & Installation

### Prerequisites
* **Grafana:** Version 10.x, 11.x, or 12.x
* **SolarWinds SAM Database:** Direct read-only database user permissions to the SolarWinds Orion SQL Server instance (`SolarWindsOrion` DB)
* **Grafana MSSQL Datasource:** Configured and pointed to your Orion SQL database instance

### Database Permissions (Least Privilege)
Ensure the SQL user used by Grafana has minimal read permissions:

```sql
USE [SolarWindsOrion];
CREATE USER [grafana_reader] FOR LOGIN [grafana_reader];
GRANT SELECT ON [dbo].[Nodes] TO [grafana_reader];
GRANT SELECT ON [dbo].[NodesData] TO [grafana_reader];
GRANT SELECT ON [dbo].[VIM_VirtualMachines] TO [grafana_reader];
GRANT SELECT ON [dbo].[HWH_HardwareInfo] TO [grafana_reader];
GRANT SELECT ON [dbo].[HWH_HardwareItem] TO [grafana_reader];
GRANT SELECT ON [dbo].[HWH_HardwareCategoryStatus] TO [grafana_reader];
GRANT SELECT ON [dbo].[APM_HardwareCategory] TO [grafana_reader];
GRANT SELECT ON [dbo].[AssetInventory_HardDrive] TO [grafana_reader];
```

### Import Dashboard to Grafana
1. Clone this repository or copy the contents of [`dashboard.json`](./dashboard.json).
2. Open Grafana and navigate to **Dashboards** > **New** > **Import**.
3. Upload the `.json` file or paste the JSON text directly.
4. Select your configured **Microsoft SQL Server (MSSQL)** datasource from the dropdown.
5. Click **Import**.

---

## 🔍 Core MSSQL Query Structure

This dashboard leverages direct SQL views for high performance. Example logic isolating active physical server hardware failures:

```sql
SELECT 
    n.Caption AS [Server], 
    c.Name AS [Category], 
    hi.DisplayName AS [Component], 
    hi.Value AS [Value], 
    'CRITICAL' AS [Status] 
FROM HWH_HardwareItem hi 
JOIN HWH_HardwareCategoryStatus hcs ON hi.HardwareCategoryStatusID = hcs.ID 
JOIN APM_HardwareCategory c ON hcs.HardwareCategoryID = c.ID 
JOIN HWH_HardwareInfo hinf ON hcs.HardwareInfoID = hinf.ID 
JOIN Nodes n ON hinf.NetObjectID = n.NodeID 
WHERE hi.Status IN (2, 14) 
  AND n.UnManaged = 0 
  AND hi.IsDisabled = 0
  AND hi.IsDeleted = 0
  AND hinf.IsDisabled = 0
  AND n.Caption IN (${Node:singlequote}) 
ORDER BY n.Caption ASC;
```

---

## 🤝 Contributing

Issues, queries, and pull requests are welcome. Feel free to submit custom panels for Dell iDRAC, HP iLO, or Cisco UCS chassis health monitoring.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/NewPanel`)
3. Commit your Changes (`git commit -m 'Add support for SAN fabric sensors'`)
4. Push to the Branch (`git push origin feature/NewPanel`)
5. Open a Pull Request

---

## 📄 License

```text
MIT License

Copyright (c) 2026 Rittu Vijai

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

## Author
Rittu Vijai — Senior Cloud Ops & Automation Engineer | Azure Observability | Defender Endpoint | KQL (Dallas-Fort Worth Metroplex)  
LinkedIn: https://www.linkedin.com/in/rittuvijai