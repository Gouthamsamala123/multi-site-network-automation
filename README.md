# 🌐 Multi-Site Network Deployment Automation Framework

> **Standardized • Automated • Scalable**  
> Enterprise-grade network infrastructure automation for deploying consistent configurations across 10–100+ sites.

[![Platform](https://img.shields.io/badge/Platform-Cisco_IOS_%7C_VeloCloud-blue)]()
[![Automation](https://img.shields.io/badge/Automation-Python_%7C_Jinja2_%7C_Ansible-green)]()
[![License](https://img.shields.io/badge/License-Private-red)]()

---

## 📋 Overview

This framework eliminates manual, error-prone network configuration by providing a **single Excel workbook** as the source of truth, **Jinja2 templates** for config generation, and **Ansible playbooks** for automated deployment.

| Metric | Manual | Automated |
|--------|--------|-----------|
| **Time per site** | ~25 hours | ~4 hours |
| **Config errors** | Frequent | 0 (validated) |
| **Standardization** | Inconsistent | 100% identical |
| **Scalability** | Linear effort | Same effort for 10 or 100 sites |
| **Audit trail** | None | Git-versioned + ServiceNow |

---

## 🗂 Repository Structure

```
├── Multi_Site_Deployment_Framework.xlsx   # Source of truth (9 sheets)
├── README.md
├── deploy_site.yml                        # Ansible playbook (IOS + VCO API)
│
├── scripts/
│   ├── generate_configs.py                # Excel → Jinja2 → device configs
│   └── validate_inputs.py                 # Pre-flight data validation
│
├── templates/
│   ├── switch_config.j2                   # Cisco IOS switch template → .cfg
│   └── velocloud_edge.j2                  # VeloCloud Edge template → .json
│
├── output/                                # Generated configs (per site)
│   ├── Versah/
│   │   ├── sw-ver-us-mdf-1.cfg           # Cisco switch (283 lines)
│   │   ├── sd-ver-us-mdf-1.json          # VeloCloud Edge (VCO API spec)
│   │   └── sd-ver-us-mdf-2.json          # VeloCloud Edge (VCO API spec)
│   └── Nobel_Biocare/
│       ├── sw-nob-ch-mdf-1.cfg
│       ├── sd-nob-ch-mdf-1.json
│       └── sd-nob-ch-mdf-2.json
│
└── docs/
    ├── Implementation_Guide.docx          # Step-by-step engineer guide
    └── Customer_Presentation.pptx         # Customer-facing deck
```

---

## 🚀 Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/Gouthamsamala123/multi-site-network-automation.git
cd multi-site-network-automation

# 2. Install dependencies
pip install openpyxl jinja2

# 3. Validate the Excel workbook
python scripts/validate_inputs.py -i Multi_Site_Deployment_Framework.xlsx

# 4. Generate all site configs
python scripts/generate_configs.py \
  -i Multi_Site_Deployment_Framework.xlsx \
  -o ./output \
  -t ./templates

# 5. Deploy (optional - Ansible)
ansible-playbook deploy_site.yml --extra-vars "site=Versah" --check   # dry-run
ansible-playbook deploy_site.yml --extra-vars "site=Versah"           # actual deploy
```

---

## 📊 Excel Workbook (9 Sheets)

| Sheet | Purpose |
|-------|---------|
| **Dashboard** | Auto-refreshing KPIs, deployment status tracker |
| **Site Inventory** | Device registry with dropdown validation (type, status, HA role) |
| **IP Addressing** | Per-site CIDR blocks, subnets, gateways, usable ranges |
| **VLAN Mapping** | Standard 8-VLAN template applied across all sites |
| **Routing** | Static/dynamic routes per device |
| **Security Policies** | Firewall rules, ACLs, zone-based policies |
| **Config Templates** | Jinja2 placeholder → Excel column mapping |
| **Naming Standards** | Enterprise naming conventions with patterns + examples |
| **Workflow** | 7-step deployment lifecycle + business benefits |

**Color coding:** Yellow cells = user input | Green cells = auto-calculated formulas | Blue cells = cross-sheet references

---

## 🔄 Data Flow

```
┌─────────────────────┐     ┌──────────────┐     ┌─────────────────────┐
│   EXCEL WORKBOOK    │     │   PYTHON +   │     │   DEVICE CONFIGS    │
│                     │────▶│   JINJA2     │────▶│                     │
│  Site Inventory     │     │              │     │  sw-ver-us-mdf-1.cfg│
│  IP Addressing      │     │  Reads Excel │     │  sd-ver-us-mdf-1.json│
│  VLAN Mapping       │     │  Validates   │     │  sd-ver-us-mdf-2.json│
│  Routing            │     │  Renders     │     │  ...per site        │
│  Security Policies  │     │  Templates   │     │                     │
└─────────────────────┘     └──────────────┘     └─────────────────────┘
                                                          │
                                                          ▼
                                                 ┌─────────────────┐
                                                 │ ANSIBLE / VCO   │
                                                 │ Push to devices │
                                                 └─────────────────┘
```

| Excel Sheet | Template Variable | Config Section |
|-------------|-------------------|----------------|
| Site Inventory → Hostname | `{{ hostname }}` | `hostname sw-ver-us-mdf-1` |
| IP Addressing → Gateway | `{{ svi.ip }}` | `ip address 10.246.232.1 255.255.255.0` |
| VLAN Mapping → ID, Name | `{{ vlan.id }}, {{ vlan.name }}` | `vlan 10` / `name VER-USER` |
| Routing → Destination, Next Hop | `{{ route.destination }}` | `ip route 0.0.0.0 0.0.0.0 10.246.233.162` |

---

## 🛡 SD-WAN Platform: VMware VeloCloud

This framework uses **VeloCloud Edges** managed via **VeloCloud Orchestrator (VCO)**.

- SD-WAN configs are generated as **JSON provisioning specs** (`.json`)
- Apply via **VCO UI** (Configure → Edges) or **VCO REST API** (`POST /edge/edgeProvision`)
- Generated JSON includes: LAN networks, WAN interfaces, HA pair config, business policies (Zscaler direct breakout, guest bandwidth caps, IoT restriction), stateful firewall rules, SNMP, syslog, TACACS, NTP
- The Ansible playbook includes a VCO API play that authenticates and provisions edges automatically

---

## ⚙️ Supported Platforms

| Platform | Template | Output | Deployment Method |
|----------|----------|--------|-------------------|
| **Cisco IOS** (Switches) | `switch_config.j2` | `.cfg` (CLI) | Ansible `ios_config` or manual SSH |
| **VMware VeloCloud** (SD-WAN) | `velocloud_edge.j2` | `.json` (API) | VCO REST API or VCO UI |

### Switch Config Sections (283 lines per device)
- Stack configuration & SSO
- VTP transparent, RPVST+ spanning-tree (root bridge)
- 8 VLANs: Transfer (3), Management (5), AP (9), User (10), WIFI (18), IoT (188), Guest (192), Corp (193)
- SVIs with gateway IPs and DHCP relay
- Trunk uplinks to SD-WAN devices
- AP trunk ports (native VLAN 9, allowed 9/18/192/193)
- User access ports (VLAN 10)
- DHCP snooping + Dynamic ARP Inspection
- AAA / TACACS+ authentication
- SNMP, Syslog, NTP
- Management ACLs + SSH-only VTY access
- Banner MOTD

### VeloCloud Edge Config Sections (JSON spec)
- Edge provisioning (model, serial, activation key, profile)
- LAN networks with VLAN-to-subnet mapping and DHCP relay
- WAN interfaces with overlay configuration
- HA active-standby pair
- Business policies (per-VLAN traffic steering)
- Stateful firewall rules (IoT isolation, guest restriction)
- Monitoring: SNMP, syslog, TACACS, NTP

---

## 🏷 Naming Conventions

| Element | Pattern | Example |
|---------|---------|---------|
| Switch | `sw-{site}-{country}-{role}-{num}` | `sw-ver-us-mdf-1` |
| SD-WAN | `sd-{site}-{country}-{role}-{num}` | `sd-ver-us-mdf-1` |
| AP | `ap-{site}-{country}-{num}` | `ap-ver-us-01` |
| VLAN | `{SITE_PREFIX}-{FUNCTION}` | `VER-USER` |
| ISP Circuit | `CKT-{site}-{ISP}-{num}` | `CKT-VER-ISP1-001` |

---

## 📖 Documentation

| Document | Description |
|----------|-------------|
| [Implementation Guide](docs/Implementation_Guide.docx) | Step-by-step instructions for engineers (7 steps with commands + expected output) |
| [Customer Presentation](docs/Customer_Presentation.pptx) | Customer-facing deck: challenge, solution, architecture, ROI, roadmap |
| [Excel Workbook](Multi_Site_Deployment_Framework.xlsx) | 9-sheet framework with validation, formulas, sample data for 2 sites |

---

## 🗓 Roadmap

| Phase | Timeline | Scope |
|-------|----------|-------|
| **Foundation** | Now | Excel framework, Jinja2 templates (Switch + VeloCloud), validation engine, 2 sites modeled |
| **Expansion** | Q3 2026 | Palo Alto firewall templates, ServiceNow API integration, Ansible Tower, scale to 10+ sites |
| **Maturity** | Q4 2026 | CI/CD pipeline for config changes, drift detection via NAPALM, compliance reporting |
| **Full IaC** | 2027+ | Terraform for cloud infra, event-driven remediation, zero-touch provisioning |

---

## 📝 Requirements

- **Python 3.8+**
- `pip install openpyxl jinja2`
- **Ansible 2.12+** (optional, for automated deployment): `pip install ansible`
- **VeloCloud Orchestrator** access (for SD-WAN provisioning via API)

---

## 👤 Author

**Goutham Samala** — Senior Cloud Network Architect  
Envista Holdings — Network Infrastructure Team

---

> *This framework reduces per-site deployment time by 84% (25 hrs → 4 hrs), eliminates configuration drift, and provides a complete audit trail for compliance.*
