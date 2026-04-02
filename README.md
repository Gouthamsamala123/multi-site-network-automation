# 🌐 Multi-Site Switch Deployment Automation

> **Standardized • Automated • Scalable**  
> Generate consistent Cisco IOS switch configurations across 10–100+ sites from a single Excel workbook.

[![Platform](https://img.shields.io/badge/Platform-Cisco_IOS-blue)]()
[![Automation](https://img.shields.io/badge/Automation-Python_%7C_Jinja2_%7C_Ansible-green)]()
[![Status](https://img.shields.io/badge/Status-Production_Ready-brightgreen)]()

---

## 📋 Overview

This framework eliminates manual, error-prone network configuration by providing a **single Excel workbook** as the source of truth, **Jinja2 templates** for config generation, and **Ansible playbooks** for automated deployment.

| Metric | Manual | Automated |
|--------|--------|-----------|
| **Time per site** | ~25 hours | ~4 hours |
| **Config errors** | Frequent | 0 (pre-validated) |
| **Standardization** | Inconsistent | 100% identical baseline |
| **Scalability** | Linear effort | Same effort for 10 or 100 sites |
| **Audit trail** | None | Git-versioned configs |

---

## 🗂 Repository Structure

```
├── Multi_Site_Deployment_Framework.xlsx   # Source of truth (9 sheets)
├── README.md
├── deploy_site.yml                        # Ansible playbook for switch deployment
│
├── scripts/
│   ├── generate_configs.py                # Excel → Jinja2 → device configs
│   └── validate_inputs.py                 # Pre-flight data validation
│
├── templates/
│   └── switch_config.j2                   # Cisco IOS switch template → .cfg
│
└── output/                                # Generated configs (per site)
    ├── Site_A/
    │   └── sw-sta-us-mdf-1.cfg
    └── Site_B/
        └── sw-stb-eu-mdf-1.cfg
```

---

## 🚀 Quick Start

```bash
# 1. Clone the repo
git clone https://github.com/Gouthamsamala123/multi-site-switch-automation.git
cd multi-site-switch-automation

# 2. Install dependencies
pip install openpyxl jinja2

# 3. Validate the Excel workbook
python scripts/validate_inputs.py -i Multi_Site_Deployment_Framework.xlsx

# 4. Generate all site configs
python scripts/generate_configs.py \
  -i Multi_Site_Deployment_Framework.xlsx \
  -o ./output \
  -t ./templates

# 5. Deploy via Ansible (optional)
ansible-playbook deploy_site.yml --extra-vars "site=Site_A" --check   # dry-run
ansible-playbook deploy_site.yml --extra-vars "site=Site_A"           # actual deploy
```

---

## 🔄 How It Works

```
┌─────────────────────┐     ┌──────────────┐     ┌─────────────────────┐
│   EXCEL WORKBOOK    │     │   PYTHON +   │     │   SWITCH CONFIGS    │
│                     │────▶│   JINJA2     │────▶│                     │
│  Site Inventory     │     │              │     │  sw-sta-us-mdf-1.cfg│
│  IP Addressing      │     │  Validates   │     │  sw-stb-eu-mdf-1.cfg│
│  VLAN Mapping       │     │  Renders     │     │  ...per site        │
│  Routing            │     │  Templates   │     │                     │
└─────────────────────┘     └──────────────┘     └─────────────────────┘
                                                          │
                                                          ▼
                                                 ┌─────────────────┐
                                                 │     ANSIBLE     │
                                                 │  Push to device │
                                                 └─────────────────┘
```

| Excel Sheet | Template Variable | Generated Config Line |
|-------------|-------------------|-----------------------|
| Site Inventory → Hostname | `{{ hostname }}` | `hostname sw-sta-us-mdf-1` |
| IP Addressing → Gateway | `{{ svi.ip }}` | `ip address 10.100.10.1 255.255.255.0` |
| VLAN Mapping → ID, Name | `{{ vlan.id }}` | `vlan 10` / `name STA-USER` |
| Routing → Next Hop | `{{ route.next_hop }}` | `ip route 0.0.0.0 0.0.0.0 10.100.11.162` |

---

## 📊 Excel Workbook (9 Sheets)

| Sheet | Purpose |
|-------|---------|
| **Dashboard** | Auto-refreshing KPIs: total sites, switches, VLANs, subnets |
| **Site Inventory** | Device registry with dropdown validation (type, status, HA role) |
| **IP Addressing** | Per-site CIDR blocks, subnets, gateways, usable ranges |
| **VLAN Mapping** | Standard 8-VLAN template applied across all sites |
| **Routing** | Static routes per device |
| **Security Policies** | ACL rules and zone definitions |
| **Config Templates** | Jinja2 placeholder → Excel column mapping reference |
| **Naming Standards** | Hostname, VLAN, and circuit naming conventions |
| **Workflow** | 7-step deployment lifecycle |

> **Color coding:** Yellow cells = user input | Green cells = auto-calculated | Blue cells = cross-sheet reference

---

## ⚙️ Switch Config Sections

Each generated `.cfg` file includes (~280 lines per device):

| Section | Description |
|---------|-------------|
| **Stack & SSO** | Stack priority, persistent MAC, redundancy mode |
| **VTP / STP** | VTP transparent, RPVST+, root bridge (priority 4096) |
| **VLANs** | 8 standard VLANs: Transfer (3), Management (5), AP (9), User (10), WIFI (18), IoT (188), Guest (192), Corp (193) |
| **SVIs** | Gateway IPs with DHCP relay on user-facing VLANs |
| **Trunk Uplinks** | SD-WAN uplink ports with allowed VLAN list |
| **AP Ports** | Trunk ports with native VLAN 9 for AP management |
| **User Ports** | Access ports on VLAN 10 with portfast and BPDU guard |
| **Routing** | Static default route to SD-WAN Transfer gateway |
| **DHCP Snooping** | Snooping + Dynamic ARP Inspection on user VLANs |
| **AAA / TACACS+** | Full AAA model with TACACS server and local fallback |
| **SNMP** | RO/RW communities, trap host, location, contact |
| **Logging / NTP** | Syslog server, NTP servers, management source interface |
| **VTY Access** | SSH-only, ACL-restricted, 15-min timeout |
| **Banner** | MOTD with hostname and site identification |

---

## 🏷 Naming Conventions

| Element | Pattern | Example |
|---------|---------|---------|
| Switch | `sw-{site}-{country}-{role}-{num}` | `sw-sta-us-mdf-1` |
| SD-WAN | `sd-{site}-{country}-{role}-{num}` | `sd-sta-us-mdf-1` |
| AP | `ap-{site}-{country}-{num}` | `ap-sta-us-01` |
| VLAN | `{SITE_PREFIX}-{FUNCTION}` | `STA-USER` |
| Circuit | `CKT-{site}-{ISP}-{num}` | `CKT-STA-ISP1-001` |

---

## 🛠 Adding a New Site

1. Copy existing rows in each Excel sheet (Site Inventory, IP Addressing, Routing)
2. Change site name, location, hostnames, and IP addresses
3. VLAN Mapping stays the same (standard template)
4. Run: `validate → generate → review → deploy`

**Total time: ~4 hours** vs ~25 hours manual

---

## 🗓 Roadmap

| Phase | Scope |
|-------|-------|
| **v1.0 (Current)** | Switch config automation with Excel + Jinja2 + Ansible |
| **v2.0** | Add SD-WAN (VeloCloud) and firewall (Palo Alto) templates |
| **v3.0** | ServiceNow API integration, CI/CD pipeline for config changes |
| **v4.0** | Drift detection, compliance reporting, zero-touch provisioning |

---

## 📝 Requirements

```
Python 3.8+
pip install openpyxl jinja2
pip install ansible          # optional, for automated deployment
```

---

## 👤 Author

**Goutham Samala** — Senior Cloud Network Architect
